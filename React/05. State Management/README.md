# State Management

> Intermediate / Mid-Level (1-3 years)

---

## Questions

41. What is Context API and when should you use it?
42. What are the performance issues with Context API?
43. What is Redux and how does it work?
44. What is the difference between Redux and Context API?
45. What are Redux Toolkit and its advantages?
46. What is Zustand and how does it compare to Redux?
47. What is Jotai and when would you use it?
48. What is Recoil state management?
49. How do you handle global state in React?
50. What is the Flux architecture pattern?

---

## Detailed Answers

### 41. What is Context API and when should you use it?

<details>
<summary>View Answer</summary>

**Context API**

The Context API is a React feature that allows you to **share data across the component tree** without having to pass props down manually at every level. It provides a way to pass data through the component tree without prop drilling.

**Basic Structure:**

```jsx
import { createContext, useContext } from 'react';

// 1. Create Context
const MyContext = createContext(defaultValue);

// 2. Provide Context
function App() {
  return (
    <MyContext.Provider value={someValue}>
      <ChildComponent />
    </MyContext.Provider>
  );
}

// 3. Consume Context
function ChildComponent() {
  const value = useContext(MyContext);
  return <div>{value}</div>;
}
```

**Three Main Parts:**
1. **createContext** - Creates a Context object
2. **Provider** - Supplies the value to components
3. **useContext** - Consumes the value in components

---

## Problem: Prop Drilling

**Without Context - Manual prop passing:**

```jsx
function App() {
  const [user, setUser] = useState({ name: 'John', role: 'admin' });
  
  return <Layout user={user} setUser={setUser} />;
}

function Layout({ user, setUser }) {
  // Layout doesn't need user, just passes it down
  return (
    <div>
      <Header user={user} />
      <Sidebar user={user} setUser={setUser} />
    </div>
  );
}

function Header({ user }) {
  // Header doesn't need user, just passes it down
  return <Navigation user={user} />;
}

function Navigation({ user }) {
  // Navigation doesn't need user, just passes it down
  return <UserMenu user={user} />;
}

function UserMenu({ user }) {
  // Finally uses user!
  return <div>Welcome, {user.name}</div>;
}

// Problem: user prop passed through 4 components
// Layout, Header, Navigation don't care about user
// Just passing it down (prop drilling)
```

**With Context - Direct access:**

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Create context
const UserContext = createContext();

function App() {
  const [user, setUser] = useState({ name: 'John', role: 'admin' });
  
  // 2. Provide value at top level
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Layout />
    </UserContext.Provider>
  );
}

function Layout() {
  // No user prop needed!
  return (
    <div>
      <Header />
      <Sidebar />
    </div>
  );
}

function Header() {
  // No user prop needed!
  return <Navigation />;
}

function Navigation() {
  // No user prop needed!
  return <UserMenu />;
}

function UserMenu() {
  // 3. Consume directly where needed
  const { user } = useContext(UserContext);
  return <div>Welcome, {user.name}</div>;
}

// Solution: No prop drilling!
// Only UserMenu accesses user
```

---

## Basic Example: Theme Context

```jsx
import { createContext, useContext, useState } from 'react';

// Create context with default value
const ThemeContext = createContext('light');

function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={theme}>
      <Toolbar />
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </ThemeContext.Provider>
  );
}

function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext);
  
  return (
    <button style={{ background: theme === 'light' ? '#fff' : '#333' }}>
      I am {theme} themed!
    </button>
  );
}
```

---

## Production Example: Auth Context

```jsx
import { createContext, useContext, useState, useEffect } from 'react';

// Create context
const AuthContext = createContext();

