# Basic Hooks

> Beginner / Junior Level (0-1 years)

---

## Questions

11. What is useState hook and how do you use it?
12. What is useEffect hook and when do you use it?
13. What is the dependency array in useEffect?
14. How do you handle events in React?
15. What is conditional rendering in React?
16. How do you render lists in React?
17. What is the purpose of keys in React lists?
18. How do you handle forms in React?
19. What are controlled vs uncontrolled components?
20. How do you pass data from child to parent component?

---

## Detailed Answers

### 11. What is useState hook and how do you use it?

<details>
<summary>View Answer</summary>

**What is useState?**

**useState** is a React Hook that lets you add state to functional components. Before hooks (React 16.8), only class components could have state. useState allows functional components to manage local state without converting to a class.

**Syntax**

```jsx
const [state, setState] = useState(initialValue);

// state: Current state value
// setState: Function to update state
// initialValue: Initial state value (any type)
```

**Basic Usage**

```jsx
import { useState } from 'react';

const Counter = () => {
  // Declare state variable
  const [count, setCount] = useState(0); // Initial value: 0
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
};
```

**Multiple State Variables**

```jsx
const UserForm = () => {
  // Each piece of state is independent
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

**Different State Types**

**1. Primitive Values**
```jsx
const [count, setCount] = useState(0);           // Number
const [name, setName] = useState('John');        // String
const [isActive, setIsActive] = useState(true);  // Boolean
const [data, setData] = useState(null);          // Null
```

**2. Objects**
```jsx
const [user, setUser] = useState({
  name: 'John',
  age: 30,
  email: 'john@example.com'
});

// ❌ WRONG: Mutating state directly
const updateNameWrong = () => {
  user.name = 'Jane'; // Don't mutate!
  setUser(user); // React won't detect the change
};

// ✅ CORRECT: Create new object
const updateNameCorrect = () => {
  setUser({
    ...user,        // Spread existing properties
    name: 'Jane'    // Override specific property
  });
};

// ✅ Update multiple properties
const updateUser = () => {
  setUser(prevUser => ({
    ...prevUser,
    name: 'Jane',
    age: 31
  }));
};

// ✅ Update nested object
const [user, setUser] = useState({
  name: 'John',
  address: {
    city: 'New York',
    country: 'USA'
  }
});

const updateCity = () => {
  setUser({
    ...user,
    address: {
      ...user.address,
      city: 'Los Angeles'
    }
  });
};
```

**3. Arrays**
```jsx
const [items, setItems] = useState(['Apple', 'Banana', 'Orange']);

// ✅ Add item
const addItem = (newItem) => {
  setItems([...items, newItem]);
  // Or: setItems(prevItems => [...prevItems, newItem]);
};

// ✅ Remove item by index
const removeItem = (index) => {
  setItems(items.filter((_, i) => i !== index));
};

// ✅ Remove item by value
const removeByValue = (value) => {
  setItems(items.filter(item => item !== value));
};

// ✅ Update item at index
const updateItem = (index, newValue) => {
  setItems(items.map((item, i) => 
    i === index ? newValue : item
  ));
};

// ✅ Insert at beginning
const addAtStart = (newItem) => {
  setItems([newItem, ...items]);
};

// ✅ Sort array
const sortItems = () => {
  setItems([...items].sort());
  // Note: [...items] creates a copy before sorting
};

// ✅ Clear array
const clearItems = () => {
  setItems([]);
};
```

**Functional Updates**

```jsx
const Counter = () => {
  const [count, setCount] = useState(0);
  
  // ❌ Problem: Stale state in rapid updates
  const incrementThreeTimesWrong = () => {
    setCount(count + 1); // Uses current count (0)
    setCount(count + 1); // Still uses current count (0)
    setCount(count + 1); // Still uses current count (0)
    // Result: count becomes 1, not 3!
  };
  
  // ✅ Solution: Functional update with previous state
  const incrementThreeTimesCorrect = () => {
    setCount(prevCount => prevCount + 1); // 0 + 1 = 1
    setCount(prevCount => prevCount + 1); // 1 + 1 = 2
    setCount(prevCount => prevCount + 1); // 2 + 1 = 3
    // Result: count becomes 3 ✓
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementThreeTimesCorrect}>+3</button>
    </div>
  );
};

// When to use functional updates:
// 1. When new state depends on previous state
// 2. When updating state rapidly
// 3. Inside callbacks that capture stale state
```

**Lazy Initialization**

```jsx
// ❌ Bad: Expensive calculation runs on every render
const Component = () => {
  const [data, setData] = useState(expensiveCalculation());
  // expensiveCalculation() called every render, even though result is ignored!
};

// ✅ Good: Lazy initialization (only runs once)
const Component = () => {
  const [data, setData] = useState(() => expensiveCalculation());
  // Function called only on initial render
};

// Real-world examples:

// 1. Loading from localStorage
const [user, setUser] = useState(() => {
  const saved = localStorage.getItem('user');
  return saved ? JSON.parse(saved) : null;
});

// 2. Complex initial state
const [gameState, setGameState] = useState(() => {
  return {
    score: 0,
    level: 1,
    inventory: initializeInventory(),
    enemies: generateEnemies(10)
  };
});

// 3. Reading from URL params
const [filters, setFilters] = useState(() => {
  const params = new URLSearchParams(window.location.search);
  return {
    category: params.get('category') || 'all',
    sort: params.get('sort') || 'date'
  };
});
```

**State Batching (React 18+)**

```jsx
const Component = () => {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  
  // React 18: Automatic batching
  const handleClick = () => {
    setCount(c => c + 1);  // Queued
    setFlag(f => !f);      // Queued
    // Both updates batched → Single re-render
  };
  
  // Even in async code (React 18+)
  const handleAsyncClick = async () => {
    await fetchData();
    setCount(c => c + 1);  // Batched
    setFlag(f => !f);      // Batched → Single re-render
  };
  
  // Force synchronous update (rare, use sparingly)
  import { flushSync } from 'react-dom';
  
  const handleSyncUpdate = () => {
    flushSync(() => {
      setCount(c => c + 1);
    });
    // DOM updated immediately
    console.log('Count updated');
  };
};
```

**Common Patterns**

**1. Toggle Boolean**
```jsx
const [isOpen, setIsOpen] = useState(false);

// Simple toggle
const toggle = () => setIsOpen(!isOpen);

// Functional toggle (safer)
const toggle = () => setIsOpen(prev => !prev);
```

**2. Increment/Decrement**
```jsx
const [count, setCount] = useState(0);

const increment = () => setCount(count + 1);
const decrement = () => setCount(count - 1);

// Or with functional updates
const increment = () => setCount(prev => prev + 1);
const decrement = () => setCount(prev => prev - 1);
```

**3. Form Input**
```jsx
const [formData, setFormData] = useState({
  username: '',
  email: '',
  password: ''
});

const handleChange = (e) => {
  const { name, value } = e.target;
  setFormData(prev => ({
    ...prev,
    [name]: value  // Dynamic key based on input name
  }));
};

// Usage
<input name="username" value={formData.username} onChange={handleChange} />
<input name="email" value={formData.email} onChange={handleChange} />
```

**4. Loading State**
```jsx
const [data, setData] = useState(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);

const fetchData = async () => {
  setLoading(true);
  setError(null);
  try {
    const response = await fetch('/api/data');
    const result = await response.json();
    setData(result);
  } catch (err) {
    setError(err.message);
  } finally {
    setLoading(false);
  }
};
```

**5. List Management**
```jsx
const [todos, setTodos] = useState([]);

// Add todo
const addTodo = (text) => {
  setTodos([...todos, {
    id: Date.now(),
    text,
    completed: false
  }]);
};

// Toggle todo
const toggleTodo = (id) => {
  setTodos(todos.map(todo =>
    todo.id === id 
      ? { ...todo, completed: !todo.completed }
      : todo
  ));
};

// Delete todo
const deleteTodo = (id) => {
  setTodos(todos.filter(todo => todo.id !== id));
};
```

**Common Mistakes**

```jsx
// ❌ Mistake 1: Direct mutation
const [user, setUser] = useState({ name: 'John', age: 30 });
user.name = 'Jane'; // Don't mutate state directly!
setUser(user); // React won't detect change

// ✅ Fix: Create new object
setUser({ ...user, name: 'Jane' });

// ❌ Mistake 2: Using state immediately after setState
const [count, setCount] = useState(0);
setCount(count + 1);
console.log(count); // Still 0! setState is async

// ✅ Fix: Use useEffect to react to state changes
useEffect(() => {
  console.log(count); // Logs updated value
}, [count]);

// ❌ Mistake 3: Not using functional updates
const increment = () => {
  setCount(count + 1);
  setCount(count + 1); // Uses stale count
};

// ✅ Fix: Use functional updates
const increment = () => {
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
};

// ❌ Mistake 4: Unnecessary state
const [products, setProducts] = useState([]);
const [productCount, setProductCount] = useState(0); // Redundant!

// ✅ Fix: Derive from state
const [products, setProducts] = useState([]);
const productCount = products.length; // Derived, not stored

// ❌ Mistake 5: Complex state structure
const [state, setState] = useState({
  user: {},
  posts: [],
  comments: [],
  likes: {},
  settings: {}
}); // Too complex, hard to update

// ✅ Fix: Split into multiple state variables
const [user, setUser] = useState({});
const [posts, setPosts] = useState([]);
const [comments, setComments] = useState([]);
```

**Production Example: Complete Form with useState**

```jsx
import { useState } from 'react';

const RegistrationForm = () => {
  // Form field states
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    password: '',
    confirmPassword: '',
    agreeToTerms: false
  });
  
  // UI states
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitSuccess, setSubmitSuccess] = useState(false);
  const [showPassword, setShowPassword] = useState(false);
  
  // Handle input changes
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
    
    // Clear error for this field when user types
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };
  
  // Validation
  const validate = () => {
    const newErrors = {};
    
    if (!formData.firstName.trim()) {
      newErrors.firstName = 'First name is required';
    }
    
    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    
    if (formData.password !== formData.confirmPassword) {
      newErrors.confirmPassword = 'Passwords do not match';
    }
    
    if (!formData.agreeToTerms) {
      newErrors.agreeToTerms = 'You must agree to terms';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  // Handle submit
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validate()) return;
    
    setIsSubmitting(true);
    
    try {
      const response = await fetch('/api/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      if (!response.ok) throw new Error('Registration failed');
      
      setSubmitSuccess(true);
      
      // Reset form
      setFormData({
        firstName: '',
        lastName: '',
        email: '',
        password: '',
        confirmPassword: '',
        agreeToTerms: false
      });
    } catch (error) {
      setErrors({ submit: error.message });
    } finally {
      setIsSubmitting(false);
    }
  };
  
  if (submitSuccess) {
    return (
      <div className="success-message">
        <h2>Registration Successful!</h2>
        <p>Please check your email to verify your account.</p>
      </div>
    );
  }
  
  return (
    <form onSubmit={handleSubmit} className="registration-form">
      <h2>Create Account</h2>
      
      {errors.submit && (
        <div className="error-banner">{errors.submit}</div>
      )}
      
      <div className="form-group">
        <label htmlFor="firstName">First Name *</label>
        <input
          id="firstName"
          name="firstName"
          value={formData.firstName}
          onChange={handleChange}
          className={errors.firstName ? 'error' : ''}
        />
        {errors.firstName && (
          <span className="error-text">{errors.firstName}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="lastName">Last Name</label>
        <input
          id="lastName"
          name="lastName"
          value={formData.lastName}
          onChange={handleChange}
        />
      </div>
      
      <div className="form-group">
        <label htmlFor="email">Email *</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          className={errors.email ? 'error' : ''}
        />
        {errors.email && (
          <span className="error-text">{errors.email}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="password">Password *</label>
        <div className="password-input">
          <input
            id="password"
            name="password"
            type={showPassword ? 'text' : 'password'}
            value={formData.password}
            onChange={handleChange}
            className={errors.password ? 'error' : ''}
          />
          <button
            type="button"
            onClick={() => setShowPassword(!showPassword)}
            className="toggle-password"
          >
            {showPassword ? 'Hide' : 'Show'}
          </button>
        </div>
        {errors.password && (
          <span className="error-text">{errors.password}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="confirmPassword">Confirm Password *</label>
        <input
          id="confirmPassword"
          name="confirmPassword"
          type="password"
          value={formData.confirmPassword}
          onChange={handleChange}
          className={errors.confirmPassword ? 'error' : ''}
        />
        {errors.confirmPassword && (
          <span className="error-text">{errors.confirmPassword}</span>
        )}
      </div>
      
      <div className="form-group checkbox">
        <label>
          <input
            name="agreeToTerms"
            type="checkbox"
            checked={formData.agreeToTerms}
            onChange={handleChange}
          />
          I agree to the Terms and Conditions *
        </label>
        {errors.agreeToTerms && (
          <span className="error-text">{errors.agreeToTerms}</span>
        )}
      </div>
      
      <button 
        type="submit" 
        disabled={isSubmitting}
        className="submit-button"
      >
        {isSubmitting ? 'Creating Account...' : 'Create Account'}
      </button>
    </form>
  );
};

export default RegistrationForm;
```

**Interview Tips:**
- useState is the most basic and commonly used Hook
- Returns array with **current state** and **updater function**
- State updates are **asynchronous** and may be **batched**
- Use **functional updates** when new state depends on previous state
- **Don't mutate** state directly - always create new objects/arrays
- Use **lazy initialization** for expensive initial calculations
- Can have **multiple useState calls** in one component
- State updates trigger **component re-render**
- Mention React 18's **automatic batching** for better performance

</details>

---

### 12. What is useEffect hook and when do you use it?

<details>
<summary>View Answer</summary>

**What is useEffect?**

**useEffect** is a React Hook that lets you perform side effects in functional components. Side effects are operations that affect things outside the component's scope: data fetching, subscriptions, timers, logging, manually changing the DOM, etc.

**Purpose**: Replaces lifecycle methods from class components:
- `componentDidMount`
- `componentDidUpdate`
- `componentWillUnmount`

**Syntax**

```jsx
useEffect(() => {
  // Side effect code here
  
  return () => {
    // Cleanup code (optional)
  };
}, [dependencies]); // Dependency array (optional)
```

**Basic Usage**

```jsx
import { useState, useEffect } from 'react';

const Example = () => {
  const [count, setCount] = useState(0);
  
  // Runs after every render
  useEffect(() => {
    console.log('Effect ran!');
    document.title = `Count: ${count}`;
  });
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

**When to Use useEffect**

**1. Data Fetching**
```jsx
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed to fetch');
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]); // Re-fetch when userId changes
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};
```

**2. Subscriptions / Event Listeners**
```jsx
const WindowSize = () => {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    // Setup: Add event listener
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    
    // Cleanup: Remove event listener
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []); // Empty array = run once on mount
  
  return (
    <div>
      Window size: {windowSize.width} x {windowSize.height}
    </div>
  );
};
```

**3. Timers / Intervals**
```jsx
const Timer = () => {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  
  useEffect(() => {
    let interval;
    
    if (isRunning) {
      interval = setInterval(() => {
        setSeconds(s => s + 1);
      }, 1000);
    }
    
    // Cleanup: Clear interval
    return () => {
      if (interval) clearInterval(interval);
    };
  }, [isRunning]); // Re-run when isRunning changes
  
  return (
    <div>
      <p>Time: {seconds}s</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'Pause' : 'Start'}
      </button>
      <button onClick={() => setSeconds(0)}>Reset</button>
    </div>
  );
};
```

**4. DOM Manipulation**
```jsx
const FocusInput = () => {
  const [shouldFocus, setShouldFocus] = useState(false);
  const inputRef = useRef(null);
  
  useEffect(() => {
    if (shouldFocus && inputRef.current) {
      inputRef.current.focus();
    }
  }, [shouldFocus]);
  
  return (
    <div>
      <input ref={inputRef} />
      <button onClick={() => setShouldFocus(true)}>Focus Input</button>
    </div>
  );
};
```

**5. Synchronizing with External Systems**
```jsx
// Sync with localStorage
const LocalStorageSync = () => {
  const [name, setName] = useState(() => {
    return localStorage.getItem('name') || '';
  });
  
  useEffect(() => {
    localStorage.setItem('name', name);
  }, [name]); // Save to localStorage whenever name changes
  
  return (
    <input 
      value={name}
      onChange={(e) => setName(e.target.value)}
      placeholder="Your name"
    />
  );
};

// Sync with browser title
const PageTitle = ({ title }) => {
  useEffect(() => {
    const previousTitle = document.title;
    document.title = title;
    
    // Restore previous title on unmount
    return () => {
      document.title = previousTitle;
    };
  }, [title]);
  
  return null; // This component doesn't render anything
};
```

**6. Analytics / Logging**
```jsx
const ProductPage = ({ productId }) => {
  useEffect(() => {
    // Log page view
    analytics.track('Product Page Viewed', {
      productId,
      timestamp: new Date()
    });
  }, [productId]);
  
  return <div>{/* Product content */}</div>;
};
```

**Effect Execution Timing**

```jsx
const LifecycleDemo = () => {
  console.log('1. Component rendering');
  
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('3. Effect executed (after render)');
    
    return () => {
      console.log('4. Cleanup (before next effect or unmount)');
    };
  });
  
  console.log('2. Render complete, returning JSX');
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

// Execution order:
// Initial render:
// 1. Component rendering
// 2. Render complete, returning JSX
// 3. Effect executed (after render)

// On button click:
// 1. Component rendering
// 2. Render complete, returning JSX
// 4. Cleanup (before next effect or unmount)
// 3. Effect executed (after render)

// On unmount:
// 4. Cleanup (before next effect or unmount)
```

**Cleanup Function**

```jsx
// When cleanup is needed:

// 1. Subscriptions
useEffect(() => {
  const subscription = dataSource.subscribe(data => {
    setData(data);
  });
  
  return () => {
    subscription.unsubscribe(); // Prevent memory leaks
  };
}, []);

// 2. Event listeners
useEffect(() => {
  const handler = (e) => console.log(e.key);
  window.addEventListener('keydown', handler);
  
  return () => {
    window.removeEventListener('keydown', handler);
  };
}, []);

// 3. Timers
useEffect(() => {
  const timer = setTimeout(() => {
    console.log('Timer fired');
  }, 1000);
  
  return () => {
    clearTimeout(timer); // Cancel timer if component unmounts
  };
}, []);

// 4. Abort API requests
useEffect(() => {
  const abortController = new AbortController();
  
  fetch('/api/data', { signal: abortController.signal })
    .then(response => response.json())
    .then(data => setData(data))
    .catch(err => {
      if (err.name !== 'AbortError') {
        setError(err.message);
      }
    });
  
  return () => {
    abortController.abort(); // Cancel request on unmount
  };
}, []);

// 5. WebSocket connections
useEffect(() => {
  const ws = new WebSocket('ws://localhost:8080');
  
  ws.onmessage = (event) => {
    setMessages(prev => [...prev, event.data]);
  };
  
  return () => {
    ws.close(); // Close connection on unmount
  };
}, []);
```

**Multiple Effects**

```jsx
const Dashboard = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [notifications, setNotifications] = useState([]);
  
  // Effect 1: Fetch user data
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]);
  
  // Effect 2: Fetch user's posts
  useEffect(() => {
    fetch(`/api/posts?userId=${userId}`)
      .then(res => res.json())
      .then(setPosts);
  }, [userId]);
  
  // Effect 3: Subscribe to notifications
  useEffect(() => {
    const subscription = notificationService.subscribe(
      userId,
      setNotifications
    );
    
    return () => subscription.unsubscribe();
  }, [userId]);
  
  // Effect 4: Update document title
  useEffect(() => {
    if (user) {
      document.title = `${user.name}'s Dashboard`;
    }
  }, [user]);
  
  // Separate concerns = cleaner, more maintainable code
};
```

**Common Patterns**

**1. Fetch on Mount**
```jsx
useEffect(() => {
  fetchData();
}, []); // Empty array = run once
```

**2. Fetch on Prop/State Change**
```jsx
useEffect(() => {
  fetchData(id);
}, [id]); // Re-fetch when id changes
```

**3. Debounced Search**
```jsx
const Search = () => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    // Debounce: Wait 500ms after user stops typing
    const timeoutId = setTimeout(() => {
      if (query) {
        fetch(`/api/search?q=${query}`)
          .then(res => res.json())
          .then(setResults);
      }
    }, 500);
    
    return () => clearTimeout(timeoutId); // Cancel previous timeout
  }, [query]);
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results.map(result => (
          <li key={result.id}>{result.name}</li>
        ))}
      </ul>
    </div>
  );
};
```

**4. Polling / Auto-refresh**
```jsx
const LiveData = () => {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    const fetchData = () => {
      fetch('/api/live-data')
        .then(res => res.json())
        .then(setData);
    };
    
    fetchData(); // Initial fetch
    
    const interval = setInterval(fetchData, 5000); // Poll every 5s
    
    return () => clearInterval(interval);
  }, []);
  
  return <div>{data?.value}</div>;
};
```

**5. Previous Value Tracking**
```jsx
const usePrevious = (value) => {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
};

