# Advanced Hooks

> Intermediate / Mid-Level (1-3 years)

---

## Questions

31. What is useContext hook and when do you use it?
32. What is useReducer hook and how is it different from useState?
33. What is useRef hook and what are its use cases?
34. What is useMemo hook and when should you use it?
35. What is useCallback hook and how does it differ from useMemo?
36. What is useLayoutEffect and how is it different from useEffect?
37. What is useImperativeHandle hook?
38. How do you create custom hooks?
39. What are the rules of hooks?
40. What is useId hook? (React 18+)

---

## Detailed Answers

### 31. What is useContext hook and when do you use it?

<details>
<summary>View Answer</summary>

**useContext Hook**

`useContext` is a React Hook that lets you **read and subscribe to context** from your component. It provides a way to access values from a Context without using a Consumer component.

**Purpose:**
- **Avoid prop drilling** - share data across component tree without passing props through every level
- **Global state management** - theme, authentication, language settings
- **Simplify context consumption** - cleaner syntax than Context.Consumer

---

**Basic Syntax:**

```jsx
import { useContext } from 'react';

const value = useContext(SomeContext);
```

**Parameters:**
- `SomeContext` - The context object created with `createContext`

**Returns:**
- The context value for the calling component
- Value is determined by the nearest `<Context.Provider>` above in the tree

---

## Creating and Using Context

**Step 1: Create Context**

```jsx
import { createContext } from 'react';

// Create context with default value
const ThemeContext = createContext('light');
```

**Step 2: Provide Context Value**

```jsx
import { useState } from 'react';
import { ThemeContext } from './ThemeContext';

function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={theme}>
      <Page />
    </ThemeContext.Provider>
  );
}
```

**Step 3: Consume Context with useContext**

```jsx
import { useContext } from 'react';
import { ThemeContext } from './ThemeContext';

function Button() {
  const theme = useContext(ThemeContext);  // Read context value
  
  return (
    <button className={`button-${theme}`}>
      Click Me
    </button>
  );
}
```

---

## useContext vs Context.Consumer

**Old Way (Context.Consumer):**

```jsx
import { ThemeContext } from './ThemeContext';

function Button() {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <button className={`button-${theme}`}>
          Click Me
        </button>
      )}
    </ThemeContext.Consumer>
  );
}
```

**New Way (useContext):**

```jsx
import { useContext } from 'react';
import { ThemeContext } from './ThemeContext';

function Button() {
  const theme = useContext(ThemeContext);  // Much cleaner!
  
  return (
    <button className={`button-${theme}`}>
      Click Me
    </button>
  );
}
```

---

## Complete Example: Theme System

**ThemeContext.js:**

```jsx
import { createContext, useState, useContext } from 'react';

// 1. Create Context
const ThemeContext = createContext();

// 2. Provider Component
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const value = {
    theme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Custom Hook for easy access
export function useTheme() {
  const context = useContext(ThemeContext);
  
  if (context === undefined) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  
  return context;
}
```

**App.js:**

```jsx
import { ThemeProvider } from './ThemeContext';
import Page from './Page';

function App() {
  return (
    <ThemeProvider>
      <Page />
    </ThemeProvider>
  );
}

export default App;
```

**Page.js:**

```jsx
import { useTheme } from './ThemeContext';
import Button from './Button';

function Page() {
  const { theme } = useTheme();
  
  return (
    <div className={`page page-${theme}`}>
      <Header />
      <Content />
      <ThemeToggle />
    </div>
  );
}

function Header() {
  const { theme } = useTheme();
  
  return (
    <header className={`header header-${theme}`}>
      <h1>My App</h1>
    </header>
  );
}

function Content() {
  const { theme } = useTheme();
  
  return (
    <main className={`content content-${theme}`}>
      <p>Content goes here</p>
    </main>
  );
}

function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme}>
      Switch to {theme === 'light' ? 'Dark' : 'Light'} Mode
    </button>
  );
}

export default Page;
```

---

## Multiple Contexts

You can use multiple contexts in the same component.

**AuthContext.js:**

```jsx
import { createContext, useState, useContext } from 'react';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  
  const login = (username, password) => {
    // Simulate login
    setUser({ username });
    setIsAuthenticated(true);
  };
  
  const logout = () => {
    setUser(null);
    setIsAuthenticated(false);
  };
  
  return (
    <AuthContext.Provider value={{ user, isAuthenticated, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

**Using Multiple Contexts:**

```jsx
import { useTheme } from './ThemeContext';
import { useAuth } from './AuthContext';

function UserProfile() {
  const { theme } = useTheme();              // Theme context
  const { user, isAuthenticated } = useAuth(); // Auth context
  
  if (!isAuthenticated) {
    return <div>Please log in</div>;
  }
  
  return (
    <div className={`profile profile-${theme}`}>
      <h2>Welcome, {user.username}!</h2>
    </div>
  );
}
```

**Combining Providers:**

```jsx
import { ThemeProvider } from './ThemeContext';
import { AuthProvider } from './AuthContext';
import { LanguageProvider } from './LanguageContext';

function App() {
  return (
    <ThemeProvider>
      <AuthProvider>
        <LanguageProvider>
          <Page />
        </LanguageProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}
```

---

## Context with Complex State

**CartContext.js:**

```jsx
import { createContext, useState, useContext, useMemo } from 'react';

const CartContext = createContext();

export function CartProvider({ children }) {
  const [items, setItems] = useState([]);
  
  const addItem = (product) => {
    setItems(prev => {
      const existing = prev.find(item => item.id === product.id);
      if (existing) {
        return prev.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prev, { ...product, quantity: 1 }];
    });
  };
  
  const removeItem = (productId) => {
    setItems(prev => prev.filter(item => item.id !== productId));
  };
  
  const updateQuantity = (productId, quantity) => {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }
    setItems(prev =>
      prev.map(item =>
        item.id === productId ? { ...item, quantity } : item
      )
    );
  };
  
  const clearCart = () => {
    setItems([]);
  };
  
  // Memoize computed values to avoid recalculation
  const total = useMemo(() => {
    return items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }, [items]);
  
  const itemCount = useMemo(() => {
    return items.reduce((count, item) => count + item.quantity, 0);
  }, [items]);
  
  const value = useMemo(() => ({
    items,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    total,
    itemCount
  }), [items, total, itemCount]);
  
  return (
    <CartContext.Provider value={value}>
      {children}
    </CartContext.Provider>
  );
}

export function useCart() {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within CartProvider');
  }
  return context;
}
```

**Using Cart Context:**

```jsx
import { useCart } from './CartContext';

function ProductCard({ product }) {
  const { addItem } = useCart();
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => addItem(product)}>
        Add to Cart
      </button>
    </div>
  );
}

function CartSummary() {
  const { items, total, itemCount } = useCart();
  
  return (
    <div className="cart-summary">
      <h2>Cart ({itemCount} items)</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name} x {item.quantity} = ${item.price * item.quantity}
          </li>
        ))}
      </ul>
      <p><strong>Total: ${total.toFixed(2)}</strong></p>
    </div>
  );
}

function CartIcon() {
  const { itemCount } = useCart();
  
  return (
    <div className="cart-icon">
      🛒 <span className="badge">{itemCount}</span>
    </div>
  );
}
```

---

## When to Use useContext

**✅ Good Use Cases:**

1. **Theme/Dark Mode**
```jsx
const { theme, toggleTheme } = useContext(ThemeContext);
```

2. **User Authentication**
```jsx
const { user, isAuthenticated, login, logout } = useContext(AuthContext);
```

3. **Internationalization (i18n)**
```jsx
const { language, t, changeLanguage } = useContext(LanguageContext);
```

4. **Global Settings**
```jsx
const { currency, dateFormat, timezone } = useContext(SettingsContext);
```

5. **Shopping Cart**
```jsx
const { items, addItem, removeItem, total } = useContext(CartContext);
```

6. **Notifications/Toasts**
```jsx
const { showSuccess, showError, showWarning } = useContext(NotificationContext);
```

**❌ Don't Use For:**

1. **Frequently Changing Values** - causes unnecessary re-renders
2. **Local Component State** - use `useState` instead
3. **Form Data** - keep local unless needed globally
4. **Derived/Computed Values** - use `useMemo` locally

---

## Performance Optimization

**Problem: All consumers re-render when context value changes**

**❌ Bad: Creating new object on every render**

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>  {/* New object every render! */}
      {children}
    </ThemeContext.Provider>
  );
}
```

**✅ Good: Memoize context value**

```jsx
import { useMemo } from 'react';

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const value = useMemo(() => ({ theme, setTheme }), [theme]);  // Only changes when theme changes
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

**Split Contexts for Better Performance:**

```jsx
// ❌ Bad: One large context
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [language, setLanguage] = useState('en');
  
  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme, language, setLanguage }}>
      {children}
    </AppContext.Provider>
  );
}

// Problem: Changing theme re-renders all components that use ANY part of context!

// ✅ Good: Split into separate contexts
const UserContext = createContext();
const ThemeContext = createContext();
const LanguageContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [language, setLanguage] = useState('en');
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <LanguageContext.Provider value={{ language, setLanguage }}>
          {children}
        </LanguageContext.Provider>
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Now: Changing theme only re-renders components that use ThemeContext!
```

---

## Production Example: Notification System

**NotificationContext.js:**

```jsx
import { createContext, useState, useContext, useCallback, useMemo } from 'react';

const NotificationContext = createContext();

export function NotificationProvider({ children }) {
  const [notifications, setNotifications] = useState([]);
  
  const addNotification = useCallback((message, type = 'info', duration = 3000) => {
    const id = Date.now() + Math.random();
    
    setNotifications(prev => [...prev, { id, message, type }]);
    
    if (duration > 0) {
      setTimeout(() => {
        removeNotification(id);
      }, duration);
    }
    
    return id;
  }, []);
  
  const removeNotification = useCallback((id) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  }, []);
  
  const showSuccess = useCallback((message, duration) => {
    return addNotification(message, 'success', duration);
  }, [addNotification]);
  
  const showError = useCallback((message, duration) => {
    return addNotification(message, 'error', duration);
  }, [addNotification]);
  
  const showWarning = useCallback((message, duration) => {
    return addNotification(message, 'warning', duration);
  }, [addNotification]);
  
  const showInfo = useCallback((message, duration) => {
    return addNotification(message, 'info', duration);
  }, [addNotification]);
  
  const value = useMemo(() => ({
    notifications,
    addNotification,
    removeNotification,
    showSuccess,
    showError,
    showWarning,
    showInfo
  }), [notifications, addNotification, removeNotification, showSuccess, showError, showWarning, showInfo]);
  
  return (
    <NotificationContext.Provider value={value}>
      {children}
      <NotificationContainer />
    </NotificationContext.Provider>
  );
}

export function useNotification() {
  const context = useContext(NotificationContext);
  if (!context) {
    throw new Error('useNotification must be used within NotificationProvider');
  }
  return context;
}

function NotificationContainer() {
  const { notifications, removeNotification } = useNotification();
  
  return (
    <div className="notification-container">
      {notifications.map(notification => (
        <Notification
          key={notification.id}
          notification={notification}
          onClose={() => removeNotification(notification.id)}
        />
      ))}
    </div>
  );
}

function Notification({ notification, onClose }) {
  return (
    <div className={`notification notification-${notification.type}`}>
      <span>{notification.message}</span>
      <button onClick={onClose}>&times;</button>
    </div>
  );
}
```

**Usage:**

```jsx
import { NotificationProvider, useNotification } from './NotificationContext';

function App() {
  return (
    <NotificationProvider>
      <Page />
    </NotificationProvider>
  );
}

function LoginForm() {
  const { showSuccess, showError } = useNotification();
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      await loginUser();
      showSuccess('Login successful!');
    } catch (error) {
      showError('Login failed. Please try again.');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <button type="submit">Login</button>
    </form>
  );
}

function ProductCard({ product }) {
  const { showSuccess } = useNotification();
  const { addItem } = useCart();
  
  const handleAddToCart = () => {
    addItem(product);
    showSuccess(`${product.name} added to cart!`);
  };
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={handleAddToCart}>Add to Cart</button>
    </div>
  );
}
```

---

## Error Handling

**Always check if context is used within provider:**

```jsx
export function useTheme() {
  const context = useContext(ThemeContext);
  
  if (context === undefined) {
    throw new Error(
      'useTheme must be used within ThemeProvider. ' +
      'Wrap your component tree with <ThemeProvider>.'
    );
  }
  
  return context;
}
```

**This helps catch bugs early:**

```jsx
// ❌ This will throw a helpful error
function Button() {
  const { theme } = useTheme();  // Error: useTheme must be used within ThemeProvider
  return <button>Click</button>;
}

// ✅ Correct usage
function App() {
  return (
    <ThemeProvider>
      <Button />  {/* Now works! */}
    </ThemeProvider>
  );
}
```

---

**Best Practices:**

1. **Create custom hooks** for each context (`useTheme`, `useAuth`)
2. **Memoize context value** with `useMemo` to prevent unnecessary re-renders
3. **Validate context usage** - throw error if used outside provider
4. **Split contexts** - don't put everything in one large context
5. **Keep context values stable** - avoid creating new objects on every render
6. **Document context** - explain what data is available and when to use it
7. **Combine related data** - group related state in one context
8. **Use meaningful names** - `ThemeContext`, `AuthContext`, not `Context1`, `Context2`

---

**Anti-patterns:**

**❌ Don't use context for everything**
```jsx
// Bad: Using context for local state
function Form() {
  const { email, setEmail } = useContext(FormContext);  // Overkill!
  // Just use useState here
}
```

**❌ Don't nest too many providers**
```jsx
// Bad: Provider hell
<Provider1>
  <Provider2>
    <Provider3>
      <Provider4>
        <Provider5>
          <App />
        </Provider5>
      </Provider4>
    </Provider3>
  </Provider2>
</Provider1>

// Better: Combine providers
<AppProvider>  {/* Wraps multiple contexts */}
  <App />
</AppProvider>
```

**❌ Don't create new objects in value**
```jsx
// Bad: Creates new object every render
<Context.Provider value={{ user, setUser }}>  {/* New object! */}

// Good: Memoize
const value = useMemo(() => ({ user, setUser }), [user]);
<Context.Provider value={value}>
```

---

**Interview Tips:**
- `useContext` reads value from **Context** created by `createContext`
- Solves **prop drilling** problem - share data without passing props through every level
- Must be used inside a **Provider** component that wraps the tree
- **All consumers re-render** when context value changes
- **Memoize context value** with `useMemo` to prevent unnecessary re-renders
- Best for **global/shared state**: theme, auth, language, settings
- **Not recommended** for frequently changing data or local component state
- Create **custom hooks** (`useTheme`, `useAuth`) for better DX and error handling
- **Split contexts** for better performance - don't put everything in one context
- Returns the value from the **nearest Provider** above in tree
- If no Provider, uses the **default value** from `createContext(defaultValue)`
- Modern alternative to **Context.Consumer** pattern (cleaner syntax)
- Different from Redux/Zustand - Context is built-in, no external dependencies

</details>

---

### 32. What is useReducer hook and how is it different from useState?

<details>
<summary>View Answer</summary>

**useReducer Hook**

`useReducer` is a React Hook for managing **complex state logic**. It's an alternative to `useState` that uses a **reducer function** to determine how state updates, similar to Redux.

**Basic Syntax:**

```jsx
import { useReducer } from 'react';

const [state, dispatch] = useReducer(reducer, initialState);
```

**Parameters:**
- `reducer` - Function that determines how state updates: `(state, action) => newState`
- `initialState` - Initial state value
- `init` (optional) - Initialization function

**Returns:**
- `state` - Current state value
- `dispatch` - Function to trigger state updates by dispatching actions

---

## Basic Example

**Counter with useReducer:**

```jsx
import { useReducer } from 'react';

// 1. Define reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  // 2. Initialize useReducer
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  );
}
```

**Same Counter with useState:**

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(count - 1)}>-</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

---

## useReducer vs useState

**When to use useState:**

```jsx
// ✅ Simple state
const [count, setCount] = useState(0);
const [name, setName] = useState('');
const [isOpen, setIsOpen] = useState(false);

// ✅ Independent state values
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [email, setEmail] = useState('');
```

**When to use useReducer:**

```jsx
// ✅ Complex state object
const [state, dispatch] = useReducer(reducer, {
  user: null,
  posts: [],
  loading: false,
  error: null
});

// ✅ Multiple related state updates
dispatch({ type: 'FETCH_SUCCESS', payload: data });
// Updates: loading=false, data=payload, error=null

// ✅ State updates depend on previous state
dispatch({ type: 'ADD_ITEM', payload: newItem });

// ✅ Complex update logic
switch (action.type) {
  case 'UPDATE_USER':
    return { ...state, user: { ...state.user, ...action.payload } };
}
```

---

**Comparison Table:**

| Feature | useState | useReducer |
|---------|----------|------------|
| **Complexity** | Simple values | Complex objects |
| **Updates** | Direct: `setState(value)` | Dispatch actions: `dispatch(action)` |
| **Logic** | In component | In reducer function |
| **Related Updates** | Multiple `setState` calls | Single action updates multiple values |
| **Testing** | Test component | Test reducer separately |
| **Predictability** | Less structured | More predictable (same action = same result) |
| **Best For** | Single values, toggles | Forms, wizards, data fetching |

---

## Reducer Function Patterns

**1. Action with Type Only**

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// Usage
dispatch({ type: 'INCREMENT' });
```

**2. Action with Payload**

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'SET_NAME':
      return { ...state, name: action.payload };
    case 'SET_AGE':
      return { ...state, age: action.payload };
    default:
      return state;
  }
}