// Custom hook for easier access
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// Provider component
export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  // Check if user is logged in on mount
  useEffect(() => {
    const checkAuth = async () => {
      try {
        const response = await fetch('/api/auth/me');
        if (response.ok) {
          const userData = await response.json();
          setUser(userData);
        }
      } catch (error) {
        console.error('Auth check failed:', error);
      } finally {
        setLoading(false);
      }
    };
    
    checkAuth();
  }, []);
  
  const login = async (email, password) => {
    setLoading(true);
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });
      
      if (response.ok) {
        const userData = await response.json();
        setUser(userData);
        return { success: true };
      }
      
      return { success: false, error: 'Invalid credentials' };
    } catch (error) {
      return { success: false, error: error.message };
    } finally {
      setLoading(false);
    }
  };
  
  const logout = async () => {
    try {
      await fetch('/api/auth/logout', { method: 'POST' });
      setUser(null);
    } catch (error) {
      console.error('Logout failed:', error);
    }
  };
  
  const value = {
    user,
    loading,
    login,
    logout,
    isAuthenticated: !!user
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Usage in App
function App() {
  return (
    <AuthProvider>
      <Router>
        <Routes>
          <Route path="/login" element={<LoginPage />} />
          <Route path="/dashboard" element={<ProtectedRoute><Dashboard /></ProtectedRoute>} />
        </Routes>
      </Router>
    </AuthProvider>
  );
}

// Usage in components
function LoginPage() {
  const { login, loading } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    const result = await login(email, password);
    if (result.success) {
      navigate('/dashboard');
    } else {
      alert(result.error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
      <button disabled={loading}>Login</button>
    </form>
  );
}

function Dashboard() {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}

function ProtectedRoute({ children }) {
  const { isAuthenticated, loading } = useAuth();
  
  if (loading) return <div>Loading...</div>;
  if (!isAuthenticated) return <Navigate to="/login" />;
  
  return children;
}
```

---

## Production Example: Multi-Language Context

```jsx
import { createContext, useContext, useState } from 'react';

const translations = {
  en: {
    welcome: 'Welcome',
    logout: 'Logout',
    settings: 'Settings'
  },
  es: {
    welcome: 'Bienvenido',
    logout: 'Cerrar sesión',
    settings: 'Configuración'
  },
  fr: {
    welcome: 'Bienvenue',
    logout: 'Se déconnecter',
    settings: 'Paramètres'
  }
};

const LanguageContext = createContext();

export function useLanguage() {
  const context = useContext(LanguageContext);
  if (!context) {
    throw new Error('useLanguage must be used within LanguageProvider');
  }
  return context;
}

export function LanguageProvider({ children }) {
  const [language, setLanguage] = useState('en');
  
  const t = (key) => {
    return translations[language][key] || key;
  };
  
  const value = {
    language,
    setLanguage,
    t
  };
  
  return (
    <LanguageContext.Provider value={value}>
      {children}
    </LanguageContext.Provider>
  );
}

// Usage
function App() {
  return (
    <LanguageProvider>
      <Header />
      <Content />
    </LanguageProvider>
  );
}

function Header() {
  const { language, setLanguage, t } = useLanguage();
  
  return (
    <header>
      <h1>{t('welcome')}</h1>
      <select value={language} onChange={(e) => setLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Español</option>
        <option value="fr">Français</option>
      </select>
    </header>
  );
}

function Content() {
  const { t } = useLanguage();
  
  return (
    <div>
      <button>{t('settings')}</button>
      <button>{t('logout')}</button>
    </div>
  );
}
```

---

## Multiple Contexts

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext();
const UserContext = createContext();
const SettingsContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState(null);
  const [settings, setSettings] = useState({});
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={{ user, setUser }}>
        <SettingsContext.Provider value={{ settings, setSettings }}>
          <MainApp />
        </SettingsContext.Provider>
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

// Usage
function Component() {
  const { theme } = useContext(ThemeContext);
  const { user } = useContext(UserContext);
  const { settings } = useContext(SettingsContext);
  
  return <div>...</div>;
}
```

**Better: Combine providers**

```jsx
function AppProviders({ children }) {
  return (
    <ThemeProvider>
      <UserProvider>
        <SettingsProvider>
          {children}
        </SettingsProvider>
      </UserProvider>
    </ThemeProvider>
  );
}

function App() {
  return (
    <AppProviders>
      <MainApp />
    </AppProviders>
  );
}
```

---

## When to Use Context

**✅ Good Use Cases:**

1. **Theme/UI preferences** - Dark/light mode, color scheme
2. **User authentication** - Current user, login/logout
3. **Internationalization** - Current language, translations
4. **Accessibility** - Screen reader settings, keyboard navigation
5. **Feature flags** - Experimental features on/off
6. **Configuration** - API endpoints, app settings
7. **Global UI state** - Sidebar open/closed, modal state

```jsx
// ✅ Good: Theme is needed by many components
const ThemeContext = createContext();

// ✅ Good: Auth status used throughout app
const AuthContext = createContext();

// ✅ Good: Language affects many components
const LanguageContext = createContext();
```

**❌ Avoid Context When:**

1. **Prop drilling is minimal** - Only 1-2 levels deep
2. **Frequently changing values** - Context re-renders all consumers
3. **Component-specific state** - Keep state local when possible
4. **Performance critical** - Context can cause unnecessary re-renders

```jsx
// ❌ Bad: Only used in one component
const FormStateContext = createContext();

// ❌ Bad: Changes every keystroke
const SearchQueryContext = createContext();

// ❌ Bad: Better as prop
function Parent() {
  return (
    <MyContext.Provider value={data}>
      <Child />  {/* Only one child - just use prop! */}
    </MyContext.Provider>
  );
}
```

---

## Best Practices

**1. Create custom hook for context**

```jsx
// ✅ Good: Custom hook with error checking
const ThemeContext = createContext();

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Usage is cleaner
function Component() {
  const { theme } = useTheme();  // ✅ Simple
  return <div>...</div>;
}

// vs
function Component() {
  const { theme } = useContext(ThemeContext);  // ❌ More verbose
  return <div>...</div>;
}
```

**2. Separate Provider component**

```jsx
// ✅ Good: Dedicated provider component
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const value = {
    theme,
    setTheme,
    toggleTheme: () => setTheme(t => t === 'light' ? 'dark' : 'light')
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Clean usage
function App() {
  return (
    <ThemeProvider>
      <MainApp />
    </ThemeProvider>
  );
}
```

**3. Memoize context value**

```jsx
import { createContext, useMemo, useState } from 'react';

// ✅ Good: Memoize to prevent unnecessary re-renders
export function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const value = useMemo(() => ({
    user,
    setUser
  }), [user]);
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

// ❌ Bad: New object on every render
export function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>  {/* New object! */}
      {children}
    </UserContext.Provider>
  );
}
```

**4. Split contexts for performance**

```jsx
// ❌ Bad: Single context with all data
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [settings, setSettings] = useState({});
  
  // Any change re-renders ALL consumers!
  return (
    <AppContext.Provider value={{ user, theme, settings, setUser, setTheme, setSettings }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ Good: Separate contexts
const UserContext = createContext();
const ThemeContext = createContext();
const SettingsContext = createContext();

function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <SettingsProvider>
          <MainApp />
        </SettingsProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

// Components only re-render when their context changes
```

---

## Context vs Props

**Use Props when:**
- Direct parent-child relationship
- Only 1-2 levels deep
- Need explicit data flow
- Performance is critical

**Use Context when:**
- Many components need the data
- Deep component tree (3+ levels)
- Global or app-wide state
- Reduces prop drilling

---

**Interview Tips:**
- Context API provides **global state** without prop drilling
- Three parts: **createContext**, **Provider**, **useContext**
- Created with `createContext(defaultValue)`
- **Provider** component wraps tree and supplies value
- **useContext** hook accesses value in any descendant
- Solves **prop drilling** problem (passing props through many levels)
- Best for **rarely changing** data (theme, auth, language)
- **Not a replacement** for Redux/state management for complex apps
- Can have **multiple contexts** in an app
- Create **custom hooks** (useAuth, useTheme) for cleaner access
- **Memoize context value** to prevent unnecessary re-renders
- All consumers **re-render** when context value changes
- **Split contexts** by concern for better performance
- Good for: **theme, auth, language, settings, feature flags**
- Avoid for: **frequently changing state, component-specific state**
- Use **Provider pattern** - separate provider component
- **Error checking** in custom hook - throw if used outside provider
- Context consumers can be **anywhere** in tree below provider
- Default value used only if **no Provider** found above
- Can **nest providers** to override values in subtrees

</details>

---

### 42. What are the performance issues with Context API?

<details>
<summary>View Answer</summary>

**Context API Performance Issues**

While Context API is great for avoiding prop drilling, it has **performance implications** that can cause unnecessary re-renders if not used carefully.

---

## Main Performance Problem

**Issue: All consumers re-render when context value changes**

When a context value changes, **every component** that uses `useContext` will re-render, even if it only uses a small part of the context value.

```jsx
import { createContext, useContext, useState } from 'react';

const AppContext = createContext();

function App() {
  const [user, setUser] = useState({ name: 'John' });
  const [theme, setTheme] = useState('light');
  const [count, setCount] = useState(0);
  
  return (
    <AppContext.Provider value={{ user, theme, count, setTheme, setCount }}>
      <UserDisplay />
      <ThemeToggle />
      <Counter />
    </AppContext.Provider>
  );
}

function UserDisplay() {
  const { user } = useContext(AppContext);
  console.log('UserDisplay rendered');
  return <div>{user.name}</div>;
}

function ThemeToggle() {
  const { theme, setTheme } = useContext(AppContext);
  console.log('ThemeToggle rendered');
  return <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>{theme}</button>;
}

function Counter() {
  const { count, setCount } = useContext(AppContext);
  console.log('Counter rendered');
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}

// Problem:
// When count changes (clicking Counter button):
// - Counter re-renders ✅ (needs count)
// - ThemeToggle re-renders ❌ (doesn't use count!)
// - UserDisplay re-renders ❌ (doesn't use count!)
//
// When theme changes:
// - ThemeToggle re-renders ✅ (needs theme)
// - Counter re-renders ❌ (doesn't use theme!)
// - UserDisplay re-renders ❌ (doesn't use theme!)
//
// All three components re-render on ANY context change!
```

---

## Problem 1: New Object on Every Render

```jsx
// ❌ BAD: Creates new object on every render
function App() {
  const [user, setUser] = useState({ name: 'John' });
  const [theme, setTheme] = useState('light');
  
  // New object created every time App renders!
  // Even if user and theme haven't changed
  return (
    <AppContext.Provider value={{ user, theme, setUser, setTheme }}>
      <Child />
    </AppContext.Provider>
  );
}

// Every App re-render causes context value to change
// All consumers re-render even if data is the same
```

**Solution: Memoize context value**

```jsx
import { useMemo } from 'react';

// ✅ GOOD: Memoize context value
function App() {
  const [user, setUser] = useState({ name: 'John' });
  const [theme, setTheme] = useState('light');
  
  // Only create new object when dependencies change
  const value = useMemo(() => ({
    user,
    theme,
    setUser,
    setTheme
  }), [user, theme]);
  
  return (
    <AppContext.Provider value={value}>
      <Child />
    </AppContext.Provider>
  );
}

// Context value only changes when user or theme changes
// Prevents unnecessary re-renders
```

---

## Problem 2: Single Large Context

```jsx
// ❌ BAD: Everything in one context
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [language, setLanguage] = useState('en');
  const [notifications, setNotifications] = useState([]);
  const [cart, setCart] = useState([]);
  
  const value = {
    user, setUser,
    theme, setTheme,
    language, setLanguage,
    notifications, setNotifications,
    cart, setCart
  };
  
  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// Component only needs user
function UserProfile() {
  const { user } = useContext(AppContext);
  return <div>{user?.name}</div>;
}

// But it re-renders when ANY value changes:
// - theme changes → UserProfile re-renders ❌
// - cart changes → UserProfile re-renders ❌
// - notifications change → UserProfile re-renders ❌
```

**Solution: Split into multiple contexts**

```jsx
// ✅ GOOD: Separate contexts by concern
const UserContext = createContext();
const ThemeContext = createContext();
const LanguageContext = createContext();
const NotificationContext = createContext();
const CartContext = createContext();

function AppProviders({ children }) {
  return (
    <UserProvider>
      <ThemeProvider>
        <LanguageProvider>
          <NotificationProvider>
            <CartProvider>
              {children}
            </CartProvider>
          </NotificationProvider>
        </LanguageProvider>
      </ThemeProvider>
    </UserProvider>
  );
}

// Now components only re-render when their context changes
function UserProfile() {
  const { user } = useContext(UserContext);
  return <div>{user?.name}</div>;
}

// UserProfile only re-renders when UserContext changes
// Theme/cart/notification changes don't affect it ✅
```

---

## Problem 3: Expensive Context Value Computation

```jsx
// ❌ BAD: Expensive computation on every render
function UserProvider({ children }) {
  const [users, setUsers] = useState([]);
  
  // Recalculated on every render, even if users unchanged!
  const sortedUsers = users.sort((a, b) => a.name.localeCompare(b.name));
  const userCount = users.length;
  const adminUsers = users.filter(u => u.role === 'admin');
  
  const value = {
    users,
    sortedUsers,
    userCount,
    adminUsers,
    setUsers
  };
  
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}
```

**Solution: Memoize expensive computations**

```jsx
import { useMemo } from 'react';

// ✅ GOOD: Memoize expensive computations
function UserProvider({ children }) {
  const [users, setUsers] = useState([]);
  
  // Only recalculate when users changes
  const sortedUsers = useMemo(() => {
    return [...users].sort((a, b) => a.name.localeCompare(b.name));
  }, [users]);
  
  const userCount = useMemo(() => users.length, [users]);
  
  const adminUsers = useMemo(() => {
    return users.filter(u => u.role === 'admin');
  }, [users]);
  
  const value = useMemo(() => ({
    users,
    sortedUsers,
    userCount,
    adminUsers,
    setUsers
  }), [users, sortedUsers, userCount, adminUsers]);
  
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}
```

---

## Solution 1: Split Context by Update Frequency

```jsx
// ✅ Split frequently changing and rarely changing data

// Rarely changes - user logs in once
const UserContext = createContext();

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

// Changes frequently - every keystroke
const SearchContext = createContext();

function SearchProvider({ children }) {
  const [query, setQuery] = useState('');
  
  const value = useMemo(() => ({ query, setQuery }), [query]);
  
  return <SearchContext.Provider value={value}>{children}</SearchContext.Provider>;
}

// Components using UserContext won't re-render when search query changes
// Components using SearchContext won't re-render when user changes
```

---

## Solution 2: Separate State and Dispatch

```jsx
// ✅ Split state and updater functions into separate contexts

const CountStateContext = createContext();
const CountDispatchContext = createContext();

function CountProvider({ children }) {
  const [count, setCount] = useState(0);
  
  return (
    <CountStateContext.Provider value={count}>
      <CountDispatchContext.Provider value={setCount}>
        {children}
      </CountDispatchContext.Provider>
    </CountStateContext.Provider>
  );
}

// Component that only displays count
function CountDisplay() {
  const count = useContext(CountStateContext);
  console.log('CountDisplay rendered');
  return <div>Count: {count}</div>;
}

// Component that only updates count
function CountControls() {
  const setCount = useContext(CountDispatchContext);
  console.log('CountControls rendered');
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <button onClick={() => setCount(c => c - 1)}>Decrement</button>
    </div>
  );
}

// When count changes:
// - CountDisplay re-renders ✅ (uses count)
// - CountControls doesn't re-render ✅ (only uses setCount)
```

---

## Solution 3: Use React.memo with Context

```jsx
import { memo } from 'react';

const AppContext = createContext();

function App() {
  const [user, setUser] = useState({ name: 'John' });
  const [count, setCount] = useState(0);
  
  const value = useMemo(() => ({ user, count, setUser, setCount }), [user, count]);
  
  return (
    <AppContext.Provider value={value}>
      <ExpensiveComponent someProp="static" />
      <Counter />
    </AppContext.Provider>
  );
}

// ✅ Memoized component won't re-render if props don't change
const ExpensiveComponent = memo(({ someProp }) => {
  // This doesn't use context
  console.log('ExpensiveComponent rendered');
  return <div>Expensive: {someProp}</div>;
});

// When context changes, ExpensiveComponent won't re-render
// because it doesn't use context and its props haven't changed

function Counter() {
  const { count, setCount } = useContext(AppContext);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```

---

## Solution 4: Context Selectors (Manual)

```jsx
// Custom implementation of context selectors

function createContextWithSelector() {
  const Context = createContext();
  
  function Provider({ value, children }) {
    const listeners = useRef(new Set());
    const valueRef = useRef(value);
    
    useEffect(() => {
      valueRef.current = value;
      listeners.current.forEach(listener => listener());
    }, [value]);
    
    const subscribe = useCallback((callback) => {
      listeners.current.add(callback);
      return () => listeners.current.delete(callback);
    }, []);
    
    const contextValue = useMemo(() => ({
      value: valueRef,
      subscribe
    }), [subscribe]);
    
    return <Context.Provider value={contextValue}>{children}</Context.Provider>;
  }
  
  function useSelector(selector) {
    const context = useContext(Context);
    const [, forceRender] = useReducer(s => s + 1, 0);
    const selectedValue = useRef();
    
    selectedValue.current = selector(context.value.current);
    
    useEffect(() => {
      return context.subscribe(() => {
        const newValue = selector(context.value.current);
        if (newValue !== selectedValue.current) {
          selectedValue.current = newValue;
          forceRender();
        }
      });
    }, [context, selector]);
    
    return selectedValue.current;
  }
  
  return { Provider, useSelector };
}

// Usage
const { Provider: AppProvider, useSelector } = createContextWithSelector();

function App() {
  const [user, setUser] = useState({ name: 'John' });
  const [theme, setTheme] = useState('light');
  
  return (
    <AppProvider value={{ user, theme, setUser, setTheme }}>
      <UserDisplay />
      <ThemeToggle />
    </AppProvider>
  );
}

function UserDisplay() {
  // Only re-renders when user changes
  const user = useSelector(state => state.user);
  return <div>{user.name}</div>;
}

function ThemeToggle() {
  // Only re-renders when theme changes
  const theme = useSelector(state => state.theme);
  const setTheme = useSelector(state => state.setTheme);
  return <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>{theme}</button>;
}
```

---

## Solution 5: Use External Library

**Libraries that solve Context performance issues:**

1. **use-context-selector** - Context with selectors
```bash
npm install use-context-selector
```

```jsx
import { createContext, useContextSelector } from 'use-context-selector';

const AppContext = createContext();

function App() {
  const [user, setUser] = useState({ name: 'John' });
  const [theme, setTheme] = useState('light');
  
  return (
    <AppContext.Provider value={{ user, theme, setUser, setTheme }}>
      <UserDisplay />
      <ThemeToggle />
    </AppContext.Provider>
  );
}

function UserDisplay() {
  // Only re-renders when user changes
  const user = useContextSelector(AppContext, state => state.user);
  return <div>{user.name}</div>;
}

function ThemeToggle() {
  // Only re-renders when theme changes
  const theme = useContextSelector(AppContext, state => state.theme);
  const setTheme = useContextSelector(AppContext, state => state.setTheme);
  return <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>{theme}</button>;
}
```

2. **Zustand** - Lightweight state management
3. **Jotai** - Atomic state management
4. **Recoil** - State management by Facebook

---

## Best Practices Summary

**1. Memoize context value**
```jsx
const value = useMemo(() => ({ data, setData }), [data]);
```

**2. Split contexts by concern**
```jsx
// Separate: UserContext, ThemeContext, SettingsContext
// Not: AppContext with everything
```

**3. Separate state and dispatch**
```jsx
// StateContext, DispatchContext
// Components using only dispatch won't re-render
```

**4. Keep context values simple**
```jsx
// ✅ Good: { user, theme }
// ❌ Bad: { user, theme, sortedUsers, filteredUsers, ... }
```

**5. Use React.memo for expensive components**
```jsx
const ExpensiveComponent = memo(Component);
```

**6. Consider alternatives for frequently changing data**
```jsx
// For frequent updates: useState, useReducer, or state management library
// Context best for: infrequent updates (theme, auth, language)
```

---

**When to Use Alternatives:**

- **Frequent updates** → useState, useReducer, or state management library
- **Complex state logic** → Redux, Zustand, Jotai
- **Need selectors** → use-context-selector, Zustand, Recoil
- **Large apps** → Consider proper state management solution

---

**Interview Tips:**
- Context causes **all consumers to re-render** when value changes
- Main issue: **unnecessary re-renders** of components that don't use changed data
- **Problem 1**: Creating new object on every render - use `useMemo`
- **Problem 2**: Single large context - split into multiple contexts
- **Problem 3**: Expensive computations - memoize with `useMemo`
- **Solution**: Split contexts by **concern** and **update frequency**
- **Solution**: Separate **state** and **dispatch** contexts
- **Solution**: Use `React.memo` for components that don't use context
- **Solution**: Context selectors (use-context-selector library)
- Context best for **infrequently changing** data
- Not ideal for **frequently changing** data (search input, animations)
- Can't **subscribe to part** of context - all or nothing
- **No built-in selector** mechanism in React Context
- Consider **state management libraries** for complex scenarios
- Libraries like **Zustand, Jotai, Recoil** solve these issues
- Always **memoize** context value object
- Split by **domain**: UserContext, ThemeContext, not AppContext
- Use **proper tools** for the job - Context isn't state management
- Performance issues are **manageable** with proper patterns
- Monitor with **React DevTools** Profiler to identify re-render issues

</details>

---

### 43. What is Redux and how does it work?

<details>
<summary>View Answer</summary>

**Redux**

Redux is a **predictable state container** for JavaScript applications. It helps manage **global application state** in a centralized store using a unidirectional data flow pattern.

**Core Concepts:**

1. **Single source of truth** - One store for entire app state
2. **State is read-only** - Only change via actions
3. **Changes made with pure functions** - Reducers

**Three Main Parts:**

```
┌─────────┐
│  Store  │  ← Single source of truth
└────┬────┘
     │
     ├→ State
     ├→ Dispatch (action)
     └→ Subscribe (listen to changes)
```

---

## Core Principles

**1. Single Store**

```jsx
import { createStore } from 'redux';

// Entire app state lives in one store
const store = createStore(rootReducer);

// State structure
{
  user: { name: 'John', id: 1 },
  todos: [{ id: 1, text: 'Learn Redux', completed: false }],
  ui: { theme: 'light', sidebarOpen: true }
}
```

**2. State is Read-Only**

```jsx
// ❌ NEVER mutate state directly
state.user.name = 'Jane';  // ❌ Wrong!
state.todos.push(newTodo);  // ❌ Wrong!

// ✅ Dispatch actions to change state
store.dispatch({ type: 'UPDATE_USER', payload: { name: 'Jane' } });
store.dispatch({ type: 'ADD_TODO', payload: newTodo });
```

**3. Pure Reducer Functions**

```jsx
// Reducer: (state, action) => newState
function userReducer(state = initialState, action) {
  switch (action.type) {
    case 'UPDATE_USER':
      return { ...state, ...action.payload };  // ✅ New object
    default:
      return state;
  }
}
```

---

## How Redux Works: Data Flow

```
┌──────────────────────────────────────────────┐
│                                              │
│   1. User clicks button                     │
│          ↓                                   │
│   2. Dispatch ACTION                         │
│          ↓                                   │
│   3. REDUCER processes action                │
│          ↓                                   │
│   4. STORE updates with new state            │
│          ↓                                   │
│   5. React components RE-RENDER              │
│          ↓                                   │
│   6. UI updates                              │
│                                              │
└──────────────────────────────────────────────┘
```

---

## Basic Example: Counter

**Step 1: Define Actions**

```jsx
// Action types
const INCREMENT = 'INCREMENT';
const DECREMENT = 'DECREMENT';
const RESET = 'RESET';

// Action creators
function increment() {
  return { type: INCREMENT };
}

function decrement() {
  return { type: DECREMENT };
}

function reset() {
  return { type: RESET };
}
```

**Step 2: Create Reducer**

```jsx
// Initial state
const initialState = {
  count: 0
};

// Reducer
function counterReducer(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return { count: state.count + 1 };
    
    case DECREMENT:
      return { count: state.count - 1 };
    
    case RESET:
      return { count: 0 };
    
    default:
      return state;
  }
}
```

**Step 3: Create Store**

```jsx
import { createStore } from 'redux';

const store = createStore(counterReducer);
```

**Step 4: Use in React**

```jsx
import { Provider, useSelector, useDispatch } from 'react-redux';

// Wrap app with Provider
function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}

// Access state and dispatch
function Counter() {
  const count = useSelector(state => state.count);
  const dispatch = useDispatch();
  
  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
      <button onClick={() => dispatch(reset())}>Reset</button>
    </div>
  );
}
```

---

## Production Example: Todo App

**1. Actions**

```jsx
// Action types
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'TOGGLE_TODO';
const DELETE_TODO = 'DELETE_TODO';
const SET_FILTER = 'SET_FILTER';

// Action creators
function addTodo(text) {
  return {
    type: ADD_TODO,
    payload: {
      id: Date.now(),
      text,
      completed: false
    }
  };
}

function toggleTodo(id) {
  return {
    type: TOGGLE_TODO,
    payload: id
  };
}

function deleteTodo(id) {
  return {
    type: DELETE_TODO,
    payload: id
  };
}

function setFilter(filter) {
  return {
    type: SET_FILTER,
    payload: filter  // 'all', 'active', 'completed'
  };
}
```

**2. Reducer**

```jsx
const initialState = {
  todos: [],
  filter: 'all'
};

function todoReducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, action.payload]
      };
    
    case TOGGLE_TODO:
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    
    case DELETE_TODO:
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    
    case SET_FILTER:
      return {
        ...state,
        filter: action.payload
      };
    
    default:
      return state;
  }
}
```

**3. Store**

```jsx
import { createStore } from 'redux';

const store = createStore(
  todoReducer,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
);
```

**4. Components**

```jsx
import { Provider, useSelector, useDispatch } from 'react-redux';
import { useState } from 'react';

function App() {
  return (
    <Provider store={store}>
      <TodoApp />
    </Provider>
  );
}

function TodoApp() {
  return (
    <div>
      <AddTodo />
      <FilterButtons />
      <TodoList />
    </div>
  );
}

function AddTodo() {
  const [text, setText] = useState('');
  const dispatch = useDispatch();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      dispatch(addTodo(text));
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="Add todo..."
      />
      <button type="submit">Add</button>
    </form>
  );
}

function FilterButtons() {
  const filter = useSelector(state => state.filter);
  const dispatch = useDispatch();
  
  return (
    <div>
      <button
        onClick={() => dispatch(setFilter('all'))}
        disabled={filter === 'all'}
      >
        All
      </button>
      <button
        onClick={() => dispatch(setFilter('active'))}
        disabled={filter === 'active'}
      >
        Active
      </button>
      <button
        onClick={() => dispatch(setFilter('completed'))}
        disabled={filter === 'completed'}
      >
        Completed
      </button>
    </div>
  );
}

function TodoList() {
  const todos = useSelector(state => state.todos);
  const filter = useSelector(state => state.filter);
  
  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });
  
  return (
    <ul>
      {filteredTodos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}

function TodoItem({ todo }) {
  const dispatch = useDispatch();
  
  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => dispatch(toggleTodo(todo.id))}
      />
      <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
        {todo.text}
      </span>
      <button onClick={() => dispatch(deleteTodo(todo.id))}>Delete</button>
    </li>
  );
}
```

---

## Multiple Reducers: combineReducers

**Split large state into multiple reducers:**

```jsx
import { combineReducers, createStore } from 'redux';

// User reducer
const userReducer = (state = null, action) => {
  switch (action.type) {
    case 'SET_USER':
      return action.payload;
    case 'LOGOUT':
      return null;
    default:
      return state;
  }
};

// Todos reducer
const todosReducer = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload];
    case 'DELETE_TODO':
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
};

// UI reducer
const uiReducer = (state = { theme: 'light' }, action) => {
  switch (action.type) {
    case 'SET_THEME':
      return { ...state, theme: action.payload };
    default:
      return state;
  }
};

// Combine into root reducer
const rootReducer = combineReducers({
  user: userReducer,
  todos: todosReducer,
  ui: uiReducer
});

// State structure:
// {
//   user: { ... },
//   todos: [ ... ],
//   ui: { ... }
// }

const store = createStore(rootReducer);

// Access in components
function Component() {
  const user = useSelector(state => state.user);
  const todos = useSelector(state => state.todos);
  const theme = useSelector(state => state.ui.theme);
  
  return <div>...</div>;
}
```

---

## Async Actions with Middleware

**Redux Thunk for async operations:**

```bash
npm install redux-thunk
```

```jsx
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';

// Action types
const FETCH_USERS_REQUEST = 'FETCH_USERS_REQUEST';
const FETCH_USERS_SUCCESS = 'FETCH_USERS_SUCCESS';
const FETCH_USERS_FAILURE = 'FETCH_USERS_FAILURE';

// Async action creator (thunk)
function fetchUsers() {
  return async (dispatch) => {
    // Start request
    dispatch({ type: FETCH_USERS_REQUEST });
    
    try {
      const response = await fetch('/api/users');
      const data = await response.json();
      
      // Success
      dispatch({
        type: FETCH_USERS_SUCCESS,
        payload: data
      });
    } catch (error) {
      // Failure
      dispatch({
        type: FETCH_USERS_FAILURE,
        payload: error.message
      });
    }
  };
}

// Reducer
const initialState = {
  users: [],
  loading: false,
  error: null
};

function usersReducer(state = initialState, action) {
  switch (action.type) {
    case FETCH_USERS_REQUEST:
      return { ...state, loading: true, error: null };
    
    case FETCH_USERS_SUCCESS:
      return { ...state, loading: false, users: action.payload };
    
    case FETCH_USERS_FAILURE:
      return { ...state, loading: false, error: action.payload };
    
    default:
      return state;
  }
}

// Create store with middleware
const store = createStore(
  usersReducer,
  applyMiddleware(thunk)
);

// Use in component
function UserList() {
  const { users, loading, error } = useSelector(state => state);
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch(fetchUsers());
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
}
```

---

## Selectors for Computed State

```jsx
// Selectors - derive data from state
const selectTodos = state => state.todos;
const selectFilter = state => state.filter;

const selectVisibleTodos = state => {
  const todos = selectTodos(state);
  const filter = selectFilter(state);
  
  switch (filter) {
    case 'active':
      return todos.filter(todo => !todo.completed);
    case 'completed':
      return todos.filter(todo => todo.completed);
    default:
      return todos;
  }
};

const selectTodoCount = state => selectTodos(state).length;
const selectCompletedCount = state => selectTodos(state).filter(t => t.completed).length;

// Use in components
function TodoList() {
  const visibleTodos = useSelector(selectVisibleTodos);
  const totalCount = useSelector(selectTodoCount);
  const completedCount = useSelector(selectCompletedCount);
  
  return (
    <div>
      <p>{completedCount} / {totalCount} completed</p>
      <ul>
        {visibleTodos.map(todo => (
          <TodoItem key={todo.id} todo={todo} />
        ))}
      </ul>
    </div>
  );
}
```

---

## Redux DevTools

**Time-travel debugging:**

```jsx
import { createStore } from 'redux';

const store = createStore(
  rootReducer,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
);
```

**Features:**
- See all actions dispatched
- Inspect state at any point
- Time-travel between actions
- Replay actions
- Export/import state

---

## Best Practices

**1. Keep reducers pure**

```jsx
// ✅ Good: Pure function
function reducer(state, action) {
  return { ...state, count: state.count + 1 };
}

// ❌ Bad: Mutates state
function reducer(state, action) {
  state.count++;
  return state;
}

// ❌ Bad: Side effects
function reducer(state, action) {
  fetch('/api/data');  // Side effect!
  return { ...state, count: state.count + 1 };
}
```

**2. Use action constants**

```jsx
// ✅ Good: Constants prevent typos
const ADD_TODO = 'ADD_TODO';
dispatch({ type: ADD_TODO });

// ❌ Bad: String literals
dispatch({ type: 'ADD_TODO' });
dispatch({ type: 'ADD_TOD0' });  // Typo! Silent failure
```

**3. Normalize state shape**

```jsx
// ❌ Bad: Nested arrays
const state = {
  posts: [
    { id: 1, title: 'Post 1', comments: [
      { id: 1, text: 'Comment 1' },
      { id: 2, text: 'Comment 2' }
    ]}
  ]
};

// ✅ Good: Normalized
const state = {
  posts: {
    byId: {
      1: { id: 1, title: 'Post 1', commentIds: [1, 2] }
    },
    allIds: [1]
  },
  comments: {
    byId: {
      1: { id: 1, text: 'Comment 1' },
      2: { id: 2, text: 'Comment 2' }
    },
    allIds: [1, 2]
  }
};
```

**4. Use Redux Toolkit (modern approach)**

```jsx
import { configureStore, createSlice } from '@reduxjs/toolkit';

// Slice combines actions and reducers
const counterSlice = createSlice({
  name: 'counter',
  initialState: { count: 0 },
  reducers: {
    increment: state => { state.count += 1 },  // Immer allows mutations!
    decrement: state => { state.count -= 1 },
    reset: state => { state.count = 0 }
  }
});

export const { increment, decrement, reset } = counterSlice.actions;

const store = configureStore({
  reducer: counterSlice.reducer
});
```

---

**Interview Tips:**
- Redux is a **predictable state container** for JavaScript apps
- **Three principles**: Single source of truth, state is read-only, changes via pure functions
- **Store** - Holds entire app state
- **Actions** - Plain objects describing what happened
- **Reducers** - Pure functions that return new state
- **Dispatch** - Send actions to store
- **Unidirectional data flow**: Action → Reducer → Store → View
- **combineReducers** - Split state logic into multiple reducers
- **Middleware** - Extends Redux (async, logging, etc.)
- **Redux Thunk** - Handle async actions
- **Selectors** - Compute derived data from state
- **Redux DevTools** - Time-travel debugging
- Reducers must be **pure** - no mutations, no side effects
- **Action creators** - Functions that return actions
- State is **immutable** - always return new objects
- **Use constants** for action types to prevent typos
- **Normalize state** - avoid deeply nested structures
- Modern Redux uses **Redux Toolkit** (recommended)
- Redux provides **predictable state updates** with clear data flow
- Not needed for **every app** - use for complex state
- Alternative to **Context API** for large apps

</details>

---

### 44. What is the difference between Redux and Context API?

<details>
<summary>View Answer</summary>

**Redux vs Context API**

While both Redux and Context API can manage global state in React applications, they have different purposes, capabilities, and use cases.

---

## Key Differences

| Feature | Context API | Redux |
|---------|-------------|-------|
| **Purpose** | Share data across components | State management solution |
| **Complexity** | Simple, built into React | More complex, external library |
| **Boilerplate** | Minimal | More setup required |
| **Performance** | All consumers re-render | Optimized with selectors |
| **DevTools** | None built-in | Redux DevTools (time-travel) |
| **Middleware** | None | Yes (thunks, saga, etc.) |
| **Async** | Manual handling | Built-in with middleware |
| **Structure** | Flexible | Strict patterns |
| **Learning Curve** | Easy | Moderate to steep |
| **Best For** | Simple global state | Complex state logic |

---

## Context API

**What it is:**
- Built-in React feature
- Designed to **avoid prop drilling**
- Share data across component tree
- Not specifically for state management

**Structure:**

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Create context
const ThemeContext = createContext();

// 2. Provider
function App() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Component />
    </ThemeContext.Provider>
  );
}

// 3. Consumer
function Component() {
  const { theme, setTheme } = useContext(ThemeContext);
  return <div>{theme}</div>;
}
```

**Pros:**
- ✅ Built into React - no installation
- ✅ Simple API - easy to learn
- ✅ Minimal boilerplate
- ✅ Good for simple state sharing
- ✅ Flexible structure

**Cons:**
- ❌ No built-in optimization - all consumers re-render
- ❌ No middleware support
- ❌ No dev tools
- ❌ Manual async handling
- ❌ Can become complex with many contexts

---

## Redux

**What it is:**
- External state management library
- Predictable state container
- Enforces unidirectional data flow
- Complete state management solution

**Structure:**

```jsx
import { createStore } from 'redux';
import { Provider, useSelector, useDispatch } from 'react-redux';

// 1. Actions
const SET_THEME = 'SET_THEME';
const setTheme = (theme) => ({ type: SET_THEME, payload: theme });

// 2. Reducer
const themeReducer = (state = { theme: 'light' }, action) => {
  switch (action.type) {
    case SET_THEME:
      return { ...state, theme: action.payload };
    default:
      return state;
  }
};

// 3. Store
const store = createStore(themeReducer);

// 4. Provider
function App() {
  return (
    <Provider store={store}>
      <Component />
    </Provider>
  );
}

// 5. Consumer
function Component() {
  const theme = useSelector(state => state.theme);
  const dispatch = useDispatch();
  
  return (
    <div>
      {theme}
      <button onClick={() => dispatch(setTheme('dark'))}>Toggle</button>
    </div>
  );
}
```

**Pros:**
- ✅ Performance optimized with selectors
- ✅ Middleware ecosystem (thunk, saga)
- ✅ Redux DevTools - time-travel debugging
- ✅ Predictable state updates
- ✅ Great for complex state logic
- ✅ Strong patterns and best practices

**Cons:**
- ❌ Extra dependency
- ❌ More boilerplate
- ❌ Steeper learning curve
- ❌ Overkill for simple apps

---

## Performance Comparison

**Context API - Re-render behavior:**

```jsx
import { createContext, useContext, useState } from 'react';

const AppContext = createContext();

function App() {
  const [user, setUser] = useState({ name: 'John' });
  const [theme, setTheme] = useState('light');
  
  return (
    <AppContext.Provider value={{ user, theme, setUser, setTheme }}>
      <UserDisplay />  {/* Uses user */}
      <ThemeDisplay />  {/* Uses theme */}
    </AppContext.Provider>
  );
}

function UserDisplay() {
  const { user } = useContext(AppContext);
  console.log('UserDisplay rendered');
  return <div>{user.name}</div>;
}

function ThemeDisplay() {
  const { theme } = useContext(AppContext);
  console.log('ThemeDisplay rendered');
  return <div>{theme}</div>;
}

// Problem:
// When theme changes:
// - ThemeDisplay re-renders ✅ (needs it)
// - UserDisplay re-renders ❌ (doesn't need it!)
//
// When user changes:
// - UserDisplay re-renders ✅ (needs it)
// - ThemeDisplay re-renders ❌ (doesn't need it!)
```

**Redux - Optimized with selectors:**

```jsx
import { createStore } from 'redux';
import { Provider, useSelector } from 'react-redux';

const initialState = {
  user: { name: 'John' },
  theme: 'light'
};

function reducer(state = initialState, action) {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload };
    case 'SET_THEME':
      return { ...state, theme: action.payload };
    default:
      return state;
  }
}