const Counter = () => {
  const [count, setCount] = useState(0);
  const previousCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {previousCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};
```

**Common Mistakes**

```jsx
// ❌ Mistake 1: Missing dependency array
useEffect(() => {
  console.log(count);
}); // Runs after EVERY render

// ✅ Fix: Add dependency array
useEffect(() => {
  console.log(count);
}, [count]); // Runs only when count changes

// ❌ Mistake 2: Missing dependencies
useEffect(() => {
  fetchData(userId); // Uses userId but not in deps
}, []); // Won't re-run when userId changes!

// ✅ Fix: Include all dependencies
useEffect(() => {
  fetchData(userId);
}, [userId]);

// ❌ Mistake 3: Not cleaning up subscriptions
useEffect(() => {
  const subscription = subscribe();
  // No cleanup = memory leak!
}, []);

// ✅ Fix: Return cleanup function
useEffect(() => {
  const subscription = subscribe();
  return () => subscription.unsubscribe();
}, []);

// ❌ Mistake 4: Async useEffect directly
useEffect(async () => {  // ❌ Wrong!
  const data = await fetchData();
}, []);

// ✅ Fix: Define async function inside
useEffect(() => {
  const fetchData = async () => {
    const data = await fetch('/api/data');
  };
  fetchData();
}, []);

// ❌ Mistake 5: Infinite loops
useEffect(() => {
  setCount(count + 1); // Updates state
}, [count]); // Depends on state it updates = infinite loop!

// ✅ Fix: Remove problematic dependency or use ref
const countRef = useRef(count);
countRef.current = count;

useEffect(() => {
  const interval = setInterval(() => {
    setCount(countRef.current + 1);
  }, 1000);
  return () => clearInterval(interval);
}, []); // Safe: no dependencies
```

**Production Example: Real-world Data Fetching**

```jsx
import { useState, useEffect } from 'react';

const ProductList = ({ category, sortBy, searchQuery }) => {
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  
  useEffect(() => {
    const abortController = new AbortController();
    
    const fetchProducts = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const params = new URLSearchParams({
          category,
          sortBy,
          search: searchQuery,
          page,
          limit: 20
        });
        
        const response = await fetch(`/api/products?${params}`, {
          signal: abortController.signal
        });
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        
        setProducts(prev => 
          page === 1 ? data.products : [...prev, ...data.products]
        );
        setHasMore(data.hasMore);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
          console.error('Fetch error:', err);
        }
      } finally {
        setLoading(false);
      }
    };
    
    fetchProducts();
    
    // Cleanup: Abort fetch if component unmounts or dependencies change
    return () => {
      abortController.abort();
    };
  }, [category, sortBy, searchQuery, page]); // Re-fetch when any of these change
  
  // Reset to page 1 when filters change
  useEffect(() => {
    setPage(1);
  }, [category, sortBy, searchQuery]);
  
  // Log analytics
  useEffect(() => {
    if (products.length > 0) {
      analytics.track('Products Viewed', {
        category,
        count: products.length,
        timestamp: new Date()
      });
    }
  }, [products, category]);
  
  const loadMore = () => {
    if (!loading && hasMore) {
      setPage(prev => prev + 1);
    }
  };
  
  if (error) {
    return (
      <div className="error">
        <p>Error loading products: {error}</p>
        <button onClick={() => setPage(1)}>Try Again</button>
      </div>
    );
  }
  
  return (
    <div className="product-list">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
      
      {loading && <div className="loading">Loading...</div>}
      
      {!loading && hasMore && (
        <button onClick={loadMore} className="load-more">
          Load More
        </button>
      )}
      
      {!loading && !hasMore && products.length > 0 && (
        <p className="end-message">No more products</p>
      )}
      
      {!loading && products.length === 0 && (
        <p className="empty-message">No products found</p>
      )}
    </div>
  );
};

export default ProductList;
```

**Interview Tips:**
- useEffect handles **side effects** in functional components
- Replaces **componentDidMount**, **componentDidUpdate**, **componentWillUnmount**
- Runs **after render** (doesn't block painting)
- Always return **cleanup function** for subscriptions/listeners/timers
- Use **dependency array** to control when effect runs
- Can have **multiple useEffect** hooks for different concerns
- **Don't** use async directly in useEffect callback
- Common use cases: data fetching, subscriptions, DOM manipulation, timers
- Mention **React 18+** features: automatic batching, useTransition for async updates

</details>

---

### 13. What is the dependency array in useEffect?

<details>
<summary>View Answer</summary>

**What is the Dependency Array?**

The **dependency array** is the second argument to `useEffect` that controls when the effect runs. It tells React which values the effect depends on, and React will re-run the effect only when those values change between renders.

**Syntax**

```jsx
useEffect(() => {
  // Effect code
}, [dep1, dep2, dep3]); // Dependency array
```

**Three Scenarios**

**1. No Dependency Array - Runs After Every Render**
```jsx
const Component = () => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  // ⚠️ Runs after EVERY render
  useEffect(() => {
    console.log('Effect ran');
    document.title = `${name}: ${count}`;
  }); // No dependency array
  
  // Effect runs when:
  // - Component mounts
  // - count changes
  // - name changes
  // - ANY state/prop changes
  // - Parent re-renders
  
  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
};

// Use case: Rarely needed, usually indicates a mistake
```

**2. Empty Dependency Array - Runs Once on Mount**
```jsx
const Component = () => {
  const [data, setData] = useState(null);
  
  // ✅ Runs ONLY ONCE after initial render
  useEffect(() => {
    console.log('Component mounted');
    
    fetch('/api/data')
      .then(res => res.json())
      .then(setData);
    
    // Cleanup runs on unmount
    return () => {
      console.log('Component unmounted');
    };
  }, []); // Empty array = componentDidMount + componentWillUnmount
  
  return <div>{data?.title}</div>;
};

// Use cases:
// - Initial data fetching
// - Setting up subscriptions
// - Adding global event listeners
// - One-time initialization
```

**3. With Dependencies - Runs When Dependencies Change**
```jsx
const Component = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  
  // ✅ Runs when userId changes
  useEffect(() => {
    console.log(`Fetching user ${userId}`);
    setLoading(true);
    
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
  }, [userId]); // Re-runs only when userId changes
  
  // Effect runs when:
  // - Component mounts (initial render)
  // - userId prop changes
  // Effect does NOT run when:
  // - Other props/state change
  // - Parent re-renders (if userId is same)
  
  if (loading) return <div>Loading...</div>;
  return <div>{user?.name}</div>;
};
```

**How Dependencies Work**

```jsx
const SearchResults = () => {
  const [query, setQuery] = useState('');
  const [category, setCategory] = useState('all');
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    console.log('Effect running...');
    
    // Fetch results based on query and category
    fetch(`/api/search?q=${query}&cat=${category}`)
      .then(res => res.json())
      .then(setResults);
    
  }, [query, category]); // Multiple dependencies
  
  // Effect behavior:
  // Render 1: query='', category='all' → Effect runs
  // Render 2: query='react', category='all' → Effect runs (query changed)
  // Render 3: query='react', category='all' → Effect SKIPPED (no changes)
  // Render 4: query='react', category='books' → Effect runs (category changed)
  // Render 5: query='vue', category='books' → Effect runs (query changed)
  
  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <select value={category} onChange={(e) => setCategory(e.target.value)}>
        <option value="all">All</option>
        <option value="books">Books</option>
        <option value="articles">Articles</option>
      </select>
      <ul>
        {results.map(r => <li key={r.id}>{r.title}</li>)}
      </ul>
    </div>
  );
};
```

**Dependency Comparison**

React uses `Object.is()` comparison (similar to `===`) to check if dependencies changed:

```jsx
// Primitive values (compared by value)
const [count, setCount] = useState(0);
useEffect(() => {
  console.log(count);
}, [count]); // count: 0 → 1 → 2 (each triggers effect)

// Objects (compared by reference)
const [user, setUser] = useState({ name: 'John' });
useEffect(() => {
  console.log(user);
}, [user]); // New object reference → effect runs

// Arrays (compared by reference)
const [items, setItems] = useState([1, 2, 3]);
useEffect(() => {
  console.log(items);
}, [items]); // New array reference → effect runs