// Usage
dispatch({ type: 'SET_NAME', payload: 'John' });
dispatch({ type: 'SET_AGE', payload: 25 });
```

**3. Multiple Payload Properties**

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'UPDATE_USER':
      return {
        ...state,
        user: {
          ...state.user,
          name: action.name,
          email: action.email
        }
      };
    default:
      return state;
  }
}

// Usage
dispatch({ 
  type: 'UPDATE_USER', 
  name: 'John', 
  email: 'john@example.com' 
});
```

---

## Complex Example: Todo App

**TodoApp with useReducer:**

```jsx
import { useReducer, useState } from 'react';

// Reducer function
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
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
    
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    
    case 'EDIT_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.id
            ? { ...todo, text: action.text }
            : todo
        )
      };
    
    case 'CLEAR_COMPLETED':
      return {
        ...state,
        todos: state.todos.filter(todo => !todo.completed)
      };
    
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload
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

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [inputValue, setInputValue] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      dispatch({ type: 'ADD_TODO', payload: inputValue });
      setInputValue('');
    }
  };
  
  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true; // 'all'
  });
  
  return (
    <div>
      <h1>Todo App</h1>
      
      <form onSubmit={handleSubmit}>
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="Add a todo..."
        />
        <button type="submit">Add</button>
      </form>
      
      <div>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'all' })}>All</button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'active' })}>Active</button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'completed' })}>Completed</button>
      </div>
      
      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: todo.id })}>
              Delete
            </button>
          </li>
        ))}
      </ul>
      
      <button onClick={() => dispatch({ type: 'CLEAR_COMPLETED' })}>
        Clear Completed
      </button>
      
      <p>
        {state.todos.filter(t => !t.completed).length} items left
      </p>
    </div>
  );
}
```

---

## Lazy Initialization

Use third parameter to lazily initialize state (useful for expensive calculations).

**Without Lazy Initialization:**

```jsx
const [state, dispatch] = useReducer(
  reducer,
  JSON.parse(localStorage.getItem('data')) || defaultData  // Runs on every render!
);
```

**With Lazy Initialization:**

```jsx
function init(initialData) {
  // This only runs once
  return JSON.parse(localStorage.getItem('data')) || initialData;
}

const [state, dispatch] = useReducer(
  reducer,
  defaultData,
  init  // Third parameter: initialization function
);
```

**Another Example:**

```jsx
function init(initialCount) {
  return { count: initialCount * 10 };
}

function Counter({ initialCount }) {
  const [state, dispatch] = useReducer(
    reducer,
    initialCount,  // Will be passed to init()
    init
  );
  
  // state.count will be initialCount * 10
}
```

---

## Production Example: Form with Validation

```jsx
import { useReducer } from 'react';

const initialState = {
  values: {
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  },
  errors: {},
  touched: {},
  isSubmitting: false,
  isValid: false
};

function formReducer(state, action) {
  switch (action.type) {
    case 'SET_FIELD_VALUE':
      return {
        ...state,
        values: {
          ...state.values,
          [action.field]: action.value
        }
      };
    
    case 'SET_FIELD_TOUCHED':
      return {
        ...state,
        touched: {
          ...state.touched,
          [action.field]: true
        }
      };
    
    case 'SET_ERRORS':
      return {
        ...state,
        errors: action.payload
      };
    
    case 'SET_SUBMITTING':
      return {
        ...state,
        isSubmitting: action.payload
      };
    
    case 'RESET_FORM':
      return initialState;
    
    case 'SUBMIT_SUCCESS':
      return {
        ...state,
        isSubmitting: false,
        values: initialState.values,
        errors: {},
        touched: {}
      };
    
    case 'SUBMIT_ERROR':
      return {
        ...state,
        isSubmitting: false,
        errors: action.payload
      };
    
    default:
      return state;
  }
}

function SignupForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);
  
  const handleChange = (field) => (e) => {
    dispatch({ type: 'SET_FIELD_VALUE', field, value: e.target.value });
  };
  
  const handleBlur = (field) => () => {
    dispatch({ type: 'SET_FIELD_TOUCHED', field });
    validateField(field, state.values[field]);
  };
  
  const validateField = (field, value) => {
    const errors = { ...state.errors };
    
    switch (field) {
      case 'username':
        if (!value) {
          errors.username = 'Username is required';
        } else if (value.length < 3) {
          errors.username = 'Username must be at least 3 characters';
        } else {
          delete errors.username;
        }
        break;
      
      case 'email':
        if (!value) {
          errors.email = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(value)) {
          errors.email = 'Email is invalid';
        } else {
          delete errors.email;
        }
        break;
      
      case 'password':
        if (!value) {
          errors.password = 'Password is required';
        } else if (value.length < 8) {
          errors.password = 'Password must be at least 8 characters';
        } else {
          delete errors.password;
        }
        break;
      
      case 'confirmPassword':
        if (value !== state.values.password) {
          errors.confirmPassword = 'Passwords do not match';
        } else {
          delete errors.confirmPassword;
        }
        break;
    }
    
    dispatch({ type: 'SET_ERRORS', payload: errors });
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Validate all fields
    const allErrors = {};
    Object.keys(state.values).forEach(field => {
      validateField(field, state.values[field]);
    });
    
    if (Object.keys(allErrors).length > 0) {
      return;
    }
    
    dispatch({ type: 'SET_SUBMITTING', payload: true });
    
    try {
      // Simulate API call
      await new Promise(resolve => setTimeout(resolve, 2000));
      dispatch({ type: 'SUBMIT_SUCCESS' });
      alert('Signup successful!');
    } catch (error) {
      dispatch({ type: 'SUBMIT_ERROR', payload: { api: error.message } });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username</label>
        <input
          type="text"
          value={state.values.username}
          onChange={handleChange('username')}
          onBlur={handleBlur('username')}
        />
        {state.touched.username && state.errors.username && (
          <span className="error">{state.errors.username}</span>
        )}
      </div>
      
      <div>
        <label>Email</label>
        <input
          type="email"
          value={state.values.email}
          onChange={handleChange('email')}
          onBlur={handleBlur('email')}
        />
        {state.touched.email && state.errors.email && (
          <span className="error">{state.errors.email}</span>
        )}
      </div>
      
      <div>
        <label>Password</label>
        <input
          type="password"
          value={state.values.password}
          onChange={handleChange('password')}
          onBlur={handleBlur('password')}
        />
        {state.touched.password && state.errors.password && (
          <span className="error">{state.errors.password}</span>
        )}
      </div>
      
      <div>
        <label>Confirm Password</label>
        <input
          type="password"
          value={state.values.confirmPassword}
          onChange={handleChange('confirmPassword')}
          onBlur={handleBlur('confirmPassword')}
        />
        {state.touched.confirmPassword && state.errors.confirmPassword && (
          <span className="error">{state.errors.confirmPassword}</span>
        )}
      </div>
      
      <button type="submit" disabled={state.isSubmitting}>
        {state.isSubmitting ? 'Submitting...' : 'Sign Up'}
      </button>
    </form>
  );
}
```

---

## Action Creators Pattern

Create functions that return actions (like Redux).

```jsx
// Action creators
const actions = {
  increment: () => ({ type: 'INCREMENT' }),
  decrement: () => ({ type: 'DECREMENT' }),
  incrementBy: (amount) => ({ type: 'INCREMENT_BY', payload: amount }),
  reset: () => ({ type: 'RESET' })
};

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch(actions.increment())}>+1</button>
      <button onClick={() => dispatch(actions.incrementBy(5))}>+5</button>
      <button onClick={() => dispatch(actions.decrement())}>-1</button>
      <button onClick={() => dispatch(actions.reset())}>Reset</button>
    </div>
  );
}
```

---

## Production Example: Data Fetching

```jsx
import { useReducer, useEffect } from 'react';

const initialState = {
  data: null,
  loading: false,
  error: null
};

function dataReducer(state, action) {
  switch (action.type) {
    case 'FETCH_START':
      return {
        ...state,
        loading: true,
        error: null
      };
    
    case 'FETCH_SUCCESS':
      return {
        data: action.payload,
        loading: false,
        error: null
      };
    
    case 'FETCH_ERROR':
      return {
        ...state,
        loading: false,
        error: action.payload
      };
    
    case 'RESET':
      return initialState;
    
    default:
      return state;
  }
}

function UserProfile({ userId }) {
  const [state, dispatch] = useReducer(dataReducer, initialState);
  
  useEffect(() => {
    const fetchUser = async () => {
      dispatch({ type: 'FETCH_START' });
      
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed to fetch user');
        const data = await response.json();
        dispatch({ type: 'FETCH_SUCCESS', payload: data });
      } catch (error) {
        dispatch({ type: 'FETCH_ERROR', payload: error.message });
      }
    };
    
    fetchUser();
  }, [userId]);
  
  if (state.loading) return <div>Loading...</div>;
  if (state.error) return <div>Error: {state.error}</div>;
  if (!state.data) return null;
  
  return (
    <div>
      <h2>{state.data.name}</h2>
      <p>Email: {state.data.email}</p>
      <p>Role: {state.data.role}</p>
    </div>
  );
}
```

---

**Best Practices:**

1. **Use descriptive action types**
```jsx
// ✅ Good
dispatch({ type: 'ADD_TODO', payload: text });

// ❌ Bad
dispatch({ type: 'add', data: text });
```

2. **Keep reducer pure**
```jsx
// ✅ Good: Pure function
function reducer(state, action) {
  return { count: state.count + 1 };
}

// ❌ Bad: Side effects
function reducer(state, action) {
  localStorage.setItem('count', state.count);  // Side effect!
  return { count: state.count + 1 };
}
```

3. **Always return new state object**
```jsx
// ✅ Good
return { ...state, count: state.count + 1 };

// ❌ Bad: Mutating state
state.count++;
return state;
```

4. **Handle default case**
```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    default:
      return state;  // Important!
  }
}
```

5. **Use TypeScript for type safety** (if using TS)
```typescript
type Action = 
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET', payload: number };

function reducer(state: State, action: Action): State {
  // TypeScript ensures only valid actions
}
```

---

**Interview Tips:**
- `useReducer` is alternative to `useState` for **complex state logic**
- Uses **reducer function** pattern: `(state, action) => newState`
- **dispatch** function sends actions to trigger state updates
- Similar to **Redux** but built-in to React (no external library)
- Better than `useState` when:
  - State is **complex object** with multiple sub-values
  - Next state depends on **previous state**
  - Multiple **related updates** happen together
  - Want to **centralize** update logic outside component
- Reducer must be **pure function** (no side effects)
- Always **return new state** object (don't mutate)
- Actions typically have `type` and optional `payload`
- Can use **lazy initialization** (third parameter) for expensive initial state
- Often combined with `useContext` for global state management
- **Easier to test** - reducer is pure function, test separately from component
- Use **action creators** for cleaner code (optional)
- Convention: action types in UPPERCASE with underscores
- Common use cases: **forms**, **wizards**, **data fetching**, **todo lists**

</details>

---

### 33. What is useRef hook and what are its use cases?

<details>
<summary>View Answer</summary>

**useRef Hook**

`useRef` is a React Hook that lets you **reference a value** that's not needed for rendering. It returns a **mutable ref object** whose `.current` property persists across re-renders.

**Basic Syntax:**

```jsx
import { useRef } from 'react';

const ref = useRef(initialValue);
```

**Parameters:**
- `initialValue` - Initial value for `ref.current` (can be any type)

**Returns:**
- An object with a single property: `{ current: initialValue }`
- `ref.current` is **mutable** and persists across re-renders
- Changing `ref.current` **does NOT trigger re-render**

---

## Key Characteristics

**1. Persists Across Re-renders**

```jsx
import { useRef, useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const renderCount = useRef(0);
  
  // This runs on every render
  renderCount.current++;
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Render count: {renderCount.current}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// renderCount.current persists across re-renders
// It increments every time but doesn't cause re-render
```

**2. Doesn't Trigger Re-render**

```jsx
import { useRef, useState } from 'react';

function Component() {
  const [count, setCount] = useState(0);
  const ref = useRef(0);
  
  const handleClick = () => {
    ref.current++;
    console.log('Ref:', ref.current);  // Updates immediately
    // Component does NOT re-render!
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Ref: {ref.current}</p>  {/* Won't update in UI */}
      <button onClick={handleClick}>Update Ref</button>
      <button onClick={() => setCount(count + 1)}>Force Re-render</button>
    </div>
  );
}
```

**3. Mutable**

```jsx
import { useRef } from 'react';

function Component() {
  const ref = useRef(0);
  
  // You can directly mutate ref.current
  ref.current = 100;  // ✅ OK - won't trigger re-render
  ref.current++;      // ✅ OK
  
  return <div>{ref.current}</div>;
}
```

---

## useRef vs useState

**Comparison:**

| Feature | useState | useRef |
|---------|----------|--------|
| **Triggers re-render** | ✅ Yes | ❌ No |
| **Persists across renders** | ✅ Yes | ✅ Yes |
| **Mutable** | ❌ No (need setState) | ✅ Yes (direct mutation) |
| **Use for** | Data that affects UI | Data that doesn't affect UI |
| **Syntax** | `[value, setValue]` | `{ current: value }` |

**Example:**

```jsx
import { useState, useRef } from 'react';

function Example() {
  // useState - triggers re-render
  const [count, setCount] = useState(0);
  
  // useRef - does NOT trigger re-render
  const renderCount = useRef(0);
  
  renderCount.current++;
  
  return (
    <div>
      <p>Count: {count}</p>  {/* Updates in UI */}
      <p>Renders: {renderCount.current}</p>  {/* Updates in UI because component re-rendered */}
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

---

## Use Case 1: Accessing DOM Elements

Most common use case - directly access and manipulate DOM nodes.

**Focus Input on Mount:**

```jsx
import { useRef, useEffect } from 'react';

function LoginForm() {
  const inputRef = useRef(null);
  
  useEffect(() => {
    // Focus input when component mounts
    inputRef.current.focus();
  }, []);
  
  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Username" />
      <input type="password" placeholder="Password" />
    </div>
  );
}
```

**Scroll to Element:**

```jsx
import { useRef } from 'react';

function Page() {
  const sectionRef = useRef(null);
  
  const scrollToSection = () => {
    sectionRef.current.scrollIntoView({ behavior: 'smooth' });
  };
  
  return (
    <div>
      <button onClick={scrollToSection}>Scroll to Section</button>
      <div style={{ height: '1000px' }}>Content...</div>
      <section ref={sectionRef}>
        <h2>Target Section</h2>
      </section>
    </div>
  );
}
```

**Measure DOM Element:**

```jsx
import { useRef, useState } from 'react';

function MeasureBox() {
  const boxRef = useRef(null);
  const [dimensions, setDimensions] = useState({});
  
  const measureBox = () => {
    const box = boxRef.current;
    setDimensions({
      width: box.offsetWidth,
      height: box.offsetHeight,
      top: box.offsetTop,
      left: box.offsetLeft
    });
  };
  
  return (
    <div>
      <div 
        ref={boxRef} 
        style={{ width: '200px', height: '100px', background: 'lightblue' }}
      >
        Box
      </div>
      <button onClick={measureBox}>Measure</button>
      {dimensions.width && (
        <p>
          Width: {dimensions.width}px, Height: {dimensions.height}px
        </p>
      )}
    </div>
  );
}
```

**Play/Pause Video:**

```jsx
import { useRef } from 'react';