const store = createStore(reducer);

function App() {
  return (
    <Provider store={store}>
      <UserDisplay />
      <ThemeDisplay />
    </Provider>
  );
}

function UserDisplay() {
  const user = useSelector(state => state.user);  // Selects only user
  console.log('UserDisplay rendered');
  return <div>{user.name}</div>;
}

function ThemeDisplay() {
  const theme = useSelector(state => state.theme);  // Selects only theme
  console.log('ThemeDisplay rendered');
  return <div>{theme}</div>;
}

// Better:
// When theme changes:
// - ThemeDisplay re-renders ✅ (needs it)
// - UserDisplay doesn't re-render ✅ (selector unchanged)
//
// When user changes:
// - UserDisplay re-renders ✅ (needs it)
// - ThemeDisplay doesn't re-render ✅ (selector unchanged)
```

---

## Async Operations

**Context API - Manual handling:**

```jsx
import { createContext, useContext, useState, useEffect } from 'react';

const UserContext = createContext();

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  // Manual async logic
  const fetchUser = async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch('/api/user');
      const data = await response.json();
      setUser(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <UserContext.Provider value={{ user, loading, error, fetchUser }}>
      {children}
    </UserContext.Provider>
  );
}

// Usage
function Component() {
  const { user, loading, error, fetchUser } = useContext(UserContext);
  
  useEffect(() => {
    fetchUser();
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{user?.name}</div>;
}
```

**Redux - With middleware (Redux Thunk):**

```jsx
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import { useSelector, useDispatch } from 'react-redux';

// Action creators
const fetchUserRequest = () => ({ type: 'FETCH_USER_REQUEST' });
const fetchUserSuccess = (user) => ({ type: 'FETCH_USER_SUCCESS', payload: user });
const fetchUserFailure = (error) => ({ type: 'FETCH_USER_FAILURE', payload: error });

// Thunk action
const fetchUser = () => async (dispatch) => {
  dispatch(fetchUserRequest());
  
  try {
    const response = await fetch('/api/user');
    const data = await response.json();
    dispatch(fetchUserSuccess(data));
  } catch (error) {
    dispatch(fetchUserFailure(error.message));
  }
};

// Reducer
const initialState = { user: null, loading: false, error: null };

function userReducer(state = initialState, action) {
  switch (action.type) {
    case 'FETCH_USER_REQUEST':
      return { ...state, loading: true, error: null };
    case 'FETCH_USER_SUCCESS':
      return { ...state, loading: false, user: action.payload };
    case 'FETCH_USER_FAILURE':
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
}

const store = createStore(userReducer, applyMiddleware(thunk));

// Usage
function Component() {
  const { user, loading, error } = useSelector(state => state);
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch(fetchUser());
  }, []);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{user?.name}</div>;
}
```

---

## DevTools

**Context API:**
- ❌ No built-in dev tools
- Can use React DevTools to inspect context values
- No time-travel debugging
- No action history

**Redux:**
- ✅ Redux DevTools Extension
- ✅ See all actions dispatched
- ✅ Time-travel debugging
- ✅ Action replay
- ✅ State export/import
- ✅ Performance monitoring

```jsx
import { createStore } from 'redux';

const store = createStore(
  reducer,
  window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
);

// Now you can:
// - See action history
// - Jump to any state
// - Replay actions
// - Export/import state
```

---

## When to Use Each

**Use Context API when:**

✅ **Simple global state**
```jsx
// Theme, language, user preferences
const ThemeContext = createContext();
const LanguageContext = createContext();
```

✅ **Avoiding prop drilling**
```jsx
// Pass data through many levels
<UserContext.Provider value={user}>
  <Layout>
    <Sidebar>
      <UserMenu />  {/* Accesses user directly */}
    </Sidebar>
  </Layout>
</UserContext.Provider>
```

✅ **Infrequently changing data**
```jsx
// Auth state changes rarely (login/logout)
const AuthContext = createContext();
```

✅ **Small to medium apps**
```jsx
// < 10 contexts, simple state logic
```

✅ **No complex state logic**
```jsx
// Just sharing values, no complex updates
```

---

**Use Redux when:**

✅ **Complex state logic**
```jsx
// Multiple related state updates
// Derived state calculations
// Complex business logic
```

✅ **Frequently changing state**
```jsx
// Real-time updates
// Form state with many fields
// Shopping cart
```

✅ **Need middleware**
```jsx
// Async operations with thunk/saga
// Logging, analytics
// API call management
```

✅ **Large teams/apps**
```jsx
// Predictable patterns
// Easier to maintain
// Better debugging
```

✅ **Need DevTools**
```jsx
// Time-travel debugging
// Action history
// State inspection
```

✅ **Performance critical**
```jsx
// Selector optimization
// Prevent unnecessary re-renders
```

---

## Combining Both

**You can use both in the same app:**

```jsx
import { Provider as ReduxProvider } from 'react-redux';
import { ThemeProvider } from './ThemeContext';
import { AuthProvider } from './AuthContext';
import store from './store';

function App() {
  return (
    // Redux for complex app state
    <ReduxProvider store={store}>
      {/* Context for simple UI state */}
      <ThemeProvider>
        <AuthProvider>
          <AppContent />
        </AuthProvider>
      </ThemeProvider>
    </ReduxProvider>
  );
}

// Redux: todos, products, cart, complex state
// Context: theme, language, simple auth
```

---

## Migration Path

**Start with Context, migrate to Redux if needed:**

```jsx
// Phase 1: Start simple with Context
function App() {
  return (
    <UserContext.Provider>
      <ThemeContext.Provider>
        <AppContent />
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Phase 2: App grows, add Redux for complex state
function App() {
  return (
    <Provider store={reduxStore}>
      <UserContext.Provider>  {/* Keep simple contexts */}
        <ThemeContext.Provider>
          <AppContent />
        </ThemeContext.Provider>
      </UserContext.Provider>
    </Provider>
  );
}

// Phase 3: Gradually move complex state to Redux
function App() {
  return (
    <Provider store={reduxStore}>  {/* All complex state */}
      <ThemeContext.Provider>  {/* Only UI preferences */}
        <AppContent />
      </ThemeContext.Provider>
    </Provider>
  );
}
```

---

## Summary

**Context API:**
- Built-in React feature
- Simple API, minimal boilerplate
- Good for simple state sharing
- No optimization, no dev tools
- Best for: theme, language, auth

**Redux:**
- External state management library
- More complex, more boilerplate
- Performance optimized with selectors
- Middleware, dev tools, strict patterns
- Best for: complex apps, frequent updates

**Choose based on:**
- App complexity
- Team size
- State complexity
- Performance needs
- Debugging requirements

---

**Interview Tips:**
- Context API is **built into React**, Redux is **external library**
- Context designed for **prop drilling**, Redux for **state management**
- Context has **all consumers re-render**, Redux uses **selectors** for optimization
- Redux has **middleware** (thunk, saga), Context doesn't
- Redux has **DevTools** with time-travel, Context doesn't
- Context has **less boilerplate**, Redux has **more structure**
- Redux better for **complex state logic**, Context for **simple sharing**
- Redux has **predictable patterns**, Context is **more flexible**
- Context good for **infrequent updates**, Redux for **frequent changes**
- Both can be used **together** in same app
- Context: theme, language, simple auth
- Redux: todos, cart, complex business logic
- **Start with Context**, migrate to Redux if needed
- Redux provides better **performance** at scale
- Context is **easier to learn**, Redux has **steeper curve**
- Redux has **larger ecosystem** (middleware, tools)
- Context is **sufficient** for many apps
- Don't use Redux if **Context works** - keep it simple
- Modern alternatives: **Zustand, Jotai, Recoil** - simpler than Redux
- Choose based on: **complexity, team size, performance needs**

</details>

---

### 45. What are Redux Toolkit and its advantages?

<details>
<summary>View Answer</summary>

**Redux Toolkit (RTK)**

Redux Toolkit is the **official, opinionated, batteries-included toolset** for efficient Redux development. It's the recommended way to write Redux logic, simplifying common use cases and reducing boilerplate.

**Installation:**

```bash
npm install @reduxjs/toolkit react-redux
```

**Tagline:** *"The official, opinionated, batteries-included toolset for efficient Redux development."*

---

## Why Redux Toolkit?

**Problems with Traditional Redux:**

1. **Too much boilerplate** - Action types, action creators, reducers
2. **Complex store setup** - Middleware, DevTools, enhancers
3. **Immutable updates are verbose** - Spread operators everywhere
4. **No built-in async handling** - Need to add thunk manually
5. **Easy to make mistakes** - Mutating state, forgetting to return

```jsx
// ❌ Traditional Redux - Lots of boilerplate

// Action types
const INCREMENT = 'counter/INCREMENT';
const DECREMENT = 'counter/DECREMENT';
const INCREMENT_BY = 'counter/INCREMENT_BY';

// Action creators
const increment = () => ({ type: INCREMENT });
const decrement = () => ({ type: DECREMENT });
const incrementBy = (amount) => ({ type: INCREMENT_BY, payload: amount });

// Reducer
const initialState = { value: 0 };

function counterReducer(state = initialState, action) {
  switch (action.type) {
    case INCREMENT:
      return { ...state, value: state.value + 1 };
    case DECREMENT:
      return { ...state, value: state.value - 1 };
    case INCREMENT_BY:
      return { ...state, value: state.value + action.payload };
    default:
      return state;
  }
}

// Store setup
import { createStore, applyMiddleware, combineReducers } from 'redux';
import thunk from 'redux-thunk';
import { composeWithDevTools } from 'redux-devtools-extension';

const rootReducer = combineReducers({ counter: counterReducer });

const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(thunk))
);
```

---

## Redux Toolkit Solution

**Same functionality with much less code:**

```jsx
import { configureStore, createSlice } from '@reduxjs/toolkit';

// ✅ Redux Toolkit - Much simpler!

// Create slice (combines actions + reducer)
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;  // ✅ Can "mutate" with Immer!
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementBy: (state, action) => {
      state.value += action.payload;
    }
  }
});