// Functions (compared by reference)
const handleClick = () => console.log('clicked');
useEffect(() => {
  document.addEventListener('click', handleClick);
  return () => document.removeEventListener('click', handleClick);
}, [handleClick]); // New function reference → effect runs
```

**Common Dependency Patterns**

**1. Props as Dependencies**
```jsx
const UserProfile = ({ userId, onUserLoad }) => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        onUserLoad(data); // Call callback with data
      });
  }, [userId, onUserLoad]); // Include all props used in effect
  
  return <div>{user?.name}</div>;
};
```

**2. State as Dependencies**
```jsx
const AutoSave = () => {
  const [content, setContent] = useState('');
  const [lastSaved, setLastSaved] = useState(null);
  
  useEffect(() => {
    // Auto-save after 2 seconds of no changes
    const timeoutId = setTimeout(() => {
      fetch('/api/save', {
        method: 'POST',
        body: JSON.stringify({ content })
      }).then(() => {
        setLastSaved(new Date());
      });
    }, 2000);
    
    return () => clearTimeout(timeoutId);
  }, [content]); // Re-run when content changes
  
  return (
    <div>
      <textarea 
        value={content}
        onChange={(e) => setContent(e.target.value)}
      />
      {lastSaved && <p>Last saved: {lastSaved.toLocaleTimeString()}</p>}
    </div>
  );
};
```

**3. Derived Values (Don't Need Dependencies)**
```jsx
const ProductList = ({ products }) => {
  const [filter, setFilter] = useState('all');
  
  // ✅ Derived value - no state needed
  const filteredProducts = products.filter(p => 
    filter === 'all' || p.category === filter
  );
  
  // ❌ Don't do this - unnecessary effect
  // const [filteredProducts, setFilteredProducts] = useState([]);
  // useEffect(() => {
  //   setFilteredProducts(products.filter(p => ...));
  // }, [products, filter]);
  
  useEffect(() => {
    console.log(`Showing ${filteredProducts.length} products`);
  }, [filteredProducts.length]); // Depend on derived value if needed
  
  return (
    <div>
      <select value={filter} onChange={(e) => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
      </select>
      {filteredProducts.map(p => <div key={p.id}>{p.name}</div>)}
    </div>
  );
};
```

**4. Refs (Usually Don't Need Dependencies)**
```jsx
const Component = () => {
  const countRef = useRef(0);
  
  useEffect(() => {
    countRef.current += 1;
    console.log(`Render count: ${countRef.current}`);
    // No dependencies needed - refs don't trigger re-renders
  });
  
  return <div>Component</div>;
};
```

**Missing Dependencies (Common Mistakes)**

```jsx
// ❌ Problem: Missing userId dependency
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`) // Uses userId
      .then(res => res.json())
      .then(setUser);
  }, []); // ❌ Empty array - won't refetch when userId changes!
  
  return <div>{user?.name}</div>;
};

// ✅ Fix: Include userId in dependencies
useEffect(() => {
  fetch(`/api/users/${userId}`)
    .then(res => res.json())
    .then(setUser);
}, [userId]); // ✅ Refetches when userId changes

// ❌ Problem: Missing function dependency
const Component = () => {
  const [count, setCount] = useState(0);
  
  const logCount = () => {
    console.log(`Count: ${count}`);
  };
  
  useEffect(() => {
    const interval = setInterval(logCount, 1000); // Uses logCount
    return () => clearInterval(interval);
  }, []); // ❌ Missing logCount - logs stale count!
  
  return <button onClick={() => setCount(c => c + 1)}>Increment</button>;
};

// ✅ Fix 1: Include function in dependencies
useEffect(() => {
  const interval = setInterval(logCount, 1000);
  return () => clearInterval(interval);
}, [logCount]); // ✅ But logCount changes every render...

// ✅ Fix 2: Use useCallback to stabilize function
const logCount = useCallback(() => {
  console.log(`Count: ${count}`);
}, [count]);

useEffect(() => {
  const interval = setInterval(logCount, 1000);
  return () => clearInterval(interval);
}, [logCount]); // ✅ Now works correctly

// ✅ Fix 3: Move function inside effect
useEffect(() => {
  const logCount = () => {
    console.log(`Count: ${count}`);
  };
  
  const interval = setInterval(logCount, 1000);
  return () => clearInterval(interval);
}, [count]); // ✅ Simple and correct
```

**Object/Array Dependencies (Tricky!)**

```jsx
// ❌ Problem: Object created on every render
const Component = ({ userId }) => {
  const config = { userId, format: 'json' }; // New object every render!
  
  useEffect(() => {
    fetchData(config);
  }, [config]); // ❌ Runs every render (new object reference)
  
  return <div>Data</div>;
};

// ✅ Fix 1: Use primitive dependencies
useEffect(() => {
  fetchData({ userId, format: 'json' });
}, [userId]); // ✅ Only userId matters

// ✅ Fix 2: useMemo to stabilize object
const config = useMemo(() => ({
  userId,
  format: 'json'
}), [userId]);

useEffect(() => {
  fetchData(config);
}, [config]); // ✅ config only changes when userId changes

// ❌ Problem: Array prop causes infinite loop
const Component = ({ items }) => { // items is new array each render from parent
  useEffect(() => {
    processItems(items);
  }, [items]); // ❌ Runs every time parent renders
};

// ✅ Fix: Depend on array length or stringify (if safe)
useEffect(() => {
  processItems(items);
}, [items.length]); // ✅ Only runs when length changes

// Or use deep comparison (careful - expensive!)
const useDeepEffect = (callback, dependencies) => {
  const previousRef = useRef();
  
  if (!isEqual(previousRef.current, dependencies)) {
    callback();
    previousRef.current = dependencies;
  }
};
```

**Functions as Dependencies**

```jsx
// ❌ Problem: Function recreated every render
const Component = () => {
  const [data, setData] = useState(null);
  
  const fetchData = () => { // New function every render
    fetch('/api/data').then(res => res.json()).then(setData);
  };
  
  useEffect(() => {
    fetchData();
  }, [fetchData]); // ❌ Infinite loop! fetchData always "new"
  
  return <div>{data}</div>;
};

// ✅ Fix 1: Move function inside effect
useEffect(() => {
  const fetchData = () => {
    fetch('/api/data').then(res => res.json()).then(setData);
  };
  fetchData();
}, []); // ✅ No external dependencies

// ✅ Fix 2: Use useCallback
const fetchData = useCallback(() => {
  fetch('/api/data').then(res => res.json()).then(setData);
}, []); // Stable function reference

useEffect(() => {
  fetchData();
}, [fetchData]); // ✅ Only runs once

// ✅ Fix 3: Don't depend on function
useEffect(() => {
  fetch('/api/data').then(res => res.json()).then(setData);
}, []); // ✅ Direct implementation
```

**ESLint Rule: exhaustive-deps**

```jsx
// ESLint plugin helps catch missing dependencies

// ⚠️ ESLint warning
useEffect(() => {
  console.log(count); // Uses count
}, []); // Warning: Missing dependency 'count'

// ✅ Follow ESLint suggestions
useEffect(() => {
  console.log(count);
}, [count]); // ✅ All dependencies included

// If you intentionally want to ignore:
useEffect(() => {
  console.log(count); // Logs initial count only
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []); // Suppress warning (use rarely!)
```

**Advanced Patterns**

**1. Conditional Dependencies**
```jsx
const Component = ({ shouldFetch, userId }) => {
  useEffect(() => {
    if (shouldFetch) {
      fetch(`/api/users/${userId}`)
        .then(res => res.json())
        .then(setUser);
    }
  }, [shouldFetch, userId]); // Both in array, condition inside effect
  
  // Don't do this:
  // useEffect(() => { ... }, shouldFetch ? [userId] : []); // ❌ Invalid
};
```

**2. Debounced Dependencies**
```jsx
const Search = () => {
  const [query, setQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');
  
  // Debounce effect
  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setDebouncedQuery(query);
    }, 500);
    
    return () => clearTimeout(timeoutId);
  }, [query]);
  
  // Search effect (depends on debounced value)
  useEffect(() => {
    if (debouncedQuery) {
      fetch(`/api/search?q=${debouncedQuery}`)
        .then(res => res.json())
        .then(setResults);
    }
  }, [debouncedQuery]);
};
```

**3. Previous Value Dependency**
```jsx
const Component = ({ value }) => {
  const previousValue = useRef(value);
  
  useEffect(() => {
    if (previousValue.current !== value) {
      console.log(`Changed from ${previousValue.current} to ${value}`);
      previousValue.current = value;
    }
  }, [value]);
};
```

**Production Example: Complete Dependency Management**

```jsx
import { useState, useEffect, useCallback, useMemo, useRef } from 'react';

const DataDashboard = ({ userId, refreshInterval = 5000 }) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [filters, setFilters] = useState({ status: 'all', sortBy: 'date' });
  
  // Stable callback for fetching
  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const params = new URLSearchParams({
        userId,
        ...filters
      });
      
      const response = await fetch(`/api/dashboard?${params}`);
      if (!response.ok) throw new Error('Fetch failed');
      
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [userId, filters]); // Recreated only when these change
  
  // Effect 1: Initial fetch and refetch on dependency change
  useEffect(() => {
    fetchData();
  }, [fetchData]); // Safe because fetchData is memoized
  
  // Effect 2: Auto-refresh interval
  useEffect(() => {
    const interval = setInterval(() => {
      fetchData();
    }, refreshInterval);
    
    return () => clearInterval(interval);
  }, [fetchData, refreshInterval]);
  
  // Effect 3: Log analytics (depends on derived value)
  const dataCount = data?.items?.length || 0;
  useEffect(() => {
    if (dataCount > 0) {
      analytics.track('Dashboard Viewed', {
        userId,
        itemCount: dataCount,
        filters
      });
    }
  }, [userId, dataCount, filters]);
  
  // Effect 4: Update document title
  useEffect(() => {
    document.title = loading 
      ? 'Loading...'
      : `Dashboard (${dataCount} items)`;
    
    return () => {
      document.title = 'App'; // Restore on unmount
    };
  }, [loading, dataCount]);
  
  // Memoized filtered data
  const filteredData = useMemo(() => {
    if (!data?.items) return [];
    
    return data.items
      .filter(item => filters.status === 'all' || item.status === filters.status)
      .sort((a, b) => {
        if (filters.sortBy === 'date') return b.date - a.date;
        return a.name.localeCompare(b.name);
      });
  }, [data, filters]);
  
  const handleFilterChange = useCallback((key, value) => {
    setFilters(prev => ({ ...prev, [key]: value }));
  }, []);
  
  if (error) return <div className="error">Error: {error}</div>;
  
  return (
    <div className="dashboard">
      <div className="filters">
        <select 
          value={filters.status}
          onChange={(e) => handleFilterChange('status', e.target.value)}
        >
          <option value="all">All</option>
          <option value="active">Active</option>
          <option value="completed">Completed</option>
        </select>
        
        <select 
          value={filters.sortBy}
          onChange={(e) => handleFilterChange('sortBy', e.target.value)}
        >
          <option value="date">Date</option>
          <option value="name">Name</option>
        </select>
        
        <button onClick={fetchData} disabled={loading}>
          {loading ? 'Refreshing...' : 'Refresh'}
        </button>
      </div>
      
      <div className="data-list">
        {filteredData.map(item => (
          <div key={item.id} className="data-item">
            {item.name}
          </div>
        ))}
      </div>
    </div>
  );
};

export default DataDashboard;
```

**Interview Tips:**
- Dependency array is the **second argument** to useEffect
- **Empty array []**: Effect runs once on mount (like componentDidMount)
- **With dependencies [a, b]**: Effect runs when a or b changes
- **No array**: Effect runs after every render (rarely needed)
- React uses **Object.is()** to compare dependencies (reference equality)
- **Include all** values used inside effect that can change between renders
- Use **useCallback** to stabilize function dependencies
- Use **useMemo** to stabilize object/array dependencies
- **ESLint** plugin helps catch missing dependencies
- Common mistake: **missing dependencies** causes stale closures
- Objects/arrays created inline are **new references** every render

</details>

---

### 14. How do you handle events in React?

<details>
<summary>View Answer</summary>

**Event Handling in React**

Event handling in React is similar to handling events in vanilla JavaScript, but with some important differences in syntax and behavior.

**Key Differences from HTML/JavaScript**

```jsx
// HTML/JavaScript
<button onclick="handleClick()">Click Me</button>

// React (camelCase, pass function reference)
<button onClick={handleClick}>Click Me</button>
```

**1. Event names use camelCase** (onClick, onChange, onSubmit)
**2. Pass function reference**, not a string
**3. Cannot return false** to prevent default behavior (must call `e.preventDefault()`)
**4. Events are SyntheticEvents** (cross-browser wrapper around native events)

**Basic Event Handling**

```jsx
import { useState } from 'react';

const ClickExample = () => {
  const [count, setCount] = useState(0);
  
  // Define event handler
  const handleClick = () => {
    setCount(count + 1);
    console.log('Button clicked!');
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Click Me</button>
    </div>
  );
};
```

**Common React Events**

**Mouse Events**
```jsx
const MouseEvents = () => {
  return (
    <div>
      <button onClick={() => console.log('Click')}>Click</button>
      <button onDoubleClick={() => console.log('Double Click')}>Double Click</button>
      <div 
        onMouseEnter={() => console.log('Mouse Enter')}
        onMouseLeave={() => console.log('Mouse Leave')}
        onMouseMove={(e) => console.log(e.clientX, e.clientY)}
        onMouseDown={() => console.log('Mouse Down')}
        onMouseUp={() => console.log('Mouse Up')}
      >
        Hover over me
      </div>
    </div>
  );
};
```

**Keyboard Events**
```jsx
const KeyboardEvents = () => {
  const handleKeyDown = (e) => {
    console.log('Key pressed:', e.key);
    console.log('Key code:', e.keyCode);
    console.log('Ctrl pressed:', e.ctrlKey);
    console.log('Shift pressed:', e.shiftKey);
    console.log('Alt pressed:', e.altKey);
  };
  
  const handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      console.log('Enter key pressed!');
    }
  };
  
  return (
    <div>
      <input 
        onKeyDown={handleKeyDown}
        onKeyPress={handleKeyPress}
        onKeyUp={(e) => console.log('Key released:', e.key)}
        placeholder="Type something..."
      />
    </div>
  );
};
```

**Form Events**
```jsx
const FormEvents = () => {
  const [formData, setFormData] = useState({ name: '', email: '' });
  
  const handleChange = (e) => {
    console.log('Input changed:', e.target.value);
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault(); // Prevent page reload
    console.log('Form submitted:', formData);
  };
  
  const handleFocus = (e) => {
    console.log('Input focused:', e.target.name);
  };
  
  const handleBlur = (e) => {
    console.log('Input blurred:', e.target.name);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        name="name"
        value={formData.name}
        onChange={handleChange}
        onFocus={handleFocus}
        onBlur={handleBlur}
      />
      <input 
        name="email"
        value={formData.email}
        onChange={handleChange}
      />
      <button type="submit">Submit</button>
    </form>
  );
};
```

**Focus Events**
```jsx
const FocusEvents = () => {
  const handleFocus = () => console.log('Focused');
  const handleBlur = () => console.log('Blurred');
  
  return (
    <input 
      onFocus={handleFocus}
      onBlur={handleBlur}
      placeholder="Focus me"
    />
  );
};
```

**Event Handler Patterns**

**1. Inline Arrow Function**
```jsx
const InlineHandler = () => {
  return (
    <button onClick={() => console.log('Clicked')}>Click</button>
  );
};

// ⚠️ Warning: Creates new function on every render
// Use sparingly, can cause performance issues with frequent re-renders
```

**2. Method Reference**
```jsx
const MethodReference = () => {
  const handleClick = () => {
    console.log('Clicked');
  };
  
  return (
    <button onClick={handleClick}>Click</button>
  );
};

// ✅ Preferred: Function created once, same reference passed each render
```

**3. Inline Function with Parameters**
```jsx
const WithParameters = () => {
  const handleClick = (id, name) => {
    console.log(`Clicked item ${id}: ${name}`);
  };
  
  return (
    <div>
      {/* Pass parameters using arrow function */}
      <button onClick={() => handleClick(1, 'Item 1')}>Item 1</button>
      <button onClick={() => handleClick(2, 'Item 2')}>Item 2</button>
    </div>
  );
};
```

**4. Using bind()**
```jsx
const UsingBind = () => {
  const handleClick = (id, name) => {
    console.log(`Clicked ${id}: ${name}`);
  };
  
  return (
    <div>
      {/* bind() creates a new function with parameters */}
      <button onClick={handleClick.bind(null, 1, 'Item 1')}>Item 1</button>
      <button onClick={handleClick.bind(null, 2, 'Item 2')}>Item 2</button>
    </div>
  );
};

// ⚠️ Warning: bind() creates new function each time, similar performance to inline arrow
```

**5. Event Handler with Data Attributes**
```jsx
const DataAttributes = () => {
  const handleClick = (e) => {
    const id = e.target.dataset.id;
    const name = e.target.dataset.name;
    console.log(`Clicked ${id}: ${name}`);
  };
  
  return (
    <div>
      <button data-id="1" data-name="Item 1" onClick={handleClick}>Item 1</button>
      <button data-id="2" data-name="Item 2" onClick={handleClick}>Item 2</button>
    </div>
  );
};

// ✅ Good: Single function reference, data in attributes
```

**Preventing Default Behavior**

```jsx
const PreventDefault = () => {
  const handleSubmit = (e) => {
    e.preventDefault(); // Prevent form submission/page reload
    console.log('Form submitted programmatically');
  };
  
  const handleLinkClick = (e) => {
    e.preventDefault(); // Prevent navigation
    console.log('Link clicked but navigation prevented');
    // Handle navigation programmatically if needed
  };
  
  return (
    <div>
      <form onSubmit={handleSubmit}>
        <input type="text" />
        <button type="submit">Submit</button>
      </form>
      
      <a href="https://example.com" onClick={handleLinkClick}>
        Click me (won't navigate)
      </a>
    </div>
  );
};
```

**Stopping Event Propagation**

```jsx
const StopPropagation = () => {
  const handleParentClick = () => {
    console.log('Parent clicked');
  };
  
  const handleChildClick = (e) => {
    e.stopPropagation(); // Prevent event from bubbling to parent
    console.log('Child clicked');
  };
  
  return (
    <div onClick={handleParentClick} style={{ padding: '20px', background: '#eee' }}>
      Parent Div
      <button onClick={handleChildClick}>
        Child Button (won't trigger parent)
      </button>
    </div>
  );
};
```

**Synthetic Events**

React wraps native browser events in a **SyntheticEvent** object:

```jsx
const SyntheticEventExample = () => {
  const handleClick = (e) => {
    console.log('SyntheticEvent:', e); // React's wrapper
    console.log('Native event:', e.nativeEvent); // Original browser event
    console.log('Event type:', e.type); // 'click'
    console.log('Target element:', e.target); // Element that triggered event
    console.log('Current target:', e.currentTarget); // Element handler is attached to
    console.log('Is default prevented?', e.defaultPrevented);
    console.log('Timestamp:', e.timeStamp);
  };
  
  return <button onClick={handleClick}>Click Me</button>;
};

// Benefits of SyntheticEvent:
// 1. Cross-browser compatibility
// 2. Consistent API across browsers
// 3. Performance optimizations (event pooling in React < 17)
// 4. Works identically in all browsers
```

**Accessing Event Properties**

```jsx
const EventProperties = () => {
  const handleEvent = (e) => {
    // Mouse event properties
    console.log('Mouse position:', e.clientX, e.clientY);
    console.log('Screen position:', e.screenX, e.screenY);
    console.log('Page position:', e.pageX, e.pageY);
    console.log('Button pressed:', e.button); // 0=left, 1=middle, 2=right
    
    // Keyboard event properties
    console.log('Key:', e.key);
    console.log('Key code:', e.keyCode);
    console.log('Alt key:', e.altKey);
    console.log('Ctrl key:', e.ctrlKey);
    console.log('Shift key:', e.shiftKey);
    console.log('Meta key:', e.metaKey); // Cmd on Mac, Win on Windows
    
    // Form input properties
    console.log('Input value:', e.target.value);
    console.log('Input name:', e.target.name);
    console.log('Checked:', e.target.checked); // For checkboxes
    console.log('Selected:', e.target.selected); // For select options
  };
  
  return (
    <div>
      <input onChange={handleEvent} />
      <button onClick={handleEvent}>Click</button>
    </div>
  );
};
```

**Common Event Handling Patterns**

**1. Toggle State**
```jsx
const ToggleExample = () => {
  const [isOpen, setIsOpen] = useState(false);
  
  const handleToggle = () => {
    setIsOpen(!isOpen);
  };
  
  return (
    <div>
      <button onClick={handleToggle}>
        {isOpen ? 'Close' : 'Open'}
      </button>
      {isOpen && <div>Content is visible</div>}
    </div>
  );
};
```

**2. Handle Multiple Inputs**
```jsx
const MultipleInputs = () => {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    age: ''
  });
  
  // Single handler for all inputs
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  return (
    <form>
      <input name="firstName" value={formData.firstName} onChange={handleChange} />
      <input name="lastName" value={formData.lastName} onChange={handleChange} />
      <input name="email" value={formData.email} onChange={handleChange} />
      <input name="age" type="number" value={formData.age} onChange={handleChange} />
    </form>
  );
};
```

**3. Debounced Input**
```jsx
import { useState, useEffect } from 'react';

const DebouncedSearch = () => {
  const [query, setQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');
  
  useEffect(() => {
    const timeoutId = setTimeout(() => {
      setDebouncedQuery(query);
    }, 500);
    
    return () => clearTimeout(timeoutId);
  }, [query]);
  
  const handleChange = (e) => {
    setQuery(e.target.value);
  };
  
  useEffect(() => {
    if (debouncedQuery) {
      console.log('Search for:', debouncedQuery);
      // Perform search API call here
    }
  }, [debouncedQuery]);
  
  return (
    <input 
      value={query}
      onChange={handleChange}
      placeholder="Search (debounced)..."
    />
  );
};
```

**4. List Item Click**
```jsx
const ListClickExample = () => {
  const [items] = useState([
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Orange' }
  ]);
  const [selectedId, setSelectedId] = useState(null);
  
  const handleItemClick = (id) => {
    setSelectedId(id);
    console.log('Selected item:', id);
  };
  
  return (
    <ul>
      {items.map(item => (
        <li 
          key={item.id}
          onClick={() => handleItemClick(item.id)}
          style={{ 
            background: selectedId === item.id ? '#e0e0e0' : 'white',
            cursor: 'pointer',
            padding: '8px'
          }}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
};
```

**5. Keyboard Shortcuts**
```jsx
import { useEffect } from 'react';

const KeyboardShortcuts = () => {
  useEffect(() => {
    const handleKeyPress = (e) => {
      // Ctrl/Cmd + S to save
      if ((e.ctrlKey || e.metaKey) && e.key === 's') {
        e.preventDefault();
        console.log('Save shortcut pressed');
      }
      
      // Ctrl/Cmd + K to search
      if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
        e.preventDefault();
        console.log('Search shortcut pressed');
      }
      
      // Escape to close
      if (e.key === 'Escape') {
        console.log('Escape pressed');
      }
    };
    
    window.addEventListener('keydown', handleKeyPress);
    
    return () => {
      window.removeEventListener('keydown', handleKeyPress);
    };
  }, []);
  
  return <div>Press Ctrl+S, Ctrl+K, or Escape</div>;
};
```

**Common Mistakes**

```jsx
// ❌ Mistake 1: Calling function instead of passing reference
const Wrong1 = () => {
  const handleClick = () => console.log('Clicked');
  
  return (
    <button onClick={handleClick()}> {/* ❌ Calls immediately! */}
      Click Me
    </button>
  );
};

// ✅ Fix: Pass function reference
<button onClick={handleClick}>Click Me</button>

// ❌ Mistake 2: Using string instead of function
<button onClick="handleClick()"> {/* ❌ Won't work in React */}
  Click Me
</button>

// ✅ Fix: Use JSX syntax with curly braces
<button onClick={handleClick}>Click Me</button>

// ❌ Mistake 3: Forgetting preventDefault
const handleSubmit = (e) => {
  // ❌ Form will reload page!
  console.log('Submitting...');
};

// ✅ Fix: Call preventDefault
const handleSubmit = (e) => {
  e.preventDefault();
  console.log('Submitting...');
};

// ❌ Mistake 4: Creating new function in render (performance issue)
const items = [1, 2, 3, 4, 5];
return (
  <div>
    {items.map(item => (
      <button onClick={() => handleClick(item)}> {/* New function each render */}
        Item {item}
      </button>
    ))}
  </div>
);

// ✅ Fix: Use data attributes or memoization
const handleClick = (e) => {
  const item = e.target.dataset.item;
  console.log('Clicked:', item);
};

return (
  <div>
    {items.map(item => (
      <button data-item={item} onClick={handleClick}>
        Item {item}
      </button>
    ))}
  </div>
);

// ❌ Mistake 5: Accessing synthetic event asynchronously
const handleClick = (e) => {
  setTimeout(() => {
    console.log(e.target.value); // ❌ Error! Event is pooled and nullified
  }, 1000);
};

// ✅ Fix: Extract value first
const handleClick = (e) => {
  const value = e.target.value; // Store value
  setTimeout(() => {
    console.log(value); // ✅ Works!
  }, 1000);
};

// Or use e.persist() in React < 17
const handleClick = (e) => {
  e.persist(); // Prevents event pooling (React < 17)
  setTimeout(() => {
    console.log(e.target.value); // Works in React < 17
  }, 1000);
};
```

**Production Example: Complete Form with Event Handling**

```jsx
import { useState } from 'react';

const ContactForm = () => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    subject: '',
    message: '',
    subscribe: false
  });
  
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  // Handle input change
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };
  
  // Handle blur (mark field as touched)
  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({
      ...prev,
      [name]: true
    }));
    
    // Validate on blur
    validateField(name, formData[name]);
  };
  
  // Validate individual field
  const validateField = (name, value) => {
    let error = '';
    
    switch (name) {
      case 'name':
        if (!value.trim()) error = 'Name is required';
        else if (value.length < 2) error = 'Name must be at least 2 characters';
        break;
      
      case 'email':
        if (!value.trim()) error = 'Email is required';
        else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) error = 'Invalid email';
        break;
      
      case 'subject':
        if (!value.trim()) error = 'Subject is required';
        break;
      
      case 'message':
        if (!value.trim()) error = 'Message is required';
        else if (value.length < 10) error = 'Message must be at least 10 characters';
        break;
    }
    
    if (error) {
      setErrors(prev => ({ ...prev, [name]: error }));
      return false;
    }
    
    return true;
  };
  
  // Validate all fields
  const validateForm = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) newErrors.name = 'Name is required';
    if (!formData.email.trim()) newErrors.email = 'Email is required';
    else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) {
      newErrors.email = 'Invalid email';
    }
    if (!formData.subject.trim()) newErrors.subject = 'Subject is required';
    if (!formData.message.trim()) newErrors.message = 'Message is required';
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  // Handle form submission
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Mark all fields as touched
    setTouched({
      name: true,
      email: true,
      subject: true,
      message: true
    });
    
    if (!validateForm()) {
      console.log('Form has errors');
      return;
    }
    
    setIsSubmitting(true);
    
    try {
      const response = await fetch('/api/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      if (!response.ok) throw new Error('Failed to submit');
      
      alert('Message sent successfully!');
      
      // Reset form
      setFormData({
        name: '',
        email: '',
        subject: '',
        message: '',
        subscribe: false
      });
      setTouched({});
      setErrors({});
    } catch (error) {
      alert('Failed to send message: ' + error.message);
    } finally {
      setIsSubmitting(false);
    }
  };
  
  // Handle reset
  const handleReset = (e) => {
    e.preventDefault();
    setFormData({
      name: '',
      email: '',
      subject: '',
      message: '',
      subscribe: false
    });
    setErrors({});
    setTouched({});
  };
  
  return (
    <form onSubmit={handleSubmit} className="contact-form">
      <h2>Contact Us</h2>
      
      <div className="form-group">
        <label htmlFor="name">Name *</label>
        <input
          id="name"
          name="name"
          type="text"
          value={formData.name}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.name && errors.name ? 'error' : ''}
        />
        {touched.name && errors.name && (
          <span className="error-text">{errors.name}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="email">Email *</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.email && errors.email ? 'error' : ''}
        />
        {touched.email && errors.email && (
          <span className="error-text">{errors.email}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="subject">Subject *</label>
        <input
          id="subject"
          name="subject"
          type="text"
          value={formData.subject}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.subject && errors.subject ? 'error' : ''}
        />
        {touched.subject && errors.subject && (
          <span className="error-text">{errors.subject}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="message">Message *</label>
        <textarea
          id="message"
          name="message"
          rows="5"
          value={formData.message}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.message && errors.message ? 'error' : ''}
        />
        {touched.message && errors.message && (
          <span className="error-text">{errors.message}</span>
        )}
      </div>
      
      <div className="form-group checkbox">
        <label>
          <input
            name="subscribe"
            type="checkbox"
            checked={formData.subscribe}
            onChange={handleChange}
          />
          Subscribe to newsletter
        </label>
      </div>
      
      <div className="form-actions">
        <button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Sending...' : 'Send Message'}
        </button>
        <button type="button" onClick={handleReset}>
          Reset
        </button>
      </div>
    </form>
  );
};

export default ContactForm;
```

**Interview Tips:**
- React uses **camelCase** for event names (onClick, onChange)
- Events are **SyntheticEvents** - cross-browser wrappers
- Pass **function reference**, not function call
- Use **e.preventDefault()** to prevent default behavior
- Use **e.stopPropagation()** to stop event bubbling
- **Avoid inline arrow functions** in lists (performance)
- Use **data attributes** for passing data to handlers
- **Extract event values** before async operations
- Common events: onClick, onChange, onSubmit, onBlur, onFocus
- Mention **event pooling** was removed in React 17

</details>

---

### 15. What is conditional rendering in React?

<details>
<summary>View Answer</summary>

**What is Conditional Rendering?**

**Conditional rendering** is the practice of rendering different UI elements or components based on certain conditions. In React, you can conditionally render components using JavaScript expressions like if statements, ternary operators, logical AND (&&), or switch statements.

**Why Use Conditional Rendering?**

- Show/hide components based on state
- Display different content for authenticated/unauthenticated users
- Render loading states
- Show error messages
- Display content based on user roles or permissions
- Responsive UI behavior

**Methods of Conditional Rendering**

**1. If-Else Statement**

```jsx
import { useState } from 'react';

const LoginStatus = () => {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  
  // Using if-else
  if (isLoggedIn) {
    return (
      <div>
        <h2>Welcome back!</h2>
        <button onClick={() => setIsLoggedIn(false)}>Logout</button>
      </div>
    );
  } else {
    return (
      <div>
        <h2>Please sign in</h2>
        <button onClick={() => setIsLoggedIn(true)}>Login</button>
      </div>
    );
  }
};
```

**2. Element Variables**

```jsx
const Greeting = ({ isLoggedIn, username }) => {
  let greeting;
  
  if (isLoggedIn) {
    greeting = <h1>Welcome back, {username}!</h1>;
  } else {
    greeting = <h1>Please sign in</h1>;
  }
  
  return (
    <div>
      {greeting}
      <p>Additional content here</p>
    </div>
  );
};
```

**3. Ternary Operator (condition ? true : false)**

```jsx
const UserStatus = ({ isLoggedIn }) => {
  return (
    <div>
      {isLoggedIn ? (
        <h2>Welcome back!</h2>
      ) : (
        <h2>Please sign in</h2>
      )}
    </div>
  );
};

// Inline ternary for simple cases
const Button = ({ isLoggedIn }) => {
  return (
    <button>
      {isLoggedIn ? 'Logout' : 'Login'}
    </button>
  );
};

// Nested ternary (use sparingly - can get complex)
const Status = ({ status }) => {
  return (
    <div>
      {status === 'loading' ? (
        <p>Loading...</p>
      ) : status === 'error' ? (
        <p>Error occurred</p>
      ) : (
        <p>Success!</p>
      )}
    </div>
  );
};
```

**4. Logical AND (&&) Operator**

```jsx
const Notification = ({ hasNotification, message }) => {
  return (
    <div>
      <h1>Dashboard</h1>
      {/* Render only if condition is true */}
      {hasNotification && (
        <div className="notification">
          {message}
        </div>
      )}
    </div>
  );
};

// Multiple conditions
const UserProfile = ({ user, isLoggedIn, isPremium }) => {
  return (
    <div>
      {isLoggedIn && <h2>Welcome, {user.name}</h2>}
      {isLoggedIn && isPremium && <span className="badge">Premium Member</span>}
      {isLoggedIn && !user.emailVerified && (
        <div className="alert">Please verify your email</div>
      )}
    </div>
  );
};
```

**⚠️ Be Careful with && Operator**

```jsx
// ❌ Problem: Renders 0 when count is 0
const Items = ({ items }) => {
  return (
    <div>
      {items.length && <p>You have {items.length} items</p>}
      {/* If items.length is 0, it renders "0" on the page! */}
    </div>
  );
};

// ✅ Fix: Use explicit comparison
const Items = ({ items }) => {
  return (
    <div>
      {items.length > 0 && <p>You have {items.length} items</p>}
      {/* Now renders nothing when items.length is 0 */}
    </div>
  );
};

// ✅ Alternative: Use Boolean conversion
const Items = ({ items }) => {
  return (
    <div>
      {!!items.length && <p>You have {items.length} items</p>}
      {/* !! converts to boolean */}
    </div>
  );
};
```

**5. Logical OR (||) Operator**

```jsx
const UserName = ({ user }) => {
  return (
    <div>
      <p>Name: {user.name || 'Guest'}</p>
      {/* Shows 'Guest' if user.name is falsy */}
    </div>
  );
};

// Nullish coalescing operator (??) - safer than ||
const UserAge = ({ user }) => {
  return (
    <div>
      <p>Age: {user.age ?? 'Not specified'}</p>
      {/* Only replaces null/undefined, not 0 or '' */}
    </div>
  );
};
```

**6. Switch Statement**

```jsx
const StatusMessage = ({ status }) => {
  const renderStatus = () => {
    switch (status) {
      case 'loading':
        return <div className="spinner">Loading...</div>;
      
      case 'success':
        return <div className="success">Operation successful!</div>;
      
      case 'error':
        return <div className="error">An error occurred</div>;
      
      case 'idle':
        return <div>Ready to start</div>;
      
      default:
        return <div>Unknown status</div>;
    }
  };
  
  return (
    <div>
      <h2>Status</h2>
      {renderStatus()}
    </div>
  );
};
```

**7. Enum Objects**

```jsx
const STATUSES = {
  loading: <div className="spinner">Loading...</div>,
  success: <div className="success">Success!</div>,
  error: <div className="error">Error occurred</div>,
  idle: <div>Ready</div>
};

const StatusDisplay = ({ status }) => {
  return (
    <div>
      <h2>Status</h2>
      {STATUSES[status] || STATUSES.idle}
    </div>
  );
};

// With functions for dynamic content
const STATUS_COMPONENTS = {
  loading: () => <div className="spinner">Loading...</div>,
  success: (data) => <div className="success">Loaded {data.count} items</div>,
  error: (error) => <div className="error">{error.message}</div>,
  idle: () => <div>Ready to start</div>
};

const DynamicStatus = ({ status, data, error }) => {
  const StatusComponent = STATUS_COMPONENTS[status];
  
  return (
    <div>
      {StatusComponent && StatusComponent(status === 'success' ? data : error)}
    </div>
  );
};
```

**8. Immediately Invoked Function Expression (IIFE)**

```jsx
const ComplexConditional = ({ status, user, isPremium }) => {
  return (
    <div>
      {(() => {
        if (status === 'loading') {
          return <div>Loading...</div>;
        }
        
        if (status === 'error') {
          return <div>Error</div>;
        }
        
        if (!user) {
          return <div>Please login</div>;
        }
        
        if (isPremium) {
          return (
            <div>
              <h2>Premium Content</h2>
              <p>Welcome, {user.name}!</p>
            </div>
          );
        }
        
        return (
          <div>
            <h2>Standard Content</h2>
            <p>Upgrade to premium for more features</p>
          </div>
        );
      })()}
    </div>
  );
};
```

**Common Patterns**

**1. Loading State**

```jsx
const DataDisplay = () => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, []);
  
  if (loading) {
    return <div className="spinner">Loading...</div>;
  }
  
  if (error) {
    return <div className="error">Error: {error}</div>;
  }
  
  if (!data) {
    return <div>No data available</div>;
  }
  
  return (
    <div>
      <h2>Data</h2>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
};
```

**2. Authentication Check**

```jsx
const ProtectedContent = ({ user, isAuthenticated }) => {
  if (!isAuthenticated) {
    return (
      <div className="login-prompt">
        <h2>Please Login</h2>
        <p>You need to be logged in to view this content</p>
        <button>Login</button>
      </div>
    );
  }
  
  return (
    <div className="protected-content">
      <h2>Welcome, {user.name}!</h2>
      <p>This is protected content</p>
    </div>
  );
};
```

**3. Role-Based Rendering**

```jsx
const Dashboard = ({ user }) => {
  const isAdmin = user.role === 'admin';
  const isModerator = user.role === 'moderator';
  const isUser = user.role === 'user';
  
  return (
    <div className="dashboard">
      <h1>Dashboard</h1>
      
      {/* All users see this */}
      <section className="profile">
        <h2>Your Profile</h2>
        <p>Name: {user.name}</p>
      </section>
      
      {/* Only moderators and admins */}
      {(isModerator || isAdmin) && (
        <section className="moderation">
          <h2>Moderation Tools</h2>
          <button>Review Reports</button>
        </section>
      )}
      
      {/* Only admins */}
      {isAdmin && (
        <section className="admin">
          <h2>Admin Panel</h2>
          <button>Manage Users</button>
          <button>System Settings</button>
        </section>
      )}
    </div>
  );
};
```

**4. Empty State**

```jsx
const TodoList = ({ todos }) => {
  if (todos.length === 0) {
    return (
      <div className="empty-state">
        <img src="/empty-icon.svg" alt="No todos" />
        <h2>No todos yet</h2>
        <p>Add your first todo to get started!</p>
        <button>Add Todo</button>
      </div>
    );
  }
  
  return (
    <ul className="todo-list">
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
};
```

**5. Error Boundary Fallback**

```jsx
const ErrorFallback = ({ error, resetError }) => {
  return (
    <div className="error-boundary">
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <button onClick={resetError}>Try Again</button>
    </div>
  );
};
```

**6. Feature Flags**

```jsx
const FeatureComponent = ({ features }) => {
  const hasNewFeature = features.includes('new-feature');
  const hasBetaAccess = features.includes('beta-access');
  
  return (
    <div>
      <h1>App</h1>
      
      {hasNewFeature && (
        <div className="new-feature">
          <h2>New Feature!</h2>
          <p>Check out our latest update</p>
        </div>
      )}
      
      {hasBetaAccess && (
        <div className="beta">
          <span className="badge">Beta Tester</span>
        </div>
      )}
    </div>
  );
};
```

**7. Conditional Styling**

```jsx
const Button = ({ isActive, isPrimary, isDisabled }) => {
  return (
    <button
      className={`
        btn
        ${isActive ? 'btn-active' : ''}
        ${isPrimary ? 'btn-primary' : 'btn-secondary'}
        ${isDisabled ? 'btn-disabled' : ''}
      `.trim()}
      disabled={isDisabled}
    >
      Click Me
    </button>
  );
};

// Using classnames library (popular solution)
import classNames from 'classnames';

const Button = ({ isActive, isPrimary, isDisabled }) => {
  const btnClass = classNames('btn', {
    'btn-active': isActive,
    'btn-primary': isPrimary,
    'btn-secondary': !isPrimary,
    'btn-disabled': isDisabled
  });
  
  return (
    <button className={btnClass} disabled={isDisabled}>
      Click Me
    </button>
  );
};
```

**8. Conditional Props**

```jsx
const Input = ({ isRequired, isDisabled, hasError }) => {
  return (
    <input
      type="text"
      {...(isRequired && { required: true })}
      {...(isDisabled && { disabled: true })}
      {...(hasError && { 'aria-invalid': true })}
      className={hasError ? 'input-error' : 'input'}
    />
  );
};
```

**Best Practices**

```jsx
// ✅ Good: Early returns for readability
const UserProfile = ({ user }) => {
  if (!user) {
    return <div>Loading...</div>;
  }
  
  if (user.banned) {
    return <div>Account suspended</div>;
  }
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
};

// ❌ Bad: Deeply nested ternaries
const Status = ({ status }) => (
  <div>
    {status === 'loading' ? (
      <Spinner />
    ) : status === 'error' ? (
      error ? (
        <Error message={error} />
      ) : (
        <DefaultError />
      )
    ) : (
      data ? (
        <Data data={data} />
      ) : (
        <Empty />
      )
    )}
  </div>
);

// ✅ Good: Extract to separate component or use if-else
const Status = ({ status, data, error }) => {
  if (status === 'loading') return <Spinner />;
  if (status === 'error') return error ? <Error message={error} /> : <DefaultError />;
  return data ? <Data data={data} /> : <Empty />;
};

// ✅ Good: Use meaningful variable names
const shouldShowWarning = user.age < 18 && !user.hasParentalConsent;
const canAccessPremiumFeatures = user.isPremium && user.emailVerified;

return (
  <div>
    {shouldShowWarning && <Warning />}
    {canAccessPremiumFeatures && <PremiumFeatures />}
  </div>
);

// ❌ Bad: Complex inline conditions
return (
  <div>
    {user.age < 18 && !user.hasParentalConsent && <Warning />}
    {user.isPremium && user.emailVerified && <PremiumFeatures />}
  </div>
);
```

**Production Example: Complete Data Fetching with States**

```jsx
import { useState, useEffect } from 'react';

const UserList = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [filter, setFilter] = useState('all');
  const [searchQuery, setSearchQuery] = useState('');
  
  useEffect(() => {
    const fetchUsers = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch('/api/users');
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
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
  
  // Loading state
  if (loading) {
    return (
      <div className="loading-container">
        <div className="spinner"></div>
        <p>Loading users...</p>
      </div>
    );
  }
  
  // Error state
  if (error) {
    return (
      <div className="error-container">
        <div className="error-icon">⚠️</div>
        <h2>Error Loading Users</h2>
        <p>{error}</p>
        <button onClick={() => window.location.reload()}>
          Try Again
        </button>
      </div>
    );
  }
  
  // Empty state
  if (users.length === 0) {
    return (
      <div className="empty-state">
        <div className="empty-icon">👥</div>
        <h2>No Users Found</h2>
        <p>There are no users in the system yet.</p>
        <button>Add First User</button>
      </div>
    );
  }
  
  // Filter users
  const filteredUsers = users.filter(user => {
    const matchesFilter = 
      filter === 'all' ||
      (filter === 'active' && user.isActive) ||
      (filter === 'inactive' && !user.isActive);
    
    const matchesSearch = 
      user.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
      user.email.toLowerCase().includes(searchQuery.toLowerCase());
    
    return matchesFilter && matchesSearch;
  });
  
  // Success state with data
  return (
    <div className="user-list-container">
      <div className="header">
        <h1>Users ({users.length})</h1>
        
        <div className="controls">
          <input
            type="text"
            placeholder="Search users..."
            value={searchQuery}
            onChange={(e) => setSearchQuery(e.target.value)}
            className="search-input"
          />
          
          <select 
            value={filter} 
            onChange={(e) => setFilter(e.target.value)}
            className="filter-select"
          >
            <option value="all">All Users</option>
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
          </select>
        </div>
      </div>
      
      {/* No results after filtering */}
      {filteredUsers.length === 0 ? (
        <div className="no-results">
          <p>No users match your search criteria</p>
          <button onClick={() => {
            setSearchQuery('');
            setFilter('all');
          }}>
            Clear Filters
          </button>
        </div>
      ) : (
        <div className="user-grid">
          {filteredUsers.map(user => (
            <div key={user.id} className="user-card">
              <img 
                src={user.avatar || '/default-avatar.png'} 
                alt={user.name}
                className="user-avatar"
              />
              
              <div className="user-info">
                <h3>{user.name}</h3>
                <p>{user.email}</p>
                
                {/* Conditional badge */}
                {user.isAdmin && (
                  <span className="badge badge-admin">Admin</span>
                )}
                
                {user.isPremium && (
                  <span className="badge badge-premium">Premium</span>
                )}
                
                {/* Status indicator */}
                <span className={`status ${user.isActive ? 'active' : 'inactive'}`}>
                  {user.isActive ? 'Active' : 'Inactive'}
                </span>
                
                {/* Conditional verification badge */}
                {user.emailVerified ? (
                  <span className="verified">✓ Verified</span>
                ) : (
                  <span className="unverified">⚠ Unverified</span>
                )}
              </div>
              
              <div className="user-actions">
                <button>View Profile</button>
                
                {/* Show edit button only for admins or own profile */}
                {(user.isAdmin || user.id === currentUserId) && (
                  <button>Edit</button>
                )}
                
                {/* Show delete button only for admins */}
                {isCurrentUserAdmin && user.id !== currentUserId && (
                  <button className="btn-danger">Delete</button>
                )}
              </div>
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

export default UserList;
```

**Interview Tips:**
- Conditional rendering shows different UI based on **conditions**
- Methods: **if-else**, **ternary** (?:), **logical AND** (&&), **switch**, **enum objects**
- Use **early returns** for better readability
- **&& operator** can render falsy values (0, '', false) - be careful!
- **Ternary** is good for simple if-else, avoid deep nesting
- Extract complex conditions to **variables** with meaningful names
- Common patterns: **loading**, **error**, **empty**, **authentication** states
- Use **null** or **false** to render nothing
- Mention **conditional styling** and **conditional props**
- Performance: Consider extracting to separate components for complex conditions

</details>

---

### 16. How do you render lists in React?

<details>
<summary>View Answer</summary>

**Rendering Lists in React**

Rendering lists is one of the most common tasks in React. You use JavaScript's **array methods** (primarily `.map()`) to transform arrays of data into arrays of JSX elements.

**Basic List Rendering**

```jsx
const SimpleList = () => {
  const fruits = ['Apple', 'Banana', 'Orange', 'Mango'];
  
  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  );
};

// Result:
// • Apple
// • Banana
// • Orange
// • Mango
```

**Why Use .map()?**

```jsx
// ❌ Don't do this (manual JSX)
const BadList = () => {
  const fruits = ['Apple', 'Banana', 'Orange'];
  
  return (
    <ul>
      <li>{fruits[0]}</li>
      <li>{fruits[1]}</li>
      <li>{fruits[2]}</li>
    </ul>
  );
};
// Problems:
// - Not scalable
// - Doesn't work with dynamic data
// - Tedious and error-prone

// ✅ Do this (using .map())
const GoodList = () => {
  const fruits = ['Apple', 'Banana', 'Orange'];
  
  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  );
};
// Benefits:
// - Scalable (works with any array size)
// - Dynamic (updates automatically)
// - Clean and maintainable
```

**Rendering Array of Objects**

```jsx
const UserList = () => {
  const users = [
    { id: 1, name: 'John Doe', email: 'john@example.com', age: 28 },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com', age: 32 },
    { id: 3, name: 'Bob Johnson', email: 'bob@example.com', age: 45 }
  ];
  
  return (
    <div className="user-list">
      {users.map(user => (
        <div key={user.id} className="user-card">
          <h3>{user.name}</h3>
          <p>Email: {user.email}</p>
          <p>Age: {user.age}</p>
        </div>
      ))}
    </div>
  );
};
```

**Key Prop is Required**

```jsx
// ⚠️ Warning: Each child should have a unique "key" prop
const NoKeyList = () => {
  const items = ['A', 'B', 'C'];
  
  return (
    <ul>
      {items.map(item => (
        <li>{item}</li>  {/* ❌ Missing key! */}
      ))}
    </ul>
  );
};

// ✅ Add key prop
const WithKeyList = () => {
  const items = ['A', 'B', 'C'];
  
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>  {/* ✅ Has key */}
      ))}
    </ul>
  );
};

// ✅ Better: Use unique ID if available
const BestKeyList = () => {
  const items = [
    { id: 'a1', name: 'Item A' },
    { id: 'b2', name: 'Item B' },
    { id: 'c3', name: 'Item C' }
  ];
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>  {/* ✅ Unique ID */}
      ))}
    </ul>
  );
};
```

**Different List Patterns**

**1. Simple Array of Primitives**
```jsx
const TagList = () => {
  const tags = ['React', 'JavaScript', 'TypeScript', 'Node.js'];
  
  return (
    <div className="tags">
      {tags.map((tag, index) => (
        <span key={index} className="tag">
          {tag}
        </span>
      ))}
    </div>
  );
};
```

**2. Array of Objects (Complex Data)**
```jsx
const ProductList = () => {
  const products = [
    { id: 101, name: 'Laptop', price: 999, inStock: true },
    { id: 102, name: 'Mouse', price: 29, inStock: true },
    { id: 103, name: 'Keyboard', price: 79, inStock: false }
  ];
  
  return (
    <div className="products">
      {products.map(product => (
        <div key={product.id} className="product-card">
          <h3>{product.name}</h3>
          <p className="price">${product.price}</p>
          <span className={`stock ${product.inStock ? 'in-stock' : 'out-of-stock'}`}>
            {product.inStock ? 'In Stock' : 'Out of Stock'}
          </span>
          <button disabled={!product.inStock}>
            {product.inStock ? 'Add to Cart' : 'Unavailable'}
          </button>
        </div>
      ))}
    </div>
  );
};
```

**3. Nested Lists**
```jsx
const NestedList = () => {
  const categories = [
    {
      id: 1,
      name: 'Fruits',
      items: ['Apple', 'Banana', 'Orange']
    },
    {
      id: 2,
      name: 'Vegetables',
      items: ['Carrot', 'Broccoli', 'Spinach']
    },
    {
      id: 3,
      name: 'Dairy',
      items: ['Milk', 'Cheese', 'Yogurt']
    }
  ];
  
  return (
    <div>
      {categories.map(category => (
        <div key={category.id} className="category">
          <h2>{category.name}</h2>
          <ul>
            {category.items.map((item, index) => (
              <li key={`${category.id}-${index}`}>{item}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
};
```

**4. List with Components**
```jsx
const UserCard = ({ user }) => (
  <div className="user-card">
    <img src={user.avatar} alt={user.name} />
    <h3>{user.name}</h3>
    <p>{user.email}</p>
  </div>
);

const UserGrid = () => {
  const users = [
    { id: 1, name: 'John', email: 'john@example.com', avatar: '/john.jpg' },
    { id: 2, name: 'Jane', email: 'jane@example.com', avatar: '/jane.jpg' },
    { id: 3, name: 'Bob', email: 'bob@example.com', avatar: '/bob.jpg' }
  ];
  
  return (
    <div className="user-grid">
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
};
```

**Array Methods for List Manipulation**

**1. Filter Before Mapping**
```jsx
const ActiveUserList = () => {
  const users = [
    { id: 1, name: 'John', isActive: true },
    { id: 2, name: 'Jane', isActive: false },
    { id: 3, name: 'Bob', isActive: true },
    { id: 4, name: 'Alice', isActive: false }
  ];
  
  return (
    <div>
      <h2>Active Users Only</h2>
      <ul>
        {users
          .filter(user => user.isActive)
          .map(user => (
            <li key={user.id}>{user.name}</li>
          ))}
      </ul>
    </div>
  );
};
// Result: John, Bob
```

**2. Sort Before Mapping**
```jsx
const SortedList = () => {
  const products = [
    { id: 1, name: 'Laptop', price: 999 },
    { id: 2, name: 'Mouse', price: 29 },
    { id: 3, name: 'Keyboard', price: 79 }
  ];
  
  return (
    <div>
      <h2>Products (Sorted by Price)</h2>
      <ul>
        {products
          .sort((a, b) => a.price - b.price)
          .map(product => (
            <li key={product.id}>
              {product.name} - ${product.price}
            </li>
          ))}
      </ul>
    </div>
  );
};
// Result: Mouse ($29), Keyboard ($79), Laptop ($999)
```

**3. Slice for Pagination**
```jsx
import { useState } from 'react';

const PaginatedList = () => {
  const [currentPage, setCurrentPage] = useState(1);
  const itemsPerPage = 5;
  
  const allItems = Array.from({ length: 23 }, (_, i) => ({
    id: i + 1,
    name: `Item ${i + 1}`
  }));
  
  const startIndex = (currentPage - 1) * itemsPerPage;
  const endIndex = startIndex + itemsPerPage;
  const currentItems = allItems.slice(startIndex, endIndex);
  const totalPages = Math.ceil(allItems.length / itemsPerPage);
  
  return (
    <div>
      <ul>
        {currentItems.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
      
      <div className="pagination">
        <button 
          onClick={() => setCurrentPage(prev => Math.max(1, prev - 1))}
          disabled={currentPage === 1}
        >
          Previous
        </button>
        
        <span>Page {currentPage} of {totalPages}</span>
        
        <button 
          onClick={() => setCurrentPage(prev => Math.min(totalPages, prev + 1))}
          disabled={currentPage === totalPages}
        >
          Next
        </button>
      </div>
    </div>
  );
};
```

**4. Chain Multiple Operations**
```jsx
const FilteredSortedList = () => {
  const products = [
    { id: 1, name: 'Laptop', price: 999, category: 'Electronics', inStock: true },
    { id: 2, name: 'Book', price: 15, category: 'Books', inStock: false },
    { id: 3, name: 'Mouse', price: 29, category: 'Electronics', inStock: true },
    { id: 4, name: 'Pen', price: 2, category: 'Stationery', inStock: true },
    { id: 5, name: 'Keyboard', price: 79, category: 'Electronics', inStock: false }
  ];
  
  return (
    <div>
      <h2>Electronics (In Stock, Sorted by Price)</h2>
      <ul>
        {products
          .filter(p => p.category === 'Electronics')  // Filter by category
          .filter(p => p.inStock)                      // Filter in stock
          .sort((a, b) => a.price - b.price)          // Sort by price
          .map(product => (
            <li key={product.id}>
              {product.name} - ${product.price}
            </li>
          ))}
      </ul>
    </div>
  );
};
// Result: Mouse ($29), Laptop ($999)
```

**Conditional Rendering in Lists**

```jsx
const ConditionalList = () => {
  const tasks = [
    { id: 1, title: 'Buy groceries', completed: false, priority: 'high' },
    { id: 2, title: 'Walk dog', completed: true, priority: 'medium' },
    { id: 3, title: 'Write code', completed: false, priority: 'high' },
    { id: 4, title: 'Read book', completed: false, priority: 'low' }
  ];
  
  return (
    <ul className="task-list">
      {tasks.map(task => (
        <li 
          key={task.id}
          className={`task ${task.completed ? 'completed' : 'pending'}`}
        >
          <span className="task-title">{task.title}</span>
          
          {/* Conditional priority badge */}
          {task.priority === 'high' && (
            <span className="badge badge-danger">High Priority</span>
          )}
          
          {/* Conditional completion status */}
          {task.completed ? (
            <span className="status-complete">✓ Completed</span>
          ) : (
            <span className="status-pending">⏳ Pending</span>
          )}
        </li>
      ))}
    </ul>
  );
};
```

**Empty List Handling**

```jsx
const SmartList = ({ items }) => {
  // Handle empty list
  if (items.length === 0) {
    return (
      <div className="empty-state">
        <p>No items to display</p>
        <button>Add First Item</button>
      </div>
    );
  }
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
};

// Alternative: Inline conditional
const SmartListInline = ({ items }) => {
  return (
    <div>
      {items.length === 0 ? (
        <div className="empty-state">
          <p>No items to display</p>
        </div>
      ) : (
        <ul>
          {items.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
};
```

**List with State Management**

```jsx
import { useState } from 'react';

const TodoList = () => {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build project', completed: false }
  ]);
  const [inputValue, setInputValue] = useState('');
  
  // Add new todo
  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos([
        ...todos,
        {
          id: Date.now(),
          text: inputValue,
          completed: false
        }
      ]);
      setInputValue('');
    }
  };
  
  // Toggle todo completion
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id
        ? { ...todo, completed: !todo.completed }
        : todo
    ));
  };
  
  // Delete todo
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  return (
    <div className="todo-app">
      <div className="input-group">
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="Add new todo..."
        />
        <button onClick={addTodo}>Add</button>
      </div>
      
      <ul className="todo-list">
        {todos.map(todo => (
          <li key={todo.id} className={todo.completed ? 'completed' : ''}>
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
    </div>
  );
};
```

**Lists with Events**

```jsx
import { useState } from 'react';

const SelectableList = () => {
  const [items] = useState([
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' },
    { id: 3, name: 'Item 3' }
  ]);
  const [selectedId, setSelectedId] = useState(null);
  
  const handleClick = (id) => {
    setSelectedId(id);
    console.log('Selected:', id);
  };
  
  return (
    <ul>
      {items.map(item => (
        <li
          key={item.id}
          onClick={() => handleClick(item.id)}
          className={selectedId === item.id ? 'selected' : ''}
          style={{
            cursor: 'pointer',
            backgroundColor: selectedId === item.id ? '#e0e0e0' : 'white'
          }}
        >
          {item.name}
        </li>
      ))}
    </ul>
  );
};
```

**Common Mistakes**

```jsx
// ❌ Mistake 1: Missing key prop
{items.map(item => <li>{item}</li>)}  // Warning!

// ✅ Fix: Add key
{items.map((item, index) => <li key={index}>{item}</li>)}

// ❌ Mistake 2: Using index as key (when order can change)
const [items, setItems] = useState(['A', 'B', 'C']);
{items.map((item, index) => <li key={index}>{item}</li>)}
// If items are reordered, React will get confused!

// ✅ Fix: Use unique ID
const [items, setItems] = useState([
  { id: 'a1', name: 'A' },
  { id: 'b2', name: 'B' },
  { id: 'c3', name: 'C' }
]);
{items.map(item => <li key={item.id}>{item.name}</li>)}

// ❌ Mistake 3: Mutating array directly
const addItem = () => {
  items.push(newItem);  // ❌ Mutates state!
  setItems(items);
};

// ✅ Fix: Create new array
const addItem = () => {
  setItems([...items, newItem]);  // ✅ New array
};

// ❌ Mistake 4: Forgetting to return in map
{items.map(item => {
  <li key={item.id}>{item.name}</li>  // ❌ No return!
})}

// ✅ Fix: Add return or use implicit return
{items.map(item => {
  return <li key={item.id}>{item.name}</li>;  // Explicit return
})}
// Or
{items.map(item => (
  <li key={item.id}>{item.name}</li>  // Implicit return with ()
))}

// ❌ Mistake 5: Wrapping map result in another array
{[items.map(item => <li key={item.id}>{item}</li>)]}  // ❌ Extra brackets

// ✅ Fix: Remove extra brackets
{items.map(item => <li key={item.id}>{item}</li>)}  // ✅ Correct
```

**Performance Optimization**

```jsx
import { useState, useMemo } from 'react';

const OptimizedList = ({ items }) => {
  const [searchQuery, setSearchQuery] = useState('');
  const [sortBy, setSortBy] = useState('name');
  
  // Memoize expensive filtering/sorting operations
  const processedItems = useMemo(() => {
    console.log('Processing items...');
    
    return items
      .filter(item =>
        item.name.toLowerCase().includes(searchQuery.toLowerCase())
      )
      .sort((a, b) => {
        if (sortBy === 'name') {
          return a.name.localeCompare(b.name);
        }
        return a.price - b.price;
      });
  }, [items, searchQuery, sortBy]);  // Only recompute when these change
  
  return (
    <div>
      <input
        value={searchQuery}
        onChange={(e) => setSearchQuery(e.target.value)}
        placeholder="Search..."
      />
      
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
        <option value="name">Sort by Name</option>
        <option value="price">Sort by Price</option>
      </select>
      
      <ul>
        {processedItems.map(item => (
          <li key={item.id}>
            {item.name} - ${item.price}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

**Production Example: Complete Product List**

```jsx
import { useState, useMemo } from 'react';

const ProductList = () => {
  const [products] = useState([
    { id: 1, name: 'Laptop', price: 999, category: 'Electronics', rating: 4.5, inStock: true },
    { id: 2, name: 'Mouse', price: 29, category: 'Electronics', rating: 4.2, inStock: true },
    { id: 3, name: 'Book', price: 15, category: 'Books', rating: 4.8, inStock: false },
    { id: 4, name: 'Desk', price: 299, category: 'Furniture', rating: 4.0, inStock: true },
    { id: 5, name: 'Chair', price: 199, category: 'Furniture', rating: 4.3, inStock: true },
    { id: 6, name: 'Keyboard', price: 79, category: 'Electronics', rating: 4.6, inStock: false }
  ]);
  
  const [filters, setFilters] = useState({
    category: 'all',
    inStockOnly: false,
    minRating: 0,
    searchQuery: ''
  });
  
  const [sortBy, setSortBy] = useState('name');
  const [sortOrder, setSortOrder] = useState('asc');
  
  // Process and filter products
  const filteredProducts = useMemo(() => {
    return products
      .filter(product => {
        // Category filter
        if (filters.category !== 'all' && product.category !== filters.category) {
          return false;
        }
        
        // Stock filter
        if (filters.inStockOnly && !product.inStock) {
          return false;
        }
        
        // Rating filter
        if (product.rating < filters.minRating) {
          return false;
        }
        
        // Search filter
        if (filters.searchQuery && !product.name.toLowerCase().includes(filters.searchQuery.toLowerCase())) {
          return false;
        }
        
        return true;
      })
      .sort((a, b) => {
        let comparison = 0;
        
        switch (sortBy) {
          case 'name':
            comparison = a.name.localeCompare(b.name);
            break;
          case 'price':
            comparison = a.price - b.price;
            break;
          case 'rating':
            comparison = a.rating - b.rating;
            break;
          default:
            comparison = 0;
        }
        
        return sortOrder === 'asc' ? comparison : -comparison;
      });
  }, [products, filters, sortBy, sortOrder]);
  
  const updateFilter = (key, value) => {
    setFilters(prev => ({ ...prev, [key]: value }));
  };
  
  return (
    <div className="product-list-container">
      <div className="filters">
        <input
          type="text"
          placeholder="Search products..."
          value={filters.searchQuery}
          onChange={(e) => updateFilter('searchQuery', e.target.value)}
          className="search-input"
        />
        
        <select
          value={filters.category}
          onChange={(e) => updateFilter('category', e.target.value)}
        >
          <option value="all">All Categories</option>
          <option value="Electronics">Electronics</option>
          <option value="Books">Books</option>
          <option value="Furniture">Furniture</option>
        </select>
        
        <label>
          <input
            type="checkbox"
            checked={filters.inStockOnly}
            onChange={(e) => updateFilter('inStockOnly', e.target.checked)}
          />
          In Stock Only
        </label>
        
        <div className="rating-filter">
          <label>Min Rating:</label>
          <input
            type="range"
            min="0"
            max="5"
            step="0.5"
            value={filters.minRating}
            onChange={(e) => updateFilter('minRating', parseFloat(e.target.value))}
          />
          <span>{filters.minRating}+</span>
        </div>
        
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="name">Sort by Name</option>
          <option value="price">Sort by Price</option>
          <option value="rating">Sort by Rating</option>
        </select>
        
        <button onClick={() => setSortOrder(sortOrder === 'asc' ? 'desc' : 'asc')}>
          {sortOrder === 'asc' ? '↑ Ascending' : '↓ Descending'}
        </button>
      </div>
      
      <div className="results-info">
        <p>Showing {filteredProducts.length} of {products.length} products</p>
      </div>
      
      {filteredProducts.length === 0 ? (
        <div className="no-results">
          <p>No products match your criteria</p>
          <button onClick={() => setFilters({ category: 'all', inStockOnly: false, minRating: 0, searchQuery: '' })}>
            Clear Filters
          </button>
        </div>
      ) : (
        <div className="product-grid">
          {filteredProducts.map(product => (
            <div key={product.id} className="product-card">
              <div className="product-header">
                <h3>{product.name}</h3>
                {!product.inStock && (
                  <span className="badge out-of-stock">Out of Stock</span>
                )}
              </div>
              
              <div className="product-details">
                <p className="category">{product.category}</p>
                <p className="price">${product.price}</p>
                
                <div className="rating">
                  <span className="stars">{'★'.repeat(Math.floor(product.rating))}</span>
                  <span className="rating-number">{product.rating}</span>
                </div>
              </div>
              
              <button 
                className="add-to-cart"
                disabled={!product.inStock}
              >
                {product.inStock ? 'Add to Cart' : 'Unavailable'}
              </button>
            </div>
          ))}
        </div>
      )}
    </div>
  );
};

export default ProductList;
```

**Interview Tips:**
- Use **`.map()`** to transform arrays into JSX elements
- Always provide **unique `key` prop** for each list item
- Prefer **stable IDs** over array indices for keys
- Can **chain array methods**: filter → sort → map
- Handle **empty lists** with conditional rendering
- Use **useMemo** for expensive list operations
- Common pattern: **filter + sort + map**
- Don't forget to **return** JSX from map callback
- Avoid **mutating arrays** - use spread operator or array methods
- Mention **keys help React identify changes** efficiently

</details>

---

### 17. What is the purpose of keys in React lists?

<details>
<summary>View Answer</summary>

**What are Keys in React?**

**Keys** are special string attributes that help React identify which items in a list have changed, been added, or been removed. They give elements a stable identity and help React optimize rendering performance.

**Why Keys are Important**

```jsx
// Without keys (problematic)
const List = ({ items }) => (
  <ul>
    {items.map(item => <li>{item}</li>)}  {/* ⚠️ Warning! */}
  </ul>
);

// With keys (correct)
const List = ({ items }) => (
  <ul>
    {items.map((item, index) => <li key={index}>{item}</li>)}  {/* ✅ Good */}
  </ul>
);
```

**What Keys Do**

1. **Help React identify elements** - React uses keys to determine which items have changed
2. **Optimize re-rendering** - Keys allow React to reuse existing DOM elements
3. **Preserve component state** - Keys maintain component state across re-renders
4. **Enable efficient updates** - React can add/remove/reorder items efficiently

**How React Uses Keys**

**Without Keys: React Re-renders Everything**
```jsx
// Initial render:
<ul>
  <li>Apple</li>
  <li>Banana</li>
  <li>Orange</li>
</ul>

// After adding 'Mango' at the beginning:
<ul>
  <li>Mango</li>   {/* React recreates this */}
  <li>Apple</li>   {/* React recreates this */}
  <li>Banana</li>  {/* React recreates this */}
  <li>Orange</li>  {/* React recreates this */}
</ul>
// React doesn't know items shifted, so it recreates ALL elements!
```

**With Keys: React Updates Efficiently**
```jsx
// Initial render:
<ul>
  <li key="apple">Apple</li>
  <li key="banana">Banana</li>
  <li key="orange">Orange</li>
</ul>

// After adding 'Mango' at the beginning:
<ul>
  <li key="mango">Mango</li>    {/* React creates ONLY this */}
  <li key="apple">Apple</li>    {/* React REUSES existing element */}
  <li key="banana">Banana</li>  {/* React REUSES existing element */}
  <li key="orange">Orange</li>  {/* React REUSES existing element */}
</ul>
// React knows these are the same items, just reordered!
```

**Key Requirements**

**1. Keys Must Be Unique Among Siblings**
```jsx
// ✅ Correct: Unique keys
const GoodList = () => {
  const items = [
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Orange' }
  ];
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
};

// ❌ Wrong: Duplicate keys
const BadList = () => {
  const items = ['Apple', 'Banana', 'Apple'];
  
  return (
    <ul>
      {items.map(item => (
        <li key={item}>{item}</li>  {/* 'Apple' appears twice! */}
      ))}
    </ul>
  );
};

// ✅ Fix: Use index when items might duplicate
const FixedList = () => {
  const items = ['Apple', 'Banana', 'Apple'];
  
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
};
```

**2. Keys Must Be Stable**
```jsx
// ❌ Wrong: Random keys
{items.map(item => (
  <li key={Math.random()}>{item}</li>  // New key every render!
))}

// ❌ Wrong: Keys that change
{items.map(item => (
  <li key={Date.now()}>{item}</li>  // New key every render!
))}

// ✅ Correct: Stable, unique keys
{items.map(item => (
  <li key={item.id}>{item.name}</li>  // Same key across renders
))}
```

**3. Keys Don't Need to Be Globally Unique**
```jsx
const TwoLists = () => {
  const fruits = [
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' }
  ];
  
  const vegetables = [
    { id: 1, name: 'Carrot' },  // Same ID as Apple - OK!
    { id: 2, name: 'Broccoli' }  // Same ID as Banana - OK!
  ];
  
  return (
    <div>
      <ul>
        {fruits.map(fruit => (
          <li key={fruit.id}>{fruit.name}</li>
        ))}
      </ul>
      
      <ul>
        {vegetables.map(veg => (
          <li key={veg.id}>{veg.name}</li>  {/* OK - different list */}
        ))}
      </ul>
    </div>
  );
};
```

**Choosing the Right Key**

**1. Best: Use Unique ID from Data**
```jsx
// ✅ Best practice: Database ID, UUID, etc.
const users = [
  { id: 'user-123', name: 'John' },
  { id: 'user-456', name: 'Jane' },
  { id: 'user-789', name: 'Bob' }
];

<ul>
  {users.map(user => (
    <li key={user.id}>{user.name}</li>
  ))}
</ul>
```

**2. Good: Generate Stable ID**
```jsx
// ✅ Good: Composite key from multiple fields
const items = [
  { firstName: 'John', lastName: 'Doe', age: 30 },
  { firstName: 'Jane', lastName: 'Smith', age: 25 }
];

<ul>
  {items.map(item => (
    <li key={`${item.firstName}-${item.lastName}`}>
      {item.firstName} {item.lastName}
    </li>
  ))}
</ul>
```

**3. Acceptable: Use Index (with Cautions)**
```jsx
// ⚠️ OK if list is static or never reordered
const staticItems = ['Red', 'Green', 'Blue'];

<ul>
  {staticItems.map((item, index) => (
    <li key={index}>{item}</li>
  ))}
</ul>

// ❌ BAD if list can be reordered, filtered, or have items added/removed
```

**When Index as Key is Problematic**

**Problem 1: Reordering Items**
```jsx
import { useState } from 'react';

const ReorderDemo = () => {
  const [items, setItems] = useState(['A', 'B', 'C']);
  
  const reverse = () => {
    setItems([...items].reverse());
  };
  
  // ❌ Using index as key
  return (
    <div>
      <button onClick={reverse}>Reverse</button>
      <ul>
        {items.map((item, index) => (
          <li key={index}>
            {item} <input type="text" />
          </li>
        ))}
      </ul>
    </div>
  );
};

// Problem:
// Before reverse: A(key=0), B(key=1), C(key=2)
// After reverse:  C(key=0), B(key=1), A(key=2)
// React thinks items are the same (same keys), just content changed
// Input values get mixed up!

// ✅ Fix: Use item value or unique ID as key
return (
  <ul>
    {items.map(item => (
      <li key={item}>
        {item} <input type="text" />
      </li>
    ))}
  </ul>
);
```

**Problem 2: Inserting Items**
```jsx
const InsertDemo = () => {
  const [items, setItems] = useState(['A', 'B', 'C']);
  
  const insertAtStart = () => {
    setItems(['NEW', ...items]);
  };
  
  // ❌ Using index as key
  return (
    <div>
      <button onClick={insertAtStart}>Insert at Start</button>
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
};

// Problem:
// Before: A(key=0), B(key=1), C(key=2)
// After:  NEW(key=0), A(key=1), B(key=2), C(key=3)
// React updates ALL items instead of just inserting one!
```

**Problem 3: Filtering Items**
```jsx
const FilterDemo = () => {
  const [filter, setFilter] = useState('');
  const allItems = [
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Cherry' }
  ];
  
  const filteredItems = allItems.filter(item =>
    item.name.toLowerCase().includes(filter.toLowerCase())
  );
  
  // ❌ Using index as key
  return (
    <div>
      <input onChange={(e) => setFilter(e.target.value)} />
      <ul>
        {filteredItems.map((item, index) => (
          <li key={index}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
};

// Problem:
// All items: Apple(key=0), Banana(key=1), Cherry(key=2)
// Filtered ("an"): Banana(key=0), (wrong key!)
// React might reuse wrong DOM elements

// ✅ Fix: Use item ID
return (
  <ul>
    {filteredItems.map(item => (
      <li key={item.id}>{item.name}</li>
    ))}
  </ul>
);
```

**Keys and Component State**

```jsx
import { useState } from 'react';

const Counter = ({ name }) => {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <span>{name}: </span>
      <button onClick={() => setCount(count + 1)}>{count}</button>
    </div>
  );
};

const KeyStateDemo = () => {
  const [items, setItems] = useState(['A', 'B', 'C']);
  
  const shuffle = () => {
    setItems([...items].sort(() => Math.random() - 0.5));
  };
  
  return (
    <div>
      <button onClick={shuffle}>Shuffle</button>
      
      {/* ❌ Using index as key - state gets mixed up */}
      <div>
        <h3>Wrong (index as key):</h3>
        {items.map((item, index) => (
          <Counter key={index} name={item} />
        ))}
      </div>
      
      {/* ✅ Using item as key - state follows item */}
      <div>
        <h3>Correct (item as key):</h3>
        {items.map(item => (
          <Counter key={item} name={item} />
        ))}
      </div>
    </div>
  );
};

// With index keys: Click counters, then shuffle
// → Count values don't move with items!
// With item keys: Click counters, then shuffle
// → Count values move with their items ✓
```

**Generating Keys**

**1. From Existing Data**
```jsx
// Use database ID
const users = [
  { id: 123, name: 'John' },
  { id: 456, name: 'Jane' }
];

{users.map(user => <User key={user.id} data={user} />)}
```

**2. Generate on Data Creation**
```jsx
import { useState } from 'react';

const TodoList = () => {
  const [todos, setTodos] = useState([]);
  
  const addTodo = (text) => {
    setTodos([
      ...todos,
      {
        id: Date.now(), // Simple unique ID
        text,
        completed: false
      }
    ]);
  };
  
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
};
```

**3. Using UUID Library**
```jsx
import { v4 as uuidv4 } from 'uuid';

const addItem = (name) => {
  setItems([
    ...items,
    {
      id: uuidv4(), // Guaranteed unique ID
      name
    }
  ]);
};

{items.map(item => <Item key={item.id} data={item} />)}
```

**4. Composite Keys**
```jsx
// Multiple fields that together are unique
const orders = [
  { customerId: 1, orderDate: '2024-01-01', productId: 'A' },
  { customerId: 1, orderDate: '2024-01-02', productId: 'B' },
  { customerId: 2, orderDate: '2024-01-01', productId: 'A' }
];

{orders.map(order => (
  <Order 
    key={`${order.customerId}-${order.orderDate}-${order.productId}`}
    data={order}
  />
))}
```

**Keys in Nested Lists**

```jsx
const NestedList = () => {
  const categories = [
    {
      id: 'cat-1',
      name: 'Fruits',
      items: [
        { id: 'fruit-1', name: 'Apple' },
        { id: 'fruit-2', name: 'Banana' }
      ]
    },
    {
      id: 'cat-2',
      name: 'Vegetables',
      items: [
        { id: 'veg-1', name: 'Carrot' },
        { id: 'veg-2', name: 'Broccoli' }
      ]
    }
  ];
  
  return (
    <div>
      {categories.map(category => (
        <div key={category.id}>  {/* Key for outer list */}
          <h2>{category.name}</h2>
          <ul>
            {category.items.map(item => (
              <li key={item.id}>  {/* Key for inner list */}
                {item.name}
              </li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
};
```

**Keys Don't Get Passed to Components**

```jsx
const Item = ({ id, name }) => {
  console.log('Props:', { id, name });
  // 'key' is NOT in props!
  
  return <div>{name}</div>;
};

const List = () => {
  const items = [
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' }
  ];
  
  return (
    <div>
      {items.map(item => (
        <Item 
          key={item.id}      // Used by React, not passed to component
          id={item.id}       // This IS passed to component
          name={item.name}
        />
      ))}
    </div>
  );
};

// If you need the key value in the component, pass it as a separate prop
```

**Common Mistakes**

```jsx
// ❌ Mistake 1: No key at all
{items.map(item => <li>{item}</li>)}  // Warning!

// ❌ Mistake 2: Non-unique keys
{items.map(item => <li key="same">{item}</li>)}  // All same key!

// ❌ Mistake 3: Unstable keys
{items.map(item => <li key={Math.random()}>{item}</li>)}  // Changes every render!

// ❌ Mistake 4: Using index when order changes
const [items, setItems] = useState(['A', 'B', 'C']);
const reverse = () => setItems([...items].reverse());
{items.map((item, index) => <li key={index}>{item}</li>)}  // Bad!

// ❌ Mistake 5: Concatenating index with non-unique value
{items.map((item, index) => (
  <li key={index + item}>{item}</li>  // 'Apple' appears twice = duplicate keys!
))}

// ✅ Correct patterns:
// 1. Use unique ID from data
{items.map(item => <li key={item.id}>{item.name}</li>)}

// 2. Generate stable ID when creating items
const addItem = () => {
  setItems([...items, { id: Date.now(), name: 'New' }]);
};

// 3. Use index ONLY if list is static and won't reorder
{['Red', 'Green', 'Blue'].map((color, i) => <li key={i}>{color}</li>)}
```

**Performance Impact**

```jsx
// Proper keys improve performance

// ❌ Without keys: React recreates all elements
const SlowList = ({ items }) => (
  <ul>
    {items.map(item => <ExpensiveComponent data={item} />)}
  </ul>
);
// When items change, EVERY component remounts (slow!)

// ✅ With keys: React reuses existing elements
const FastList = ({ items }) => (
  <ul>
    {items.map(item => (
      <ExpensiveComponent key={item.id} data={item} />
    ))}
  </ul>
);
// When items change, only affected components update (fast!)
```

**Production Example: Dynamic Todo List with Proper Keys**

```jsx
import { useState } from 'react';

const TodoApp = () => {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false, createdAt: Date.now() - 3600000 },
    { id: 2, text: 'Build project', completed: false, createdAt: Date.now() - 1800000 },
    { id: 3, text: 'Deploy app', completed: false, createdAt: Date.now() }
  ]);
  const [inputValue, setInputValue] = useState('');
  const [sortBy, setSortBy] = useState('created'); // 'created' or 'text'
  const [filterBy, setFilterBy] = useState('all'); // 'all', 'active', 'completed'
  
  // Generate unique ID for new todos
  const generateId = () => {
    return Date.now() + Math.random(); // Simple unique ID
  };
  
  // Add todo
  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos([
        ...todos,
        {
          id: generateId(),
          text: inputValue,
          completed: false,
          createdAt: Date.now()
        }
      ]);
      setInputValue('');
    }
  };
  
  // Toggle todo
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id
        ? { ...todo, completed: !todo.completed }
        : todo
    ));
  };
  
  // Delete todo
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  // Edit todo
  const editTodo = (id, newText) => {
    setTodos(todos.map(todo =>
      todo.id === id
        ? { ...todo, text: newText }
        : todo
    ));
  };
  
  // Filter todos
  const getFilteredTodos = () => {
    switch (filterBy) {
      case 'active':
        return todos.filter(todo => !todo.completed);
      case 'completed':
        return todos.filter(todo => todo.completed);
      default:
        return todos;
    }
  };
  
  // Sort todos
  const getSortedTodos = (filteredTodos) => {
    const sorted = [...filteredTodos];
    
    if (sortBy === 'text') {
      sorted.sort((a, b) => a.text.localeCompare(b.text));
    } else {
      sorted.sort((a, b) => b.createdAt - a.createdAt);
    }
    
    return sorted;
  };
  
  const displayedTodos = getSortedTodos(getFilteredTodos());
  
  return (
    <div className="todo-app">
      <h1>Todo List</h1>
      
      <div className="input-section">
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="Add new todo..."
        />
        <button onClick={addTodo}>Add</button>
      </div>
      
      <div className="controls">
        <div>
          <label>Filter: </label>
          <select value={filterBy} onChange={(e) => setFilterBy(e.target.value)}>
            <option value="all">All</option>
            <option value="active">Active</option>
            <option value="completed">Completed</option>
          </select>
        </div>
        
        <div>
          <label>Sort by: </label>
          <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
            <option value="created">Date Created</option>
            <option value="text">Alphabetical</option>
          </select>
        </div>
      </div>
      
      <div className="stats">
        <span>Total: {todos.length}</span>
        <span>Active: {todos.filter(t => !t.completed).length}</span>
        <span>Completed: {todos.filter(t => t.completed).length}</span>
      </div>
      
      {displayedTodos.length === 0 ? (
        <p className="empty">No todos to display</p>
      ) : (
        <ul className="todo-list">
          {displayedTodos.map(todo => (
            <TodoItem
              key={todo.id}  // ✅ Stable, unique key
              todo={todo}
              onToggle={toggleTodo}
              onDelete={deleteTodo}
              onEdit={editTodo}
            />
          ))}
        </ul>
      )}
    </div>
  );
};

const TodoItem = ({ todo, onToggle, onDelete, onEdit }) => {
  const [isEditing, setIsEditing] = useState(false);
  const [editText, setEditText] = useState(todo.text);
  
  const handleSave = () => {
    if (editText.trim()) {
      onEdit(todo.id, editText);
      setIsEditing(false);
    }
  };
  
  return (
    <li className={`todo-item ${todo.completed ? 'completed' : ''}`}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      
      {isEditing ? (
        <input
          value={editText}
          onChange={(e) => setEditText(e.target.value)}
          onBlur={handleSave}
          onKeyPress={(e) => e.key === 'Enter' && handleSave()}
          autoFocus
        />
      ) : (
        <span onClick={() => setIsEditing(true)}>{todo.text}</span>
      )}
      
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
};

export default TodoApp;
```

**Interview Tips:**
- Keys help React **identify which items changed**
- Keys must be **unique among siblings**
- Keys should be **stable** (not random or based on Date.now() unless during creation)
- Best practice: Use **unique ID from data** (database ID, UUID)
- Avoid **index as key** when list can be reordered/filtered/modified
- Index is **OK for static lists** that never change
- Keys are **not passed as props** to components
- Keys enable React to **preserve component state** correctly
- Without keys, React may **recreate all DOM elements** (poor performance)
- Mention **key reconciliation algorithm** helps optimize rendering

</details>

---

### 18. How do you handle forms in React?

<details>
<summary>View Answer</summary>

**Form Handling in React**

Forms in React are handled differently than traditional HTML forms. React uses **controlled components** to manage form state, giving you full control over form data and validation.

**Traditional HTML Form (Uncontrolled)**

```html
<!-- Plain HTML -->
<form>
  <input type="text" name="username" />
  <button type="submit">Submit</button>
</form>

<script>
  // DOM manages the state
  const input = document.querySelector('input');
  console.log(input.value); // Access value from DOM
</script>
```

**React Form (Controlled)**

```jsx
import { useState } from 'react';

const LoginForm = () => {
  const [username, setUsername] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault(); // Prevent page reload
    console.log('Username:', username);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={username}  // React state controls value
        onChange={(e) => setUsername(e.target.value)}  // Update state on change
      />
      <button type="submit">Submit</button>
    </form>
  );
};

// React state is the "single source of truth"
```

**Basic Form with Single Input**

```jsx
import { useState } from 'react';

const SimpleForm = () => {
  const [name, setName] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    alert(`Hello, ${name}!`);
    setName(''); // Clear form
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <label>
        Name:
        <input
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder="Enter your name"
        />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
};
```

**Form with Multiple Inputs**

**Approach 1: Multiple State Variables**

```jsx
const MultiInputForm = () => {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ firstName, lastName, email, age });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={firstName}
        onChange={(e) => setFirstName(e.target.value)}
        placeholder="First Name"
      />
      
      <input
        type="text"
        value={lastName}
        onChange={(e) => setLastName(e.target.value)}
        placeholder="Last Name"
      />
      
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      
      <input
        type="number"
        value={age}
        onChange={(e) => setAge(e.target.value)}
        placeholder="Age"
      />
      
      <button type="submit">Submit</button>
    </form>
  );
};
```

**Approach 2: Single State Object (Recommended)**

```jsx
const BetterMultiInputForm = () => {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    age: ''
  });
  
  // Single handler for all inputs
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prevData => ({
      ...prevData,
      [name]: value  // Dynamic property name
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form Data:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="firstName"
        type="text"
        value={formData.firstName}
        onChange={handleChange}
        placeholder="First Name"
      />
      
      <input
        name="lastName"
        type="text"
        value={formData.lastName}
        onChange={handleChange}
        placeholder="Last Name"
      />
      
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      
      <input
        name="age"
        type="number"
        value={formData.age}
        onChange={handleChange}
        placeholder="Age"
      />
      
      <button type="submit">Submit</button>
    </form>
  );
};
```

**Different Input Types**

**1. Text Input**
```jsx
const [text, setText] = useState('');