function VideoPlayer({ src }) {
  const videoRef = useRef(null);
  
  const handlePlay = () => {
    videoRef.current.play();
  };
  
  const handlePause = () => {
    videoRef.current.pause();
  };
  
  const handleRestart = () => {
    videoRef.current.currentTime = 0;
    videoRef.current.play();
  };
  
  return (
    <div>
      <video ref={videoRef} src={src} width="400" />
      <div>
        <button onClick={handlePlay}>Play</button>
        <button onClick={handlePause}>Pause</button>
        <button onClick={handleRestart}>Restart</button>
      </div>
    </div>
  );
}
```

---

## Use Case 2: Storing Mutable Values

Store values that should persist but don't need to trigger re-renders.

**Previous Value:**

```jsx
import { useRef, useEffect, useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();
  
  useEffect(() => {
    prevCountRef.current = count;
  }, [count]);
  
  const prevCount = prevCountRef.current;
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Custom usePrevious Hook:**

```jsx
import { useRef, useEffect } from 'react';

function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

// Usage
function Component() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Now: {count}, Before: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Store Timer ID:**

```jsx
import { useRef, useState } from 'react';

function Stopwatch() {
  const [time, setTime] = useState(0);
  const intervalRef = useRef(null);
  
  const start = () => {
    if (intervalRef.current) return; // Already running
    
    intervalRef.current = setInterval(() => {
      setTime(t => t + 1);
    }, 1000);
  };
  
  const stop = () => {
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };
  
  const reset = () => {
    stop();
    setTime(0);
  };
  
  return (
    <div>
      <p>Time: {time}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

**Track if Component is Mounted:**

```jsx
import { useRef, useEffect } from 'react';

function useIsMounted() {
  const isMountedRef = useRef(false);
  
  useEffect(() => {
    isMountedRef.current = true;
    return () => {
      isMountedRef.current = false;
    };
  }, []);
  
  return isMountedRef;
}

// Usage
function Component() {
  const isMountedRef = useIsMounted();
  
  const fetchData = async () => {
    const data = await fetch('/api/data');
    
    // Only update state if component is still mounted
    if (isMountedRef.current) {
      setData(data);
    }
  };
  
  return <div>...</div>;
}
```

---

## Use Case 3: Avoiding Stale Closures

Keep latest value accessible in callbacks.

**Problem: Stale Closure**

```jsx
import { useState, useEffect } from 'react';

function Component() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count);  // Always logs 0! (stale closure)
      setCount(count + 1); // Always sets to 1!
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);  // Empty deps - closure captures initial count (0)
  
  return <div>Count: {count}</div>;
}
```

**Solution 1: useRef**

```jsx
import { useState, useEffect, useRef } from 'react';

function Component() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  
  // Keep ref in sync
  useEffect(() => {
    countRef.current = count;
  }, [count]);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log(countRef.current);  // Always has latest value!
      setCount(countRef.current + 1);
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);  // Empty deps OK now
  
  return <div>Count: {count}</div>;
}
```

**Solution 2: Functional setState**

```jsx
import { useState, useEffect } from 'react';

function Component() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(c => c + 1);  // Use function form - gets latest count
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  return <div>Count: {count}</div>;
}
```

---

## Use Case 4: Imperative Handle (forwardRef)

Expose imperative methods to parent components.

**Custom Input Component:**

```jsx
import { useRef, useImperativeHandle, forwardRef } from 'react';

const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef();
  
  // Expose custom methods to parent
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    clear: () => {
      inputRef.current.value = '';
    },
    getValue: () => {
      return inputRef.current.value;
    }
  }));
  
  return <input ref={inputRef} {...props} />;
});

// Parent component
function Form() {
  const inputRef = useRef();
  
  const handleSubmit = () => {
    const value = inputRef.current.getValue();
    console.log('Value:', value);
    inputRef.current.clear();
  };
  
  return (
    <div>
      <CustomInput ref={inputRef} placeholder="Enter text" />
      <button onClick={handleSubmit}>Submit</button>
      <button onClick={() => inputRef.current.focus()}>Focus</button>
    </div>
  );
}
```

---

## Use Case 5: Caching Expensive Computations

**Cache Function References:**

```jsx
import { useRef } from 'react';

function Component() {
  const expensiveFunctionRef = useRef(() => {
    // Expensive computation
    return someExpensiveCalculation();
  });
  
  // Function reference never changes
  const result = expensiveFunctionRef.current();
  
  return <div>{result}</div>;
}
```

---

## Production Example: Search with Debounce

```jsx
import { useState, useRef, useEffect } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isLoading, setIsLoading] = useState(false);
  const timeoutRef = useRef(null);
  
  useEffect(() => {
    // Clear previous timeout
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    if (!query) {
      setResults([]);
      return;
    }
    
    setIsLoading(true);
    
    // Set new timeout
    timeoutRef.current = setTimeout(async () => {
      try {
        const response = await fetch(`/api/search?q=${query}`);
        const data = await response.json();
        setResults(data);
      } catch (error) {
        console.error('Search error:', error);
      } finally {
        setIsLoading(false);
      }
    }, 500); // 500ms debounce
    
    // Cleanup
    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, [query]);
  
  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      {isLoading && <div>Loading...</div>}
      <ul>
        {results.map(result => (
          <li key={result.id}>{result.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Production Example: Click Outside to Close

```jsx
import { useRef, useEffect } from 'react';

function useClickOutside(callback) {
  const ref = useRef();
  
  useEffect(() => {
    const handleClick = (event) => {
      if (ref.current && !ref.current.contains(event.target)) {
        callback();
      }
    };
    
    document.addEventListener('mousedown', handleClick);
    
    return () => {
      document.removeEventListener('mousedown', handleClick);
    };
  }, [callback]);
  
  return ref;
}

// Usage
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useClickOutside(() => setIsOpen(false));
  
  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && (
        <div className="dropdown-menu">
          <div>Option 1</div>
          <div>Option 2</div>
          <div>Option 3</div>
        </div>
      )}
    </div>
  );
}
```

---

## Production Example: Infinite Scroll

```jsx
import { useRef, useEffect, useState } from 'react';

function InfiniteScroll() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(false);
  const observerRef = useRef();
  const lastItemRef = useRef();
  
  useEffect(() => {
    const loadMore = async () => {
      setIsLoading(true);
      const response = await fetch(`/api/items?page=${page}`);
      const data = await response.json();
      setItems(prev => [...prev, ...data]);
      setIsLoading(false);
    };
    
    loadMore();
  }, [page]);
  
  useEffect(() => {
    // Create intersection observer
    observerRef.current = new IntersectionObserver((entries) => {
      if (entries[0].isIntersecting && !isLoading) {
        setPage(prev => prev + 1);
      }
    });
    
    // Observe last item
    if (lastItemRef.current) {
      observerRef.current.observe(lastItemRef.current);
    }
    
    // Cleanup
    return () => {
      if (observerRef.current) {
        observerRef.current.disconnect();
      }
    };
  }, [isLoading, items]);
  
  return (
    <div>
      <ul>
        {items.map((item, index) => (
          <li 
            key={item.id}
            ref={index === items.length - 1 ? lastItemRef : null}
          >
            {item.name}
          </li>
        ))}
      </ul>
      {isLoading && <div>Loading more...</div>}
    </div>
  );
}
```

---

## Common Patterns

**1. Combining Multiple Refs:**

```jsx
import { useRef } from 'react';

function Form() {
  const inputRefs = useRef([]);
  
  const focusInput = (index) => {
    inputRefs.current[index]?.focus();
  };
  
  return (
    <div>
      <input ref={el => inputRefs.current[0] = el} />
      <input ref={el => inputRefs.current[1] = el} />
      <input ref={el => inputRefs.current[2] = el} />
      <button onClick={() => focusInput(0)}>Focus First</button>
    </div>
  );
}
```

**2. Ref Callback Pattern:**

```jsx
import { useRef, useState } from 'react';

function Component() {
  const [height, setHeight] = useState(0);
  
  const measuredRef = useRef((node) => {
    if (node !== null) {
      setHeight(node.getBoundingClientRect().height);
    }
  });
  
  return (
    <div>
      <div ref={measuredRef.current}>
        Content
      </div>
      <p>Height: {height}px</p>
    </div>
  );
}
```

---

**Best Practices:**

1. **Don't read/write ref.current during render**
```jsx
// ❌ Bad: Reading during render
function Component() {
  const ref = useRef(0);
  ref.current++;  // Don't do this!
  return <div>{ref.current}</div>;
}

// ✅ Good: Update in event handler or effect
function Component() {
  const ref = useRef(0);
  
  useEffect(() => {
    ref.current++;  // OK in effect
  });
  
  const handleClick = () => {
    ref.current++;  // OK in event handler
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

2. **Don't use ref for data that affects rendering**
```jsx
// ❌ Bad: Should use useState
function Component() {
  const countRef = useRef(0);
  
  return (
    <div>
      <p>{countRef.current}</p>  {/* Won't update! */}
      <button onClick={() => countRef.current++}>Increment</button>
    </div>
  );
}

// ✅ Good: Use useState for UI data
function Component() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>{count}</p>  {/* Updates correctly */}
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

3. **Initialize ref correctly for DOM elements**
```jsx
// ✅ Good: Initialize as null for DOM refs
const inputRef = useRef(null);

// ✅ Good: Initialize with value for data refs
const countRef = useRef(0);
const dataRef = useRef({ name: 'John' });
```

---

**Interview Tips:**
- `useRef` returns **mutable ref object** with `.current` property
- **Persists across re-renders** - value doesn't reset
- **Doesn't trigger re-render** when `.current` changes
- **Two main use cases**:
  1. **Access DOM elements** (most common) - focus, scroll, measure
  2. **Store mutable values** - timers, previous values, flags
- Different from `useState`: ref changes don't cause re-renders
- `ref.current` is **mutable** - can be changed directly
- Great for **storing timer IDs** (setTimeout, setInterval)
- Use `forwardRef` + `useImperativeHandle` to expose methods to parent
- Avoid reading/writing `ref.current` during render (use effects/handlers)
- Common pattern: **track previous value** with custom `usePrevious` hook
- Helps avoid **stale closures** in callbacks/effects
- Initial value only used on first render, ignored on subsequent renders
- Can store **any value** - primitives, objects, functions, DOM nodes
- Use `null` as initial value for DOM refs
- Multiple refs can reference same DOM node if needed

</details>

---

### 34. What is useMemo hook and when should you use it?

<details>
<summary>View Answer</summary>

**useMemo Hook**

`useMemo` is a React Hook that lets you **cache the result** of an expensive calculation between re-renders. It **memoizes** (remembers) the computed value and only recalculates when dependencies change.

**Basic Syntax:**

```jsx
import { useMemo } from 'react';

const cachedValue = useMemo(() => {
  // Expensive calculation
  return computeExpensiveValue(a, b);
}, [a, b]);  // Dependencies
```

**Parameters:**
- `calculateValue` - Function that computes the value to cache (must be pure)
- `dependencies` - Array of values that trigger recalculation when changed

**Returns:**
- Cached result of the calculation
- Recalculates only when dependencies change

---

## How useMemo Works

**Without useMemo:**

```jsx
import { useState } from 'react';

function Component() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([1, 2, 3, 4, 5]);
  
  // This runs on EVERY render (even when count changes)!
  const expensiveResult = items.reduce((sum, item) => {
    console.log('Computing...');
    return sum + item * 1000;
  }, 0);
  
  return (
    <div>
      <p>Result: {expensiveResult}</p>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// Problem: Expensive calculation runs even when only count changes!
```

**With useMemo:**

```jsx
import { useState, useMemo } from 'react';

function Component() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([1, 2, 3, 4, 5]);
  
  // Only recalculates when items changes
  const expensiveResult = useMemo(() => {
    console.log('Computing...');
    return items.reduce((sum, item) => sum + item * 1000, 0);
  }, [items]);  // Dependency: items
  
  return (
    <div>
      <p>Result: {expensiveResult}</p>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// Solution: Calculation only runs when items changes!
// Clicking "Increment" doesn't trigger recalculation
```

---

## Basic Examples

**1. Expensive Calculation**

```jsx
import { useState, useMemo } from 'react';

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

function Component() {
  const [num, setNum] = useState(10);
  const [count, setCount] = useState(0);
  
  // Memoize expensive fibonacci calculation
  const fib = useMemo(() => {
    console.log('Calculating fibonacci...');
    return fibonacci(num);
  }, [num]);
  
  return (
    <div>
      <p>Fibonacci({num}) = {fib}</p>
      <button onClick={() => setNum(num + 1)}>Increase N</button>
      
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      {/* Incrementing count doesn't recalculate fibonacci! */}
    </div>
  );
}
```

**2. Filtering Large Lists**

```jsx
import { useState, useMemo } from 'react';

function UserList({ users }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState('name');
  
  // Memoize filtered and sorted users
  const filteredUsers = useMemo(() => {
    console.log('Filtering and sorting...');
    
    let result = users.filter(user =>
      user.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
    
    result.sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      if (sortBy === 'age') return a.age - b.age;
      return 0;
    });
    
    return result;
  }, [users, searchTerm, sortBy]);  // Recalculate when any of these change
  
  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search..."
      />
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
        <option value="name">Name</option>
        <option value="age">Age</option>
      </select>
      
      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>{user.name} - {user.age}</li>
        ))}
      </ul>
    </div>
  );
}
```

**3. Complex Object Creation**

```jsx
import { useMemo } from 'react';

function Chart({ data, options }) {
  // Memoize chart configuration
  const chartConfig = useMemo(() => {
    console.log('Creating chart config...');
    return {
      type: 'line',
      data: {
        labels: data.map(d => d.label),
        datasets: [{
          label: 'Sales',
          data: data.map(d => d.value),
          borderColor: options.color
        }]
      },
      options: {
        responsive: true,
        animation: options.animate
      }
    };
  }, [data, options.color, options.animate]);
  
  return <LineChart config={chartConfig} />;
}
```

---

## When to Use useMemo

**✅ Good Use Cases:**

**1. Expensive Calculations**
```jsx
const sortedData = useMemo(() => {
  return data.sort((a, b) => a.value - b.value);
}, [data]);
```

**2. Referential Equality**
```jsx
// Object/array created every render
const config = { theme: 'dark', size: 'large' };

// Memoized - same reference if dependencies don't change
const config = useMemo(() => ({
  theme: 'dark',
  size: 'large'
}), []);
```

**3. Preventing Child Re-renders**
```jsx
const memoizedProps = useMemo(() => ({
  data: processData(rawData),
  onUpdate: handleUpdate
}), [rawData]);

<ExpensiveComponent {...memoizedProps} />
```

**4. Derived State**
```jsx
const stats = useMemo(() => ({
  total: items.length,
  completed: items.filter(i => i.done).length,
  pending: items.filter(i => !i.done).length
}), [items]);
```

---

**❌ Don't Use When:**

**1. Simple Calculations**
```jsx
// ❌ Bad: Unnecessary memoization
const sum = useMemo(() => a + b, [a, b]);

// ✅ Good: Just calculate directly
const sum = a + b;
```

**2. Values Used Only Once**
```jsx
// ❌ Bad: Memoizing when not reused
const fullName = useMemo(() => `${firstName} ${lastName}`, [firstName, lastName]);

return <div>{fullName}</div>;  // Only used once

// ✅ Good: Direct calculation
return <div>{`${firstName} ${lastName}`}</div>;
```

**3. Premature Optimization**
```jsx
// ❌ Bad: Memoizing everything "just in case"
const a = useMemo(() => x + 1, [x]);
const b = useMemo(() => y * 2, [y]);
const c = useMemo(() => z - 3, [z]);

// ✅ Good: Only optimize when needed
const a = x + 1;
const b = y * 2;
const c = z - 3;
```

---

## Referential Equality Problem

**Problem: New object/array on every render**

```jsx
import { useState } from 'react';
import React from 'react';