// Export actions (auto-generated)
export const { increment, decrement, incrementBy } = counterSlice.actions;

// Configure store (includes thunk + DevTools automatically)
const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  }
});
```

**Benefits:**
- **80% less code** for same functionality
- **No action type constants** - generated automatically
- **No action creators** - generated automatically
- **Can "mutate" state** - Immer handles immutability
- **Store setup is simple** - one function call
- **Thunk included** by default
- **DevTools enabled** automatically

---

## Core APIs

### 1. configureStore

**Simplifies store setup:**

```jsx
import { configureStore } from '@reduxjs/toolkit';

// ❌ Traditional Redux
import { createStore, applyMiddleware, combineReducers } from 'redux';
import thunk from 'redux-thunk';
import { composeWithDevTools } from 'redux-devtools-extension';

const rootReducer = combineReducers({ counter: counterReducer });
const store = createStore(
  rootReducer,
  composeWithDevTools(applyMiddleware(thunk))
);

// ✅ Redux Toolkit
const store = configureStore({
  reducer: {
    counter: counterReducer
  }
});

// Automatically includes:
// - Redux Thunk middleware
// - Redux DevTools Extension
// - combineReducers
// - Development checks (immutability, serializability)
```

**With middleware:**

```jsx
import { configureStore } from '@reduxjs/toolkit';
import logger from 'redux-logger';

const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(logger),
  devTools: process.env.NODE_ENV !== 'production'
});
```

---

### 2. createSlice

**Combines reducers and actions:**

```jsx
import { createSlice } from '@reduxjs/toolkit';

const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      // ✅ Can push directly - Immer handles immutability
      state.push({
        id: Date.now(),
        text: action.payload,
        completed: false
      });
    },
    toggleTodo: (state, action) => {
      const todo = state.find(t => t.id === action.payload);
      if (todo) {
        // ✅ Can mutate directly
        todo.completed = !todo.completed;
      }
    },
    deleteTodo: (state, action) => {
      // ✅ Can use filter (returns new array)
      return state.filter(t => t.id !== action.payload);
    }
  }
});

// Actions auto-generated
export const { addTodo, toggleTodo, deleteTodo } = todosSlice.actions;

// Reducer auto-generated
export default todosSlice.reducer;
```

**With prepare callbacks:**

```jsx
const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: {
      reducer: (state, action) => {
        state.push(action.payload);
      },
      prepare: (text) => {
        // Prepare payload before reducer
        return {
          payload: {
            id: Date.now(),
            text,
            completed: false
          }
        };
      }
    }
  }
});

// Usage
dispatch(addTodo('Learn Redux Toolkit'));  // Only pass text
```

---

### 3. createAsyncThunk

**Handles async operations:**

```jsx
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Create async thunk
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async () => {
    const response = await fetch('/api/users');
    return response.json();
  }
);

// Create slice
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    users: [],
    loading: 'idle',  // 'idle' | 'pending' | 'succeeded' | 'failed'
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = 'pending';
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = 'succeeded';
        state.users = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = 'failed';
        state.error = action.error.message;
      });
  }
});

// Usage in component
function UserList() {
  const { users, loading, error } = useSelector(state => state.users);
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch(fetchUsers());
  }, []);
  
  if (loading === 'pending') return <div>Loading...</div>;
  if (loading === 'failed') return <div>Error: {error}</div>;
  
  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

**With parameters:**

```jsx
export const fetchUserById = createAsyncThunk(
  'users/fetchUserById',
  async (userId, { rejectWithValue }) => {
    try {
      const response = await fetch(`/api/users/${userId}`);
      if (!response.ok) {
        return rejectWithValue('User not found');
      }
      return response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

// Usage
dispatch(fetchUserById(123));
```

---

### 4. createEntityAdapter

**Manages normalized state:**

```jsx
import { createSlice, createEntityAdapter } from '@reduxjs/toolkit';

// Create adapter
const usersAdapter = createEntityAdapter({
  selectId: (user) => user.id,
  sortComparer: (a, b) => a.name.localeCompare(b.name)
});

// Initial state from adapter
const initialState = usersAdapter.getInitialState({
  loading: 'idle',
  error: null
});

// State structure:
// {
//   ids: [1, 2, 3],
//   entities: {
//     1: { id: 1, name: 'John' },
//     2: { id: 2, name: 'Jane' },
//     3: { id: 3, name: 'Bob' }
//   },
//   loading: 'idle',
//   error: null
// }

const usersSlice = createSlice({
  name: 'users',
  initialState,
  reducers: {
    // Adapter provides CRUD methods
    addUser: usersAdapter.addOne,
    addUsers: usersAdapter.addMany,
    updateUser: usersAdapter.updateOne,
    removeUser: usersAdapter.removeOne
  },
  extraReducers: (builder) => {
    builder.addCase(fetchUsers.fulfilled, (state, action) => {
      usersAdapter.setAll(state, action.payload);
    });
  }
});

// Selectors
export const {
  selectAll: selectAllUsers,
  selectById: selectUserById,
  selectIds: selectUserIds
} = usersAdapter.getSelectors((state) => state.users);

// Usage
function Component() {
  const allUsers = useSelector(selectAllUsers);
  const user = useSelector(state => selectUserById(state, 123));
  const userIds = useSelector(selectUserIds);
  
  return <div>...</div>;
}
```

---

## Production Example: Todo App

```jsx
import { configureStore, createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { Provider, useSelector, useDispatch } from 'react-redux';

// Async thunk for fetching todos
export const fetchTodos = createAsyncThunk(
  'todos/fetchTodos',
  async () => {
    const response = await fetch('/api/todos');
    return response.json();
  }
);

export const addTodoAsync = createAsyncThunk(
  'todos/addTodo',
  async (text) => {
    const response = await fetch('/api/todos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text })
    });
    return response.json();
  }
);

// Slice
const todosSlice = createSlice({
  name: 'todos',
  initialState: {
    items: [],
    status: 'idle',
    error: null,
    filter: 'all'
  },
  reducers: {
    toggleTodo: (state, action) => {
      const todo = state.items.find(t => t.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
    deleteTodo: (state, action) => {
      state.items = state.items.filter(t => t.id !== action.payload);
    },
    setFilter: (state, action) => {
      state.filter = action.payload;
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      })
      .addCase(addTodoAsync.fulfilled, (state, action) => {
        state.items.push(action.payload);
      });
  }
});

export const { toggleTodo, deleteTodo, setFilter } = todosSlice.actions;

// Selectors
export const selectFilteredTodos = (state) => {
  const { items, filter } = state.todos;
  switch (filter) {
    case 'active':
      return items.filter(t => !t.completed);
    case 'completed':
      return items.filter(t => t.completed);
    default:
      return items;
  }
};

// Store
const store = configureStore({
  reducer: {
    todos: todosSlice.reducer
  }
});

// Components
function App() {
  return (
    <Provider store={store}>
      <TodoApp />
    </Provider>
  );
}

function TodoApp() {
  const dispatch = useDispatch();
  const { status, error } = useSelector(state => state.todos);
  
  useEffect(() => {
    dispatch(fetchTodos());
  }, []);
  
  if (status === 'loading') return <div>Loading...</div>;
  if (status === 'failed') return <div>Error: {error}</div>;
  
  return (
    <div>
      <AddTodo />
      <FilterButtons />
      <TodoList />
    </div>
  );
}

function AddTodo() {
  const [text, setText] = useState('');
  const dispatch = useDispatch();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      dispatch(addTodoAsync(text));
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">Add</button>
    </form>
  );
}

function FilterButtons() {
  const filter = useSelector(state => state.todos.filter);
  const dispatch = useDispatch();
  
  return (
    <div>
      <button onClick={() => dispatch(setFilter('all'))}>All</button>
      <button onClick={() => dispatch(setFilter('active'))}>Active</button>
      <button onClick={() => dispatch(setFilter('completed'))}>Completed</button>
    </div>
  );
}

function TodoList() {
  const todos = useSelector(selectFilteredTodos);
  const dispatch = useDispatch();
  
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => dispatch(toggleTodo(todo.id))}
          />
          <span>{todo.text}</span>
          <button onClick={() => dispatch(deleteTodo(todo.id))}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## Advantages of Redux Toolkit

**1. Less Boilerplate**
- **80% less code** compared to traditional Redux
- No action type constants
- No action creators
- Auto-generated actions from reducers

**2. Simpler Immutable Updates**
```jsx
// ❌ Traditional Redux - verbose
return {
  ...state,
  todos: state.todos.map(todo =>
    todo.id === action.id
      ? { ...todo, completed: !todo.completed }
      : todo
  )
};

// ✅ Redux Toolkit - simple with Immer
const todo = state.todos.find(t => t.id === action.id);
if (todo) {
  todo.completed = !todo.completed;
}
```

**3. Built-in Async Handling**
- `createAsyncThunk` for async operations
- Automatic pending/fulfilled/rejected actions
- No need to install redux-thunk separately

**4. Better Store Setup**
- One function: `configureStore`
- Auto-includes middleware and DevTools
- Better defaults
- Development checks

**5. Entity Management**
- `createEntityAdapter` for normalized state
- Built-in CRUD operations
- Auto-generated selectors

**6. Better Developer Experience**
- TypeScript support out of the box
- Better error messages
- Integrated DevTools
- Modern JavaScript features

**7. Best Practices Built-in**
- Prevents common mistakes
- Encourages good patterns
- Official Redux recommendation

---

## Migration from Redux

**Gradual migration is possible:**

```jsx
import { configureStore } from '@reduxjs/toolkit';
import oldReducer from './oldReducer';  // Traditional Redux
import newSlice from './newSlice';      // Redux Toolkit

const store = configureStore({
  reducer: {
    old: oldReducer,     // Keep old code
    new: newSlice.reducer  // Add new RTK slices
  }
});