<input
  type="text"
  value={text}
  onChange={(e) => setText(e.target.value)}
/>
```

**2. Textarea**
```jsx
const [message, setMessage] = useState('');

<textarea
  value={message}
  onChange={(e) => setMessage(e.target.value)}
  rows={5}
  placeholder="Enter your message..."
/>
```

**3. Select Dropdown**
```jsx
const [country, setCountry] = useState('');

<select value={country} onChange={(e) => setCountry(e.target.value)}>
  <option value="">Select Country</option>
  <option value="usa">USA</option>
  <option value="uk">UK</option>
  <option value="canada">Canada</option>
</select>
```

**4. Radio Buttons**
```jsx
const [gender, setGender] = useState('');

<div>
  <label>
    <input
      type="radio"
      name="gender"
      value="male"
      checked={gender === 'male'}
      onChange={(e) => setGender(e.target.value)}
    />
    Male
  </label>
  
  <label>
    <input
      type="radio"
      name="gender"
      value="female"
      checked={gender === 'female'}
      onChange={(e) => setGender(e.target.value)}
    />
    Female
  </label>
  
  <label>
    <input
      type="radio"
      name="gender"
      value="other"
      checked={gender === 'other'}
      onChange={(e) => setGender(e.target.value)}
    />
    Other
  </label>