const ExpensiveComponent = React.memo(({ config }) => {
  console.log('ExpensiveComponent rendered');
  return <div>{config.theme}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  // New object created every render!
  const config = { theme: 'dark', size: 'large' };
  
  return (
    <div>
      <ExpensiveComponent config={config} />  {/* Always re-renders! */}
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
}

// Problem: config is new object every time, so ExpensiveComponent re-renders
// even though the values inside config haven't changed
```

**Solution: useMemo for referential equality**

```jsx
import { useState, useMemo } from 'react';
import React from 'react';

const ExpensiveComponent = React.memo(({ config }) => {
  console.log('ExpensiveComponent rendered');
  return <div>{config.theme}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  // Same object reference if dependencies don't change
  const config = useMemo(() => ({
    theme: 'dark',
    size: 'large'
  }), []);  // Empty deps - never changes
  
  return (
    <div>
      <ExpensiveComponent config={config} />  {/* Doesn't re-render! */}
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
}

// Solution: config has same reference, so ExpensiveComponent doesn't re-render
```

---

## Production Example: Data Table

```jsx
import { useState, useMemo } from 'react';
import React from 'react';

function DataTable({ data }) {
  const [sortColumn, setSortColumn] = useState('name');
  const [sortDirection, setSortDirection] = useState('asc');
  const [searchTerm, setSearchTerm] = useState('');
  const [filterStatus, setFilterStatus] = useState('all');
  const [currentPage, setCurrentPage] = useState(1);
  const itemsPerPage = 10;
  
  // 1. Filter data
  const filteredData = useMemo(() => {
    console.log('Filtering data...');
    return data.filter(item => {
      const matchesSearch = item.name.toLowerCase().includes(searchTerm.toLowerCase());
      const matchesStatus = filterStatus === 'all' || item.status === filterStatus;
      return matchesSearch && matchesStatus;
    });
  }, [data, searchTerm, filterStatus]);
  
  // 2. Sort data
  const sortedData = useMemo(() => {
    console.log('Sorting data...');
    const sorted = [...filteredData].sort((a, b) => {
      const aVal = a[sortColumn];
      const bVal = b[sortColumn];
      
      if (typeof aVal === 'string') {
        return sortDirection === 'asc'
          ? aVal.localeCompare(bVal)
          : bVal.localeCompare(aVal);
      }
      
      return sortDirection === 'asc' ? aVal - bVal : bVal - aVal;
    });
    return sorted;
  }, [filteredData, sortColumn, sortDirection]);
  
  // 3. Paginate data
  const paginatedData = useMemo(() => {
    console.log('Paginating data...');
    const startIndex = (currentPage - 1) * itemsPerPage;
    const endIndex = startIndex + itemsPerPage;
    return sortedData.slice(startIndex, endIndex);
  }, [sortedData, currentPage, itemsPerPage]);
  
  // 4. Calculate statistics
  const stats = useMemo(() => {
    console.log('Calculating stats...');
    return {
      total: data.length,
      filtered: filteredData.length,
      active: filteredData.filter(item => item.status === 'active').length,
      inactive: filteredData.filter(item => item.status === 'inactive').length,
      totalPages: Math.ceil(sortedData.length / itemsPerPage)
    };
  }, [data, filteredData, sortedData, itemsPerPage]);
  
  const handleSort = (column) => {
    if (sortColumn === column) {
      setSortDirection(sortDirection === 'asc' ? 'desc' : 'asc');
    } else {
      setSortColumn(column);
      setSortDirection('asc');
    }
  };
  
  return (
    <div>
      <div className="controls">
        <input
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          placeholder="Search..."
        />
        <select value={filterStatus} onChange={(e) => setFilterStatus(e.target.value)}>
          <option value="all">All</option>
          <option value="active">Active</option>
          <option value="inactive">Inactive</option>
        </select>
      </div>
      
      <div className="stats">
        <span>Total: {stats.total}</span>
        <span>Filtered: {stats.filtered}</span>
        <span>Active: {stats.active}</span>
        <span>Inactive: {stats.inactive}</span>
      </div>
      
      <table>
        <thead>
          <tr>
            <th onClick={() => handleSort('name')}>
              Name {sortColumn === 'name' && (sortDirection === 'asc' ? '↑' : '↓')}
            </th>
            <th onClick={() => handleSort('age')}>
              Age {sortColumn === 'age' && (sortDirection === 'asc' ? '↑' : '↓')}
            </th>
            <th onClick={() => handleSort('status')}>
              Status {sortColumn === 'status' && (sortDirection === 'asc' ? '↑' : '↓')}
            </th>
          </tr>
        </thead>
        <tbody>
          {paginatedData.map(item => (
            <tr key={item.id}>
              <td>{item.name}</td>
              <td>{item.age}</td>
              <td>{item.status}</td>
            </tr>
          ))}
        </tbody>
      </table>
      
      <div className="pagination">
        <button
          onClick={() => setCurrentPage(currentPage - 1)}
          disabled={currentPage === 1}
        >
          Previous
        </button>
        <span>Page {currentPage} of {stats.totalPages}</span>
        <button
          onClick={() => setCurrentPage(currentPage + 1)}
          disabled={currentPage === stats.totalPages}
        >
          Next
        </button>
      </div>
    </div>
  );
}
```

---

## Production Example: Shopping Cart

```jsx
import { useMemo } from 'react';

function ShoppingCart({ items, taxRate, discountCode }) {
  // Calculate subtotal
  const subtotal = useMemo(() => {
    console.log('Calculating subtotal...');
    return items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }, [items]);
  
  // Apply discount
  const discount = useMemo(() => {
    console.log('Calculating discount...');
    if (discountCode === 'SAVE10') return subtotal * 0.1;
    if (discountCode === 'SAVE20') return subtotal * 0.2;
    return 0;
  }, [subtotal, discountCode]);
  
  // Calculate tax
  const tax = useMemo(() => {
    console.log('Calculating tax...');
    return (subtotal - discount) * taxRate;
  }, [subtotal, discount, taxRate]);
  
  // Calculate total
  const total = useMemo(() => {
    console.log('Calculating total...');
    return subtotal - discount + tax;
  }, [subtotal, discount, tax]);
  
  // Calculate savings
  const savings = useMemo(() => {
    const originalTotal = items.reduce((sum, item) => {
      const originalPrice = item.originalPrice || item.price;
      return sum + (originalPrice * item.quantity);
    }, 0);
    return originalTotal - total;
  }, [items, total]);
  
  return (
    <div className="shopping-cart">
      <h2>Shopping Cart</h2>
      
      <div className="items">
        {items.map(item => (
          <div key={item.id} className="item">
            <span>{item.name}</span>
            <span>{item.quantity} x ${item.price}</span>
            <span>${item.price * item.quantity}</span>
          </div>
        ))}
      </div>
      
      <div className="summary">
        <div>Subtotal: ${subtotal.toFixed(2)}</div>
        {discount > 0 && (
          <div className="discount">Discount: -${discount.toFixed(2)}</div>
        )}
        <div>Tax ({(taxRate * 100).toFixed(1)}%): ${tax.toFixed(2)}</div>
        <div className="total">Total: ${total.toFixed(2)}</div>
        {savings > 0 && (
          <div className="savings">You saved: ${savings.toFixed(2)}</div>
        )}
      </div>
    </div>
  );
}
```

---

## useMemo vs useCallback

**useMemo** - Memoizes **value**
```jsx
const value = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

**useCallback** - Memoizes **function**
```jsx
const callback = useCallback(() => doSomething(a, b), [a, b]);
```

**Equivalence:**
```jsx
// These are equivalent:
const callback = useCallback(fn, deps);
const callback = useMemo(() => fn, deps);
```

---

**Best Practices:**

1. **Only memoize when needed**
```jsx
// ❌ Premature optimization
const sum = useMemo(() => a + b, [a, b]);

// ✅ Optimize expensive operations
const sorted = useMemo(() => largeArray.sort(), [largeArray]);
```

2. **List all dependencies**
```jsx
// ❌ Missing dependency
const result = useMemo(() => compute(a, b), [a]);  // Missing b!

// ✅ Complete dependencies
const result = useMemo(() => compute(a, b), [a, b]);
```

3. **Don't memoize primitives unnecessarily**
```jsx
// ❌ Unnecessary
const doubled = useMemo(() => count * 2, [count]);

// ✅ Direct calculation
const doubled = count * 2;
```

4. **Combine with React.memo for components**
```jsx
const MyComponent = React.memo(({ data }) => {
  const processed = useMemo(() => processData(data), [data]);
  return <div>{processed}</div>;
});
```

---

**Interview Tips:**
- `useMemo` **caches/memoizes** the result of expensive calculations
- Returns **cached value** until dependencies change
- **Prevents unnecessary recalculations** on every render
- Use for **expensive computations**, **filtering/sorting large lists**, **complex object creation**
- Helps with **referential equality** - same object/array reference across renders
- Important for **preventing child re-renders** when passing objects/arrays as props
- Don't overuse - **premature optimization** can hurt performance
- **Not for simple calculations** like `a + b` or string concatenation
- Dependencies array determines **when to recalculate**
- Runs during render, so function **must be pure** (no side effects)
- Similar to `useCallback` but memoizes **values** instead of **functions**
- Equivalence: `useMemo(() => fn, deps)` = `useCallback(fn, deps)`
- Use with `React.memo` for **maximum optimization** of child components
- **Profile first** - only optimize when you have measurable performance issues
- Empty dependency array `[]` means value **never recalculates**
- React may **discard memoized value** during memory pressure (don't rely on it for correctness)

</details>

---

### 35. What is useCallback hook and how does it differ from useMemo?

<details>
<summary>View Answer</summary>

**useCallback Hook**

`useCallback` is a React Hook that lets you **cache a function definition** between re-renders. It returns a **memoized callback function** that only changes when dependencies change.

**Basic Syntax:**

```jsx
import { useCallback } from 'react';

const cachedFn = useCallback(() => {
  // Function logic
}, [dependencies]);
```

**Parameters:**
- `fn` - The function to memoize
- `dependencies` - Array of values that trigger function recreation when changed

**Returns:**
- Same function reference until dependencies change
- New function only created when dependencies change

---

## Why useCallback?

**Problem: Functions recreated on every render**

```jsx
import React, { useState } from 'react';

const ExpensiveChild = React.memo(({ onClick }) => {
  console.log('ExpensiveChild rendered');
  return <button onClick={onClick}>Click Me</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  // New function created on EVERY render!
  const handleClick = () => {
    console.log('Button clicked');
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <ExpensiveChild onClick={handleClick} />  {/* Always re-renders! */}
    </div>
  );
}

// Problem: handleClick is new function every time
// ExpensiveChild re-renders even with React.memo because onClick prop changes
```

**Solution: useCallback**

```jsx
import React, { useState, useCallback } from 'react';

const ExpensiveChild = React.memo(({ onClick }) => {
  console.log('ExpensiveChild rendered');
  return <button onClick={onClick}>Click Me</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  // Same function reference across renders
  const handleClick = useCallback(() => {
    console.log('Button clicked');
  }, []);  // Empty deps - function never changes
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <ExpensiveChild onClick={handleClick} />  {/* Doesn't re-render! */}
    </div>
  );
}

// Solution: handleClick has same reference
// ExpensiveChild doesn't re-render when count changes
```

---

## useCallback vs useMemo

**Key Difference:**
- `useCallback` - Memoizes the **function itself**
- `useMemo` - Memoizes the **return value** of a function

**Comparison:**

```jsx
import { useCallback, useMemo } from 'react';

// useCallback - returns the function
const handleClick = useCallback(() => {
  doSomething(a, b);
}, [a, b]);

// useMemo - returns the result of calling the function
const value = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);
```

**Equivalence:**

```jsx
// These are equivalent:
const memoizedCallback = useCallback(fn, deps);
const memoizedCallback = useMemo(() => fn, deps);

// useCallback is syntactic sugar for useMemo with a function
```

**Side-by-Side Example:**

```jsx
import { useState, useCallback, useMemo } from 'react';

function Component() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([1, 2, 3, 4, 5]);
  
  // useCallback - memoize function
  const handleIncrement = useCallback(() => {
    setCount(c => c + 1);
  }, []);  // Function reference never changes
  
  // useMemo - memoize computed value
  const total = useMemo(() => {
    return items.reduce((sum, item) => sum + item, 0);
  }, [items]);  // Value recalculated only when items change
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Total: {total}</p>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
}
```

**Comparison Table:**

| Feature | useCallback | useMemo |
|---------|-------------|----------|
| **Memoizes** | Function reference | Computed value |
| **Returns** | Same function | Result of function call |
| **Use For** | Event handlers, callbacks | Expensive calculations |
| **Syntax** | `useCallback(fn, deps)` | `useMemo(() => value, deps)` |
| **Example** | `onClick`, `onChange` | Sorted arrays, filtered lists |

---

## Basic Examples

**1. Preventing Child Re-renders**

```jsx
import React, { useState, useCallback } from 'react';

const Button = React.memo(({ onClick, children }) => {
  console.log('Button rendered:', children);
  return <button onClick={onClick}>{children}</button>;
});

function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  // Without useCallback - new function every render
  // const addTodo = () => {
  //   setTodos([...todos, { id: Date.now(), text: input }]);
  //   setInput('');
  // };
  
  // With useCallback - same function reference
  const addTodo = useCallback(() => {
    setTodos(prev => [...prev, { id: Date.now(), text: input }]);
    setInput('');
  }, [input]);  // Recreate when input changes
  
  const clearTodos = useCallback(() => {
    setTodos([]);
  }, []);
  
  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <Button onClick={addTodo}>Add Todo</Button>
      <Button onClick={clearTodos}>Clear All</Button>
      {/* clearTodos button doesn't re-render when typing! */}
    </div>
  );
}
```

**2. Event Handlers with Dependencies**

```jsx
import { useState, useCallback } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  const [category, setCategory] = useState('all');
  
  const handleSearch = useCallback(() => {
    console.log('Searching:', query, 'in', category);
    // API call with current query and category
    fetch(`/api/search?q=${query}&category=${category}`);
  }, [query, category]);  // Recreate when query or category changes
  
  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <select value={category} onChange={(e) => setCategory(e.target.value)}>
        <option value="all">All</option>
        <option value="books">Books</option>
        <option value="movies">Movies</option>
      </select>
      <button onClick={handleSearch}>Search</button>
    </div>
  );
}
```

**3. Callback Props**

```jsx
import React, { useState, useCallback } from 'react';

const ListItem = React.memo(({ item, onDelete }) => {
  console.log('ListItem rendered:', item.id);
  return (
    <li>
      {item.text}
      <button onClick={() => onDelete(item.id)}>Delete</button>
    </li>
  );
});

function List() {
  const [items, setItems] = useState([
    { id: 1, text: 'Item 1' },
    { id: 2, text: 'Item 2' },
    { id: 3, text: 'Item 3' }
  ]);
  
  const handleDelete = useCallback((id) => {
    setItems(prev => prev.filter(item => item.id !== id));
  }, []);  // No dependencies - uses functional update
  
  return (
    <ul>
      {items.map(item => (
        <ListItem key={item.id} item={item} onDelete={handleDelete} />
      ))}
    </ul>
  );
}
```

---

## Common Patterns

**Pattern 1: useCallback with useEffect**

```jsx
import { useState, useCallback, useEffect } from 'react';

function DataFetcher({ userId }) {
  const [data, setData] = useState(null);
  
  // Memoize fetch function
  const fetchData = useCallback(async () => {
    const response = await fetch(`/api/users/${userId}`);
    const data = await response.json();
    setData(data);
  }, [userId]);  // Recreate when userId changes
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);  // Safe to include in dependencies
  
  return <div>{data ? data.name : 'Loading...'}</div>;
}
```

**Pattern 2: Multiple Callbacks**

```jsx
import { useState, useCallback } from 'react';

function Form() {
  const [values, setValues] = useState({ name: '', email: '', age: '' });
  
  const handleChange = useCallback((field) => (e) => {
    setValues(prev => ({ ...prev, [field]: e.target.value }));
  }, []);
  
  const handleSubmit = useCallback((e) => {
    e.preventDefault();
    console.log('Submit:', values);
  }, [values]);
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={values.name} onChange={handleChange('name')} />
      <input value={values.email} onChange={handleChange('email')} />
      <input value={values.age} onChange={handleChange('age')} />
      <button type="submit">Submit</button>
    </form>
  );
}
```

**Pattern 3: Callback with Context**

```jsx
import { createContext, useContext, useState, useCallback } from 'react';

const TodoContext = createContext();

function TodoProvider({ children }) {
  const [todos, setTodos] = useState([]);
  
  const addTodo = useCallback((text) => {
    setTodos(prev => [...prev, { id: Date.now(), text, completed: false }]);
  }, []);
  
  const toggleTodo = useCallback((id) => {
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, []);
  
  const deleteTodo = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);
  
  return (
    <TodoContext.Provider value={{ todos, addTodo, toggleTodo, deleteTodo }}>
      {children}
    </TodoContext.Provider>
  );
}
```

---

## Production Example: Optimized List

```jsx
import React, { useState, useCallback, useMemo } from 'react';

const TodoItem = React.memo(({ todo, onToggle, onDelete, onEdit }) => {
  console.log('TodoItem rendered:', todo.id);
  
  return (
    <li className={todo.completed ? 'completed' : ''}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span>{todo.text}</span>
      <button onClick={() => onEdit(todo.id, prompt('Edit:', todo.text))}>Edit</button>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
});

function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build app', completed: false },
    { id: 3, text: 'Deploy', completed: false }
  ]);
  const [filter, setFilter] = useState('all');
  const [input, setInput] = useState('');
  
  // Memoized callbacks - stable references
  const addTodo = useCallback(() => {
    if (input.trim()) {
      setTodos(prev => [
        ...prev,
        { id: Date.now(), text: input, completed: false }
      ]);
      setInput('');
    }
  }, [input]);
  
  const toggleTodo = useCallback((id) => {
    setTodos(prev =>
      prev.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  }, []);
  
  const deleteTodo = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);
  
  const editTodo = useCallback((id, newText) => {
    if (newText && newText.trim()) {
      setTodos(prev =>
        prev.map(todo =>
          todo.id === id ? { ...todo, text: newText } : todo
        )
      );
    }
  }, []);
  
  const clearCompleted = useCallback(() => {
    setTodos(prev => prev.filter(todo => !todo.completed));
  }, []);
  
  // Memoized filtered list
  const filteredTodos = useMemo(() => {
    console.log('Filtering todos...');
    if (filter === 'active') return todos.filter(t => !t.completed);
    if (filter === 'completed') return todos.filter(t => t.completed);
    return todos;
  }, [todos, filter]);
  
  // Memoized stats
  const stats = useMemo(() => ({
    total: todos.length,
    completed: todos.filter(t => t.completed).length,
    active: todos.filter(t => !t.completed).length
  }), [todos]);
  
  return (
    <div className="todo-app">
      <h1>Todo List</h1>
      
      <div className="input-section">
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="Add a todo..."
        />
        <button onClick={addTodo}>Add</button>
      </div>
      
      <div className="filters">
        <button onClick={() => setFilter('all')}>All ({stats.total})</button>
        <button onClick={() => setFilter('active')}>Active ({stats.active})</button>
        <button onClick={() => setFilter('completed')}>Completed ({stats.completed})</button>
      </div>
      
      <ul className="todo-list">
        {filteredTodos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={toggleTodo}
            onDelete={deleteTodo}
            onEdit={editTodo}
          />
        ))}
      </ul>
      
      {stats.completed > 0 && (
        <button onClick={clearCompleted}>Clear Completed</button>
      )}
    </div>
  );
}
```

---

## Production Example: Form with Validation

```jsx
import { useState, useCallback, useMemo } from 'react';

function SignupForm() {
  const [values, setValues] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  });
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  // Validation function
  const validate = useCallback((name, value) => {
    switch (name) {
      case 'username':
        if (!value) return 'Username is required';
        if (value.length < 3) return 'Username must be at least 3 characters';
        break;
      case 'email':
        if (!value) return 'Email is required';
        if (!/\S+@\S+\.\S+/.test(value)) return 'Email is invalid';
        break;
      case 'password':
        if (!value) return 'Password is required';
        if (value.length < 8) return 'Password must be at least 8 characters';
        break;
      case 'confirmPassword':
        if (value !== values.password) return 'Passwords do not match';
        break;
    }
    return '';
  }, [values.password]);
  
  const handleChange = useCallback((name) => (e) => {
    const value = e.target.value;
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Validate on change if field was touched
    if (touched[name]) {
      const error = validate(name, value);
      setErrors(prev => ({ ...prev, [name]: error }));
    }
  }, [touched, validate]);
  
  const handleBlur = useCallback((name) => () => {
    setTouched(prev => ({ ...prev, [name]: true }));
    const error = validate(name, values[name]);
    setErrors(prev => ({ ...prev, [name]: error }));
  }, [values, validate]);
  
  const handleSubmit = useCallback((e) => {
    e.preventDefault();
    
    // Validate all fields
    const newErrors = {};
    Object.keys(values).forEach(key => {
      const error = validate(key, values[key]);
      if (error) newErrors[key] = error;
    });
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      setTouched({ username: true, email: true, password: true, confirmPassword: true });
      return;
    }
    
    console.log('Form submitted:', values);
  }, [values, validate]);
  
  const isValid = useMemo(() => {
    return Object.keys(values).every(key => !validate(key, values[key]));
  }, [values, validate]);
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="username"
          value={values.username}
          onChange={handleChange('username')}
          onBlur={handleBlur('username')}
          placeholder="Username"
        />
        {touched.username && errors.username && (
          <span className="error">{errors.username}</span>
        )}
      </div>
      
      <div>
        <input
          name="email"
          value={values.email}
          onChange={handleChange('email')}
          onBlur={handleBlur('email')}
          placeholder="Email"
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>
      
      <div>
        <input
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange('password')}
          onBlur={handleBlur('password')}
          placeholder="Password"
        />
        {touched.password && errors.password && (
          <span className="error">{errors.password}</span>
        )}
      </div>
      
      <div>
        <input
          type="password"
          name="confirmPassword"
          value={values.confirmPassword}
          onChange={handleChange('confirmPassword')}
          onBlur={handleBlur('confirmPassword')}
          placeholder="Confirm Password"
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span className="error">{errors.confirmPassword}</span>
        )}
      </div>
      
      <button type="submit" disabled={!isValid}>Sign Up</button>
    </form>
  );
}
```

---

## When to Use useCallback

**✅ Use useCallback when:**

1. **Passing callbacks to optimized child components**
```jsx
<ExpensiveChild onClick={memoizedCallback} />
```

2. **Callback is used as dependency in useEffect/useMemo**
```jsx
useEffect(() => {
  fetchData();
}, [fetchData]);  // fetchData should be memoized
```

3. **Creating event handlers in loops**
```jsx
{items.map(item => (
  <Item key={item.id} onDelete={handleDelete} />
))}
```

4. **Passing callbacks to context**
```jsx
<Context.Provider value={{ callback1, callback2 }}>
```

**❌ Don't use when:**

1. **Callback used only in same component**
```jsx
// ❌ Unnecessary
const handleClick = useCallback(() => alert('Hi'), []);
<button onClick={handleClick}>Click</button>

// ✅ Direct
<button onClick={() => alert('Hi')}>Click</button>
```

2. **Premature optimization**
```jsx
// ❌ Overkill
const add = useCallback(() => a + b, [a, b]);
```

3. **Inline event handlers**
```jsx
// ❌ Don't memoize inline
<button onClick={useCallback(() => alert('Hi'), [])}>Click</button>
```

---

**Best Practices:**

1. **Combine with React.memo**
```jsx
const Child = React.memo(({ onClick }) => {
  return <button onClick={onClick}>Click</button>;
});

const memoizedCallback = useCallback(() => {...}, []);
<Child onClick={memoizedCallback} />
```

2. **Use functional updates**
```jsx
// ✅ No dependencies needed
const increment = useCallback(() => {
  setCount(c => c + 1);
}, []);

// ❌ Needs count in dependencies
const increment = useCallback(() => {
  setCount(count + 1);
}, [count]);
```

3. **List all dependencies**
```jsx
// ❌ Missing dependency
const callback = useCallback(() => {
  doSomething(a, b);
}, [a]);  // Missing b!

// ✅ Complete
const callback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

4. **Don't overuse**
```jsx
// ❌ Too much memoization
const handleClick = useCallback(() => alert('Hi'), []);
const handleHover = useCallback(() => console.log('Hover'), []);
const handleFocus = useCallback(() => console.log('Focus'), []);

// ✅ Only memoize what matters
<button onClick={() => alert('Hi')}>Click</button>
```

---

**Interview Tips:**
- `useCallback` **memoizes function reference** to prevent recreation on every render
- Returns **same function** until dependencies change
- Main purpose: **prevent child re-renders** when passing callbacks as props
- Use with **React.memo** for maximum optimization
- Different from `useMemo`: **memoizes function vs return value**
- Equivalence: `useCallback(fn, deps)` = `useMemo(() => fn, deps)`
- Common use: **event handlers**, **callbacks passed to children**, **dependencies in useEffect**
- Use **functional updates** (`setState(prev => ...)`) to avoid dependencies
- Don't overuse - **only optimize when needed** (child re-renders, deps in effects)
- Dependencies array determines **when function is recreated**
- Empty deps `[]` means function **never changes**
- Must include **all values used inside** the callback in dependencies
- Helps prevent **unnecessary re-renders** of memoized child components
- Not needed for inline handlers in same component
- Profile first - premature optimization can make code harder to read

</details>

---

### 36. What is useLayoutEffect and how is it different from useEffect?

<details>
<summary>View Answer</summary>

**useLayoutEffect Hook**

`useLayoutEffect` is a React Hook that fires **synchronously** after all DOM mutations but **before the browser paints**. It's identical to `useEffect` in API but different in timing.

**Basic Syntax:**

```jsx
import { useLayoutEffect } from 'react';

useLayoutEffect(() => {
  // Effect logic
  
  return () => {
    // Cleanup
  };
}, [dependencies]);
```

**Same API as useEffect:**
- Takes effect function and dependency array
- Can return cleanup function
- Runs on mount and when dependencies change

---

## useLayoutEffect vs useEffect

**Timing Difference:**

```
Render Phase:
1. React updates DOM
2. ✅ useLayoutEffect runs (blocks paint)
3. Browser paints screen
4. ✅ useEffect runs (after paint)
```

**Visual Timeline:**

```jsx
Component renders
   ↓
DOM updated
   ↓
useLayoutEffect runs ← SYNCHRONOUS (blocks paint)
   ↓
Browser paints screen ← User sees changes
   ↓
useEffect runs ← ASYNCHRONOUS (after paint)
```

**Comparison Table:**

| Feature | useEffect | useLayoutEffect |
|---------|-----------|------------------|
| **Timing** | After paint | Before paint |
| **Execution** | Asynchronous | Synchronous |
| **Blocks painting** | No | Yes |
| **User sees** | Changes before effect | Changes after effect |
| **Use for** | Most side effects | DOM measurements, mutations |
| **Performance** | Better (non-blocking) | Can slow down if heavy |

---

## Why useLayoutEffect?

**Problem with useEffect: Visual Flicker**

```jsx
import { useState, useEffect, useRef } from 'react';

function Tooltip() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const tooltipRef = useRef();
  
  useEffect(() => {
    // This runs AFTER browser paints
    const rect = tooltipRef.current.getBoundingClientRect();
    
    // Adjust position if tooltip goes off screen
    if (rect.right > window.innerWidth) {
      setPosition({ x: window.innerWidth - rect.width, y: 0 });
    }
  }, []);
  
  return (
    <div 
      ref={tooltipRef}
      style={{ position: 'absolute', left: position.x, top: position.y }}
    >
      Tooltip Content
    </div>
  );
}

// Problem: User sees tooltip flash in wrong position, then jump to correct position!
// 1. Tooltip renders at (0, 0)
// 2. Browser paints it at (0, 0) ← USER SEES THIS
// 3. useEffect runs, adjusts position
// 4. Browser paints again at new position ← FLICKER!
```

**Solution with useLayoutEffect:**

```jsx
import { useState, useLayoutEffect, useRef } from 'react';

function Tooltip() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const tooltipRef = useRef();
  
  useLayoutEffect(() => {
    // This runs BEFORE browser paints
    const rect = tooltipRef.current.getBoundingClientRect();
    
    // Adjust position if tooltip goes off screen
    if (rect.right > window.innerWidth) {
      setPosition({ x: window.innerWidth - rect.width, y: 0 });
    }
  }, []);
  
  return (
    <div 
      ref={tooltipRef}
      style={{ position: 'absolute', left: position.x, top: position.y }}
    >
      Tooltip Content
    </div>
  );
}

// Solution: Position adjusted before paint - no flicker!
// 1. Tooltip renders at (0, 0)
// 2. useLayoutEffect runs, adjusts position ← BEFORE PAINT
// 3. Browser paints at correct position ← USER SEES CORRECT POSITION
```

---

## Use Cases for useLayoutEffect

**1. Reading Layout/Measuring DOM**

```jsx
import { useState, useLayoutEffect, useRef } from 'react';

function DynamicHeight() {
  const [height, setHeight] = useState(0);
  const divRef = useRef();
  
  useLayoutEffect(() => {
    // Measure DOM element before paint
    const rect = divRef.current.getBoundingClientRect();
    setHeight(rect.height);
  }, []);
  
  return (
    <div>
      <div ref={divRef}>Content with dynamic height</div>
      <p>Height: {height}px</p>
    </div>
  );
}
```

**2. Mutating DOM Before Paint**

```jsx
import { useLayoutEffect, useRef } from 'react';

function ScrollToBottom() {
  const listRef = useRef();
  
  useLayoutEffect(() => {
    // Scroll to bottom before user sees the list
    listRef.current.scrollTop = listRef.current.scrollHeight;
  });
  
  return (
    <div ref={listRef} style={{ height: '300px', overflow: 'auto' }}>
      {messages.map(msg => (
        <div key={msg.id}>{msg.text}</div>
      ))}
    </div>
  );
}
```

**3. Synchronizing with DOM**

```jsx
import { useLayoutEffect, useRef } from 'react';

function AutoResizeTextarea({ value }) {
  const textareaRef = useRef();
  
  useLayoutEffect(() => {
    // Adjust height based on content before paint
    const textarea = textareaRef.current;
    textarea.style.height = 'auto';
    textarea.style.height = `${textarea.scrollHeight}px`;
  }, [value]);
  
  return (
    <textarea
      ref={textareaRef}
      value={value}
      style={{ resize: 'none', overflow: 'hidden' }}
    />
  );
}
```

**4. Positioning Elements**

```jsx
import { useState, useLayoutEffect, useRef } from 'react';

function Dropdown({ isOpen, children }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const dropdownRef = useRef();
  const triggerRef = useRef();
  
  useLayoutEffect(() => {
    if (isOpen && dropdownRef.current && triggerRef.current) {
      const triggerRect = triggerRef.current.getBoundingClientRect();
      const dropdownRect = dropdownRef.current.getBoundingClientRect();
      
      let top = triggerRect.bottom;
      let left = triggerRect.left;
      
      // Adjust if goes off screen
      if (left + dropdownRect.width > window.innerWidth) {
        left = window.innerWidth - dropdownRect.width;
      }
      
      if (top + dropdownRect.height > window.innerHeight) {
        top = triggerRect.top - dropdownRect.height;
      }
      
      setPosition({ top, left });
    }
  }, [isOpen]);
  
  return (
    <div>
      <button ref={triggerRef}>Toggle</button>
      {isOpen && (
        <div
          ref={dropdownRef}
          style={{
            position: 'fixed',
            top: position.top,
            left: position.left
          }}
        >
          {children}
        </div>
      )}
    </div>
  );
}
```

---

## Production Example: Tooltip Positioning

```jsx
import { useState, useLayoutEffect, useRef } from 'react';

function Tooltip({ children, content, position = 'top' }) {
  const [isVisible, setIsVisible] = useState(false);
  const [tooltipStyle, setTooltipStyle] = useState({});
  const triggerRef = useRef();
  const tooltipRef = useRef();
  
  useLayoutEffect(() => {
    if (isVisible && tooltipRef.current && triggerRef.current) {
      const triggerRect = triggerRef.current.getBoundingClientRect();
      const tooltipRect = tooltipRef.current.getBoundingClientRect();
      
      let top, left;
      
      switch (position) {
        case 'top':
          top = triggerRect.top - tooltipRect.height - 8;
          left = triggerRect.left + (triggerRect.width - tooltipRect.width) / 2;
          break;
        case 'bottom':
          top = triggerRect.bottom + 8;
          left = triggerRect.left + (triggerRect.width - tooltipRect.width) / 2;
          break;
        case 'left':
          top = triggerRect.top + (triggerRect.height - tooltipRect.height) / 2;
          left = triggerRect.left - tooltipRect.width - 8;
          break;
        case 'right':
          top = triggerRect.top + (triggerRect.height - tooltipRect.height) / 2;
          left = triggerRect.right + 8;
          break;
      }
      
      // Keep tooltip in viewport
      if (left < 0) left = 8;
      if (left + tooltipRect.width > window.innerWidth) {
        left = window.innerWidth - tooltipRect.width - 8;
      }
      if (top < 0) top = 8;
      if (top + tooltipRect.height > window.innerHeight) {
        top = window.innerHeight - tooltipRect.height - 8;
      }
      
      setTooltipStyle({
        position: 'fixed',
        top: `${top}px`,
        left: `${left}px`,
        zIndex: 1000
      });
    }
  }, [isVisible, position]);
  
  return (
    <>
      <span
        ref={triggerRef}
        onMouseEnter={() => setIsVisible(true)}
        onMouseLeave={() => setIsVisible(false)}
      >
        {children}
      </span>
      {isVisible && (
        <div ref={tooltipRef} style={tooltipStyle} className="tooltip">
          {content}
        </div>
      )}
    </>
  );
}

// Usage
function App() {
  return (
    <div>
      <Tooltip content="This is a tooltip" position="top">
        <button>Hover me</button>
      </Tooltip>
    </div>
  );
}
```

---

## Production Example: Modal with Auto-focus

```jsx
import { useLayoutEffect, useRef } from 'react';

function Modal({ isOpen, onClose, children }) {
  const modalRef = useRef();
  const previousActiveElement = useRef();
  
  useLayoutEffect(() => {
    if (isOpen) {
      // Save previously focused element
      previousActiveElement.current = document.activeElement;
      
      // Focus modal before paint
      const focusableElements = modalRef.current.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      
      if (focusableElements.length > 0) {
        focusableElements[0].focus();
      } else {
        modalRef.current.focus();
      }
    }
    
    return () => {
      // Restore focus when modal closes
      if (previousActiveElement.current) {
        previousActiveElement.current.focus();
      }
    };
  }, [isOpen]);
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={modalRef}
        className="modal-content"
        onClick={(e) => e.stopPropagation()}
        tabIndex={-1}
      >
        {children}
      </div>
    </div>
  );
}
```

---

## When to Use Each

**Use useEffect (99% of cases):**

```jsx
// ✅ Data fetching
useEffect(() => {
  fetch('/api/data').then(...);
}, []);

// ✅ Subscriptions
useEffect(() => {
  const unsubscribe = subscribe();
  return unsubscribe;
}, []);

// ✅ Timers
useEffect(() => {
  const timer = setTimeout(...);
  return () => clearTimeout(timer);
}, []);

// ✅ Event listeners
useEffect(() => {
  window.addEventListener('resize', handler);
  return () => window.removeEventListener('resize', handler);
}, []);

// ✅ Side effects that don't affect layout
useEffect(() => {
  logAnalytics();
}, []);
```

**Use useLayoutEffect (rare cases):**

```jsx
// ✅ Measuring DOM
useLayoutEffect(() => {
  const height = ref.current.offsetHeight;
  setHeight(height);
}, []);

// ✅ Positioning elements
useLayoutEffect(() => {
  const rect = ref.current.getBoundingClientRect();
  setPosition({ x: rect.left, y: rect.top });
}, []);

// ✅ Scrolling before paint
useLayoutEffect(() => {
  ref.current.scrollTop = 0;
}, [page]);

// ✅ Preventing visual flicker
useLayoutEffect(() => {
  ref.current.style.transform = 'translateX(0)';
}, []);

// ✅ Reading/writing DOM to avoid flash
useLayoutEffect(() => {
  const width = ref.current.scrollWidth;
  if (width > maxWidth) {
    ref.current.style.fontSize = '12px';
  }
}, [content]);
```

---

## Visual Flicker Example

**Problem: useEffect causes flicker**

```jsx
import { useState, useEffect, useRef } from 'react';

function AnimatedBox() {
  const [show, setShow] = useState(false);
  const boxRef = useRef();
  
  useEffect(() => {
    if (show) {
      // This runs AFTER paint
      boxRef.current.style.opacity = '0';
      boxRef.current.style.transform = 'translateY(-20px)';
      
      setTimeout(() => {
        boxRef.current.style.opacity = '1';
        boxRef.current.style.transform = 'translateY(0)';
      }, 10);
    }
  }, [show]);
  
  if (!show) return null;
  
  return <div ref={boxRef}>Animated Box</div>;
}

// Problem: Box flashes visible before animation starts
```

**Solution: useLayoutEffect prevents flicker**

```jsx
import { useState, useLayoutEffect, useRef } from 'react';

function AnimatedBox() {
  const [show, setShow] = useState(false);
  const boxRef = useRef();
  
  useLayoutEffect(() => {
    if (show) {
      // This runs BEFORE paint
      boxRef.current.style.opacity = '0';
      boxRef.current.style.transform = 'translateY(-20px)';
      
      // Force reflow
      boxRef.current.offsetHeight;
      
      boxRef.current.style.transition = 'all 0.3s';
      boxRef.current.style.opacity = '1';
      boxRef.current.style.transform = 'translateY(0)';
    }
  }, [show]);
  
  if (!show) return null;
  
  return <div ref={boxRef}>Animated Box</div>;
}

// Solution: Initial styles applied before paint - no flicker
```

---

**Best Practices:**

1. **Default to useEffect**
```jsx
// Start with useEffect
useEffect(() => {
  // Your code
}, []);

// Only switch to useLayoutEffect if you see visual issues
```

2. **Keep useLayoutEffect fast**
```jsx
// ❌ Bad: Heavy computation blocks paint
useLayoutEffect(() => {
  const result = expensiveCalculation();  // Blocks UI!
  setData(result);
}, []);

// ✅ Good: Quick DOM operations only
useLayoutEffect(() => {
  const height = ref.current.offsetHeight;
  setHeight(height);
}, []);
```

3. **Measure first, then mutate**
```jsx
useLayoutEffect(() => {
  // Read layout
  const rect = ref.current.getBoundingClientRect();
  
  // Then mutate
  if (rect.width > maxWidth) {
    ref.current.style.fontSize = 'smaller';
  }
}, []);
```

4. **SSR: Use useEffect**
```jsx
// useLayoutEffect doesn't run on server
// Can cause hydration warnings

// Better: Detect environment
const useIsomorphicLayoutEffect = 
  typeof window !== 'undefined' ? useLayoutEffect : useEffect;

function Component() {
  useIsomorphicLayoutEffect(() => {
    // Safe on both server and client
  }, []);
}
```

---

**Interview Tips:**
- `useLayoutEffect` runs **synchronously after DOM updates but before paint**
- `useEffect` runs **asynchronously after paint**
- **Same API** as useEffect but different timing
- Use `useLayoutEffect` for **DOM measurements** and **preventing visual flicker**
- Use `useEffect` for **99% of cases** - data fetching, subscriptions, side effects
- `useLayoutEffect` **blocks browser painting** - can hurt performance if slow
- Prevents **visual flash** when reading/writing DOM (tooltips, positioning, scrolling)
- Runs **before user sees changes** - ideal for layout adjustments
- Common use cases: **measuring elements**, **positioning popovers**, **scroll position**, **animations**
- **SSR warning**: useLayoutEffect doesn't run on server (use conditional)
- Default to `useEffect`, switch to `useLayoutEffect` only if visual issues
- Timing: `render → DOM update → useLayoutEffect → paint → useEffect`
- Both support **cleanup functions** and **dependency arrays**
- Choose based on **when effect needs to run**, not what it does
- Use for **synchronous DOM mutations** that must happen before paint

</details>

---

### 37. What is useImperativeHandle hook?

<details>
<summary>View Answer</summary>

**useImperativeHandle Hook**

`useImperativeHandle` is a React Hook that lets you **customize the ref value** that is exposed to parent components. It's used with `forwardRef` to control what instance value gets exposed when using a ref.

**Basic Syntax:**

```jsx
import { useImperativeHandle, forwardRef } from 'react';

const Component = forwardRef((props, ref) => {
  useImperativeHandle(ref, () => {
    return {
      // Expose these methods to parent
      method1() { /*...*/ },
      method2() { /*...*/ }
    };
  }, [dependencies]);
  
  return <div>...</div>;
});
```

**Parameters:**
- `ref` - The ref passed from parent via `forwardRef`
- `createHandle` - Function that returns the object to expose
- `dependencies` - Array of values that trigger recreation

**Purpose:**
- **Encapsulate internal implementation** - don't expose entire DOM node
- **Expose only specific methods** - controlled API for parent
- **Provide custom imperative API** - go beyond standard DOM methods

---

## Why useImperativeHandle?

**Problem: Exposing entire DOM node**

```jsx
import { forwardRef, useRef } from 'react';

// Without useImperativeHandle
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

function Parent() {
  const inputRef = useRef();
  
  const handleClick = () => {
    // Parent has access to ENTIRE input DOM node
    inputRef.current.focus();              // ✅ Good
    inputRef.current.value = 'hacked!';    // ❌ Parent can do anything!
    inputRef.current.style.color = 'red';  // ❌ Breaks encapsulation
  };
  
  return (
    <div>
      <Input ref={inputRef} />
      <button onClick={handleClick}>Focus</button>
    </div>
  );
}

// Problem: Parent has full access to DOM internals
```

**Solution: Controlled API with useImperativeHandle**

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

// With useImperativeHandle
const Input = forwardRef((props, ref) => {
  const inputRef = useRef();
  
  // Expose only specific methods
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    clear: () => {
      inputRef.current.value = '';
    }
    // Don't expose: value setter, style, other DOM methods
  }));
  
  return <input ref={inputRef} {...props} />;
});

function Parent() {
  const inputRef = useRef();
  
  const handleClick = () => {
    inputRef.current.focus();   // ✅ Allowed
    inputRef.current.clear();   // ✅ Allowed
    // inputRef.current.value = 'x';  // ❌ Not available!
    // inputRef.current.style = ...;   // ❌ Not available!
  };
  
  return (
    <div>
      <Input ref={inputRef} />
      <button onClick={handleClick}>Focus</button>
    </div>
  );
}

// Solution: Parent can only use exposed methods
```

---

## Basic Examples

**1. Custom Input with Focus and Clear**

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef();
  
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    blur: () => {
      inputRef.current.blur();
    },
    clear: () => {
      inputRef.current.value = '';
    },
    getValue: () => {
      return inputRef.current.value;
    },
    setValue: (value) => {
      inputRef.current.value = value;
    }
  }));
  
  return <input ref={inputRef} {...props} />;
});