// Gradually convert old reducers to RTK slices
```

---

## Best Practices

**1. Use createSlice for all state**
```jsx
// ✅ Good
const userSlice = createSlice({ /*...*/ });
```

**2. Use createAsyncThunk for async**
```jsx
// ✅ Good
const fetchUser = createAsyncThunk('user/fetch', async (id) => {
  return await api.getUser(id);
});
```

**3. Use createEntityAdapter for collections**
```jsx
// ✅ Good for lists of items
const usersAdapter = createEntityAdapter();
```

**4. Co-locate selectors with slices**
```jsx
export const selectFilteredTodos = (state) => {
  // Selector logic
};
```

**5. Use RTK Query for data fetching (advanced)**
```jsx
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

const api = createApi({
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  endpoints: (builder) => ({
    getUsers: builder.query({ query: () => 'users' })
  })
});
```

---

**Interview Tips:**
- Redux Toolkit is the **official recommended** way to write Redux
- Reduces boilerplate by **~80%** compared to traditional Redux
- **createSlice** combines actions and reducers in one place
- **configureStore** simplifies store setup with good defaults
- **createAsyncThunk** handles async operations automatically
- Uses **Immer** internally - can "mutate" state safely
- **Auto-generates** action creators and action types
- Includes **Redux Thunk** middleware by default
- **DevTools** enabled automatically in development
- **createEntityAdapter** manages normalized state
- Better **TypeScript** support out of the box
- Encourages **best practices** and prevents common mistakes
- **Backwards compatible** - can mix with traditional Redux
- **RTK Query** for advanced data fetching/caching
- Main APIs: configureStore, createSlice, createAsyncThunk, createEntityAdapter
- **Immer** allows direct mutations in reducers (converted to immutable updates)
- **extraReducers** for handling external actions (async thunks)
- **prepare callbacks** for customizing action payloads
- Significantly **improves developer experience**
- Now the **standard** for Redux development

</details>

---

### 46. What is Zustand and how does it compare to Redux?

<details>
<summary>View Answer</summary>

**Zustand**

Zustand is a **small, fast, and scalable** state management solution for React. It has a minimal API, requires no boilerplate, and works without providers or context.

**Installation:**

```bash
npm install zustand
```

**Tagline:** *"A small, fast and scalable bearbones state-management solution."*

---

## Basic Example

**Zustand - Simple and Clean:**

```jsx
import { create } from 'zustand';

// 1. Create store
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 })
}));

// 2. Use in component - No Provider needed!
function Counter() {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);
  const decrement = useStore((state) => state.decrement);
  
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}

// That's it! No Provider wrapper needed
function App() {
  return <Counter />;
}
```

**Redux Toolkit - More Setup:**

```jsx
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { Provider, useSelector, useDispatch } from 'react-redux';

// 1. Create slice
const counterSlice = createSlice({
  name: 'counter',
  initialState: { count: 0 },
  reducers: {
    increment: (state) => { state.count += 1 },
    decrement: (state) => { state.count -= 1 },
    reset: (state) => { state.count = 0 }
  }
});

const { increment, decrement, reset } = counterSlice.actions;

// 2. Create store
const store = configureStore({
  reducer: { counter: counterSlice.reducer }
});

// 3. Use in component
function Counter() {
  const count = useSelector((state) => state.counter.count);
  const dispatch = useDispatch();
  
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
    </div>
  );
}

// 4. Need Provider wrapper
function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}
```

---

## Key Differences

| Feature | Zustand | Redux Toolkit |
|---------|---------|---------------|
| **Size** | ~1.2KB | ~11KB |
| **Setup** | One function | Slice + Store + Provider |
| **Provider** | Not needed | Required |
| **Boilerplate** | Minimal | More (but less than Redux) |
| **Selectors** | Built-in | useSelector |
| **DevTools** | Optional plugin | Built-in |
| **Middleware** | Simple | Robust ecosystem |
| **Async** | Direct in store | createAsyncThunk |
| **Learning Curve** | Very easy | Moderate |
| **TypeScript** | Great | Excellent |
| **Flux Pattern** | No | Yes |
| **Best For** | Small to medium apps | Large apps |

---

## Zustand Features

### 1. No Provider Needed

```jsx
import { create } from 'zustand';

const useStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user })
}));

// Use anywhere - no Provider wrapper!
function App() {
  return (
    <div>
      <Header />  {/* Can use store */}
      <Main />    {/* Can use store */}
      <Footer />  {/* Can use store */}
    </div>
  );
}

function Header() {
  const user = useStore((state) => state.user);
  return <div>Welcome, {user?.name}</div>;
}
```

### 2. Direct State Mutations

```jsx
const useStore = create((set) => ({
  todos: [],
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, { id: Date.now(), text, completed: false }]
  })),
  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  })),
  deleteTodo: (id) => set((state) => ({
    todos: state.todos.filter(todo => todo.id !== id)
  }))
}));
```

### 3. Selective Subscriptions

```jsx
const useStore = create((set) => ({
  user: { name: 'John', age: 30 },
  theme: 'light',
  setUser: (user) => set({ user }),
  setTheme: (theme) => set({ theme })
}));

// Only subscribes to user - won't re-render when theme changes
function UserDisplay() {
  const user = useStore((state) => state.user);
  console.log('UserDisplay rendered');
  return <div>{user.name}</div>;
}

// Only subscribes to theme - won't re-render when user changes
function ThemeDisplay() {
  const theme = useStore((state) => state.theme);
  console.log('ThemeDisplay rendered');
  return <div>{theme}</div>;
}

// When theme changes:
// - ThemeDisplay re-renders ✅
// - UserDisplay doesn't re-render ✅
```

### 4. Multiple Stores

```jsx
// Separate stores for different concerns
const useUserStore = create((set) => ({
  user: null,
  login: (user) => set({ user }),
  logout: () => set({ user: null })
}));

const useCartStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  clearCart: () => set({ items: [] })
}));

const useThemeStore = create((set) => ({
  theme: 'light',
  setTheme: (theme) => set({ theme })
}));

// Use in components
function Component() {
  const user = useUserStore((state) => state.user);
  const items = useCartStore((state) => state.items);
  const theme = useThemeStore((state) => state.theme);
  
  return <div>...</div>;
}
```

---

## Production Example: Todo App (Zustand)

```jsx
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

// Create store with middleware
const useTodoStore = create(
  devtools(
    persist(
      (set, get) => ({
        // State
        todos: [],
        filter: 'all',
        
        // Actions
        addTodo: (text) => set((state) => ({
          todos: [...state.todos, {
            id: Date.now(),
            text,
            completed: false
          }]
        })),
        
        toggleTodo: (id) => set((state) => ({
          todos: state.todos.map(todo =>
            todo.id === id
              ? { ...todo, completed: !todo.completed }
              : todo
          )
        })),
        
        deleteTodo: (id) => set((state) => ({
          todos: state.todos.filter(todo => todo.id !== id)
        })),
        
        setFilter: (filter) => set({ filter }),
        
        // Computed/Selectors
        getFilteredTodos: () => {
          const { todos, filter } = get();
          switch (filter) {
            case 'active':
              return todos.filter(t => !t.completed);
            case 'completed':
              return todos.filter(t => t.completed);
            default:
              return todos;
          }
        }
      }),
      { name: 'todo-storage' }  // Persist to localStorage
    )
  )
);

// Components
function TodoApp() {
  return (
    <div>
      <AddTodo />
      <FilterButtons />
      <TodoList />
    </div>
  );
}

function AddTodo() {
  const [text, setText] = useState('');
  const addTodo = useTodoStore((state) => state.addTodo);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      addTodo(text);
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">Add</button>
    </form>
  );
}

function FilterButtons() {
  const filter = useTodoStore((state) => state.filter);
  const setFilter = useTodoStore((state) => state.setFilter);
  
  return (
    <div>
      <button onClick={() => setFilter('all')}>All</button>
      <button onClick={() => setFilter('active')}>Active</button>
      <button onClick={() => setFilter('completed')}>Completed</button>
    </div>
  );
}