</div>
```

**5. Checkbox (Single)**
```jsx
const [agreed, setAgreed] = useState(false);

<label>
  <input
    type="checkbox"
    checked={agreed}
    onChange={(e) => setAgreed(e.target.checked)}  // Note: e.target.checked
  />
  I agree to terms and conditions
</label>
```

**6. Checkboxes (Multiple)**
```jsx
const [hobbies, setHobbies] = useState([]);

const handleCheckboxChange = (e) => {
  const { value, checked } = e.target;
  
  if (checked) {
    setHobbies([...hobbies, value]); // Add to array
  } else {
    setHobbies(hobbies.filter(hobby => hobby !== value)); // Remove from array
  }
};

<div>
  <label>
    <input
      type="checkbox"
      value="reading"
      checked={hobbies.includes('reading')}
      onChange={handleCheckboxChange}
    />
    Reading
  </label>
  
  <label>
    <input
      type="checkbox"
      value="sports"
      checked={hobbies.includes('sports')}
      onChange={handleCheckboxChange}
    />
    Sports
  </label>
  
  <label>
    <input
      type="checkbox"
      value="music"
      checked={hobbies.includes('music')}
      onChange={handleCheckboxChange}
    />
    Music
  </label>
</div>

// Selected: {hobbies.join(', ')}
```

**7. File Input**
```jsx
const [file, setFile] = useState(null);