// Usage
function Form() {
  const inputRef = useRef();
  
  return (
    <div>
      <CustomInput ref={inputRef} placeholder="Enter text" />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
      <button onClick={() => inputRef.current.clear()}>Clear</button>
      <button onClick={() => alert(inputRef.current.getValue())}>Get Value</button>
    </div>
  );
}
```

**2. Video Player with Controls**

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const VideoPlayer = forwardRef(({ src }, ref) => {
  const videoRef = useRef();
  
  useImperativeHandle(ref, () => ({
    play: () => {
      videoRef.current.play();
    },
    pause: () => {
      videoRef.current.pause();
    },
    stop: () => {
      videoRef.current.pause();
      videoRef.current.currentTime = 0;
    },
    seek: (time) => {
      videoRef.current.currentTime = time;
    },
    setVolume: (level) => {
      videoRef.current.volume = Math.max(0, Math.min(1, level));
    },
    getCurrentTime: () => {
      return videoRef.current.currentTime;
    },
    getDuration: () => {
      return videoRef.current.duration;
    }
  }));
  
  return <video ref={videoRef} src={src} />;
});

// Usage
function App() {
  const playerRef = useRef();
  
  return (
    <div>
      <VideoPlayer ref={playerRef} src="video.mp4" />
      <button onClick={() => playerRef.current.play()}>Play</button>
      <button onClick={() => playerRef.current.pause()}>Pause</button>
      <button onClick={() => playerRef.current.stop()}>Stop</button>
      <button onClick={() => playerRef.current.seek(30)}>Skip to 30s</button>
    </div>
  );
}
```