function TodoList() {
  const getFilteredTodos = useTodoStore((state) => state.getFilteredTodos);
  const toggleTodo = useTodoStore((state) => state.toggleTodo);
  const deleteTodo = useTodoStore((state) => state.deleteTodo);
  const todos = getFilteredTodos();
  
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          <span>{todo.text}</span>
          <button onClick={() => deleteTodo(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## Async Operations

**Zustand - Direct in actions:**

```jsx
import { create } from 'zustand';

const useUserStore = create((set) => ({
  user: null,
  loading: false,
  error: null,
  
  // Async action - just a regular async function
  fetchUser: async (id) => {
    set({ loading: true, error: null });
    
    try {
      const response = await fetch(`/api/users/${id}`);
      const data = await response.json();
      set({ user: data, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  }
}));

// Usage
function UserProfile({ userId }) {
  const { user, loading, error, fetchUser } = useUserStore();
  
  useEffect(() => {
    fetchUser(userId);
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{user?.name}</div>;
}
```

**Redux Toolkit - createAsyncThunk:**

```jsx
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

const fetchUser = createAsyncThunk(
  'user/fetchUser',
  async (id) => {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: { user: null, loading: false, error: null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});
```

---

## Middleware

**Zustand - Simple middleware:**

```jsx
import { create } from 'zustand';
import { devtools, persist, subscribeWithSelector } from 'zustand/middleware';

const useStore = create(
  subscribeWithSelector(
    devtools(
      persist(
        (set) => ({
          count: 0,
          increment: () => set((state) => ({ count: state.count + 1 }))
        }),
        { name: 'my-store' }
      )
    )
  )
);

// Built-in middleware:
// - devtools: Redux DevTools integration
// - persist: LocalStorage persistence
// - subscribeWithSelector: Fine-grained subscriptions
// - combine: Combine multiple stores
// - immer: Immer integration for mutations
```

**Redux Toolkit - Middleware ecosystem:**

```jsx
import { configureStore } from '@reduxjs/toolkit';
import logger from 'redux-logger';

const store = configureStore({
  reducer: { /*...*/ },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware()
      .concat(logger)
      .concat(customMiddleware)
});

// Large middleware ecosystem:
// - redux-thunk (included)
// - redux-saga
// - redux-observable
// - redux-logger
// - Many more...
```

---

## When to Use Each

**Use Zustand when:**

✅ **Small to medium apps** - Less than 50k LOC
✅ **Want minimal setup** - No boilerplate
✅ **No Provider needed** - Simpler component tree
✅ **Fast prototyping** - Quick to set up
✅ **Modern React** - Hooks-based
✅ **Multiple stores** - Split by feature
✅ **Simple state logic** - Direct updates
✅ **Performance** - Selective subscriptions

**Use Redux Toolkit when:**

✅ **Large enterprise apps** - 50k+ LOC
✅ **Complex state logic** - Many interdependencies
✅ **Strict patterns** - Team needs structure
✅ **Time-travel debugging** - Redux DevTools
✅ **Middleware ecosystem** - saga, observable, etc.
✅ **Established codebase** - Already using Redux
✅ **Team experience** - Team knows Redux
✅ **Predictable updates** - Strict unidirectional flow

---

## Performance Comparison

**Zustand:**
- Smaller bundle size (~1.2KB)
- Selective subscriptions by default
- Fast updates
- No Provider re-renders

**Redux Toolkit:**
- Larger bundle (~11KB)
- Requires selector optimization
- Mature performance optimizations
- Provider can cause re-renders

---

## Migration

**From Redux to Zustand:**

```jsx
// Redux
const counterSlice = createSlice({
  name: 'counter',
  initialState: { count: 0 },
  reducers: {
    increment: (state) => { state.count += 1 }
  }
});

// Zustand equivalent
const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 }))
}));
```

---

## Best Practices

**Zustand:**

```jsx
// 1. Split by domain
const useUserStore = create(/*...*/);
const useCartStore = create(/*...*/);

// 2. Use middleware
const useStore = create(
  devtools(
    persist(/*...*/, { name: 'store' })
  )
);

// 3. Selective subscriptions
const user = useStore((state) => state.user);  // Only user

// 4. Actions in store
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 }))
}));
```

---

**Interview Tips:**
- Zustand is a **lightweight** state management library (~1.2KB)
- **No Provider** needed - simpler than Redux
- Uses **hooks** directly - `useStore(selector)`
- **Minimal boilerplate** compared to Redux
- **Selective subscriptions** - only re-render when selected state changes
- Can have **multiple stores** - split by feature
- **Async** operations are just regular async functions
- Supports **middleware** - devtools, persist, etc.
- **TypeScript** support is excellent
- **No actions/reducers** - direct state updates
- Redux Toolkit is **larger** (~11KB) but more structured
- Redux has **strict patterns** (Flux), Zustand is flexible
- Redux has **robust middleware ecosystem**, Zustand is simpler
- Redux has **built-in DevTools**, Zustand needs plugin
- Zustand is **easier to learn** - less concepts
- Redux better for **large teams** needing structure
- Zustand better for **small to medium apps** needing speed
- Both support **async**, **TypeScript**, **DevTools**
- Zustand has **better performance** with selective subscriptions
- Can **use both** in same app for different purposes

</details>

---

### 47. What is Jotai and when would you use it?

<details>
<summary>View Answer</summary>

**Jotai**

Jotai is a **primitive and flexible** state management library for React. It uses an **atomic model** where state is split into small, independent pieces called "atoms" that can be composed together.

**Installation:**

```bash
npm install jotai
```

**Tagline:** *"Primitive and flexible state management for React."*

---

## Core Concept: Atoms

**Atom = piece of state**

```jsx
import { atom, useAtom } from 'jotai';

// 1. Create atom (outside component)
const countAtom = atom(0);

// 2. Use in component
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// No Provider needed at basic level!
function App() {
  return <Counter />;
}
```

---

## Why Jotai?

**Comparison to other solutions:**

```jsx
// Redux/Zustand - Global store with all state
const store = {
  user: { name: 'John' },
  theme: 'light',
  count: 0,
  todos: []
};

// Jotai - Individual atoms
const userAtom = atom({ name: 'John' });
const themeAtom = atom('light');
const countAtom = atom(0);
const todosAtom = atom([]);

// Components only subscribe to atoms they use
// Changing userAtom doesn't affect components using themeAtom
```

---

## Basic Usage

**1. Primitive Atoms**

```jsx
import { atom, useAtom } from 'jotai';

// Create atoms
const nameAtom = atom('John');
const ageAtom = atom(30);
const themeAtom = atom('light');

// Use in components
function UserProfile() {
  const [name, setName] = useAtom(nameAtom);
  const [age, setAge] = useAtom(ageAtom);
  
  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <input value={age} onChange={(e) => setAge(Number(e.target.value))} />
    </div>
  );
}

function ThemeToggle() {
  const [theme, setTheme] = useAtom(themeAtom);
  
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      {theme}
    </button>
  );
}
```

**2. Read-Only Access**

```jsx
import { useAtomValue } from 'jotai';

// Only read, no setter
function Display() {
  const count = useAtomValue(countAtom);  // Read-only
  console.log('Display rendered');
  return <div>Count: {count}</div>;
}

// Won't re-render when other atoms change
```

**3. Write-Only Access**

```jsx
import { useSetAtom } from 'jotai';

// Only setter, no value
function Controls() {
  const setCount = useSetAtom(countAtom);  // Write-only
  console.log('Controls rendered');
  
  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>+</button>
      <button onClick={() => setCount((c) => c - 1)}>-</button>
    </div>
  );
}

// Won't re-render when count changes (no read access)
```

---

## Derived Atoms

**Compute values from other atoms:**

```jsx
import { atom } from 'jotai';

// Base atoms
const firstNameAtom = atom('John');
const lastNameAtom = atom('Doe');

// Derived atom (read-only)
const fullNameAtom = atom((get) => {
  return `${get(firstNameAtom)} ${get(lastNameAtom)}`;
});

// Usage
function FullName() {
  const fullName = useAtomValue(fullNameAtom);
  return <div>{fullName}</div>;  // "John Doe"
}

// Changing firstNameAtom or lastNameAtom automatically updates fullNameAtom
```

**Writable derived atoms:**

```jsx
// Base atom
const tempCelsiusAtom = atom(0);

// Derived atom (read + write)
const tempFahrenheitAtom = atom(
  (get) => get(tempCelsiusAtom) * 9/5 + 32,  // Read
  (get, set, newValue) => {                   // Write
    set(tempCelsiusAtom, (newValue - 32) * 5/9);
  }
);

// Usage
function Temperature() {
  const [celsius, setCelsius] = useAtom(tempCelsiusAtom);
  const [fahrenheit, setFahrenheit] = useAtom(tempFahrenheitAtom);
  
  return (
    <div>
      <input value={celsius} onChange={(e) => setCelsius(Number(e.target.value))} />
      <input value={fahrenheit} onChange={(e) => setFahrenheit(Number(e.target.value))} />
    </div>
  );
}

// Updating either input updates both (they're synced)
```

---

## Production Example: Todo App

```jsx
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';
import { atomWithStorage } from 'jotai/utils';

// Atoms
const todosAtom = atomWithStorage('todos', []);  // Persists to localStorage
const filterAtom = atom('all');

// Derived atoms
const filteredTodosAtom = atom((get) => {
  const todos = get(todosAtom);
  const filter = get(filterAtom);
  
  switch (filter) {
    case 'active':
      return todos.filter(t => !t.completed);
    case 'completed':
      return todos.filter(t => t.completed);
    default:
      return todos;
  }
});

const todoStatsAtom = atom((get) => {
  const todos = get(todosAtom);
  return {
    total: todos.length,
    completed: todos.filter(t => t.completed).length,
    active: todos.filter(t => !t.completed).length
  };
});

// Components
function TodoApp() {
  return (
    <div>
      <AddTodo />
      <FilterButtons />
      <TodoList />
      <TodoStats />
    </div>
  );
}

function AddTodo() {
  const [text, setText] = useState('');
  const setTodos = useSetAtom(todosAtom);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      setTodos((prev) => [...prev, {
        id: Date.now(),
        text,
        completed: false
      }]);
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">Add</button>
    </form>
  );
}

function FilterButtons() {
  const [filter, setFilter] = useAtom(filterAtom);
  
  return (
    <div>
      <button onClick={() => setFilter('all')}>All</button>
      <button onClick={() => setFilter('active')}>Active</button>
      <button onClick={() => setFilter('completed')}>Completed</button>
    </div>
  );
}

function TodoList() {
  const filteredTodos = useAtomValue(filteredTodosAtom);
  
  return (
    <ul>
      {filteredTodos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}

function TodoItem({ todo }) {
  const setTodos = useSetAtom(todosAtom);
  
  const toggle = () => {
    setTodos((todos) =>
      todos.map(t => t.id === todo.id ? { ...t, completed: !t.completed } : t)
    );
  };
  
  const remove = () => {
    setTodos((todos) => todos.filter(t => t.id !== todo.id));
  };
  
  return (
    <li>
      <input type="checkbox" checked={todo.completed} onChange={toggle} />
      <span>{todo.text}</span>
      <button onClick={remove}>Delete</button>
    </li>
  );
}

function TodoStats() {
  const stats = useAtomValue(todoStatsAtom);
  
  return (
    <div>
      <p>Total: {stats.total}</p>
      <p>Completed: {stats.completed}</p>
      <p>Active: {stats.active}</p>
    </div>
  );
}
```

---

## Async Atoms

**Handle async operations:**

```jsx
import { atom, useAtom } from 'jotai';
import { Suspense } from 'react';

// Async atom
const userAtom = atom(async (get) => {
  const response = await fetch('/api/user');
  return response.json();
});

// Usage with Suspense
function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <UserProfile />
    </Suspense>
  );
}

function UserProfile() {
  const [user] = useAtom(userAtom);  // Suspends until loaded
  return <div>{user.name}</div>;
}
```

**Async with parameters:**

```jsx
import { atomFamily } from 'jotai/utils';

// Atom family - creates atoms with parameters
const userAtomFamily = atomFamily((userId) =>
  atom(async () => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  })
);

// Usage
function User({ userId }) {
  const [user] = useAtom(userAtomFamily(userId));
  return <div>{user.name}</div>;
}

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <User userId={1} />
      <User userId={2} />
      <User userId={3} />
    </Suspense>
  );
}
```

---

## Utilities

**1. atomWithStorage - LocalStorage persistence**

```jsx
import { atomWithStorage } from 'jotai/utils';

const themeAtom = atomWithStorage('theme', 'light');

// Automatically syncs with localStorage
function ThemeToggle() {
  const [theme, setTheme] = useAtom(themeAtom);
  return <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>{theme}</button>;
}
```

**2. atomWithReset - Resettable atoms**

```jsx
import { atomWithReset, useResetAtom } from 'jotai/utils';

const countAtom = atomWithReset(0);

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const resetCount = useResetAtom(countAtom);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <button onClick={resetCount}>Reset</button>
    </div>
  );
}
```

**3. atomWithReducer - Reducer pattern**

```jsx
import { atomWithReducer } from 'jotai/utils';

const countReducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
};

const countAtom = atomWithReducer(0, countReducer);

function Counter() {
  const [count, dispatch] = useAtom(countAtom);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
    </div>
  );
}
```

**4. atomFamily - Dynamic atoms**

```jsx
import { atomFamily } from 'jotai/utils';

const todoAtomFamily = atomFamily((id) =>
  atom({
    id,
    text: '',
    completed: false
  })
);

// Each ID gets its own atom
const todo1 = todoAtomFamily(1);
const todo2 = todoAtomFamily(2);
const todo3 = todoAtomFamily(3);
```

---

## Provider (Optional)

**Scope atoms to subtree:**

```jsx
import { Provider } from 'jotai';

const countAtom = atom(0);

function App() {
  return (
    <div>
      {/* Default scope */}
      <Counter />  {/* count starts at 0 */}
      
      {/* Separate scope */}
      <Provider>
        <Counter />  {/* count starts at 0, independent */}
      </Provider>
      
      {/* Another scope */}
      <Provider>
        <Counter />  {/* count starts at 0, independent */}
      </Provider>
    </div>
  );
}

// Each Provider creates isolated state
```

---

## When to Use Jotai

**✅ Use Jotai when:**

1. **Bottom-up state management** - Start small, grow naturally
2. **Fine-grained reactivity** - Only re-render what changes
3. **Derived state** - Compute values from other atoms
4. **TypeScript** - Excellent type inference
5. **React Suspense** - Built-in async support
6. **Small apps** - Minimal setup
7. **Colocation** - Define atoms near components
8. **Flexible structure** - No enforced patterns

**Examples:**

```jsx
// ✅ Good: Fine-grained state
const userNameAtom = atom('');
const userAgeAtom = atom(0);
const userEmailAtom = atom('');

// ✅ Good: Derived state
const userDataAtom = atom((get) => ({
  name: get(userNameAtom),
  age: get(userAgeAtom),
  email: get(userEmailAtom)
}));

// ✅ Good: Async with Suspense
const todosAtom = atom(async () => {
  const res = await fetch('/api/todos');
  return res.json();
});
```

**❌ Avoid Jotai when:**

1. **Need strict patterns** - Use Redux
2. **Large team** - May prefer Redux structure
3. **Complex middleware** - Redux has more options
4. **Time-travel debugging** - Redux DevTools better
5. **Legacy codebase** - Already using Redux

---

## Jotai vs Others

**vs Redux:**
- Jotai: Bottom-up, atomic, minimal
- Redux: Top-down, single store, structured

**vs Zustand:**
- Jotai: Atoms, React-only, Suspense
- Zustand: Store-based, framework-agnostic, simpler async

**vs Recoil:**
- Jotai: Simpler API, smaller bundle
- Recoil: More features, Facebook-backed

**vs Context:**
- Jotai: Optimized, no Provider bloat
- Context: Built-in, but re-render issues

---

## Best Practices

**1. Define atoms at module level**

```jsx
// ✅ Good: Outside component
const countAtom = atom(0);

function Counter() {
  const [count] = useAtom(countAtom);
  return <div>{count}</div>;
}

// ❌ Bad: Inside component
function Counter() {
  const countAtom = atom(0);  // New atom every render!
  const [count] = useAtom(countAtom);
  return <div>{count}</div>;
}
```

**2. Use derived atoms for computed values**

```jsx
// ✅ Good: Derived atom
const totalAtom = atom((get) => {
  const items = get(itemsAtom);
  return items.reduce((sum, item) => sum + item.price, 0);
});

// ❌ Bad: Calculate in component
function Total() {
  const items = useAtomValue(itemsAtom);
  const total = items.reduce((sum, item) => sum + item.price, 0);
  return <div>{total}</div>;
}
```

**3. Split read/write when possible**

```jsx
// ✅ Good: Only what's needed
function Display() {
  const count = useAtomValue(countAtom);  // Read-only
  return <div>{count}</div>;
}

function Controls() {
  const setCount = useSetAtom(countAtom);  // Write-only
  return <button onClick={() => setCount(c => c + 1)}>+</button>;
}
```

**4. Use atomFamily for dynamic state**

```jsx
// ✅ Good: Atom family for list items
const todoAtomFamily = atomFamily((id) => atom({ id, text: '', done: false }));
```

---

**Interview Tips:**
- Jotai is an **atomic state management** library for React
- State is split into small pieces called **atoms**
- **atom()** creates a piece of state
- **useAtom()** is like useState but for atoms
- **useAtomValue()** for read-only access
- **useSetAtom()** for write-only access
- **Derived atoms** compute values from other atoms
- **No Provider needed** at basic level (optional for scoping)
- **Fine-grained reactivity** - only re-render what changes
- **Built-in Suspense** support for async
- **atomFamily** for dynamic atoms with parameters
- **atomWithStorage** for localStorage persistence
- **Bottom-up approach** - start small, compose atoms
- **TypeScript** has excellent type inference
- **Smaller bundle** than Recoil, similar concept
- **Minimal boilerplate** compared to Redux
- Good for **flexible, incremental** state management
- Atoms are **independent** - changing one doesn't affect others
- Can use **multiple atoms** or **combine into derived atoms**
- **React-only** - designed specifically for React

</details>

---

### 48. What is Recoil state management?

<details>
<summary>View Answer</summary>

**Recoil**

Recoil is a **state management library** for React developed by Facebook. It uses an **atomic and graph-based** approach where state is split into atoms, and components subscribe only to the atoms they use.

**Installation:**

```bash
npm install recoil
```

**Tagline:** *"A state management library for React."*

---

## Core Concepts

**1. Atoms - Units of state**
```jsx
import { atom } from 'recoil';

const countState = atom({
  key: 'countState',  // Unique ID
  default: 0          // Default value
});
```

**2. Selectors - Derived state**
```jsx
import { selector } from 'recoil';

const doubleCountState = selector({
  key: 'doubleCountState',
  get: ({ get }) => {
    const count = get(countState);
    return count * 2;
  }
});
```

**3. Hooks - Access state**
```jsx
import { useRecoilState, useRecoilValue, useSetRecoilState } from 'recoil';

const [count, setCount] = useRecoilState(countState);  // Read + Write
const count = useRecoilValue(countState);              // Read only
const setCount = useSetRecoilState(countState);        // Write only
```

---

## Basic Setup

**Wrap app with RecoilRoot:**

```jsx
import { RecoilRoot } from 'recoil';

function App() {
  return (
    <RecoilRoot>
      <Counter />
    </RecoilRoot>
  );
}
```

**Create and use atoms:**

```jsx
import { atom, useRecoilState } from 'recoil';

// 1. Define atom
const countState = atom({
  key: 'countState',
  default: 0
});

// 2. Use in component
function Counter() {
  const [count, setCount] = useRecoilState(countState);
  
  return (
    <div>
      <h1>{count}</h1>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
    </div>
  );
}
```

---

## Atoms

**Simple atoms:**

```jsx
import { atom } from 'recoil';

// Primitive values
const nameState = atom({
  key: 'nameState',
  default: 'John'
});

const ageState = atom({
  key: 'ageState',
  default: 30
});

const themeState = atom({
  key: 'themeState',
  default: 'light'
});

// Objects
const userState = atom({
  key: 'userState',
  default: {
    name: 'John',
    email: 'john@example.com',
    age: 30
  }
});

// Arrays
const todosState = atom({
  key: 'todosState',
  default: []
});
```

**Atom families - Dynamic atoms:**

```jsx
import { atomFamily } from 'recoil';

// Create atoms based on parameters
const todoItemState = atomFamily({
  key: 'todoItemState',
  default: (id) => ({
    id,
    text: '',
    completed: false
  })
});

// Usage
function TodoItem({ id }) {
  const [todo, setTodo] = useRecoilState(todoItemState(id));
  
  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={(e) => setTodo({ ...todo, completed: e.target.checked })}
      />
      <span>{todo.text}</span>
    </div>
  );
}
```

---

## Selectors

**Read-only selectors:**

```jsx
import { selector } from 'recoil';

const firstNameState = atom({ key: 'firstName', default: 'John' });
const lastNameState = atom({ key: 'lastName', default: 'Doe' });

// Derived state
const fullNameState = selector({
  key: 'fullNameState',
  get: ({ get }) => {
    const firstName = get(firstNameState);
    const lastName = get(lastNameState);
    return `${firstName} ${lastName}`;
  }
});

// Usage
function FullName() {
  const fullName = useRecoilValue(fullNameState);
  return <div>{fullName}</div>;  // "John Doe"
}
```

**Writable selectors:**

```jsx
const tempCelsiusState = atom({
  key: 'tempCelsius',
  default: 0
});

const tempFahrenheitState = selector({
  key: 'tempFahrenheit',
  get: ({ get }) => {
    const celsius = get(tempCelsiusState);
    return (celsius * 9) / 5 + 32;
  },
  set: ({ set }, newValue) => {
    const celsius = ((newValue - 32) * 5) / 9;
    set(tempCelsiusState, celsius);
  }
});

// Usage
function Temperature() {
  const [celsius, setCelsius] = useRecoilState(tempCelsiusState);
  const [fahrenheit, setFahrenheit] = useRecoilState(tempFahrenheitState);
  
  return (
    <div>
      <input value={celsius} onChange={(e) => setCelsius(Number(e.target.value))} />
      <input value={fahrenheit} onChange={(e) => setFahrenheit(Number(e.target.value))} />
    </div>
  );
}
```

**Selector families:**

```jsx
import { selectorFamily } from 'recoil';

const userNameQuery = selectorFamily({
  key: 'userNameQuery',
  get: (userID) => async ({ get }) => {
    const response = await fetch(`/api/users/${userID}`);
    const data = await response.json();
    return data.name;
  }
});

// Usage
function UserName({ userID }) {
  const userName = useRecoilValue(userNameQuery(userID));
  return <div>{userName}</div>;
}
```

---

## Async Queries

**Async selectors with Suspense:**

```jsx
import { selector } from 'recoil';
import { Suspense } from 'react';

const currentUserQuery = selector({
  key: 'currentUserQuery',
  get: async ({ get }) => {
    const response = await fetch('/api/user');
    return response.json();
  }
});

// Must use Suspense
function App() {
  return (
    <RecoilRoot>
      <Suspense fallback={<div>Loading...</div>}>
        <UserProfile />
      </Suspense>
    </RecoilRoot>
  );
}

function UserProfile() {
  const user = useRecoilValue(currentUserQuery);  // Suspends
  return <div>{user.name}</div>;
}
```

**With error boundary:**

```jsx
import { ErrorBoundary } from 'react-error-boundary';

function App() {
  return (
    <RecoilRoot>
      <ErrorBoundary fallback={<div>Error loading user</div>}>
        <Suspense fallback={<div>Loading...</div>}>
          <UserProfile />
        </Suspense>
      </ErrorBoundary>
    </RecoilRoot>
  );
}
```

---

## Production Example: Todo App

```jsx
import { atom, selector, useRecoilState, useRecoilValue, useSetRecoilState } from 'recoil';
import { RecoilRoot } from 'recoil';

// Atoms
const todoListState = atom({
  key: 'todoListState',
  default: []
});

const todoListFilterState = atom({
  key: 'todoListFilterState',
  default: 'all'
});

// Selectors
const filteredTodoListState = selector({
  key: 'filteredTodoListState',
  get: ({ get }) => {
    const filter = get(todoListFilterState);
    const list = get(todoListState);
    
    switch (filter) {
      case 'completed':
        return list.filter((item) => item.completed);
      case 'active':
        return list.filter((item) => !item.completed);
      default:
        return list;
    }
  }
});

const todoListStatsState = selector({
  key: 'todoListStatsState',
  get: ({ get }) => {
    const todoList = get(todoListState);
    const totalNum = todoList.length;
    const completedNum = todoList.filter((item) => item.completed).length;
    const activeNum = totalNum - completedNum;
    const percentCompleted = totalNum === 0 ? 0 : (completedNum / totalNum) * 100;
    
    return {
      totalNum,
      completedNum,
      activeNum,
      percentCompleted
    };
  }
});

// Components
function TodoApp() {
  return (
    <RecoilRoot>
      <div>
        <TodoItemCreator />
        <TodoListFilters />
        <TodoListStats />
        <TodoList />
      </div>
    </RecoilRoot>
  );
}

function TodoItemCreator() {
  const [inputValue, setInputValue] = useState('');
  const setTodoList = useSetRecoilState(todoListState);
  
  const addItem = () => {
    if (inputValue.trim()) {
      setTodoList((oldTodoList) => [
        ...oldTodoList,
        {
          id: Date.now(),
          text: inputValue,
          completed: false
        }
      ]);
      setInputValue('');
    }
  };
  
  return (
    <div>
      <input value={inputValue} onChange={(e) => setInputValue(e.target.value)} />
      <button onClick={addItem}>Add</button>
    </div>
  );
}

function TodoListFilters() {
  const [filter, setFilter] = useRecoilState(todoListFilterState);
  
  return (
    <div>
      <button onClick={() => setFilter('all')}>All</button>
      <button onClick={() => setFilter('active')}>Active</button>
      <button onClick={() => setFilter('completed')}>Completed</button>
    </div>
  );
}

function TodoListStats() {
  const { totalNum, completedNum, activeNum, percentCompleted } = useRecoilValue(todoListStatsState);
  
  return (
    <div>
      <p>Total: {totalNum}</p>
      <p>Completed: {completedNum}</p>
      <p>Active: {activeNum}</p>
      <p>Progress: {percentCompleted.toFixed(0)}%</p>
    </div>
  );
}

function TodoList() {
  const todoList = useRecoilValue(filteredTodoListState);
  
  return (
    <ul>
      {todoList.map((todoItem) => (
        <TodoItem key={todoItem.id} item={todoItem} />
      ))}
    </ul>
  );
}

function TodoItem({ item }) {
  const [todoList, setTodoList] = useRecoilState(todoListState);
  
  const toggleItemCompletion = () => {
    setTodoList((list) =>
      list.map((i) =>
        i.id === item.id ? { ...i, completed: !i.completed } : i
      )
    );
  };
  
  const deleteItem = () => {
    setTodoList((list) => list.filter((i) => i.id !== item.id));
  };
  
  return (
    <li>
      <input type="checkbox" checked={item.completed} onChange={toggleItemCompletion} />
      <span>{item.text}</span>
      <button onClick={deleteItem}>Delete</button>
    </li>
  );
}
```

---

## Advanced Features

**1. Atom effects - Side effects**

```jsx
const myState = atom({
  key: 'myState',
  default: 0,
  effects: [
    // Log state changes
    ({ onSet }) => {
      onSet((newValue, oldValue) => {
        console.log('State changed:', oldValue, '->', newValue);
      });
    },
    // Persist to localStorage
    ({ setSelf, onSet }) => {
      const savedValue = localStorage.getItem('myState');
      if (savedValue != null) {
        setSelf(JSON.parse(savedValue));
      }
      
      onSet((newValue) => {
        localStorage.setItem('myState', JSON.stringify(newValue));
      });
    }
  ]
});
```

**2. Snapshot - Read state outside components**

```jsx
import { useRecoilCallback } from 'recoil';

function Debug() {
  const logState = useRecoilCallback(({ snapshot }) => async () => {
    const count = await snapshot.getPromise(countState);
    console.log('Count:', count);
  });
  
  return <button onClick={logState}>Log State</button>;
}
```

**3. Loadable - Handle async state manually**

```jsx
import { useRecoilValueLoadable } from 'recoil';

function UserProfile() {
  const userLoadable = useRecoilValueLoadable(currentUserQuery);
  
  switch (userLoadable.state) {
    case 'loading':
      return <div>Loading...</div>;
    case 'hasValue':
      return <div>{userLoadable.contents.name}</div>;
    case 'hasError':
      return <div>Error: {userLoadable.contents.message}</div>;
  }
}
```

---

## Recoil vs Others

**vs Redux:**
- Recoil: Atomic, React-focused, less boilerplate
- Redux: Single store, more structure, larger ecosystem

**vs Zustand:**
- Recoil: Atoms, Suspense, Facebook-backed
- Zustand: Simpler, smaller bundle, no Provider

**vs Jotai:**
- Recoil: More features, atom effects
- Jotai: Simpler API, smaller bundle, similar concept

**vs Context:**
- Recoil: Fine-grained updates, async built-in
- Context: Built-in React, simpler but performance issues

---

## Best Practices

**1. Use unique keys**

```jsx
// ✅ Good: Descriptive unique keys
const userNameState = atom({ key: 'userNameState', default: '' });
const userAgeState = atom({ key: 'userAgeState', default: 0 });

// ❌ Bad: Generic keys
const state1 = atom({ key: 'state1', default: '' });
const state2 = atom({ key: 'state2', default: 0 });
```

**2. Use selectors for derived state**

```jsx
// ✅ Good: Selector
const totalPriceState = selector({
  key: 'totalPriceState',
  get: ({ get }) => {
    const items = get(cartItemsState);
    return items.reduce((sum, item) => sum + item.price, 0);
  }
});

// ❌ Bad: Calculate in component
function Total() {
  const items = useRecoilValue(cartItemsState);
  const total = items.reduce((sum, item) => sum + item.price, 0);
  return <div>{total}</div>;
}
```

**3. Use Suspense for async**

```jsx
// ✅ Good: Let Suspense handle loading
function App() {
  return (
    <Suspense fallback={<Loading />}>
      <UserProfile />
    </Suspense>
  );
}

// ❌ Bad: Manual loading state
function UserProfile() {
  const [loading, setLoading] = useState(true);
  // Manual async handling...
}
```

**4. Split read/write when possible**

```jsx
// ✅ Good: Only what's needed
function Display() {
  const count = useRecoilValue(countState);  // Read-only
  return <div>{count}</div>;
}

function Controls() {
  const setCount = useSetRecoilState(countState);  // Write-only
  return <button onClick={() => setCount(c => c + 1)}>+</button>;
}
```

---

**Interview Tips:**
- Recoil is a **state management library** developed by Facebook
- Uses **atomic model** - state split into atoms
- **RecoilRoot** wraps app (like Provider)
- **atom()** creates a piece of state with unique key
- **selector()** creates derived state from atoms
- **useRecoilState()** is like useState for atoms
- **useRecoilValue()** for read-only access
- **useSetRecoilState()** for write-only access
- **Built-in Suspense** support for async queries
- **Selectors** can be synchronous or asynchronous
- **Atom families** create dynamic atoms with parameters
- **Selector families** for parameterized derived state
- **Atom effects** for side effects (localStorage, logging)
- **Fine-grained subscriptions** - only re-render what changes
- Similar to **Jotai** but more features and complexity
- **Graph-based** - tracks dependencies between atoms
- Good for **complex state dependencies**
- **TypeScript** support is good
- Requires **unique keys** for all atoms and selectors
- Still **experimental** but production-ready

</details>

---

### 49. How do you handle global state in React?

<details>
<summary>View Answer</summary>

**Global State in React**

Global state is data that needs to be **accessible by multiple components** throughout your application. There are several approaches to manage it, each with different trade-offs.

---

## Approaches to Global State

### 1. Context API (Built-in)

**Best for: Simple global state, theme, auth**

```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const AppContext = createContext();

// Provider component
export function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  const value = {
    user,
    setUser,
    theme,
    setTheme
  };
  
  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// Custom hook
export function useApp() {
  const context = useContext(AppContext);
  if (!context) throw new Error('useApp must be used within AppProvider');
  return context;
}

// Usage
function App() {
  return (
    <AppProvider>
      <Header />
      <Main />
    </AppProvider>
  );
}

function Header() {
  const { user, theme } = useApp();
  return <header>{user?.name} - {theme}</header>;
}
```

**Pros:**
- ✅ Built into React - no extra dependencies
- ✅ Simple API
- ✅ Good for small to medium apps

**Cons:**
- ❌ Performance issues - all consumers re-render
- ❌ No dev tools
- ❌ Manual optimization needed

---

### 2. Redux / Redux Toolkit

**Best for: Large apps, complex state, team collaboration**

```jsx
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { Provider, useSelector, useDispatch } from 'react-redux';

// Create slice
const appSlice = createSlice({
  name: 'app',
  initialState: {
    user: null,
    theme: 'light'
  },
  reducers: {
    setUser: (state, action) => {
      state.user = action.payload;
    },
    setTheme: (state, action) => {
      state.theme = action.payload;
    }
  }
});

export const { setUser, setTheme } = appSlice.actions;

// Create store
const store = configureStore({
  reducer: {
    app: appSlice.reducer
  }
});

// Usage
function App() {
  return (
    <Provider store={store}>
      <Header />
      <Main />
    </Provider>
  );
}

function Header() {
  const { user, theme } = useSelector((state) => state.app);
  const dispatch = useDispatch();
  
  return (
    <header>
      {user?.name} - {theme}
      <button onClick={() => dispatch(setTheme('dark'))}>Toggle</button>
    </header>
  );
}
```

**Pros:**
- ✅ Excellent dev tools (time-travel debugging)
- ✅ Middleware ecosystem
- ✅ Predictable state updates
- ✅ Great for large teams

**Cons:**
- ❌ More boilerplate (even with RTK)
- ❌ Learning curve
- ❌ Extra dependency

---

### 3. Zustand

**Best for: Small to medium apps, minimal setup**

```jsx
import { create } from 'zustand';

// Create store - no Provider needed!
const useStore = create((set) => ({
  user: null,
  theme: 'light',
  setUser: (user) => set({ user }),
  setTheme: (theme) => set({ theme })
}));

// Usage - no Provider wrapper
function App() {
  return (
    <div>
      <Header />
      <Main />
    </div>
  );
}

function Header() {
  const user = useStore((state) => state.user);
  const theme = useStore((state) => state.theme);
  const setTheme = useStore((state) => state.setTheme);
  
  return (
    <header>
      {user?.name} - {theme}
      <button onClick={() => setTheme('dark')}>Toggle</button>
    </header>
  );
}
```

**Pros:**
- ✅ Minimal boilerplate
- ✅ No Provider needed
- ✅ Small bundle size (~1KB)
- ✅ Good performance with selectors

**Cons:**
- ❌ Less structure than Redux
- ❌ Smaller ecosystem

---

### 4. Jotai

**Best for: Atomic state, bottom-up approach**

```jsx
import { atom, useAtom } from 'jotai';

// Create atoms
const userAtom = atom(null);
const themeAtom = atom('light');

// Usage - no Provider needed
function App() {
  return (
    <div>
      <Header />
      <Main />
    </div>
  );
}

function Header() {
  const [user] = useAtom(userAtom);
  const [theme, setTheme] = useAtom(themeAtom);
  
  return (
    <header>
      {user?.name} - {theme}
      <button onClick={() => setTheme('dark')}>Toggle</button>
    </header>
  );
}
```

**Pros:**
- ✅ Atomic model - fine-grained updates
- ✅ Minimal boilerplate
- ✅ Built-in Suspense support
- ✅ Great TypeScript support

**Cons:**
- ❌ Different mental model
- ❌ Less mature than Redux

---

### 5. Recoil

**Best for: Complex state graphs, Facebook ecosystem**

```jsx
import { atom, useRecoilState, RecoilRoot } from 'recoil';

// Create atoms
const userState = atom({
  key: 'userState',
  default: null
});

const themeState = atom({
  key: 'themeState',
  default: 'light'
});

// Usage
function App() {
  return (
    <RecoilRoot>
      <Header />
      <Main />
    </RecoilRoot>
  );
}

function Header() {
  const [user] = useRecoilState(userState);
  const [theme, setTheme] = useRecoilState(themeState);
  
  return (
    <header>
      {user?.name} - {theme}
      <button onClick={() => setTheme('dark')}>Toggle</button>
    </header>
  );
}
```

**Pros:**
- ✅ Atomic model with more features
- ✅ Built-in async support
- ✅ Facebook-backed

**Cons:**
- ❌ Still experimental
- ❌ Requires RecoilRoot
- ❌ More complex than Jotai

---

## Comparison Table

| Solution | Bundle Size | Setup | Performance | DevTools | Best For |
|----------|-------------|-------|-------------|----------|----------|
| Context API | 0KB (built-in) | Simple | Poor* | No | Simple state |
| Redux Toolkit | ~11KB | Moderate | Good | Excellent | Large apps |
| Zustand | ~1KB | Minimal | Excellent | Optional | Small/Medium |
| Jotai | ~3KB | Minimal | Excellent | Optional | Atomic state |
| Recoil | ~18KB | Moderate | Excellent | Yes | Complex graphs |

*Context performance is poor without optimization

---

## Decision Guide

**Choose Context API when:**
- ✅ Small app (< 10 global states)
- ✅ Infrequently changing data (theme, auth)
- ✅ Don't want dependencies
- ✅ Simple requirements

**Choose Redux Toolkit when:**
- ✅ Large application (50k+ LOC)
- ✅ Complex state logic
- ✅ Large team (need structure)
- ✅ Need time-travel debugging
- ✅ Middleware requirements

**Choose Zustand when:**
- ✅ Medium app
- ✅ Want minimal setup
- ✅ Don't need strict patterns
- ✅ Bundle size matters

**Choose Jotai when:**
- ✅ Bottom-up state management
- ✅ Atomic model appeals to you
- ✅ Need Suspense integration
- ✅ TypeScript project

**Choose Recoil when:**
- ✅ Complex derived state
- ✅ Need graph-based state
- ✅ Facebook ecosystem

---

## Combining Approaches

**You can use multiple solutions in one app:**

```jsx
import { Provider as ReduxProvider } from 'react-redux';
import { ThemeProvider } from './ThemeContext';  // Context
import store from './store';  // Redux

function App() {
  return (
    // Redux for complex app state
    <ReduxProvider store={store}>
      {/* Context for simple UI state */}
      <ThemeProvider>
        <AppContent />
      </ThemeProvider>
    </ReduxProvider>
  );
}

// Redux: todos, users, cart, complex business logic
// Context: theme, locale, simple UI preferences
```

---

## Best Practices

**1. Start Simple**

```jsx
// Start with Context
const UserContext = createContext();

// Migrate to Redux/Zustand only if needed
// Don't over-engineer early
```

**2. Split by Domain**

```jsx
// ✅ Good: Split concerns
const UserContext = createContext();      // User state
const ThemeContext = createContext();     // UI state
const CartContext = createContext();      // Cart state

// ❌ Bad: Everything in one
const AppContext = createContext();  // All state together
```

**3. Optimize Context**

```jsx
import { useMemo } from 'react';

// ✅ Good: Memoize value
function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

// ❌ Bad: New object every render
function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>  {/* New object! */}
      {children}
    </UserContext.Provider>
  );
}
```

**4. Use Selectors**

```jsx
// ✅ Good: Selective subscription
const user = useStore((state) => state.user);
const theme = useStore((state) => state.theme);

// ❌ Bad: Subscribe to entire store
const store = useStore();
const user = store.user;
const theme = store.theme;
```

**5. Separate State and Dispatch**

```jsx
// ✅ Good: Split contexts
const StateContext = createContext();
const DispatchContext = createContext();

function Provider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Components using only dispatch won't re-render when state changes
```

---

## Production Example: E-commerce App

**Using Multiple Solutions:**

```jsx
import { configureStore, createSlice } from '@reduxjs/toolkit';
import { Provider as ReduxProvider } from 'react-redux';
import { create } from 'zustand';
import { createContext, useContext } from 'react';

// Redux: Complex business logic (products, cart, orders)
const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] },
  reducers: {
    addToCart: (state, action) => {
      state.items.push(action.payload);
    },
    removeFromCart: (state, action) => {
      state.items = state.items.filter(item => item.id !== action.payload);
    }
  }
});

const reduxStore = configureStore({
  reducer: {
    cart: cartSlice.reducer
  }
});

// Zustand: Medium complexity (filters, sorting)
const useFilterStore = create((set) => ({
  category: 'all',
  sortBy: 'price',
  setCategory: (category) => set({ category }),
  setSortBy: (sortBy) => set({ sortBy })
}));

// Context: Simple UI state (theme, sidebar)
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// App
function App() {
  return (
    <ReduxProvider store={reduxStore}>
      <ThemeProvider>
        <ProductList />
        <Cart />
      </ThemeProvider>
    </ReduxProvider>
  );
}

// Components use appropriate state management
function ProductList() {
  const category = useFilterStore((state) => state.category);  // Zustand
  const { theme } = useContext(ThemeContext);                   // Context
  const cartItems = useSelector((state) => state.cart.items);  // Redux
  
  return <div>...</div>;
}
```

---

## Migration Strategy

**Phase 1: Start with Context**
```jsx
// Simple apps - Context is enough
function App() {
  return (
    <UserProvider>
      <ThemeProvider>
        <Content />
      </ThemeProvider>
    </UserProvider>
  );
}
```

**Phase 2: Add Zustand for growing complexity**
```jsx
// Medium apps - Add Zustand for complex state
const useCartStore = create(/*...*/);
const useFilterStore = create(/*...*/);

function App() {
  return (
    <ThemeProvider>  {/* Keep simple Context */}
      <Content />    {/* Use Zustand stores */}
    </ThemeProvider>
  );
}
```

**Phase 3: Consider Redux for large apps**
```jsx
// Large apps - Migrate to Redux if needed
function App() {
  return (
    <Provider store={reduxStore}>
      <ThemeProvider>  {/* Keep Context for UI */}
        <Content />
      </ThemeProvider>
    </Provider>
  );
}
```

---

**Interview Tips:**
- Global state is data **shared across multiple components**
- **Context API** - built-in, simple, but performance issues
- **Redux Toolkit** - structured, dev tools, middleware, larger apps
- **Zustand** - minimal, no Provider, ~1KB, great for medium apps
- **Jotai** - atomic model, fine-grained updates, Suspense support
- **Recoil** - graph-based, Facebook-backed, experimental
- Choose based on **app size, complexity, team size**
- Can **combine multiple solutions** in one app
- Start **simple** (Context), migrate if needed
- **Split by domain** - UserContext, ThemeContext, not AppContext
- **Optimize Context** with useMemo to prevent re-renders
- Use **selectors** for performance in Redux/Zustand
- **Don't over-engineer** - Context is fine for many apps
- Redux better for **large teams** needing structure
- Zustand/Jotai better for **small teams** needing speed
- Context good for **infrequent updates** (theme, auth)
- Redux good for **frequent updates** with complex logic
- Consider **bundle size** - Context (0KB), Zustand (~1KB), Redux (~11KB)
- **Dev tools** important? Redux has best tools
- Modern trend: **smaller, simpler** libraries (Zustand, Jotai)
- No single "best" solution - depends on **requirements**

</details>

---

### 50. What is the Flux architecture pattern?

<details>
<summary>View Answer</summary>

**Flux Architecture**

Flux is an **application architecture** pattern developed by Facebook for building **client-side web applications**. It complements React's composable view components by utilizing a **unidirectional data flow**.

**Core Principle:** Unidirectional data flow

```
Action → Dispatcher → Store → View → Action
   ↑                                      |
   └──────────────────────────────────────┘
         (Unidirectional loop)
```

---

## Four Main Parts

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   1. ACTION                                             │
│      { type: 'ADD_TODO', payload: { text: 'Learn' } }  │
│                        ↓                                │
│   2. DISPATCHER                                         │
│      Receives actions, broadcasts to stores            │
│                        ↓                                │
│   3. STORE                                              │
│      Holds state, business logic                       │
│                        ↓                                │
│   4. VIEW (React Component)                             │
│      Renders UI, triggers actions                      │
│                        ↓                                │
│      User interaction                                   │
│                        ↓                                │
│   Back to ACTION ────────────────────────────────────┐ │
│                                                       ↓ │
└───────────────────────────────────────────────────────┘
```

---

## 1. Actions

**Plain objects describing what happened:**

```jsx
// Action type constants
const ADD_TODO = 'ADD_TODO';
const TOGGLE_TODO = 'TOGGLE_TODO';
const DELETE_TODO = 'DELETE_TODO';

// Action creators (functions that create actions)
function addTodo(text) {
  return {
    type: ADD_TODO,
    payload: {
      id: Date.now(),
      text: text,
      completed: false
    }
  };
}

function toggleTodo(id) {
  return {
    type: TOGGLE_TODO,
    payload: { id }
  };
}

function deleteTodo(id) {
  return {
    type: DELETE_TODO,
    payload: { id }
  };
}

// Actions describe WHAT happened, not HOW to update state
```

---

## 2. Dispatcher

**Central hub that broadcasts actions to stores:**

```jsx
class Dispatcher {
  constructor() {
    this.callbacks = [];
  }
  
  // Register a callback (store subscribes)
  register(callback) {
    this.callbacks.push(callback);
    return this.callbacks.length - 1; // Return ID
  }
  
  // Dispatch action to all stores
  dispatch(action) {
    this.callbacks.forEach(callback => {
      callback(action);
    });
  }
}

// Single dispatcher instance
const AppDispatcher = new Dispatcher();

// Dispatch an action
AppDispatcher.dispatch(addTodo('Learn Flux'));
```

---

## 3. Stores

**Hold application state and business logic:**

```jsx
import { EventEmitter } from 'events';

class TodoStore extends EventEmitter {
  constructor() {
    super();
    this.todos = [];
    
    // Register with dispatcher
    AppDispatcher.register((action) => {
      switch (action.type) {
        case ADD_TODO:
          this.todos.push(action.payload);
          this.emit('change');
          break;
          
        case TOGGLE_TODO:
          this.todos = this.todos.map(todo =>
            todo.id === action.payload.id
              ? { ...todo, completed: !todo.completed }
              : todo
          );
          this.emit('change');
          break;
          
        case DELETE_TODO:
          this.todos = this.todos.filter(todo => todo.id !== action.payload.id);
          this.emit('change');
          break;
      }
    });
  }
  
  // Get all todos
  getTodos() {
    return this.todos;
  }
  
  // Get filtered todos
  getActiveTodos() {
    return this.todos.filter(todo => !todo.completed);
  }
  
  getCompletedTodos() {
    return this.todos.filter(todo => todo.completed);
  }
}

// Single store instance
const todoStore = new TodoStore();

export default todoStore;
```

---

## 4. Views (React Components)

**React components that render UI and trigger actions:**

```jsx
import React, { useState, useEffect } from 'react';
import todoStore from './TodoStore';
import { addTodo, toggleTodo, deleteTodo } from './actions';
import AppDispatcher from './Dispatcher';

function TodoApp() {
  const [todos, setTodos] = useState(todoStore.getTodos());
  
  useEffect(() => {
    // Subscribe to store changes
    const handleChange = () => {
      setTodos(todoStore.getTodos());
    };
    
    todoStore.on('change', handleChange);
    
    // Cleanup
    return () => {
      todoStore.removeListener('change', handleChange);
    };
  }, []);
  
  const handleAddTodo = (text) => {
    // Dispatch action
    AppDispatcher.dispatch(addTodo(text));
  };
  
  const handleToggleTodo = (id) => {
    AppDispatcher.dispatch(toggleTodo(id));
  };
  
  const handleDeleteTodo = (id) => {
    AppDispatcher.dispatch(deleteTodo(id));
  };
  
  return (
    <div>
      <AddTodo onAdd={handleAddTodo} />
      <TodoList
        todos={todos}
        onToggle={handleToggleTodo}
        onDelete={handleDeleteTodo}
      />
    </div>
  );
}

function AddTodo({ onAdd }) {
  const [text, setText] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (text.trim()) {
      onAdd(text);
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">Add</button>
    </form>
  );
}

function TodoList({ todos, onToggle, onDelete }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => onToggle(todo.id)}
          />
          <span>{todo.text}</span>
          <button onClick={() => onDelete(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

## Unidirectional Data Flow

**Flow visualization:**

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  USER CLICKS BUTTON                                     │
│         ↓                                               │
│  Component dispatches ACTION                            │
│         ↓                                               │
│  DISPATCHER receives action                             │
│         ↓                                               │
│  DISPATCHER broadcasts to all STORES                    │
│         ↓                                               │
│  STORE updates its state                                │
│         ↓                                               │
│  STORE emits 'change' event                             │
│         ↓                                               │
│  VIEW listens to 'change' event                         │
│         ↓                                               │
│  VIEW re-renders with new data                          │
│         ↓                                               │
│  USER SEES UPDATED UI                                   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Key Point:** Data flows in **ONE DIRECTION**. Views cannot directly modify stores.

---

## Flux vs MVC

**Traditional MVC (Bidirectional):**

```
     Controller
    ↗    ↓    ↖
View ←→ Model
  ↓       ↑
   ↖──────┘
   
(Data flows in multiple directions - hard to debug)
```

**Flux (Unidirectional):**

```
Action → Dispatcher → Store → View
   ↑                             |
   └─────────────────────────────┘
   
(Data flows in one direction - predictable)
```

**Problems with MVC:**
- Bidirectional data flow
- Models can update each other
- Views can update models directly
- Hard to track changes
- Cascading updates

**Flux Solutions:**
- Unidirectional data flow
- Stores can't update each other directly
- Views can't update stores directly
- Clear data flow
- Predictable updates

---

## Multiple Stores

**Flux allows multiple stores:**

```jsx
// UserStore
class UserStore extends EventEmitter {
  constructor() {
    super();
    this.user = null;
    
    AppDispatcher.register((action) => {
      switch (action.type) {
        case 'LOGIN':
          this.user = action.payload;
          this.emit('change');
          break;
        case 'LOGOUT':
          this.user = null;
          this.emit('change');
          break;
      }
    });
  }
  
  getUser() {
    return this.user;
  }
}

// TodoStore
class TodoStore extends EventEmitter {
  constructor() {
    super();
    this.todos = [];
    
    AppDispatcher.register((action) => {
      switch (action.type) {
        case 'ADD_TODO':
          this.todos.push(action.payload);
          this.emit('change');
          break;
      }
    });
  }
  
  getTodos() {
    return this.todos;
  }
}

// Create instances
const userStore = new UserStore();
const todoStore = new TodoStore();
```

**Each store manages its own domain:**
- UserStore: Authentication, profile
- TodoStore: Todo items
- CartStore: Shopping cart
- NotificationStore: Notifications

---

## Flux vs Redux

**Flux (Original):**
- Multiple stores
- Each store registers with dispatcher
- Stores are objects (OOP)
- Manual subscriptions
- More boilerplate

**Redux (Inspired by Flux):**
- Single store
- No dispatcher (dispatch built into store)
- Reducers are pure functions
- React-Redux for subscriptions
- Less boilerplate (with Redux Toolkit)

```jsx
// Flux: Multiple stores
const userStore = new UserStore();
const todoStore = new TodoStore();
const cartStore = new CartStore();

// Redux: Single store, combined reducers
const store = createStore(combineReducers({
  user: userReducer,
  todos: todoReducer,
  cart: cartReducer
}));
```

---

## Benefits of Flux

**1. Predictable State Updates**
```
Action → Dispatcher → Store → View
(Clear, linear flow)
```

**2. Easier Debugging**
```
Action logged → Store updated → View rendered
(Can trace exact sequence)
```

**3. Separation of Concerns**
- Actions: What happened
- Dispatcher: Broadcast system
- Stores: Business logic
- Views: UI rendering

**4. Testability**
```jsx
// Test stores independently
const store = new TodoStore();
AppDispatcher.dispatch(addTodo('Test'));
assert(store.getTodos().length === 1);
```

**5. Scalability**
- Add new stores without affecting others
- Clear patterns to follow
- Team can work on different stores

---

## Real-World Example: Facebook Chat

**Why Facebook created Flux:**

Problem: Unread message count bug
- User sees notification (1 unread message)
- User opens chat
- Message count shows (0)
- Notification still shows (1)

With MVC: Multiple models updating each other caused issues

With Flux:
```
MARK_AS_READ action
       ↓
   Dispatcher
       ↓
   ├─→ MessageStore (updates messages)
   └─→ NotificationStore (updates count)
       ↓
   Both views update consistently
```

---

## Modern Alternatives

**Flux inspired many libraries:**

1. **Redux** - Single store, no dispatcher
2. **Zustand** - Simplified state management
3. **MobX** - Observable state
4. **Recoil** - Atomic state
5. **Jotai** - Primitive atoms

**Most popular: Redux**
- Simplified Flux concepts
- Single store instead of multiple
- No dispatcher (dispatch built-in)
- Pure functions (reducers)
- Time-travel debugging

---

## When to Use Flux Pattern

**✅ Good for:**
- Large applications
- Complex state interactions
- Multiple data sources
- Team needs structure
- Predictability is critical

**❌ Overkill for:**
- Small applications
- Simple state
- Prototypes
- Single developer
- Tight deadlines

**Modern recommendation:**
- Use **Redux Toolkit** (simplified Flux)
- Or **Zustand** (even simpler)
- Or **Context API** (built-in React)

---

**Interview Tips:**
- Flux is an **architecture pattern** by Facebook
- **Unidirectional data flow** is the core principle
- **Four parts**: Actions, Dispatcher, Stores, Views
- **Actions** describe what happened (plain objects)
- **Dispatcher** broadcasts actions to stores
- **Stores** hold state and business logic
- **Views** are React components that render UI
- Data flows **one way**: Action → Dispatcher → Store → View
- Views **cannot directly modify** stores
- Created to solve **bidirectional data flow** problems in MVC
- Flux allows **multiple stores** (one per domain)
- **Redux** was inspired by Flux but simplified
- Redux has **single store**, Flux has **multiple stores**
- Redux has no **dispatcher** (built into store)
- Flux uses **EventEmitter** for store subscriptions
- **Predictable** state updates
- Easier to **debug** than MVC
- Good **separation of concerns**
- Used at **Facebook** for chat application
- Modern apps typically use **Redux** instead
- Pattern still relevant - understanding helps with **Redux**

</details>