const handleFileChange = (e) => {
  const selectedFile = e.target.files[0];
  setFile(selectedFile);
  console.log('Selected:', selectedFile.name, selectedFile.size);
};

<input
  type="file"
  onChange={handleFileChange}
  accept="image/*"  // Only images
/>

{file && <p>Selected: {file.name}</p>}
```

**8. Date Input**
```jsx
const [date, setDate] = useState('');

<input
  type="date"
  value={date}
  onChange={(e) => setDate(e.target.value)}
/>
```

**Form Validation**

**1. Basic Validation**
```jsx
const FormWithValidation = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  
  const validate = () => {
    const newErrors = {};
    
    if (!email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!password) {
      newErrors.password = 'Password is required';
    } else if (password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    if (validate()) {
      console.log('Form is valid!', { email, password });
      // Submit to API
    } else {
      console.log('Form has errors');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Email"
          className={errors.email ? 'error' : ''}
        />
        {errors.email && <span className="error-text">{errors.email}</span>}
      </div>
      
      <div>
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          placeholder="Password"
          className={errors.password ? 'error' : ''}
        />
        {errors.password && <span className="error-text">{errors.password}</span>}
      </div>
      
      <button type="submit">Submit</button>
    </form>
  );
};
```

**2. Real-time Validation**
```jsx
const RealtimeValidation = () => {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: ''
  });
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Validate on change
    validateField(name, value);
  };
  
  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
  };
  
  const validateField = (name, value) => {
    let error = '';
    
    switch (name) {
      case 'username':
        if (!value) error = 'Username is required';
        else if (value.length < 3) error = 'Username must be at least 3 characters';
        break;
      
      case 'email':
        if (!value) error = 'Email is required';
        else if (!/\S+@\S+\.\S+/.test(value)) error = 'Email is invalid';
        break;
      
      case 'password':
        if (!value) error = 'Password is required';
        else if (value.length < 8) error = 'Must be at least 8 characters';
        else if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
          error = 'Must contain uppercase, lowercase, and number';
        }
        break;
    }
    
    setErrors(prev => ({ ...prev, [name]: error }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Mark all as touched
    setTouched({ username: true, email: true, password: true });
    
    // Validate all fields
    Object.keys(formData).forEach(key => {
      validateField(key, formData[key]);
    });
    
    // Check if no errors
    if (Object.values(errors).every(error => !error)) {
      console.log('Form submitted:', formData);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          name="username"
          value={formData.username}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Username"
        />
        {touched.username && errors.username && (
          <span className="error">{errors.username}</span>
        )}
      </div>
      
      <div>
        <input
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Email"
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>
      
      <div>
        <input
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Password"
        />
        {touched.password && errors.password && (
          <span className="error">{errors.password}</span>
        )}
      </div>
      
      <button type="submit">Register</button>
    </form>
  );
};
```

**Form Submission**

**1. Simple Submission**
```jsx
const handleSubmit = (e) => {
  e.preventDefault(); // Prevent default form submission
  
  console.log('Form data:', formData);
  
  // Process data
  alert('Form submitted!');
};
```

**2. Async Submission with API**
```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  setIsSubmitting(true);
  setError(null);
  
  try {
    const response = await fetch('/api/submit', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(formData),
    });
    
    if (!response.ok) {
      throw new Error('Submission failed');
    }
    
    const data = await response.json();
    console.log('Success:', data);
    
    // Reset form
    setFormData({ name: '', email: '' });
    alert('Form submitted successfully!');
    
  } catch (error) {
    setError(error.message);
    console.error('Error:', error);
  } finally {
    setIsSubmitting(false);
  }
};
```

**3. FormData API (for file uploads)**
```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  
  const formData = new FormData();
  formData.append('name', name);
  formData.append('email', email);
  formData.append('file', file); // File object
  
  try {
    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData, // Don't set Content-Type header
    });
    
    const data = await response.json();
    console.log('Success:', data);
  } catch (error) {
    console.error('Error:', error);
  }
};
```

**Form Reset**

```jsx
const ContactForm = () => {
  const initialState = {
    name: '',
    email: '',
    message: ''
  };
  
  const [formData, setFormData] = useState(initialState);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitted:', formData);
    
    // Reset to initial state
    setFormData(initialState);
  };
  
  const handleReset = () => {
    setFormData(initialState);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      <button type="submit">Submit</button>
      <button type="button" onClick={handleReset}>Reset</button>
    </form>
  );
};
```

**Custom Form Hook (Reusable)**

```jsx
import { useState } from 'react';