**3. Modal with Open/Close API**

```jsx
import { forwardRef, useState, useImperativeHandle } from 'react';

const Modal = forwardRef(({ title, children }, ref) => {
  const [isOpen, setIsOpen] = useState(false);
  
  useImperativeHandle(ref, () => ({
    open: () => {
      setIsOpen(true);
    },
    close: () => {
      setIsOpen(false);
    },
    toggle: () => {
      setIsOpen(prev => !prev);
    },
    isOpen: () => {
      return isOpen;
    }
  }));
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay">
      <div className="modal-content">
        <h2>{title}</h2>
        {children}
        <button onClick={() => setIsOpen(false)}>Close</button>
      </div>
    </div>
  );
});

// Usage
function App() {
  const modalRef = useRef();
  
  return (
    <div>
      <button onClick={() => modalRef.current.open()}>Open Modal</button>
      <Modal ref={modalRef} title="My Modal">
        <p>Modal content goes here</p>
      </Modal>
    </div>
  );
}
```

---

## With Dependencies

**Recreate handle when dependencies change:**

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const Counter = forwardRef(({ step = 1 }, ref) => {
  const [count, setCount] = useState(0);
  
  useImperativeHandle(ref, () => ({
    increment: () => {
      setCount(c => c + step);  // Uses current step
    },
    decrement: () => {
      setCount(c => c - step);  // Uses current step
    },
    reset: () => {
      setCount(0);
    },
    getCount: () => {
      return count;
    }
  }), [step, count]);  // Recreate when step or count changes
  
  return <div>Count: {count}</div>;
});
```

---

## Production Example: Form Component

```jsx
import { forwardRef, useRef, useImperativeHandle, useState } from 'react';

const Form = forwardRef(({ onSubmit, children }, ref) => {
  const formRef = useRef();
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  useImperativeHandle(ref, () => ({
    // Submit form programmatically
    submit: async () => {
      setIsSubmitting(true);
      setErrors({});
      
      const formData = new FormData(formRef.current);
      const data = Object.fromEntries(formData);
      
      try {
        await onSubmit(data);
      } catch (error) {
        setErrors(error.errors || {});
      } finally {
        setIsSubmitting(false);
      }
    },
    
    // Reset form
    reset: () => {
      formRef.current.reset();
      setErrors({});
      setIsSubmitting(false);
    },
    
    // Get form values
    getValues: () => {
      const formData = new FormData(formRef.current);
      return Object.fromEntries(formData);
    },
    
    // Set form values
    setValues: (values) => {
      Object.keys(values).forEach(key => {
        const input = formRef.current.elements[key];
        if (input) input.value = values[key];
      });
    },
    
    // Validate
    validate: () => {
      const formData = new FormData(formRef.current);
      const data = Object.fromEntries(formData);
      const newErrors = {};
      
      // Simple validation
      Object.keys(data).forEach(key => {
        if (!data[key]) {
          newErrors[key] = `${key} is required`;
        }
      });
      
      setErrors(newErrors);
      return Object.keys(newErrors).length === 0;
    },
    
    // Focus first error
    focusFirstError: () => {
      const firstErrorField = Object.keys(errors)[0];
      if (firstErrorField) {
        formRef.current.elements[firstErrorField]?.focus();
      }
    },
    
    // Check if form is dirty
    isDirty: () => {
      const inputs = formRef.current.querySelectorAll('input, textarea, select');
      return Array.from(inputs).some(input => {
        return input.value !== input.defaultValue;
      });
    }
  }), [errors, isSubmitting, onSubmit]);
  
  return (
    <form ref={formRef}>
      {children}
      {Object.keys(errors).length > 0 && (
        <div className="errors">
          {Object.entries(errors).map(([field, error]) => (
            <div key={field}>{error}</div>
          ))}
        </div>
      )}
    </form>
  );
});

// Usage
function App() {
  const formRef = useRef();
  
  const handleSave = () => {
    if (formRef.current.validate()) {
      formRef.current.submit();
    } else {
      formRef.current.focusFirstError();
    }
  };
  
  const handleCancel = () => {
    if (formRef.current.isDirty()) {
      if (confirm('Discard changes?')) {
        formRef.current.reset();
      }
    }
  };
  
  return (
    <div>
      <Form ref={formRef} onSubmit={async (data) => console.log(data)}>
        <input name="username" placeholder="Username" />
        <input name="email" type="email" placeholder="Email" />
        <textarea name="bio" placeholder="Bio" />
      </Form>
      <button onClick={handleSave}>Save</button>
      <button onClick={handleCancel}>Cancel</button>
      <button onClick={() => formRef.current.reset()}>Reset</button>
    </div>
  );
}
```

---

## Production Example: Slider Component

```jsx
import { forwardRef, useRef, useImperativeHandle, useState } from 'react';

const Slider = forwardRef(({ min = 0, max = 100, defaultValue = 0, onChange }, ref) => {
  const [value, setValue] = useState(defaultValue);
  const sliderRef = useRef();
  
  useImperativeHandle(ref, () => ({
    getValue: () => value,
    
    setValue: (newValue) => {
      const clampedValue = Math.max(min, Math.min(max, newValue));
      setValue(clampedValue);
      onChange?.(clampedValue);
    },
    
    increment: (step = 1) => {
      const newValue = Math.min(max, value + step);
      setValue(newValue);
      onChange?.(newValue);
    },
    
    decrement: (step = 1) => {
      const newValue = Math.max(min, value - step);
      setValue(newValue);
      onChange?.(newValue);
    },
    
    reset: () => {
      setValue(defaultValue);
      onChange?.(defaultValue);
    },
    
    setToMin: () => {
      setValue(min);
      onChange?.(min);
    },
    
    setToMax: () => {
      setValue(max);
      onChange?.(max);
    },
    
    getPercentage: () => {
      return ((value - min) / (max - min)) * 100;
    }
  }), [value, min, max, defaultValue, onChange]);
  
  const handleChange = (e) => {
    const newValue = Number(e.target.value);
    setValue(newValue);
    onChange?.(newValue);
  };
  
  return (
    <div className="slider">
      <input
        ref={sliderRef}
        type="range"
        min={min}
        max={max}
        value={value}
        onChange={handleChange}
      />
      <span>{value}</span>
    </div>
  );
});

// Usage
function VolumeControl() {
  const sliderRef = useRef();
  
  return (
    <div>
      <Slider ref={sliderRef} min={0} max={100} defaultValue={50} />
      <button onClick={() => sliderRef.current.increment(10)}>+10</button>
      <button onClick={() => sliderRef.current.decrement(10)}>-10</button>
      <button onClick={() => sliderRef.current.setToMax()}>Max</button>
      <button onClick={() => sliderRef.current.setToMin()}>Mute</button>
      <button onClick={() => alert(sliderRef.current.getValue())}>Show Value</button>
    </div>
  );
}
```

---

## Production Example: Carousel

```jsx
import { forwardRef, useRef, useImperativeHandle, useState } from 'react';

const Carousel = forwardRef(({ items, autoplay = false, interval = 3000 }, ref) => {
  const [currentIndex, setCurrentIndex] = useState(0);
  const [isPlaying, setIsPlaying] = useState(autoplay);
  const timerRef = useRef();
  
  useImperativeHandle(ref, () => ({
    next: () => {
      setCurrentIndex(prev => (prev + 1) % items.length);
    },
    
    previous: () => {
      setCurrentIndex(prev => (prev - 1 + items.length) % items.length);
    },
    
    goTo: (index) => {
      if (index >= 0 && index < items.length) {
        setCurrentIndex(index);
      }
    },
    
    play: () => {
      setIsPlaying(true);
    },
    
    pause: () => {
      setIsPlaying(false);
      if (timerRef.current) {
        clearInterval(timerRef.current);
      }
    },
    
    reset: () => {
      setCurrentIndex(0);
      setIsPlaying(autoplay);
    },
    
    getCurrentIndex: () => currentIndex,
    
    getTotalSlides: () => items.length,
    
    isPlaying: () => isPlaying
  }), [currentIndex, items.length, isPlaying, autoplay]);
  
  // Auto-play logic
  useEffect(() => {
    if (isPlaying) {
      timerRef.current = setInterval(() => {
        setCurrentIndex(prev => (prev + 1) % items.length);
      }, interval);
    }
    
    return () => {
      if (timerRef.current) {
        clearInterval(timerRef.current);
      }
    };
  }, [isPlaying, items.length, interval]);
  
  return (
    <div className="carousel">
      <div className="carousel-content">
        {items[currentIndex]}
      </div>
      <div className="carousel-indicators">
        {items.map((_, index) => (
          <button
            key={index}
            className={index === currentIndex ? 'active' : ''}
            onClick={() => setCurrentIndex(index)}
          />
        ))}
      </div>
    </div>
  );
});

// Usage
function App() {
  const carouselRef = useRef();
  
  const slides = [
    <div>Slide 1</div>,
    <div>Slide 2</div>,
    <div>Slide 3</div>
  ];
  
  return (
    <div>
      <Carousel ref={carouselRef} items={slides} autoplay />
      <button onClick={() => carouselRef.current.previous()}>Previous</button>
      <button onClick={() => carouselRef.current.next()}>Next</button>
      <button onClick={() => carouselRef.current.play()}>Play</button>
      <button onClick={() => carouselRef.current.pause()}>Pause</button>
      <button onClick={() => carouselRef.current.goTo(0)}>Go to First</button>
    </div>
  );
}
```

---

## Best Practices

**1. Always use with forwardRef**

```jsx
// ✅ Correct
const Component = forwardRef((props, ref) => {
  useImperativeHandle(ref, () => ({ /*...*/ }));
  return <div>...</div>;
});

// ❌ Wrong: useImperativeHandle without forwardRef
function Component({ ref }) {  // ref as prop doesn't work
  useImperativeHandle(ref, () => ({ /*...*/ }));
  return <div>...</div>;
}
```

**2. Expose minimal API**

```jsx
// ❌ Bad: Exposing too much
useImperativeHandle(ref, () => ({
  inputRef: inputRef.current,  // Don't expose raw ref
  internalState: state,        // Don't expose internal state
  // ... 20 methods
}));

// ✅ Good: Minimal, focused API
useImperativeHandle(ref, () => ({
  focus: () => inputRef.current.focus(),
  clear: () => inputRef.current.value = ''
}));
```

**3. Include dependencies**

```jsx
// ❌ Bad: Missing dependencies
useImperativeHandle(ref, () => ({
  increment: () => setCount(count + step)  // Uses count and step
}), []);  // Missing dependencies!

// ✅ Good: Complete dependencies
useImperativeHandle(ref, () => ({
  increment: () => setCount(count + step)
}), [count, step]);

// ✅ Better: Use functional update
useImperativeHandle(ref, () => ({
  increment: () => setCount(c => c + step)
}), [step]);  // Only step needed
```

**4. Document the API**

```jsx
/**
 * Custom Input Component
 * 
 * Exposed methods:
 * - focus(): Focus the input
 * - blur(): Blur the input
 * - clear(): Clear the input value
 * - getValue(): Get current value
 * - setValue(value): Set input value
 */