// Custom hook for form handling
const useForm = (initialValues, validate) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    const newValue = type === 'checkbox' ? checked : value;
    
    setValues(prev => ({
      ...prev,
      [name]: newValue
    }));
    
    // Clear error when user types
    if (errors[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: ''
      }));
    }
  };
  
  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({
      ...prev,
      [name]: true
    }));
  };
  
  const handleSubmit = async (onSubmit) => (e) => {
    e.preventDefault();
    
    // Mark all as touched
    const allTouched = Object.keys(values).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {});
    setTouched(allTouched);
    
    // Validate
    const validationErrors = validate ? validate(values) : {};
    setErrors(validationErrors);
    
    // Submit if no errors
    if (Object.keys(validationErrors).length === 0) {
      setIsSubmitting(true);
      try {
        await onSubmit(values);
      } finally {
        setIsSubmitting(false);
      }
    }
  };
  
  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
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
};

// Usage
const LoginForm = () => {
  const validate = (values) => {
    const errors = {};
    
    if (!values.email) {
      errors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(values.email)) {
      errors.email = 'Email is invalid';
    }
    
    if (!values.password) {
      errors.password = 'Password is required';
    } else if (values.password.length < 8) {
      errors.password = 'Password must be at least 8 characters';
    }
    
    return errors;
  };
  
  const { 
    values, 
    errors, 
    touched, 
    isSubmitting,
    handleChange, 
    handleBlur, 
    handleSubmit,
    reset
  } = useForm(
    { email: '', password: '' },
    validate
  );
  
  const onSubmit = async (formData) => {
    console.log('Submitting:', formData);
    // API call here
    await new Promise(resolve => setTimeout(resolve, 1000));
    alert('Login successful!');
    reset();
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <input
          name="email"
          type="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Email"
        />
        {touched.email && errors.email && (
          <span className="error">{errors.email}</span>
        )}
      </div>
      
      <div>
        <input
          name="password"
          type="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="Password"
        />
        {touched.password && errors.password && (
          <span className="error">{errors.password}</span>
        )}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

**Common Form Patterns**

**1. Multi-step Form**
```jsx
const MultiStepForm = () => {
  const [step, setStep] = useState(1);
  const [formData, setFormData] = useState({
    // Step 1
    name: '',
    email: '',
    // Step 2
    address: '',
    city: '',
    // Step 3
    cardNumber: '',
    cvv: ''
  });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const nextStep = () => setStep(step + 1);
  const prevStep = () => setStep(step - 1);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Final data:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {step === 1 && (
        <div>
          <h2>Step 1: Personal Info</h2>
          <input name="name" value={formData.name} onChange={handleChange} placeholder="Name" />
          <input name="email" value={formData.email} onChange={handleChange} placeholder="Email" />
          <button type="button" onClick={nextStep}>Next</button>
        </div>
      )}
      
      {step === 2 && (
        <div>
          <h2>Step 2: Address</h2>
          <input name="address" value={formData.address} onChange={handleChange} placeholder="Address" />
          <input name="city" value={formData.city} onChange={handleChange} placeholder="City" />
          <button type="button" onClick={prevStep}>Back</button>
          <button type="button" onClick={nextStep}>Next</button>
        </div>
      )}
      
      {step === 3 && (
        <div>
          <h2>Step 3: Payment</h2>
          <input name="cardNumber" value={formData.cardNumber} onChange={handleChange} placeholder="Card Number" />
          <input name="cvv" value={formData.cvv} onChange={handleChange} placeholder="CVV" />
          <button type="button" onClick={prevStep}>Back</button>
          <button type="submit">Submit</button>
        </div>
      )}
    </form>
  );
};
```

**2. Dynamic Form Fields**
```jsx
const DynamicFieldsForm = () => {
  const [fields, setFields] = useState([{ id: 1, value: '' }]);
  
  const addField = () => {
    setFields([...fields, { id: Date.now(), value: '' }]);
  };
  
  const removeField = (id) => {
    setFields(fields.filter(field => field.id !== id));
  };
  
  const handleFieldChange = (id, value) => {
    setFields(fields.map(field =>
      field.id === id ? { ...field, value } : field
    ));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Values:', fields.map(f => f.value));
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Add Multiple Emails</h2>
      {fields.map(field => (
        <div key={field.id}>
          <input
            type="email"
            value={field.value}
            onChange={(e) => handleFieldChange(field.id, e.target.value)}
            placeholder="Email"
          />
          {fields.length > 1 && (
            <button type="button" onClick={() => removeField(field.id)}>
              Remove
            </button>
          )}
        </div>
      ))}
      <button type="button" onClick={addField}>Add Email</button>
      <button type="submit">Submit</button>
    </form>
  );
};
```

**Production Example: Complete Registration Form**

```jsx
import { useState } from 'react';

const RegistrationForm = () => {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    password: '',
    confirmPassword: '',
    phone: '',
    country: '',
    gender: '',
    hobbies: [],
    agreeToTerms: false,
    newsletter: false
  });
  
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitSuccess, setSubmitSuccess] = useState(false);
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    
    if (type === 'checkbox') {
      if (name === 'hobbies') {
        const updatedHobbies = checked
          ? [...formData.hobbies, value]
          : formData.hobbies.filter(h => h !== value);
        setFormData(prev => ({ ...prev, hobbies: updatedHobbies }));
      } else {
        setFormData(prev => ({ ...prev, [name]: checked }));
      }
    } else {
      setFormData(prev => ({ ...prev, [name]: value }));
    }
    
    // Clear error when user types
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };
  
  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    validateField(name, formData[name]);
  };
  
  const validateField = (name, value) => {
    let error = '';
    
    switch (name) {
      case 'firstName':
        if (!value.trim()) error = 'First name is required';
        break;
      
      case 'lastName':
        if (!value.trim()) error = 'Last name is required';
        break;
      
      case 'email':
        if (!value.trim()) {
          error = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(value)) {
          error = 'Email is invalid';
        }
        break;
      
      case 'password':
        if (!value) {
          error = 'Password is required';
        } else if (value.length < 8) {
          error = 'Password must be at least 8 characters';
        } else if (!/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/.test(value)) {
          error = 'Must contain uppercase, lowercase, and number';
        }
        break;
      
      case 'confirmPassword':
        if (!value) {
          error = 'Please confirm your password';
        } else if (value !== formData.password) {
          error = 'Passwords do not match';
        }
        break;
      
      case 'phone':
        if (!value.trim()) {
          error = 'Phone is required';
        } else if (!/^\d{10}$/.test(value.replace(/\D/g, ''))) {
          error = 'Phone must be 10 digits';
        }
        break;
      
      case 'country':
        if (!value) error = 'Please select a country';
        break;
      
      case 'gender':
        if (!value) error = 'Please select gender';
        break;
      
      case 'agreeToTerms':
        if (!formData.agreeToTerms) error = 'You must agree to terms';
        break;
    }
    
    if (error) {
      setErrors(prev => ({ ...prev, [name]: error }));
    }
  };
  
  const validateForm = () => {
    const newErrors = {};
    
    if (!formData.firstName.trim()) newErrors.firstName = 'First name is required';
    if (!formData.lastName.trim()) newErrors.lastName = 'Last name is required';
    
    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    
    if (formData.password !== formData.confirmPassword) {
      newErrors.confirmPassword = 'Passwords do not match';
    }
    
    if (!formData.phone.trim()) {
      newErrors.phone = 'Phone is required';
    }
    
    if (!formData.country) newErrors.country = 'Please select a country';
    if (!formData.gender) newErrors.gender = 'Please select gender';
    if (!formData.agreeToTerms) newErrors.agreeToTerms = 'You must agree to terms';
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Mark all fields as touched
    const allTouched = Object.keys(formData).reduce((acc, key) => {
      acc[key] = true;
      return acc;
    }, {});
    setTouched(allTouched);
    
    if (!validateForm()) {
      console.log('Form has errors');
      return;
    }
    
    setIsSubmitting(true);
    
    try {
      const response = await fetch('/api/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      if (!response.ok) throw new Error('Registration failed');
      
      const data = await response.json();
      console.log('Success:', data);
      
      setSubmitSuccess(true);
      
      // Reset form
      setFormData({
        firstName: '',
        lastName: '',
        email: '',
        password: '',
        confirmPassword: '',
        phone: '',
        country: '',
        gender: '',
        hobbies: [],
        agreeToTerms: false,
        newsletter: false
      });
      setTouched({});
      
    } catch (error) {
      setErrors({ submit: error.message });
    } finally {
      setIsSubmitting(false);
    }
  };
  
  if (submitSuccess) {
    return (
      <div className="success-message">
        <h2>✓ Registration Successful!</h2>
        <p>Please check your email to verify your account.</p>
      </div>
    );
  }
  
  return (
    <form onSubmit={handleSubmit} className="registration-form">
      <h1>Create Account</h1>
      
      {errors.submit && (
        <div className="error-banner">{errors.submit}</div>
      )}
      
      <div className="form-row">
        <div className="form-group">
          <label htmlFor="firstName">First Name *</label>
          <input
            id="firstName"
            name="firstName"
            type="text"
            value={formData.firstName}
            onChange={handleChange}
            onBlur={handleBlur}
            className={touched.firstName && errors.firstName ? 'error' : ''}
          />
          {touched.firstName && errors.firstName && (
            <span className="error-text">{errors.firstName}</span>
          )}
        </div>
        
        <div className="form-group">
          <label htmlFor="lastName">Last Name *</label>
          <input
            id="lastName"
            name="lastName"
            type="text"
            value={formData.lastName}
            onChange={handleChange}
            onBlur={handleBlur}
            className={touched.lastName && errors.lastName ? 'error' : ''}
          />
          {touched.lastName && errors.lastName && (
            <span className="error-text">{errors.lastName}</span>
          )}
        </div>
      </div>
      
      <div className="form-group">
        <label htmlFor="email">Email *</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.email && errors.email ? 'error' : ''}
        />
        {touched.email && errors.email && (
          <span className="error-text">{errors.email}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="password">Password *</label>
        <input
          id="password"
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.password && errors.password ? 'error' : ''}
        />
        {touched.password && errors.password && (
          <span className="error-text">{errors.password}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="confirmPassword">Confirm Password *</label>
        <input
          id="confirmPassword"
          name="confirmPassword"
          type="password"
          value={formData.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.confirmPassword && errors.confirmPassword ? 'error' : ''}
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span className="error-text">{errors.confirmPassword}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="phone">Phone *</label>
        <input
          id="phone"
          name="phone"
          type="tel"
          value={formData.phone}
          onChange={handleChange}
          onBlur={handleBlur}
          placeholder="1234567890"
          className={touched.phone && errors.phone ? 'error' : ''}
        />
        {touched.phone && errors.phone && (
          <span className="error-text">{errors.phone}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="country">Country *</label>
        <select
          id="country"
          name="country"
          value={formData.country}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.country && errors.country ? 'error' : ''}
        >
          <option value="">Select Country</option>
          <option value="usa">United States</option>
          <option value="uk">United Kingdom</option>
          <option value="canada">Canada</option>
          <option value="india">India</option>
        </select>
        {touched.country && errors.country && (
          <span className="error-text">{errors.country}</span>
        )}
      </div>
      
      <div className="form-group">
        <label>Gender *</label>
        <div className="radio-group">
          <label>
            <input
              type="radio"
              name="gender"
              value="male"
              checked={formData.gender === 'male'}
              onChange={handleChange}
            />
            Male
          </label>
          <label>
            <input
              type="radio"
              name="gender"
              value="female"
              checked={formData.gender === 'female'}
              onChange={handleChange}
            />
            Female
          </label>
          <label>
            <input
              type="radio"
              name="gender"
              value="other"
              checked={formData.gender === 'other'}
              onChange={handleChange}
            />
            Other
          </label>
        </div>
        {touched.gender && errors.gender && (
          <span className="error-text">{errors.gender}</span>
        )}
      </div>
      
      <div className="form-group">
        <label>Hobbies</label>
        <div className="checkbox-group">
          <label>
            <input
              type="checkbox"
              name="hobbies"
              value="reading"
              checked={formData.hobbies.includes('reading')}
              onChange={handleChange}
            />
            Reading
          </label>
          <label>
            <input
              type="checkbox"
              name="hobbies"
              value="sports"
              checked={formData.hobbies.includes('sports')}
              onChange={handleChange}
            />
            Sports
          </label>
          <label>
            <input
              type="checkbox"
              name="hobbies"
              value="music"
              checked={formData.hobbies.includes('music')}
              onChange={handleChange}
            />
            Music
          </label>
        </div>
      </div>
      
      <div className="form-group">
        <label>
          <input
            type="checkbox"
            name="agreeToTerms"
            checked={formData.agreeToTerms}
            onChange={handleChange}
          />
          I agree to the Terms and Conditions *
        </label>
        {touched.agreeToTerms && errors.agreeToTerms && (
          <span className="error-text">{errors.agreeToTerms}</span>
        )}
      </div>
      
      <div className="form-group">
        <label>
          <input
            type="checkbox"
            name="newsletter"
            checked={formData.newsletter}
            onChange={handleChange}
          />
          Subscribe to newsletter
        </label>
      </div>
      
      <button type="submit" disabled={isSubmitting} className="submit-button">
        {isSubmitting ? 'Creating Account...' : 'Create Account'}
      </button>
    </form>
  );
};

export default RegistrationForm;
```

**Interview Tips:**
- React uses **controlled components** - state is single source of truth
- Always use **e.preventDefault()** in submit handler to prevent page reload
- Use **single state object** for multiple form fields with dynamic key updates
- Handle different inputs: text, textarea, select, radio, **checkbox** (use `checked`), file
- Implement **validation** - basic, real-time, or on blur
- Track **touched fields** to show errors only after user interaction
- Use **async/await** for API submissions
- Can create **custom hooks** (useForm) for reusable form logic
- Common patterns: multi-step forms, dynamic fields, file uploads
- Popular libraries: **Formik**, **React Hook Form** for complex forms

</details>

---

### 19. What are controlled vs uncontrolled components?

<details>
<summary>View Answer</summary>

**Controlled vs Uncontrolled Components**

The difference between controlled and uncontrolled components relates to **who manages the form input's state**.

**Controlled Components**

In controlled components, React state is the **single source of truth**. The component controls the input value.

**Key Characteristics:**
- Input value is controlled by React state
- Every change triggers a state update
- Value flows from state → input
- React has full control over input data

**Basic Example:**
```jsx
import { useState } from 'react';

const ControlledInput = () => {
  const [value, setValue] = useState('');
  
  return (
    <div>
      <input
        type="text"
        value={value}  // React state controls the value
        onChange={(e) => setValue(e.target.value)}  // Update state on change
      />
      <p>Current value: {value}</p>
    </div>
  );
};
```

**Uncontrolled Components**

In uncontrolled components, the **DOM manages the state**. React doesn't control the input value.

**Key Characteristics:**
- DOM holds the input value
- Use `ref` to access value when needed
- No state updates on every keystroke
- Similar to traditional HTML forms

**Basic Example:**
```jsx
import { useRef } from 'react';

const UncontrolledInput = () => {
  const inputRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Value:', inputRef.current.value);  // Access DOM directly
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        ref={inputRef}  // Ref to access DOM element
        defaultValue=""  // Use defaultValue, not value
      />
      <button type="submit">Submit</button>
    </form>
  );
};
```

**Key Differences**

| Aspect | Controlled | Uncontrolled |
|--------|-----------|-------------|
| **State Management** | React state | DOM |
| **Value Access** | `value` prop from state | `ref.current.value` |
| **Initial Value** | `value={state}` | `defaultValue="..."` |
| **Updates** | Every keystroke | Only when accessed |
| **Validation** | Real-time, easy | Manual, on submit |
| **Performance** | Re-renders on change | No re-renders |
| **Data Flow** | One-way (state → input) | Input manages itself |
| **React Control** | Full control | Limited control |

**Controlled Component Examples**

**1. Text Input**
```jsx
const [name, setName] = useState('');

<input
  type="text"
  value={name}
  onChange={(e) => setName(e.target.value)}
/>
```

**2. Textarea**
```jsx
const [message, setMessage] = useState('');

<textarea
  value={message}
  onChange={(e) => setMessage(e.target.value)}
/>
```

**3. Select**
```jsx
const [country, setCountry] = useState('usa');

<select value={country} onChange={(e) => setCountry(e.target.value)}>
  <option value="usa">USA</option>
  <option value="uk">UK</option>
</select>
```

**4. Checkbox**
```jsx
const [agreed, setAgreed] = useState(false);

<input
  type="checkbox"
  checked={agreed}
  onChange={(e) => setAgreed(e.target.checked)}
/>
```

**5. Radio Buttons**
```jsx
const [gender, setGender] = useState('');

<input
  type="radio"
  name="gender"
  value="male"
  checked={gender === 'male'}
  onChange={(e) => setGender(e.target.value)}
/>
```

**Uncontrolled Component Examples**

**1. Text Input with Ref**
```jsx
import { useRef } from 'react';

const UncontrolledForm = () => {
  const nameRef = useRef(null);
  const emailRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({
      name: nameRef.current.value,
      email: emailRef.current.value
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={nameRef}
        type="text"
        defaultValue="John"  // Initial value
      />
      <input
        ref={emailRef}
        type="email"
        defaultValue="john@example.com"
      />
      <button type="submit">Submit</button>
    </form>
  );
};
```

**2. File Input (Always Uncontrolled)**
```jsx
const FileUpload = () => {
  const fileRef = useRef(null);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const file = fileRef.current.files[0];
    console.log('Selected file:', file.name, file.size);
    
    // Upload file
    const formData = new FormData();
    formData.append('file', file);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={fileRef}
        type="file"
        accept="image/*"
      />
      <button type="submit">Upload</button>
    </form>
  );
};

// Note: File inputs are ALWAYS uncontrolled
// You cannot set the value of a file input programmatically for security reasons
```

**3. Clearing Uncontrolled Input**
```jsx
const ClearableInput = () => {
  const inputRef = useRef(null);
  
  const handleClear = () => {
    inputRef.current.value = '';  // Directly manipulate DOM
    inputRef.current.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" defaultValue="Clear me" />
      <button onClick={handleClear}>Clear</button>
    </div>
  );
};
```

**When to Use Controlled Components**

**Use controlled when you need:**

1. **Real-time validation**
```jsx
const [email, setEmail] = useState('');
const [error, setError] = useState('');

const handleChange = (e) => {
  const value = e.target.value;
  setEmail(value);
  
  // Validate in real-time
  if (!value.includes('@')) {
    setError('Email must contain @');
  } else {
    setError('');
  }
};

<input value={email} onChange={handleChange} />
{error && <span className="error">{error}</span>}
```

2. **Input formatting**
```jsx
const [phone, setPhone] = useState('');

const handleChange = (e) => {
  let value = e.target.value.replace(/\D/g, ''); // Remove non-digits
  
  // Format as (123) 456-7890
  if (value.length > 6) {
    value = `(${value.slice(0, 3)}) ${value.slice(3, 6)}-${value.slice(6, 10)}`;
  } else if (value.length > 3) {
    value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
  }
  
  setPhone(value);
};

<input value={phone} onChange={handleChange} placeholder="(123) 456-7890" />
```

3. **Conditional enabling/disabling**
```jsx
const [username, setUsername] = useState('');
const [password, setPassword] = useState('');

<form>
  <input value={username} onChange={(e) => setUsername(e.target.value)} />
  <input value={password} onChange={(e) => setPassword(e.target.value)} />
  <button disabled={!username || !password}>  // Disable if empty
    Login
  </button>
</form>
```

4. **Dynamic form values**
```jsx
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');

useEffect(() => {
  setFullName(`${firstName} ${lastName}`.trim());
}, [firstName, lastName]);

<input value={firstName} onChange={(e) => setFirstName(e.target.value)} />
<input value={lastName} onChange={(e) => setLastName(e.target.value)} />
<input value={fullName} readOnly />  // Derived value
```

5. **Character count/limits**
```jsx
const [text, setText] = useState('');
const maxLength = 100;

const handleChange = (e) => {
  const value = e.target.value;
  if (value.length <= maxLength) {
    setText(value);
  }
};

<textarea value={text} onChange={handleChange} />
<p>{text.length}/{maxLength} characters</p>
```

**When to Use Uncontrolled Components**

**Use uncontrolled when:**

1. **Simple forms** - No real-time validation needed
2. **Performance critical** - Avoid re-renders on every keystroke
3. **Integrating non-React code** - Working with jQuery or other libraries
4. **File uploads** - File inputs are always uncontrolled
5. **Quick prototypes** - Less boilerplate code
6. **Large forms** - Where not all fields need validation

**Example: Simple Contact Form (Uncontrolled)**
```jsx
const SimpleContactForm = () => {
  const nameRef = useRef();
  const emailRef = useRef();
  const messageRef = useRef();
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    const formData = {
      name: nameRef.current.value,
      email: emailRef.current.value,
      message: messageRef.current.value
    };
    
    console.log('Submitting:', formData);
    
    // Send to API
    fetch('/api/contact', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    });
    
    // Reset form
    e.target.reset();
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} name="name" placeholder="Name" />
      <input ref={emailRef} name="email" type="email" placeholder="Email" />
      <textarea ref={messageRef} name="message" placeholder="Message" />
      <button type="submit">Send</button>
    </form>
  );
};
```

**Hybrid Approach**

You can mix controlled and uncontrolled components:

```jsx
const HybridForm = () => {
  // Controlled - needs validation
  const [email, setEmail] = useState('');
  const [emailError, setEmailError] = useState('');
  
  // Uncontrolled - simple inputs
  const nameRef = useRef();
  const phoneRef = useRef();
  const fileRef = useRef();  // File inputs must be uncontrolled
  
  const validateEmail = (value) => {
    if (!value.includes('@')) {
      setEmailError('Invalid email');
      return false;
    }
    setEmailError('');
    return true;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    if (!validateEmail(email)) return;
    
    const formData = {
      name: nameRef.current.value,
      email: email,  // From state
      phone: phoneRef.current.value,
      file: fileRef.current.files[0]
    };
    
    console.log('Submit:', formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Uncontrolled */}
      <input ref={nameRef} placeholder="Name" />
      
      {/* Controlled - needs validation */}
      <div>
        <input
          type="email"
          value={email}
          onChange={(e) => {
            setEmail(e.target.value);
            validateEmail(e.target.value);
          }}
          placeholder="Email"
        />
        {emailError && <span className="error">{emailError}</span>}
      </div>
      
      {/* Uncontrolled */}
      <input ref={phoneRef} placeholder="Phone" />
      
      {/* Always uncontrolled */}
      <input ref={fileRef} type="file" />
      
      <button type="submit">Submit</button>
    </form>
  );
};
```

**Converting Between Controlled and Uncontrolled**

**❌ Common Mistake: Switching Between Controlled and Uncontrolled**
```jsx
// BAD: Starting as uncontrolled (undefined), then controlled
const [value, setValue] = useState();  // undefined!

<input
  value={value}  // undefined initially (uncontrolled)
  onChange={(e) => setValue(e.target.value)}  // Then becomes controlled
/>

// Warning: A component is changing an uncontrolled input to be controlled
```

**✅ Solution: Always Initialize State**
```jsx
// GOOD: Always controlled
const [value, setValue] = useState('');  // Empty string, not undefined

<input
  value={value}
  onChange={(e) => setValue(e.target.value)}
/>
```

**Production Example: Login Form Comparison**

**Controlled Version:**
```jsx
import { useState } from 'react';

const ControlledLoginForm = () => {
  const [formData, setFormData] = useState({ email: '', password: '' });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Real-time validation
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };
  
  const validate = () => {
    const newErrors = {};
    
    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    
    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validate()) return;
    
    setIsSubmitting(true);
    
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      const data = await response.json();
      console.log('Login success:', data);
      
    } catch (error) {
      setErrors({ submit: 'Login failed' });
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Login (Controlled)</h2>
      
      <div>
        <input
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <input
          name="password"
          type="password"
          value={formData.password}
          onChange={handleChange}
          placeholder="Password"
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>
      
      {errors.submit && <div className="error">{errors.submit}</div>}
      
      <button type="submit" disabled={isSubmitting || !formData.email || !formData.password}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
      
      <p>Email: {formData.email}</p>
      <p>Password length: {formData.password.length}</p>
    </form>
  );
};
```

**Uncontrolled Version:**
```jsx
import { useRef, useState } from 'react';

const UncontrolledLoginForm = () => {
  const emailRef = useRef();
  const passwordRef = useRef();
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    
    const email = emailRef.current.value;
    const password = passwordRef.current.value;
    
    // Validation on submit only
    if (!email || !password) {
      setError('All fields are required');
      return;
    }
    
    if (!/\S+@\S+\.\S+/.test(email)) {
      setError('Email is invalid');
      return;
    }
    
    if (password.length < 8) {
      setError('Password must be at least 8 characters');
      return;
    }
    
    setIsSubmitting(true);
    
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password })
      });
      
      const data = await response.json();
      console.log('Login success:', data);
      
      // Clear form
      e.target.reset();
      
    } catch (error) {
      setError('Login failed');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <h2>Login (Uncontrolled)</h2>
      
      <input
        ref={emailRef}
        type="email"
        name="email"
        placeholder="Email"
        defaultValue=""
      />
      
      <input
        ref={passwordRef}
        type="password"
        name="password"
        placeholder="Password"
        defaultValue=""
      />
      
      {error && <div className="error">{error}</div>}
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
};
```

**Comparison Summary:**

| Feature | Controlled | Uncontrolled |
|---------|-----------|-------------|
| **Code** | More verbose | Less code |
| **Validation** | Real-time | On submit only |
| **Performance** | Re-renders on change | No re-renders |
| **Control** | Full React control | Limited control |
| **Best for** | Complex forms, validation | Simple forms, quick prototypes |
| **React way** | ✅ Recommended | ⚠️ Use sparingly |

**Interview Tips:**
- **Controlled**: React state controls input value, updates on every change
- **Uncontrolled**: DOM manages state, access via `ref` when needed
- Use **controlled** for validation, formatting, dynamic values (React's recommended approach)
- Use **uncontrolled** for simple forms, file uploads, or performance reasons
- **Never switch** between controlled/uncontrolled - always initialize state
- **File inputs** are always uncontrolled (security reasons)
- Controlled uses `value` prop; uncontrolled uses `defaultValue` prop
- Controlled enables real-time validation; uncontrolled validates on submit
- Can use **hybrid approach** - controlled for complex fields, uncontrolled for simple ones
- Most React apps prefer **controlled components** for consistency and predictability

</details>

---

### 20. How do you pass data from child to parent component?

<details>
<summary>View Answer</summary>

**Passing Data from Child to Parent**

In React, data flows **one-way (downward)** from parent to child through props. To pass data **upward** from child to parent, you pass a **callback function** from parent to child.

**The Pattern:**
1. Parent defines a function
2. Parent passes function to child as a prop
3. Child calls the function with data
4. Parent receives the data

**Basic Example**

```jsx
import { useState } from 'react';

// Child Component
const ChildComponent = ({ onSendData }) => {
  const sendDataToParent = () => {
    const data = 'Hello from Child!';
    onSendData(data);  // Call parent's function
  };
  
  return (
    <button onClick={sendDataToParent}>
      Send Data to Parent
    </button>
  );
};

// Parent Component
const ParentComponent = () => {
  const [dataFromChild, setDataFromChild] = useState('');
  
  const handleDataFromChild = (data) => {
    console.log('Received from child:', data);
    setDataFromChild(data);
  };
  
  return (
    <div>
      <h2>Parent Component</h2>
      <p>Data from child: {dataFromChild}</p>
      
      <ChildComponent onSendData={handleDataFromChild} />
    </div>
  );
};
```

**How It Works:**
```
1. Parent creates function: handleDataFromChild
2. Parent passes to child: onSendData={handleDataFromChild}
3. Child receives: onSendData as prop
4. Child calls: onSendData('data')
5. Parent's function executes: handleDataFromChild('data')
6. Parent updates state: setDataFromChild('data')
```

**Input Example: Passing User Input**

```jsx
// Child: Input Component
const InputField = ({ onInputChange }) => {
  const handleChange = (e) => {
    onInputChange(e.target.value);  // Send value to parent
  };
  
  return (
    <input
      type="text"
      onChange={handleChange}
      placeholder="Type something..."
    />
  );
};

// Parent Component
const ParentWithInput = () => {
  const [inputValue, setInputValue] = useState('');
  
  return (
    <div>
      <h2>Parent</h2>
      <p>You typed: {inputValue}</p>
      
      <InputField onInputChange={setInputValue} />  
      {/* Pass setState directly */}
    </div>
  );
};
```

**Form Example: Submit Data from Child**

```jsx
// Child: Form Component
const UserForm = ({ onSubmit }) => {
  const [formData, setFormData] = useState({ name: '', email: '' });
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);  // Send form data to parent
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Name"
      />
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <button type="submit">Submit</button>
    </form>
  );
};

// Parent Component
const ParentWithForm = () => {
  const [userData, setUserData] = useState(null);
  
  const handleFormSubmit = (data) => {
    console.log('Form submitted:', data);
    setUserData(data);
    // Can also send to API here
  };
  
  return (
    <div>
      <h2>User Registration</h2>
      
      <UserForm onSubmit={handleFormSubmit} />
      
      {userData && (
        <div>
          <h3>Submitted Data:</h3>
          <p>Name: {userData.name}</p>
          <p>Email: {userData.email}</p>
        </div>
      )}
    </div>
  );
};
```

**Passing Multiple Parameters**

```jsx
// Child Component
const ProductCard = ({ product, onAddToCart }) => {
  const handleClick = () => {
    // Pass multiple values
    onAddToCart(product.id, product.name, product.price);
  };
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={handleClick}>Add to Cart</button>
    </div>
  );
};

// Parent Component
const ShoppingPage = () => {
  const [cart, setCart] = useState([]);
  
  const handleAddToCart = (id, name, price) => {
    const item = { id, name, price, quantity: 1 };
    setCart(prev => [...prev, item]);
    console.log('Added to cart:', item);
  };
  
  const products = [
    { id: 1, name: 'Laptop', price: 999 },
    { id: 2, name: 'Phone', price: 599 }
  ];
  
  return (
    <div>
      <h2>Products</h2>
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={handleAddToCart}
        />
      ))}
      
      <h3>Cart ({cart.length} items)</h3>
    </div>
  );
};
```

**Passing Event Objects**

```jsx
// Child Component
const CustomButton = ({ onClick, label }) => {
  const handleClick = (e) => {
    // Pass event object to parent
    onClick(e);
  };
  
  return <button onClick={handleClick}>{label}</button>;
};

// Parent Component
const ParentWithEvent = () => {
  const handleButtonClick = (event) => {
    console.log('Button clicked!');
    console.log('Event type:', event.type);
    console.log('Target:', event.target);
    event.stopPropagation();  // Can manipulate event in parent
  };
  
  return (
    <div onClick={() => console.log('Div clicked')}>
      <CustomButton onClick={handleButtonClick} label="Click Me" />
    </div>
  );
};
```

**Lifting State Up Pattern**

When multiple children need to share the same state, "lift the state up" to their common parent.

```jsx
import { useState } from 'react';

// Child 1: Temperature Input
const TemperatureInput = ({ scale, temperature, onTemperatureChange }) => {
  const handleChange = (e) => {
    onTemperatureChange(e.target.value);
  };
  
  return (
    <fieldset>
      <legend>Enter temperature in {scale}:</legend>
      <input value={temperature} onChange={handleChange} />
    </fieldset>
  );
};

// Child 2: Boiling Verdict
const BoilingVerdict = ({ celsius }) => {
  if (celsius >= 100) {
    return <p>The water would boil.</p>;
  }
  return <p>The water would not boil.</p>;
};

// Parent: Manages Shared State
const TemperatureCalculator = () => {
  const [temperature, setTemperature] = useState('');
  const [scale, setScale] = useState('c');
  
  const handleCelsiusChange = (temp) => {
    setScale('c');
    setTemperature(temp);
  };
  
  const handleFahrenheitChange = (temp) => {
    setScale('f');
    setTemperature(temp);
  };
  
  const toCelsius = (fahrenheit) => {
    return ((fahrenheit - 32) * 5) / 9;
  };
  
  const toFahrenheit = (celsius) => {
    return (celsius * 9) / 5 + 32;
  };
  
  const celsius = scale === 'f' ? toCelsius(temperature) : temperature;
  const fahrenheit = scale === 'c' ? toFahrenheit(temperature) : temperature;
  
  return (
    <div>
      <TemperatureInput
        scale="Celsius"
        temperature={celsius}
        onTemperatureChange={handleCelsiusChange}
      />
      
      <TemperatureInput
        scale="Fahrenheit"
        temperature={fahrenheit}
        onTemperatureChange={handleFahrenheitChange}
      />
      
      <BoilingVerdict celsius={parseFloat(celsius)} />
    </div>
  );
};
```

**Passing Data Through Multiple Levels**

**Approach 1: Prop Drilling (Not Recommended for Deep Nesting)**
```jsx
// Grandchild Component
const GrandchildInput = ({ onDataChange }) => {
  return (
    <input
      onChange={(e) => onDataChange(e.target.value)}
      placeholder="Type here..."
    />
  );
};

// Child Component (Middle Layer)
const ChildWrapper = ({ onDataChange }) => {
  return (
    <div>
      <h3>Child Component</h3>
      <GrandchildInput onDataChange={onDataChange} />  
      {/* Just passing through */}
    </div>
  );
};

// Parent Component
const GrandparentComponent = () => {
  const [data, setData] = useState('');
  
  return (
    <div>
      <h2>Grandparent Component</h2>
      <p>Data: {data}</p>
      
      <ChildWrapper onDataChange={setData} />
    </div>
  );
};
```

**Approach 2: Context API (Better for Deep Nesting)**
```jsx
import { createContext, useContext, useState } from 'react';

// Create Context
const DataContext = createContext();

// Deeply Nested Child
const DeeplyNestedChild = () => {
  const { updateData } = useContext(DataContext);
  
  return (
    <input
      onChange={(e) => updateData(e.target.value)}
      placeholder="Type here..."
    />
  );
};

// Middle Components (Don't need to know about data)
const MiddleComponent = () => {
  return (
    <div>
      <h3>Middle Component</h3>
      <DeeplyNestedChild />
    </div>
  );
};

// Parent Component (Provides Context)
const RootComponent = () => {
  const [data, setData] = useState('');
  
  return (
    <DataContext.Provider value={{ data, updateData: setData }}>
      <div>
        <h2>Root Component</h2>
        <p>Data: {data}</p>
        <MiddleComponent />
      </div>
    </DataContext.Provider>
  );
};
```

**Todo List Example: Complete Pattern**

```jsx
import { useState } from 'react';

// Child 1: Add Todo Form
const AddTodoForm = ({ onAddTodo }) => {
  const [inputValue, setInputValue] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      onAddTodo(inputValue);  // Send new todo to parent
      setInputValue('');  // Clear input
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="Add new todo..."
      />
      <button type="submit">Add</button>
    </form>
  );
};

// Child 2: Todo Item
const TodoItem = ({ todo, onToggle, onDelete }) => {
  return (
    <div className="todo-item">
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}  // Send id to parent
      />
      <span
        style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
      >
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </div>
  );
};

// Child 3: Todo List
const TodoList = ({ todos, onToggle, onDelete }) => {
  return (
    <div className="todo-list">
      {todos.map(todo => (
        <TodoItem
          key={todo.id}
          todo={todo}
          onToggle={onToggle}  // Pass callbacks down
          onDelete={onDelete}
        />
      ))}
    </div>
  );
};

// Parent: Manages All State
const TodoApp = () => {
  const [todos, setTodos] = useState([]);
  
  const addTodo = (text) => {
    const newTodo = {
      id: Date.now(),
      text,
      completed: false
    };
    setTodos([...todos, newTodo]);
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };
  
  return (
    <div className="todo-app">
      <h1>Todo List</h1>
      
      <AddTodoForm onAddTodo={addTodo} />
      
      <TodoList
        todos={todos}
        onToggle={toggleTodo}
        onDelete={deleteTodo}
      />
      
      <div className="stats">
        <p>Total: {todos.length}</p>
        <p>Completed: {todos.filter(t => t.completed).length}</p>
      </div>
    </div>
  );
};

export default TodoApp;
```

**Custom Hook Pattern**

Create reusable logic for parent-child communication:

```jsx
import { useState } from 'react';

// Custom Hook
const useChildData = (initialValue = null) => {
  const [data, setData] = useState(initialValue);
  const [history, setHistory] = useState([]);
  
  const handleChildData = (newData) => {
    setData(newData);
    setHistory(prev => [...prev, newData]);
  };
  
  const clearData = () => {
    setData(initialValue);
  };
  
  return {
    data,
    history,
    handleChildData,
    clearData
  };
};

// Child Component
const DataSender = ({ onSend }) => {
  const [input, setInput] = useState('');
  
  const handleSend = () => {
    onSend(input);
    setInput('');
  };
  
  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Enter data"
      />
      <button onClick={handleSend}>Send</button>
    </div>
  );
};

// Parent Component Using Custom Hook
const ParentUsingHook = () => {
  const { data, history, handleChildData, clearData } = useChildData();
  
  return (
    <div>
      <h2>Parent Component</h2>
      <p>Current data: {data}</p>
      <p>History: {history.join(', ')}</p>
      
      <DataSender onSend={handleChildData} />
      
      <button onClick={clearData}>Clear</button>
    </div>
  );
};
```

**Production Example: Search Filter**

```jsx
import { useState } from 'react';

// Child: Search Input
const SearchBar = ({ onSearch, placeholder }) => {
  const [query, setQuery] = useState('');
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    onSearch(value);  // Send search query to parent in real-time
  };
  
  const handleClear = () => {
    setQuery('');
    onSearch('');
  };
  
  return (
    <div className="search-bar">
      <input
        type="text"
        value={query}
        onChange={handleChange}
        placeholder={placeholder}
      />
      {query && (
        <button onClick={handleClear} className="clear-btn">
          ✕
        </button>
      )}
    </div>
  );
};

// Child: Product Card
const ProductCard = ({ product }) => {
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>{product.category}</p>
      <p>${product.price}</p>
    </div>
  );
};

// Parent: Product List with Search
const ProductPage = () => {
  const [searchQuery, setSearchQuery] = useState('');
  
  const allProducts = [
    { id: 1, name: 'Laptop', category: 'Electronics', price: 999 },
    { id: 2, name: 'Phone', category: 'Electronics', price: 699 },
    { id: 3, name: 'Desk', category: 'Furniture', price: 299 },
    { id: 4, name: 'Chair', category: 'Furniture', price: 199 },
    { id: 5, name: 'Monitor', category: 'Electronics', price: 399 }
  ];
  
  // Filter products based on search query from child
  const filteredProducts = allProducts.filter(product =>
    product.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
    product.category.toLowerCase().includes(searchQuery.toLowerCase())
  );
  
  const handleSearch = (query) => {
    console.log('Searching for:', query);
    setSearchQuery(query);
  };
  
  return (
    <div className="product-page">
      <h1>Products</h1>
      
      <SearchBar
        onSearch={handleSearch}
        placeholder="Search products..."
      />
      
      <div className="results-info">
        <p>Showing {filteredProducts.length} of {allProducts.length} products</p>
        {searchQuery && <p>Search term: "{searchQuery}"</p>}
      </div>
      
      <div className="product-grid">
        {filteredProducts.length > 0 ? (
          filteredProducts.map(product => (
            <ProductCard key={product.id} product={product} />
          ))
        ) : (
          <p>No products found matching "{searchQuery}"</p>
        )}
      </div>
    </div>
  );
};

export default ProductPage;
```

**Common Patterns Summary**

1. **Direct Callback**
```jsx
<Child onData={(data) => handleData(data)} />
```

2. **Pass setState Directly**
```jsx
<Child onData={setData} />
```

3. **Multiple Callbacks**
```jsx
<Child onAdd={handleAdd} onEdit={handleEdit} onDelete={handleDelete} />
```

4. **Event Handler with ID**
```jsx
<Child onClick={(id) => handleClick(id)} />
```

5. **Form Submission**
```jsx
<Form onSubmit={(formData) => handleSubmit(formData)} />
```

**Interview Tips:**
- Data flows **down** (parent → child) via props, but can flow **up** (child → parent) via **callbacks**
- Parent passes a **function** as prop, child **calls** it with data
- This is called **"lifting state up"** - state lives in parent, children communicate through callbacks
- Pattern: Parent defines handler → passes as prop → child calls with data
- Can pass **multiple parameters** in callback: `callback(id, data, extra)`
- For deep nesting, use **Context API** instead of prop drilling
- Common naming: `onSomething` for callback props (e.g., `onSubmit`, `onChange`, `onDelete`)
- Child manages **local state** (like input value), parent manages **shared state**
- This pattern enables **component composition** and **reusability**
- Real examples: forms, search filters, todo lists, modal dialogs

</details>