const CustomInput = forwardRef((props, ref) => {
  // ...
});
```

---

**When to Use:**

**✅ Good Use Cases:**

1. **Custom form controls** - focus, clear, validate
2. **Media players** - play, pause, seek
3. **Modals/dialogs** - open, close, toggle
4. **Complex widgets** - carousels, sliders, tooltips
5. **Animations** - play, pause, reset

**❌ Avoid When:**

1. **Can use props instead** - prefer declarative over imperative
2. **Simple components** - don't add complexity unnecessarily
3. **Exposing internal state** - breaks encapsulation

---

**Interview Tips:**
- `useImperativeHandle` **customizes the ref value** exposed to parent components
- Used with `forwardRef` to **control what parent can access**
- **Encapsulates implementation** - parent only sees exposed methods
- Prevents parent from accessing **entire DOM node** or **internal state**
- **Imperative API** for components that need programmatic control
- Common use cases: **form controls**, **media players**, **modals**, **animations**
- Takes **three parameters**: ref, createHandle function, dependencies
- **Recreates handle** when dependencies change
- **Prefer declarative** (props/state) over imperative when possible
- Exposes **minimal API** - only what parent truly needs
- Use **functional updates** in methods to avoid dependency issues
- Makes components more **reusable** with controlled interface
- Different from regular `useRef` - this customizes what **others** see via ref
- Can expose **computed values** or **derived state** in addition to methods
- Always **document the exposed API** for other developers

</details>

---

### 38. How do you create custom hooks?

<details>
<summary>View Answer</summary>

**Custom Hooks**

Custom Hooks are **JavaScript functions** whose names start with `use` and that can call other Hooks. They let you **extract and reuse stateful logic** between components.

**Basic Structure:**

```jsx
import { useState, useEffect } from 'react';

function useCustomHook(initialValue) {
  const [state, setState] = useState(initialValue);
  
  useEffect(() => {
    // Side effect logic
  }, []);
  
  // Return values/functions for component to use
  return [state, setState];
}
```

**Key Rules:**
1. **Name must start with "use"** - `useCustomHook`, `useFetch`, `useLocalStorage`
2. **Can call other Hooks** - `useState`, `useEffect`, `useContext`, etc.
3. **Must follow Rules of Hooks** - only call at top level, only in React functions
4. **Return anything** - values, functions, objects, arrays

---

## Why Custom Hooks?

**Problem: Duplicated logic across components**

```jsx
// Component 1 - duplicated logic
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{user.name}</div>;
}

// Component 2 - same logic duplicated!
function UserSettings() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);
  
  // ... same logic
}
```

**Solution: Extract to custom hook**

```jsx
// Custom hook
function useUser() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);
  
  return { user, loading, error };
}

// Component 1 - clean!
function UserProfile() {
  const { user, loading, error } = useUser();
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{user.name}</div>;
}

// Component 2 - reusing same logic!
function UserSettings() {
  const { user, loading, error } = useUser();
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>Settings for {user.name}</div>;
}
```

---

## Basic Examples

**1. useToggle - Boolean State**

```jsx
import { useState } from 'react';

function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);
  
  const toggle = () => setValue(v => !v);
  const setTrue = () => setValue(true);
  const setFalse = () => setValue(false);
  
  return [value, toggle, setTrue, setFalse];
}

// Usage
function Component() {
  const [isOpen, toggle, open, close] = useToggle(false);
  
  return (
    <div>
      <p>{isOpen ? 'Open' : 'Closed'}</p>
      <button onClick={toggle}>Toggle</button>
      <button onClick={open}>Open</button>
      <button onClick={close}>Close</button>
    </div>
  );
}
```

**2. useCounter - Counter Logic**

```jsx
import { useState } from 'react';

function useCounter(initialValue = 0, step = 1) {
  const [count, setCount] = useState(initialValue);
  
  const increment = () => setCount(c => c + step);
  const decrement = () => setCount(c => c - step);
  const reset = () => setCount(initialValue);
  const set = (value) => setCount(value);
  
  return { count, increment, decrement, reset, set };
}

// Usage
function Component() {
  const { count, increment, decrement, reset } = useCounter(0, 5);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+5</button>
      <button onClick={decrement}>-5</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

**3. useLocalStorage - Persistent State**

```jsx
import { useState, useEffect } from 'react';

function useLocalStorage(key, initialValue) {
  // Get initial value from localStorage or use initialValue
  const [value, setValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // Update localStorage when value changes
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  }, [key, value]);
  
  return [value, setValue];
}

// Usage
function Component() {
  const [name, setName] = useLocalStorage('name', '');
  
  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <p>Saved: {name}</p>
    </div>
  );
}
```

**4. useDebounce - Delayed Value**

```jsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage
function SearchBox() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);
  
  useEffect(() => {
    if (debouncedQuery) {
      console.log('Searching for:', debouncedQuery);
      // API call here
    }
  }, [debouncedQuery]);
  
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

---

## Advanced Custom Hooks

**1. useFetch - Data Fetching**

```jsx
import { useState, useEffect } from 'react';

function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let isMounted = true;
    
    const fetchData = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch(url, options);
        if (!response.ok) throw new Error(response.statusText);
        const json = await response.json();
        
        if (isMounted) {
          setData(json);
          setLoading(false);
        }
      } catch (err) {
        if (isMounted) {
          setError(err.message);
          setLoading(false);
        }
      }
    };
    
    fetchData();
    
    return () => {
      isMounted = false;
    };
  }, [url]);
  
  const refetch = () => {
    setLoading(true);
    setError(null);
  };
  
  return { data, loading, error, refetch };
}

// Usage
function UserList() {
  const { data, loading, error, refetch } = useFetch('/api/users');
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div>
      <button onClick={refetch}>Refresh</button>
      <ul>
        {data.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

**2. useWindowSize - Responsive Hook**

```jsx
import { useState, useEffect } from 'react';

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// Usage
function Component() {
  const { width, height } = useWindowSize();
  
  return (
    <div>
      <p>Width: {width}px</p>
      <p>Height: {height}px</p>
      {width < 768 && <p>Mobile view</p>}
    </div>
  );
}
```

**3. useOnClickOutside - Click Outside Detection**

```jsx
import { useEffect } from 'react';

function useOnClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      if (!ref.current || ref.current.contains(event.target)) {
        return;
      }
      handler(event);
    };
    
    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);
    
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}

// Usage
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef();
  
  useOnClickOutside(dropdownRef, () => setIsOpen(false));
  
  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && <div>Dropdown content</div>}
    </div>
  );
}
```

**4. useInterval - Interval Hook**

```jsx
import { useEffect, useRef } from 'react';

function useInterval(callback, delay) {
  const savedCallback = useRef();
  
  // Remember latest callback
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);
  
  // Set up interval
  useEffect(() => {
    if (delay === null) return;
    
    const tick = () => savedCallback.current();
    const id = setInterval(tick, delay);
    
    return () => clearInterval(id);
  }, [delay]);
}

// Usage
function Timer() {
  const [count, setCount] = useState(0);
  const [isRunning, setIsRunning] = useState(true);
  
  useInterval(
    () => setCount(count + 1),
    isRunning ? 1000 : null  // null pauses the interval
  );
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'Pause' : 'Resume'}
      </button>
    </div>
  );
}
```

---

## Production Example: useForm

```jsx
import { useState } from 'react';

function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = (name) => (e) => {
    const value = e.target.value;
    setValues(prev => ({ ...prev, [name]: value }));
    
    // Validate if field was touched
    if (touched[name] && validate) {
      const fieldErrors = validate({ ...values, [name]: value });
      setErrors(prev => ({ ...prev, [name]: fieldErrors[name] }));
    }
  };
  
  const handleBlur = (name) => () => {
    setTouched(prev => ({ ...prev, [name]: true }));
    
    if (validate) {
      const fieldErrors = validate(values);
      setErrors(prev => ({ ...prev, [name]: fieldErrors[name] }));
    }
  };
  
  const handleSubmit = (onSubmit) => async (e) => {
    e.preventDefault();
    
    // Mark all fields as touched
    const allTouched = {};
    Object.keys(values).forEach(key => {
      allTouched[key] = true;
    });
    setTouched(allTouched);
    
    // Validate
    if (validate) {
      const validationErrors = validate(values);
      setErrors(validationErrors);
      
      if (Object.keys(validationErrors).length > 0) {
        return;
      }
    }
    
    // Submit
    setIsSubmitting(true);
    try {
      await onSubmit(values);
    } catch (error) {
      setErrors({ submit: error.message });
    } finally {
      setIsSubmitting(false);
    }
  };
  
  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
    setIsSubmitting(false);
  };
  
  return {
    values,
    errors,
    touched,
    isSubmitting,
    handleChange,
    handleBlur,
    handleSubmit,
    reset
  };
}

// Usage
function SignupForm() {
  const { values, errors, touched, handleChange, handleBlur, handleSubmit } = useForm(
    { username: '', email: '', password: '' },
    (values) => {
      const errors = {};
      if (!values.username) errors.username = 'Required';
      if (!values.email) errors.email = 'Required';
      if (!values.password) errors.password = 'Required';
      return errors;
    }
  );
  
  return (
    <form onSubmit={handleSubmit(async (data) => console.log(data))}>
      <input
        value={values.username}
        onChange={handleChange('username')}
        onBlur={handleBlur('username')}
      />
      {touched.username && errors.username && <span>{errors.username}</span>}
      
      <input
        value={values.email}
        onChange={handleChange('email')}
        onBlur={handleBlur('email')}
      />
      {touched.email && errors.email && <span>{errors.email}</span>}
      
      <input
        type="password"
        value={values.password}
        onChange={handleChange('password')}
        onBlur={handleBlur('password')}
      />
      {touched.password && errors.password && <span>{errors.password}</span>}
      
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

---

## Production Example: useAsync

```jsx
import { useState, useCallback } from 'react';

function useAsync(asyncFunction) {
  const [status, setStatus] = useState('idle');  // 'idle' | 'pending' | 'success' | 'error'
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  const execute = useCallback(async (...params) => {
    setStatus('pending');
    setData(null);
    setError(null);
    
    try {
      const response = await asyncFunction(...params);
      setData(response);
      setStatus('success');
      return response;
    } catch (err) {
      setError(err);
      setStatus('error');
      throw err;
    }
  }, [asyncFunction]);
  
  const reset = () => {
    setStatus('idle');
    setData(null);
    setError(null);
  };
  
  return {
    execute,
    status,
    data,
    error,
    isIdle: status === 'idle',
    isPending: status === 'pending',
    isSuccess: status === 'success',
    isError: status === 'error',
    reset
  };
}

// Usage
function UserProfile({ userId }) {
  const fetchUser = async (id) => {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  };
  
  const { execute, data, isPending, isError, error } = useAsync(fetchUser);
  
  useEffect(() => {
    execute(userId);
  }, [userId]);
  
  if (isPending) return <div>Loading...</div>;
  if (isError) return <div>Error: {error.message}</div>;
  if (!data) return null;
  
  return <div>{data.name}</div>;
}
```

---

**Best Practices:**

**1. Name starts with "use"**
```jsx
// ✅ Good
function useCustomHook() { /*...*/ }
function useFetch() { /*...*/ }
function useLocalStorage() { /*...*/ }

// ❌ Bad
function customHook() { /*...*/ }
function fetchData() { /*...*/ }
```

**2. Extract reusable logic**
```jsx
// ✅ Good: Reusable across components
function useWindowSize() {
  // Logic that multiple components need
}

// ❌ Bad: Component-specific logic
function useUserProfileLogic() {
  // Logic only used in UserProfile component
}
```

**3. Return useful values**
```jsx
// ✅ Good: Return object for named access
function useUser() {
  return { user, loading, error, refetch };
}

// ✅ Good: Return array for custom naming
function useToggle() {
  return [isOpen, toggle, open, close];
}
```

**4. Keep hooks focused**
```jsx
// ✅ Good: Single responsibility
function useFetch(url) { /*...*/ }
function useLocalStorage(key) { /*...*/ }

// ❌ Bad: Too many responsibilities
function useEverything() {
  // Fetching, local storage, form handling, etc.
}
```

---

**Interview Tips:**
- Custom hooks are **JavaScript functions** that start with `use` and can call other hooks
- **Extract and reuse** stateful logic across components
- Must follow **Rules of Hooks** - only call at top level, only in React functions
- **Name must start with "use"** - `useFetch`, `useLocalStorage`, `useToggle`
- Can **return anything** - values, functions, objects, arrays
- Common patterns: **data fetching**, **form handling**, **subscriptions**, **timers**
- Helps avoid **code duplication** and makes logic testable
- Can call **other hooks** - useState, useEffect, useContext, other custom hooks
- Each call to custom hook has **isolated state** - not shared between components
- Use **object return** for named access or **array return** for custom naming
- **Not a React feature** - just a pattern that follows from hooks design
- Popular examples: useLocalStorage, useFetch, useDebounce, useToggle
- Compose hooks together to build more complex functionality
- Keep hooks **focused** on single responsibility
- Can be **tested independently** from components

</details>

---

### 39. What are the rules of hooks?

<details>
<summary>View Answer</summary>

**Rules of Hooks**

React Hooks must follow two fundamental rules to work correctly. These rules ensure that hooks are called in the same order on every render, which is critical for React to preserve state between calls.

---

## The Two Rules

**Rule 1: Only Call Hooks at the Top Level**

**Don't call Hooks inside:**
- ❌ Loops
- ❌ Conditions
- ❌ Nested functions

**Always call Hooks at the top level** of your React function, before any early returns.

```jsx
// ❌ BAD: Hook inside condition
function Component({ condition }) {
  if (condition) {
    const [value, setValue] = useState(0);  // ❌ Wrong!
  }
  return <div>...</div>;
}

// ❌ BAD: Hook inside loop
function Component({ items }) {
  items.forEach(item => {
    const [value, setValue] = useState(0);  // ❌ Wrong!
  });
  return <div>...</div>;
}

// ❌ BAD: Hook after early return
function Component({ isLoading }) {
  if (isLoading) return <div>Loading...</div>;
  const [value, setValue] = useState(0);  // ❌ Wrong!
  return <div>{value}</div>;
}

// ✅ GOOD: Hooks at top level
function Component({ condition, isLoading }) {
  const [value, setValue] = useState(0);  // ✅ Correct!
  
  if (isLoading) return <div>Loading...</div>;
  if (condition) return <div>{value}</div>;
  
  return <div>...</div>;
}
```

**Rule 2: Only Call Hooks from React Functions**

**Only call Hooks from:**
- ✅ React function components
- ✅ Custom Hooks

**Don't call Hooks from:**
- ❌ Regular JavaScript functions
- ❌ Class components
- ❌ Event handlers
- ❌ Utility functions

```jsx
// ❌ BAD: Hook in regular function
function calculateTotal() {
  const [total, setTotal] = useState(0);  // ❌ Wrong!
  return total;
}

// ❌ BAD: Hook in class component
class Component extends React.Component {
  render() {
    const [value, setValue] = useState(0);  // ❌ Wrong!
    return <div>{value}</div>;
  }
}

// ❌ BAD: Hook in event handler
function Component() {
  const handleClick = () => {
    const [value, setValue] = useState(0);  // ❌ Wrong!
  };
  return <button onClick={handleClick}>Click</button>;
}

// ✅ GOOD: Hook in function component
function Component() {
  const [value, setValue] = useState(0);  // ✅ Correct!
  return <div>{value}</div>;
}

// ✅ GOOD: Hook in custom hook
function useCustomHook() {
  const [value, setValue] = useState(0);  // ✅ Correct!
  return [value, setValue];
}
```

---

## Why These Rules?

**React relies on call order to manage state:**

```jsx
function Component() {
  // First render:
  const [name, setName] = useState('');      // Hook 1 - creates state slot 0
  const [age, setAge] = useState(0);         // Hook 2 - creates state slot 1
  const [email, setEmail] = useState('');    // Hook 3 - creates state slot 2
  
  // Second render:
  // React expects the same order:
  // Hook 1 → returns state from slot 0 (name)
  // Hook 2 → returns state from slot 1 (age)
  // Hook 3 → returns state from slot 2 (email)
}
```

**If order changes, React gets confused:**

```jsx
function Component({ showAge }) {
  const [name, setName] = useState('');      // Always Hook 1
  
  // ❌ This changes the order!
  if (showAge) {
    const [age, setAge] = useState(0);       // Sometimes Hook 2
  }
  
  const [email, setEmail] = useState('');    // Hook 2 or 3?
  
  // React doesn't know which state belongs to which hook!
}
```

---

## Common Mistakes

**1. Conditional Hooks**

```jsx
// ❌ BAD
function Form({ isEditing }) {
  const [name, setName] = useState('');
  
  if (isEditing) {
    const [originalName, setOriginalName] = useState('');  // ❌ Conditional!
  }
  
  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}

// ✅ GOOD: Move condition inside
function Form({ isEditing }) {
  const [name, setName] = useState('');
  const [originalName, setOriginalName] = useState('');  // ✅ Always called
  
  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      {isEditing && <p>Original: {originalName}</p>}
    </div>
  );
}
```

**2. Early Returns Before Hooks**

```jsx
// ❌ BAD
function Component({ data }) {
  if (!data) return null;  // Early return!
  
  const [count, setCount] = useState(0);  // ❌ Hook after early return
  return <div>{count}</div>;
}

// ✅ GOOD: Hooks before early return
function Component({ data }) {
  const [count, setCount] = useState(0);  // ✅ Hook first
  
  if (!data) return null;  // Early return after hooks
  return <div>{count}</div>;
}
```

**3. Hooks in Loops**

```jsx
// ❌ BAD
function TodoList({ items }) {
  return (
    <div>
      {items.map(item => {
        const [checked, setChecked] = useState(false);  // ❌ Hook in loop!
        return <div key={item.id}>{item.text}</div>;
      })}
    </div>
  );
}

// ✅ GOOD: Separate component with hook
function TodoItem({ item }) {
  const [checked, setChecked] = useState(false);  // ✅ Hook in component
  return <div>{item.text}</div>;
}

function TodoList({ items }) {
  return (
    <div>
      {items.map(item => (
        <TodoItem key={item.id} item={item} />
      ))}
    </div>
  );
}
```

**4. Hooks in Event Handlers**

```jsx
// ❌ BAD
function Component() {
  const handleClick = () => {
    const [count, setCount] = useState(0);  // ❌ Hook in event handler!
    setCount(count + 1);
  };
  
  return <button onClick={handleClick}>Click</button>;
}

// ✅ GOOD: Hook at top level
function Component() {
  const [count, setCount] = useState(0);  // ✅ Hook at top level
  
  const handleClick = () => {
    setCount(count + 1);
  };
  
  return <button onClick={handleClick}>Click {count}</button>;
}
```

**5. Hooks in Regular Functions**

```jsx
// ❌ BAD
function calculatePrice() {
  const [discount, setDiscount] = useState(0);  // ❌ Not a React function!
  return 100 - discount;
}

function Component() {
  const price = calculatePrice();
  return <div>Price: ${price}</div>;
}

// ✅ GOOD: Custom hook (starts with "use")
function usePrice() {
  const [discount, setDiscount] = useState(0);  // ✅ Custom hook
  return { price: 100 - discount, discount, setDiscount };
}

function Component() {
  const { price } = usePrice();
  return <div>Price: ${price}</div>;
}
```

---

## Handling Conditional Logic

**Pattern 1: Call hook unconditionally, use state conditionally**

```jsx
// ✅ GOOD
function Component({ shouldTrack }) {
  const [count, setCount] = useState(0);  // Always call hook
  
  const increment = () => {
    if (shouldTrack) {  // Condition inside
      setCount(c => c + 1);
    }
  };
  
  return <button onClick={increment}>Count: {count}</button>;
}
```

**Pattern 2: Initialize with conditional value**

```jsx
// ✅ GOOD
function Component({ isEditing, initialValue }) {
  // Hook always called, but value depends on condition
  const [value, setValue] = useState(() => {
    return isEditing ? initialValue : '';
  });
  
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
}
```

**Pattern 3: Multiple states, render conditionally**

```jsx
// ✅ GOOD
function Component({ mode }) {
  const [editValue, setEditValue] = useState('');      // Always called
  const [viewValue, setViewValue] = useState('');      // Always called
  
  if (mode === 'edit') {
    return <input value={editValue} onChange={(e) => setEditValue(e.target.value)} />;
  }
  
  return <div>{viewValue}</div>;
}
```

---

## ESLint Plugin

**React provides an ESLint plugin to enforce these rules:**

```bash
npm install eslint-plugin-react-hooks --save-dev
```

**Configure in .eslintrc:**

```json
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",      // Enforce Rules of Hooks
    "react-hooks/exhaustive-deps": "warn"       // Verify effect dependencies
  }
}
```

**This will catch violations:**

```jsx
function Component({ condition }) {
  if (condition) {
    const [value, setValue] = useState(0);
    // ⚠️ ESLint error: React Hook "useState" is called conditionally.
  }
}
```

---

## Summary

**The Two Rules:**

1. **Only call Hooks at the top level**
   - Not inside loops, conditions, or nested functions
   - Before any early returns
   - Ensures consistent call order

2. **Only call Hooks from React functions**
   - React function components
   - Custom Hooks (start with "use")
   - Not regular JavaScript functions

**Why:**
- React uses **call order** to match hooks with their state
- Breaking rules causes **state mismatch** and bugs
- Use **ESLint plugin** to catch violations

**Common Patterns:**
- Call hook **unconditionally**, use value **conditionally**
- **Initialize with conditional value**, not conditional hook
- Create **separate component** if you need hook in loop
- Move logic to **custom hook** if sharing between functions

---

**Interview Tips:**
- There are **two fundamental rules** for using hooks
- **Rule 1**: Only call hooks at the **top level** (not in loops, conditions, nested functions)
- **Rule 2**: Only call hooks from **React function components** or **custom hooks**
- React relies on **consistent call order** to preserve state between renders
- If hook order changes, React can't match hooks to their state correctly
- **Early returns** must come after all hooks
- **Conditional logic** should be inside hooks, not around them
- **ESLint plugin** `eslint-plugin-react-hooks` enforces these rules
- Custom hooks must **start with "use"** to follow convention
- Hooks in **loops** should be in a separate component
- **Event handlers** can use hook state/functions but can't call hooks themselves
- These rules enable React's hooks to be **simple and predictable**
- Breaking rules leads to **bugs** like state mismatch, stale closures
- Always **call hooks unconditionally**, handle conditionals inside
- Use **lazy initialization** if initial state depends on condition
- The **same number** of hooks must be called on every render

</details>

---

### 40. What is useId hook? (React 18+)

<details>
<summary>View Answer</summary>

**useId Hook (React 18+)**

`useId` is a React Hook introduced in React 18 that **generates unique IDs** that are stable across server and client renders. It's primarily used for **accessibility attributes** that require unique IDs.

**Basic Syntax:**

```jsx
import { useId } from 'react';

function Component() {
  const id = useId();
  return <div id={id}>...</div>;
}
```

**Key Features:**
- **Generates unique IDs** - Different on each call
- **Stable across renders** - Same ID on every render
- **SSR compatible** - Same ID on server and client
- **No arguments** - Just call `useId()`
- **Returns string** - Use directly in attributes

---

## Why useId?

**Problem: Manual ID generation has issues**

```jsx
// ❌ BAD: Hardcoded IDs can conflict
function Form() {
  return (
    <div>
      <label htmlFor="email">Email:</label>
      <input id="email" type="email" />
    </div>
  );
}

// If you render Form twice, both inputs have id="email"!
// This breaks accessibility and label clicks
function App() {
  return (
    <div>
      <Form />  {/* First form: id="email" */}
      <Form />  {/* Second form: id="email" - CONFLICT! */}
    </div>
  );
}
```

```jsx
// ❌ BAD: Random IDs break SSR
function Form() {
  const id = Math.random().toString();  // Different on server and client!
  
  return (
    <div>
      <label htmlFor={id}>Email:</label>
      <input id={id} type="email" />
    </div>
  );
}

// Server renders: id="0.123"
// Client hydrates: id="0.789"
// Mismatch error!
```

**Solution: useId generates unique, stable IDs**

```jsx
import { useId } from 'react';

// ✅ GOOD: useId generates unique IDs
function Form() {
  const id = useId();
  
  return (
    <div>
      <label htmlFor={id}>Email:</label>
      <input id={id} type="email" />
    </div>
  );
}

// Each instance gets unique ID
function App() {
  return (
    <div>
      <Form />  {/* id=":r1:" */}
      <Form />  {/* id=":r2:" */}
    </div>
  );
}

// IDs are the same on server and client!
```

---

## Basic Examples

**1. Form Input with Label**

```jsx
import { useId } from 'react';

function EmailInput() {
  const id = useId();
  
  return (
    <div>
      <label htmlFor={id}>Email:</label>
      <input id={id} type="email" name="email" />
    </div>
  );
}

// Each EmailInput instance gets unique ID
function Form() {
  return (
    <div>
      <EmailInput />  {/* id=":r1:" */}
      <EmailInput />  {/* id=":r2:" */}
    </div>
  );
}
```

**2. Multiple Related Elements**

```jsx
import { useId } from 'react';

function PasswordInput() {
  const id = useId();
  
  return (
    <div>
      <label htmlFor={id}>Password:</label>
      <input
        id={id}
        type="password"
        aria-describedby={`${id}-hint`}
      />
      <p id={`${id}-hint`}>Must be at least 8 characters</p>
    </div>
  );
}

// Uses same ID base for related elements
// input: id=":r1:"
// hint: id=":r1:-hint"
```

**3. Radio Group**

```jsx
import { useId } from 'react';

function RadioGroup({ options, name, label }) {
  const id = useId();
  
  return (
    <fieldset>
      <legend id={id}>{label}</legend>
      {options.map((option, index) => {
        const optionId = `${id}-${index}`;
        return (
          <div key={option.value}>
            <input
              type="radio"
              id={optionId}
              name={name}
              value={option.value}
              aria-labelledby={id}
            />
            <label htmlFor={optionId}>{option.label}</label>
          </div>
        );
      })}
    </fieldset>
  );
}

// Usage
function Form() {
  return (
    <RadioGroup
      name="size"
      label="Size"
      options={[
        { value: 's', label: 'Small' },
        { value: 'm', label: 'Medium' },
        { value: 'l', label: 'Large' }
      ]}
    />
  );
}
```

---

## Production Example: Form Field Component

```jsx
import { useId } from 'react';

function FormField({
  label,
  type = 'text',
  error,
  hint,
  required,
  ...inputProps
}) {
  const id = useId();
  const errorId = error ? `${id}-error` : undefined;
  const hintId = hint ? `${id}-hint` : undefined;
  
  // Build aria-describedby with all descriptions
  const describedBy = [hintId, errorId].filter(Boolean).join(' ');
  
  return (
    <div className="form-field">
      <label htmlFor={id}>
        {label}
        {required && <span aria-label="required">*</span>}
      </label>
      
      <input
        id={id}
        type={type}
        aria-required={required}
        aria-invalid={!!error}
        aria-describedby={describedBy || undefined}
        {...inputProps}
      />
      
      {hint && (
        <p id={hintId} className="hint">
          {hint}
        </p>
      )}
      
      {error && (
        <p id={errorId} className="error" role="alert">
          {error}
        </p>
      )}
    </div>
  );
}

// Usage
function SignupForm() {
  const [errors, setErrors] = useState({});
  
  return (
    <form>
      <FormField
        label="Username"
        name="username"
        required
        hint="3-20 characters, letters and numbers only"
        error={errors.username}
      />
      
      <FormField
        label="Email"
        name="email"
        type="email"
        required
        error={errors.email}
      />
      
      <FormField
        label="Password"
        name="password"
        type="password"
        required
        hint="At least 8 characters with uppercase, lowercase, and numbers"
        error={errors.password}
      />
      
      <button type="submit">Sign Up</button>
    </form>
  );
}

// Each FormField gets unique IDs:
// Username input: id=":r1:", hint: ":r1:-hint"
// Email input: id=":r2:", error: ":r2:-error"
// Password input: id=":r3:", hint: ":r3:-hint", error: ":r3:-error"
```

---

## Production Example: Accessible Modal

```jsx
import { useId } from 'react';

function Modal({ isOpen, onClose, title, children }) {
  const titleId = useId();
  const descriptionId = useId();
  
  if (!isOpen) return null;
  
  return (
    <div
      className="modal-overlay"
      role="dialog"
      aria-modal="true"
      aria-labelledby={titleId}
      aria-describedby={descriptionId}
      onClick={onClose}
    >
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <h2 id={titleId}>{title}</h2>
        <div id={descriptionId}>{children}</div>
        <button onClick={onClose} aria-label="Close modal">
          ×
        </button>
      </div>
    </div>
  );
}

// Usage
function App() {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>
      <Modal
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
        title="Confirm Action"
      >
        <p>Are you sure you want to continue?</p>
      </Modal>
    </div>
  );
}
```

---

## Production Example: Tooltip

```jsx
import { useId, useState } from 'react';

function Tooltip({ children, content }) {
  const [isVisible, setIsVisible] = useState(false);
  const id = useId();
  
  return (
    <span className="tooltip-wrapper">
      <span
        onMouseEnter={() => setIsVisible(true)}
        onMouseLeave={() => setIsVisible(false)}
        onFocus={() => setIsVisible(true)}
        onBlur={() => setIsVisible(false)}
        aria-describedby={id}
      >
        {children}
      </span>
      {isVisible && (
        <span id={id} role="tooltip" className="tooltip-content">
          {content}
        </span>
      )}
    </span>
  );
}

// Usage
function App() {
  return (
    <div>
      <p>
        Hover over <Tooltip content="This is a tooltip">this text</Tooltip> for more info.
      </p>
      <p>
        Or <Tooltip content="Another tooltip">this one</Tooltip> too.
      </p>
    </div>
  );
}
```

---

## Production Example: Tabbed Interface

```jsx
import { useId, useState } from 'react';

function Tabs({ tabs }) {
  const [activeIndex, setActiveIndex] = useState(0);
  const id = useId();
  
  return (
    <div>
      <div role="tablist" aria-label="Content tabs">
        {tabs.map((tab, index) => {
          const tabId = `${id}-tab-${index}`;
          const panelId = `${id}-panel-${index}`;
          const isActive = index === activeIndex;
          
          return (
            <button
              key={index}
              id={tabId}
              role="tab"
              aria-selected={isActive}
              aria-controls={panelId}
              tabIndex={isActive ? 0 : -1}
              onClick={() => setActiveIndex(index)}
            >
              {tab.label}
            </button>
          );
        })}
      </div>
      
      {tabs.map((tab, index) => {
        const tabId = `${id}-tab-${index}`;
        const panelId = `${id}-panel-${index}`;
        const isActive = index === activeIndex;
        
        return (
          <div
            key={index}
            id={panelId}
            role="tabpanel"
            aria-labelledby={tabId}
            hidden={!isActive}
          >
            {tab.content}
          </div>
        );
      })}
    </div>
  );
}

// Usage
function App() {
  return (
    <Tabs
      tabs={[
        { label: 'Profile', content: <div>Profile content</div> },
        { label: 'Settings', content: <div>Settings content</div> },
        { label: 'History', content: <div>History content</div> }
      ]}
    />
  );
}
```

---

## Don't Use for Keys

```jsx
// ❌ BAD: Don't use useId for list keys
function TodoList({ items }) {
  const id = useId();  // Same ID on every render!
  
  return (
    <ul>
      {items.map((item, index) => (
        <li key={`${id}-${index}`}>  // ❌ All items share same base ID
          {item.text}
        </li>
      ))}
    </ul>
  );
}

// ✅ GOOD: Use stable item IDs for keys
function TodoList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>  // ✅ Use item's own ID
          {item.text}
        </li>
      ))}
    </ul>
  );
}
```

---

## SSR Compatibility

**useId generates the same ID on server and client:**

```jsx
import { useId } from 'react';

function Form() {
  const id = useId();
  
  return (
    <div>
      <label htmlFor={id}>Email:</label>
      <input id={id} type="email" />
    </div>
  );
}

// Server renders: <input id=":r1:" />
// Client hydrates: <input id=":r1:" />
// ✅ No hydration mismatch!
```

**Compare to Math.random():**

```jsx
// ❌ BAD: Causes hydration mismatch
function Form() {
  const id = Math.random().toString();
  
  return (
    <div>
      <label htmlFor={id}>Email:</label>
      <input id={id} type="email" />
    </div>
  );
}

// Server renders: <input id="0.123" />
// Client hydrates: <input id="0.789" />
// ⚠️ Hydration mismatch error!
```

---

## Best Practices

**1. Use for accessibility attributes**

```jsx
// ✅ GOOD: Form labels
const id = useId();
<label htmlFor={id}>Name:</label>
<input id={id} />

// ✅ GOOD: ARIA attributes
const id = useId();
<button aria-describedby={id}>Help</button>
<div id={id} role="tooltip">Help text</div>

// ✅ GOOD: Multiple related elements
const id = useId();
<input id={id} aria-describedby={`${id}-hint ${id}-error`} />
<p id={`${id}-hint`}>Hint text</p>
<p id={`${id}-error`}>Error text</p>
```

**2. Don't use for keys in lists**

```jsx
// ❌ BAD: useId for keys
const id = useId();
items.map((item, i) => <li key={`${id}-${i}`}>...</li>)

// ✅ GOOD: Use item ID or index directly
items.map(item => <li key={item.id}>...</li>)
items.map((item, i) => <li key={i}>...</li>)
```

**3. Call at component top level**

```jsx
// ✅ GOOD: Call at top level
function Component() {
  const id = useId();
  return <div id={id}>...</div>;
}

// ❌ BAD: Conditional call
function Component({ needsId }) {
  if (needsId) {
    const id = useId();  // ❌ Violates Rules of Hooks
  }
}
```

**4. Reuse same ID for related elements**

```jsx
// ✅ GOOD: One useId, multiple derived IDs
function FormField() {
  const id = useId();
  return (
    <div>
      <label htmlFor={id}>Label</label>
      <input id={id} aria-describedby={`${id}-hint`} />
      <p id={`${id}-hint`}>Hint</p>
    </div>
  );
}

// ❌ BAD: Multiple useId calls for same field
function FormField() {
  const inputId = useId();
  const hintId = useId();
  // Unnecessary - can derive hintId from inputId
}
```

---

**Use Cases:**

**✅ Good Use Cases:**
- Form `<label htmlFor>` and `<input id>`
- `aria-labelledby`, `aria-describedby` relationships
- Modal `aria-labelledby` for titles
- Tab panels and tab controls
- Tooltips with `aria-describedby`
- Error messages linked to inputs

**❌ Avoid:**
- List item `key` props
- CSS class names
- Database IDs or API responses
- When you already have a stable ID

---

**Interview Tips:**
- `useId` generates **unique IDs** stable across renders
- Introduced in **React 18** for accessibility
- **SSR compatible** - same ID on server and client
- **No hydration mismatch** unlike Math.random() or Date.now()
- Primary use: **accessibility attributes** (htmlFor, aria-describedby, aria-labelledby)
- Takes **no arguments**, returns **string**
- Each call to useId returns a **different ID**
- Same component instance gets **same ID** across renders
- Can **derive multiple IDs** from one useId: `${id}-hint`, `${id}-error`
- **Don't use for list keys** - use item IDs instead
- Follows **Rules of Hooks** - call at top level only
- Generated IDs look like **`:r1:`, `:r2:`** etc.
- Solves problem of **ID conflicts** when reusing components
- Better than **manual counters** which break with SSR
- Part of React's **concurrent features** infrastructure
- Used for **associating labels** with inputs accessibly
- Enables **reusable form components** without ID conflicts
- Format is **implementation detail** - don't rely on it
- Works with React's **streaming SSR** and Suspense
- Alternative to **global ID counters** or **uuid libraries** for UI IDs

</details>
