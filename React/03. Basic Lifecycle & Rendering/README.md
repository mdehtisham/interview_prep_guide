# Basic Lifecycle & Rendering

> Beginner / Junior Level (0-1 years)

---

## Questions

21. What is the component lifecycle in React?
22. What happens during component mounting?
23. What is re-rendering in React?
24. When does a component re-render?
25. What is the difference between `createElement` and JSX?
26. What are React fragments and why use them?
27. How do you apply CSS styles in React?
28. What is the children prop?
29. What is prop drilling?
30. How do you conditionally apply CSS classes in React?

---

## Detailed Answers

### 21. What is the component lifecycle in React?

<details>
<summary>View Answer</summary>

**Component Lifecycle in React**

Every React component goes through a **lifecycle** - a series of phases from creation to removal. Understanding the lifecycle helps you know when to fetch data, update DOM, clean up resources, and optimize performance.

**Three Main Phases:**

1. **Mounting** - Component is created and inserted into the DOM
2. **Updating** - Component re-renders due to changes in props or state
3. **Unmounting** - Component is removed from the DOM

**Component Lifecycle Diagram:**

```
MOUNTING (Birth)
   |
   ├── constructor()
   ├── getDerivedStateFromProps()
   ├── render()
   └── componentDidMount()  ← Component is in DOM now
   |
   |
UPDATING (Growth)
   |
   ├── getDerivedStateFromProps()
   ├── shouldComponentUpdate()
   ├── render()
   ├── getSnapshotBeforeUpdate()
   └── componentDidUpdate()  ← Component updated
   |
   ├─→ (Can update multiple times)
   |
   |
UNMOUNTING (Death)
   |
   └── componentWillUnmount()  ← Cleanup before removal
```

**Lifecycle Methods (Class Components)**

**Mounting Phase:**

```jsx
class MyComponent extends React.Component {
  // 1. Constructor - Initialize state
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    console.log('1. Constructor');
  }
  
  // 2. getDerivedStateFromProps - Sync state with props
  static getDerivedStateFromProps(props, state) {
    console.log('2. getDerivedStateFromProps');
    return null;  // Return object to update state, or null
  }
  
  // 3. Render - Return JSX
  render() {
    console.log('3. Render');
    return <div>Count: {this.state.count}</div>;
  }
  
  // 4. ComponentDidMount - After component is in DOM
  componentDidMount() {
    console.log('4. ComponentDidMount');
    // Perfect for:
    // - API calls
    // - Subscriptions
    // - DOM manipulation
    // - Timers
  }
}
```

**Updating Phase:**

```jsx
class MyComponent extends React.Component {
  // 1. getDerivedStateFromProps (same as mounting)
  static getDerivedStateFromProps(props, state) {
    console.log('1. getDerivedStateFromProps (update)');
    return null;
  }
  
  // 2. shouldComponentUpdate - Optimization
  shouldComponentUpdate(nextProps, nextState) {
    console.log('2. shouldComponentUpdate');
    // Return false to skip re-render
    return true;  // Default: always re-render
  }
  
  // 3. Render - Re-render component
  render() {
    console.log('3. Render (update)');
    return <div>Count: {this.state.count}</div>;
  }
  
  // 4. getSnapshotBeforeUpdate - Before DOM updates
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('4. getSnapshotBeforeUpdate');
    // Capture info from DOM (e.g., scroll position)
    return null;
  }
  
  // 5. componentDidUpdate - After update
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('5. componentDidUpdate');
    // Perfect for:
    // - Update DOM after state change
    // - Network requests based on prop changes
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser(this.props.userId);
    }
  }
}
```

**Unmounting Phase:**

```jsx
class MyComponent extends React.Component {
  componentWillUnmount() {
    console.log('ComponentWillUnmount - cleanup');
    // Perfect for:
    // - Clear timers
    // - Cancel network requests
    // - Unsubscribe from subscriptions
    // - Remove event listeners
  }
}
```

**Complete Lifecycle Example (Class Component):**

```jsx
import React from 'react';

class LifecycleDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
      data: null
    };
    console.log('Constructor: Component is being created');
  }
  
  static getDerivedStateFromProps(props, state) {
    console.log('getDerivedStateFromProps: Sync props to state');
    // Rarely used - only when state depends on props
    if (props.initialCount !== undefined && state.count === 0) {
      return { count: props.initialCount };
    }
    return null;
  }
  
  componentDidMount() {
    console.log('ComponentDidMount: Component is now in DOM');
    
    // Fetch data after mount
    fetch('https://api.example.com/data')
      .then(res => res.json())
      .then(data => this.setState({ data }))
      .catch(err => console.error(err));
    
    // Set up timer
    this.timerId = setInterval(() => {
      console.log('Timer tick');
    }, 1000);
    
    // Add event listener
    window.addEventListener('resize', this.handleResize);
  }
  
  shouldComponentUpdate(nextProps, nextState) {
    console.log('shouldComponentUpdate: Deciding whether to re-render');
    
    // Optimization: only re-render if count changed
    if (this.state.count === nextState.count) {
      return false;  // Skip re-render
    }
    return true;  // Proceed with re-render
  }
  
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('getSnapshotBeforeUpdate: Just before DOM update');
    
    // Example: Capture scroll position
    if (prevState.count < this.state.count) {
      return { scrollPosition: window.scrollY };
    }
    return null;
  }
  
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('ComponentDidUpdate: Component just updated');
    
    if (snapshot !== null) {
      console.log('Previous scroll position:', snapshot.scrollPosition);
    }
    
    // Fetch data if props changed
    if (prevProps.userId !== this.props.userId) {
      this.fetchUserData(this.props.userId);
    }
  }
  
  componentWillUnmount() {
    console.log('ComponentWillUnmount: Component is being removed');
    
    // Cleanup timer
    clearInterval(this.timerId);
    
    // Remove event listener
    window.removeEventListener('resize', this.handleResize);
    
    // Cancel pending requests
    // this.abortController.abort();
  }
  
  handleResize = () => {
    console.log('Window resized');
  }
  
  increment = () => {
    this.setState(prev => ({ count: prev.count + 1 }));
  }
  
  render() {
    console.log('Render: Generating JSX');
    
    return (
      <div>
        <h2>Lifecycle Demo</h2>
        <p>Count: {this.state.count}</p>
        <button onClick={this.increment}>Increment</button>
        {this.state.data && <p>Data: {JSON.stringify(this.state.data)}</p>}
      </div>
    );
  }
}

export default LifecycleDemo;
```

**Function Components with Hooks (Modern Approach)**

Function components don't have lifecycle methods. Instead, they use **Hooks** to achieve the same functionality:

**Lifecycle to Hooks Mapping:**

| Class Component | Function Component (Hooks) |
|-----------------|---------------------------|
| `constructor` | `useState` |
| `componentDidMount` | `useEffect(() => {}, [])` |
| `componentDidUpdate` | `useEffect(() => {})` |
| `componentWillUnmount` | `useEffect(() => { return () => {} }, [])` |
| `shouldComponentUpdate` | `React.memo()`, `useMemo`, `useCallback` |
| `getDerivedStateFromProps` | Update state during render |

**Function Component Lifecycle with useEffect:**

```jsx
import { useState, useEffect } from 'react';

const LifecycleFunctional = () => {
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);
  
  // ComponentDidMount - Runs once after initial render
  useEffect(() => {
    console.log('Component Mounted');
    
    // Fetch data
    fetch('https://api.example.com/data')
      .then(res => res.json())
      .then(data => setData(data));
    
  }, []);  // Empty array = run once on mount
  
  // ComponentDidUpdate - Runs on every render
  useEffect(() => {
    console.log('Component Updated');
    console.log('Count is:', count);
  });  // No dependency array = run on every render
  
  // ComponentDidUpdate (specific dependency) - Runs when count changes
  useEffect(() => {
    console.log('Count changed to:', count);
    
    // Only runs when count changes
    if (count > 0) {
      document.title = `Count: ${count}`;
    }
  }, [count]);  // Runs when count changes
  
  // ComponentWillUnmount - Cleanup
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Tick');
    }, 1000);
    
    const handleResize = () => console.log('Resized');
    window.addEventListener('resize', handleResize);
    
    // Cleanup function (like componentWillUnmount)
    return () => {
      console.log('Component Unmounting - Cleanup');
      clearInterval(timer);
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  
  return (
    <div>
      <h2>Lifecycle (Functional)</h2>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      {data && <p>Data: {JSON.stringify(data)}</p>}
    </div>
  );
};
```

**useEffect Patterns:**

**1. Mount Only (componentDidMount)**
```jsx
useEffect(() => {
  console.log('Mounted');
  // Runs once after first render
}, []);  // Empty dependency array
```

**2. Update Only (componentDidUpdate)**
```jsx
useEffect(() => {
  console.log('Updated');
  // Runs on every render (after initial mount)
});  // No dependency array
```

**3. Specific Dependency**
```jsx
useEffect(() => {
  console.log('User ID changed:', userId);
  fetchUser(userId);
}, [userId]);  // Runs when userId changes
```

**4. Cleanup (componentWillUnmount)**
```jsx
useEffect(() => {
  const subscription = subscribeToData();
  
  return () => {
    console.log('Cleanup');
    subscription.unsubscribe();
  };
}, []);
```

**5. Combined (Mount + Cleanup)**
```jsx
useEffect(() => {
  console.log('Mount');
  const timer = setInterval(() => console.log('Tick'), 1000);
  
  return () => {
    console.log('Unmount');
    clearInterval(timer);
  };
}, []);
```

**Common Lifecycle Use Cases**

**1. Fetching Data on Mount**

```jsx
// Class Component
componentDidMount() {
  fetch('/api/users')
    .then(res => res.json())
    .then(users => this.setState({ users }));
}

// Function Component
useEffect(() => {
  fetch('/api/users')
    .then(res => res.json())
    .then(users => setUsers(users));
}, []);
```

**2. Setting Up Subscriptions**

```jsx
// Class Component
componentDidMount() {
  this.subscription = DataStore.subscribe(data => {
    this.setState({ data });
  });
}

componentWillUnmount() {
  this.subscription.unsubscribe();
}

// Function Component
useEffect(() => {
  const subscription = DataStore.subscribe(data => {
    setData(data);
  });
  
  return () => subscription.unsubscribe();
}, []);
```

**3. Updating Based on Prop Changes**

```jsx
// Class Component
componentDidUpdate(prevProps) {
  if (prevProps.userId !== this.props.userId) {
    this.fetchUser(this.props.userId);
  }
}

// Function Component
useEffect(() => {
  fetchUser(userId);
}, [userId]);  // Automatically handles changes
```

**4. Timers**

```jsx
// Class Component
componentDidMount() {
  this.timer = setInterval(() => {
    this.tick();
  }, 1000);
}

componentWillUnmount() {
  clearInterval(this.timer);
}

// Function Component
useEffect(() => {
  const timer = setInterval(() => {
    tick();
  }, 1000);
  
  return () => clearInterval(timer);
}, []);
```

**5. Event Listeners**

```jsx
// Class Component
componentDidMount() {
  window.addEventListener('scroll', this.handleScroll);
}

componentWillUnmount() {
  window.removeEventListener('scroll', this.handleScroll);
}

// Function Component
useEffect(() => {
  const handleScroll = () => console.log('Scrolling');
  window.addEventListener('scroll', handleScroll);
  
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

**Lifecycle Visualization:**

```
User visits page
       |
       v
   MOUNTING
       |
   constructor()
   useState() initialization
       |
   render() / JSX evaluation
       |
   React updates DOM
       |
   componentDidMount()
   useEffect(() => {}, [])
       |
       |
   Component is visible
       |
       v
User clicks button / Props change / setState called
       |
       v
   UPDATING
       |
   shouldComponentUpdate() [optional]
   React.memo comparison [optional]
       |
   render() / JSX evaluation
       |
   React updates DOM
       |
   componentDidUpdate()
   useEffect(() => {}, [deps])
       |
   ├──> Can update many times
   |
       v
User navigates away / Component removed
       |
       v
   UNMOUNTING
       |
   componentWillUnmount()
   useEffect cleanup: return () => {}
       |
   Component removed from DOM
```

**Production Example: User Profile Component**

```jsx
import { useState, useEffect } from 'react';

const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [isOnline, setIsOnline] = useState(false);
  
  // Mount: Initial data fetch
  useEffect(() => {
    console.log('Component Mounted');
    
    // Set page title
    document.title = 'User Profile';
    
    // Track component mount (analytics)
    trackEvent('profile_view', { userId });
    
    return () => {
      console.log('Component Unmounting');
      document.title = 'App';  // Reset title
    };
  }, []);  // Run once on mount
  
  // Update: Fetch user when userId changes
  useEffect(() => {
    console.log('Fetching user:', userId);
    
    setLoading(true);
    setError(null);
    
    // Create abort controller for cleanup
    const abortController = new AbortController();
    
    fetch(`/api/users/${userId}`, {
      signal: abortController.signal
    })
      .then(res => {
        if (!res.ok) throw new Error('User not found');
        return res.json();
      })
      .then(data => {
        setUser(data);
        setLoading(false);
        
        // Update page title with user name
        document.title = `${data.name}'s Profile`;
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          setError(err.message);
          setLoading(false);
        }
      });
    
    // Cleanup: Cancel fetch if component unmounts or userId changes
    return () => {
      console.log('Canceling fetch for user:', userId);
      abortController.abort();
    };
  }, [userId]);  // Re-run when userId changes
  
  // Mount + Unmount: WebSocket connection
  useEffect(() => {
    console.log('Setting up WebSocket');
    
    // Connect to WebSocket
    const ws = new WebSocket('wss://api.example.com/status');
    
    ws.onopen = () => {
      console.log('WebSocket connected');
      ws.send(JSON.stringify({ userId }));
    };
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.userId === userId) {
        setIsOnline(data.online);
      }
    };
    
    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
    
    // Cleanup: Close WebSocket
    return () => {
      console.log('Closing WebSocket');
      ws.close();
    };
  }, [userId]);
  
  // Update: Save to localStorage when user changes
  useEffect(() => {
    if (user) {
      console.log('Saving to localStorage');
      localStorage.setItem('lastViewedUser', userId);
    }
  }, [user, userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return null;
  
  return (
    <div className="user-profile">
      <div className="header">
        <img src={user.avatar} alt={user.name} />
        <h1>{user.name}</h1>
        <span className={`status ${isOnline ? 'online' : 'offline'}`}>
          {isOnline ? 'Online' : 'Offline'}
        </span>
      </div>
      <div className="info">
        <p>Email: {user.email}</p>
        <p>Bio: {user.bio}</p>
      </div>
    </div>
  );
};

export default UserProfile;
```

**Lifecycle Antipatterns (Avoid These)**

**❌ Infinite Loop**
```jsx
// BAD: Will cause infinite loop
useEffect(() => {
  setCount(count + 1);  // Updates state
}, [count]);  // Depends on state it updates
```

**❌ Missing Cleanup**
```jsx
// BAD: Timer keeps running after unmount
useEffect(() => {
  setInterval(() => {
    console.log('Tick');
  }, 1000);
  // Missing cleanup!
}, []);

// GOOD:
useEffect(() => {
  const timer = setInterval(() => {
    console.log('Tick');
  }, 1000);
  
  return () => clearInterval(timer);
}, []);
```

**❌ Async Function in useEffect**
```jsx
// BAD: useEffect callback can't be async
useEffect(async () => {
  const data = await fetchData();
  setData(data);
}, []);

// GOOD:
useEffect(() => {
  const fetchData = async () => {
    const data = await fetchData();
    setData(data);
  };
  fetchData();
}, []);
```

**Interview Tips:**
- React component lifecycle has **three phases**: Mounting, Updating, Unmounting
- **Class components** use lifecycle methods (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`)
- **Function components** use `useEffect` hook to handle all lifecycle phases
- **Empty dependency array** `[]` in useEffect = runs once on mount (componentDidMount)
- **No dependency array** = runs on every render (componentDidUpdate)
- **Return function** in useEffect = cleanup (componentWillUnmount)
- Common uses: **data fetching**, **subscriptions**, **timers**, **event listeners**
- Always **cleanup** timers, subscriptions, and event listeners to prevent memory leaks
- Modern React prefers **function components with hooks** over class components
- useEffect can have **multiple instances** for different concerns (separation of concerns)

</details>

---

### 22. What happens during component mounting?

<details>
<summary>View Answer</summary>

**Component Mounting Phase**

Mounting is the **first phase** of a component's lifecycle - when a component is **created and inserted into the DOM** for the first time.

**Mounting Process:**

```
1. Component instance created
   ↓
2. Initial props and state set
   ↓
3. Render method called (JSX → Virtual DOM)
   ↓
4. React creates actual DOM elements
   ↓
5. Elements inserted into DOM
   ↓
6. componentDidMount / useEffect runs
   ↓
7. Component is now visible and interactive
```

**Class Component Mounting Sequence:**

```jsx
class MyComponent extends React.Component {
  // 1. CONSTRUCTOR - First thing called
  constructor(props) {
    super(props);
    
    // Initialize state
    this.state = {
      count: 0,
      data: null
    };
    
    // Bind methods (if not using arrow functions)
    this.handleClick = this.handleClick.bind(this);
    
    console.log('1. Constructor called');
  }
  
  // 2. getDerivedStateFromProps (rarely used)
  static getDerivedStateFromProps(props, state) {
    console.log('2. getDerivedStateFromProps');
    
    // Sync state with props if needed
    if (props.initialCount !== state.count) {
      return { count: props.initialCount };
    }
    
    return null;  // No state update needed
  }
  
  // 3. RENDER - Generate JSX
  render() {
    console.log('3. Render called');
    
    // Return JSX (what should appear on screen)
    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        <button onClick={this.handleClick}>Click Me</button>
      </div>
    );
  }
  
  // 4. componentDidMount - After component is in DOM
  componentDidMount() {
    console.log('4. ComponentDidMount - Component is now in DOM!');
    
    // Perfect place for:
    // - API calls
    // - Subscriptions
    // - Timers
    // - DOM manipulation
    // - Third-party library initialization
    
    fetch('/api/data')
      .then(res => res.json())
      .then(data => this.setState({ data }));
  }
  
  handleClick = () => {
    this.setState(prev => ({ count: prev.count + 1 }));
  }
}
```

**Console Output During Mounting:**
```
1. Constructor called
2. getDerivedStateFromProps
3. Render called
4. ComponentDidMount - Component is now in DOM!
```

**Function Component Mounting:**

```jsx
import { useState, useEffect } from 'react';

const MyComponent = (props) => {
  // 1. Function body runs (like constructor)
  console.log('1. Component function called');
  
  // Initialize state
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);
  
  // 2. useEffect with empty dependency = componentDidMount
  useEffect(() => {
    console.log('2. Component mounted - now in DOM!');
    
    // Same use cases as componentDidMount
    fetch('/api/data')
      .then(res => res.json())
      .then(data => setData(data));
    
    // Cleanup function (optional)
    return () => {
      console.log('Component will unmount');
    };
  }, []);  // Empty array = run once on mount
  
  console.log('3. About to render');
  
  // 3. Return JSX
  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>Click Me</button>
    </div>
  );
};
```

**Mounting Order (Parent and Children):**

```jsx
// Parent Component
const Parent = () => {
  console.log('Parent render');
  
  useEffect(() => {
    console.log('Parent mounted');
  }, []);
  
  return (
    <div>
      <h1>Parent</h1>
      <ChildA />
      <ChildB />
    </div>
  );
};

// Child A
const ChildA = () => {
  console.log('ChildA render');
  
  useEffect(() => {
    console.log('ChildA mounted');
  }, []);
  
  return <div>Child A</div>;
};

// Child B
const ChildB = () => {
  console.log('ChildB render');
  
  useEffect(() => {
    console.log('ChildB mounted');
  }, []);
  
  return <div>Child B</div>;
};
```

**Console Output (Mounting Order):**
```
Parent render
ChildA render
ChildB render
ChildA mounted      ← Children mount first
ChildB mounted      ← Then siblings
Parent mounted      ← Parent mounts last
```

**Key Point:** Children mount before parents! React mounts from **bottom-up** (leaves to root).

**What to Do During Mounting**

**✅ Do These in componentDidMount / useEffect:**

**1. Fetch Data from API**
```jsx
useEffect(() => {
  const fetchUsers = async () => {
    try {
      const response = await fetch('/api/users');
      const users = await response.json();
      setUsers(users);
    } catch (error) {
      setError(error.message);
    }
  };
  
  fetchUsers();
}, []);  // Empty array = mount only
```

**2. Set Up Subscriptions**
```jsx
useEffect(() => {
  // Subscribe to data source
  const subscription = DataStore.subscribe((data) => {
    setData(data);
  });
  
  // Cleanup subscription on unmount
  return () => {
    subscription.unsubscribe();
  };
}, []);
```

**3. Set Up Timers**
```jsx
useEffect(() => {
  const timer = setInterval(() => {
    setTime(new Date());
  }, 1000);
  
  // Cleanup timer on unmount
  return () => {
    clearInterval(timer);
  };
}, []);
```

**4. Add Event Listeners**
```jsx
useEffect(() => {
  const handleResize = () => {
    setWindowWidth(window.innerWidth);
  };
  
  window.addEventListener('resize', handleResize);
  
  // Cleanup listener on unmount
  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []);
```

**5. Initialize Third-Party Libraries**
```jsx
useEffect(() => {
  // Initialize chart library
  const chart = new Chart(chartRef.current, {
    type: 'bar',
    data: chartData
  });
  
  // Cleanup
  return () => {
    chart.destroy();
  };
}, []);
```

**6. Focus Input Element**
```jsx
import { useRef, useEffect } from 'react';

const AutoFocusInput = () => {
  const inputRef = useRef(null);
  
  useEffect(() => {
    // DOM is ready, focus the input
    inputRef.current.focus();
  }, []);
  
  return <input ref={inputRef} type="text" />;
};
```

**7. Track Analytics**
```jsx
useEffect(() => {
  // Track page view
  analytics.track('page_view', {
    page: 'Home',
    timestamp: Date.now()
  });
}, []);
```

**❌ Don't Do These:**

**1. Don't Call setState in Constructor**
```jsx
// ❌ BAD
constructor(props) {
  super(props);
  this.state = { count: 0 };
  this.setState({ count: 1 });  // Don't do this!
}

// ✅ GOOD
constructor(props) {
  super(props);
  this.state = { count: 1 };  // Just set initial state directly
}
```

**2. Don't Perform Side Effects in Render**
```jsx
// ❌ BAD
const Component = () => {
  console.log('Rendering...');
  
  fetch('/api/data')  // Don't fetch in render!
    .then(res => res.json())
    .then(data => setData(data));
  
  return <div>Component</div>;
};

// ✅ GOOD
const Component = () => {
  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(data => setData(data));
  }, []);
  
  return <div>Component</div>;
};
```

**3. Don't Access DOM Directly in Render**
```jsx
// ❌ BAD
const Component = () => {
  document.getElementById('myDiv').style.color = 'red';  // Don't do this!
  return <div id="myDiv">Text</div>;
};

// ✅ GOOD
const Component = () => {
  const divRef = useRef(null);
  
  useEffect(() => {
    divRef.current.style.color = 'red';  // Do this after mount
  }, []);
  
  return <div ref={divRef}>Text</div>;
};
```

**Mounting with Async Data:**

```jsx
import { useState, useEffect } from 'react';

const UserList = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    console.log('Component mounted - fetching users...');
    
    // Set loading state
    setLoading(true);
    
    // Fetch data
    fetch('https://jsonplaceholder.typicode.com/users')
      .then(response => {
        if (!response.ok) {
          throw new Error('Failed to fetch');
        }
        return response.json();
      })
      .then(data => {
        console.log('Data fetched successfully');
        setUsers(data);
        setLoading(false);
      })
      .catch(err => {
        console.error('Error fetching data:', err);
        setError(err.message);
        setLoading(false);
      });
  }, []);  // Empty array = run once on mount
  
  // Render loading state
  if (loading) return <div>Loading users...</div>;
  
  // Render error state
  if (error) return <div>Error: {error}</div>;
  
  // Render data
  return (
    <div>
      <h1>Users</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
};
```

**Production Example: Dashboard Component**

```jsx
import { useState, useEffect, useRef } from 'react';

const Dashboard = ({ userId }) => {
  const [userData, setUserData] = useState(null);
  const [stats, setStats] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const [currentTime, setCurrentTime] = useState(new Date());
  const chartRef = useRef(null);
  const timerRef = useRef(null);
  
  // Mount: Initial setup
  useEffect(() => {
    console.log('Dashboard mounted');
    
    // Set page title
    document.title = 'Dashboard';
    
    // Track analytics
    trackPageView('dashboard');
    
    // Return cleanup function
    return () => {
      console.log('Dashboard unmounting');
      document.title = 'App';
    };
  }, []);  // Run once on mount
  
  // Mount: Fetch user data
  useEffect(() => {
    console.log('Fetching user data for:', userId);
    
    setLoading(true);
    setError(null);
    
    // Fetch user profile
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUserData(data);
        console.log('User data loaded');
      })
      .catch(err => {
        console.error('Error fetching user:', err);
        setError(err.message);
      })
      .finally(() => {
        setLoading(false);
      });
  }, [userId]);
  
  // Mount: Fetch statistics
  useEffect(() => {
    console.log('Fetching statistics');
    
    fetch('/api/stats')
      .then(res => res.json())
      .then(data => {
        setStats(data);
        console.log('Statistics loaded');
      })
      .catch(err => {
        console.error('Error fetching stats:', err);
      });
  }, []);
  
  // Mount: Set up real-time clock
  useEffect(() => {
    console.log('Starting clock');
    
    timerRef.current = setInterval(() => {
      setCurrentTime(new Date());
    }, 1000);
    
    // Cleanup timer
    return () => {
      console.log('Stopping clock');
      if (timerRef.current) {
        clearInterval(timerRef.current);
      }
    };
  }, []);
  
  // Mount: Initialize chart library
  useEffect(() => {
    console.log('Initializing chart');
    
    if (chartRef.current && stats) {
      const chart = new Chart(chartRef.current, {
        type: 'line',
        data: {
          labels: stats.labels,
          datasets: [{
            label: 'Sales',
            data: stats.values
          }]
        }
      });
      
      // Cleanup chart
      return () => {
        console.log('Destroying chart');
        chart.destroy();
      };
    }
  }, [stats]);  // Re-initialize when stats change
  
  // Mount: WebSocket connection
  useEffect(() => {
    console.log('Connecting to WebSocket');
    
    const ws = new WebSocket('wss://api.example.com/live');
    
    ws.onopen = () => {
      console.log('WebSocket connected');
      ws.send(JSON.stringify({ userId }));
    };
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      console.log('WebSocket message:', data);
      // Update UI with real-time data
    };
    
    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
    
    // Cleanup WebSocket
    return () => {
      console.log('Closing WebSocket');
      ws.close();
    };
  }, [userId]);
  
  // Mount: Keyboard shortcuts
  useEffect(() => {
    console.log('Setting up keyboard shortcuts');
    
    const handleKeyPress = (e) => {
      if (e.ctrlKey && e.key === 'r') {
        e.preventDefault();
        console.log('Refresh shortcut pressed');
        // Refresh data
      }
    };
    
    window.addEventListener('keydown', handleKeyPress);
    
    // Cleanup event listener
    return () => {
      console.log('Removing keyboard shortcuts');
      window.removeEventListener('keydown', handleKeyPress);
    };
  }, []);
  
  if (loading) {
    return (
      <div className="dashboard-loading">
        <div className="spinner" />
        <p>Loading dashboard...</p>
      </div>
    );
  }
  
  if (error) {
    return (
      <div className="dashboard-error">
        <h2>Error Loading Dashboard</h2>
        <p>{error}</p>
        <button onClick={() => window.location.reload()}>Retry</button>
      </div>
    );
  }
  
  return (
    <div className="dashboard">
      <header>
        <h1>Dashboard</h1>
        <div className="clock">{currentTime.toLocaleTimeString()}</div>
      </header>
      
      <div className="user-info">
        <h2>Welcome, {userData?.name}!</h2>
        <p>Email: {userData?.email}</p>
      </div>
      
      <div className="stats">
        <div className="stat-card">
          <h3>Total Sales</h3>
          <p>{stats?.totalSales}</p>
        </div>
        <div className="stat-card">
          <h3>Active Users</h3>
          <p>{stats?.activeUsers}</p>
        </div>
        <div className="stat-card">
          <h3>Revenue</h3>
          <p>${stats?.revenue}</p>
        </div>
      </div>
      
      <div className="chart-container">
        <canvas ref={chartRef} />
      </div>
    </div>
  );
};

export default Dashboard;
```

**Console Output (Dashboard Mounting):**
```
Dashboard mounted
Fetching user data for: 123
Fetching statistics
Starting clock
Connecting to WebSocket
Setting up keyboard shortcuts
User data loaded
Statistics loaded
Initializing chart
WebSocket connected
```

**Mounting Lifecycle Timeline:**

```
Time 0ms:    Component function/constructor called
Time 1ms:    State initialized
Time 2ms:    Render method called
Time 5ms:    Virtual DOM created
Time 10ms:   React updates actual DOM
Time 15ms:   Component inserted into DOM
Time 16ms:   componentDidMount / useEffect runs
Time 20ms:   API calls start
Time 100ms:  API responses arrive
Time 101ms:  setState called with API data
Time 102ms:  Component re-renders (UPDATE phase begins)
```

**Interview Tips:**
- Mounting is when component is **created and inserted into DOM** for the first time
- Mounting order: Constructor → Render → DOM update → **componentDidMount/useEffect**
- **componentDidMount** (class) or **useEffect(() => {}, [])** (hooks) runs **after** component is in DOM
- Perfect time for: **API calls**, **subscriptions**, **timers**, **event listeners**
- **Children mount before parents** (bottom-up mounting)
- Always **return cleanup function** from useEffect for subscriptions/timers/listeners
- **Don't call setState in constructor** - set initial state directly
- **Don't perform side effects in render** - use componentDidMount/useEffect instead
- DOM is **not ready during render** - wait for componentDidMount/useEffect
- Use **empty dependency array []** in useEffect to run code only on mount

</details>

---

### 23. What is re-rendering in React?

<details>
<summary>View Answer</summary>

**Re-rendering in React**

Re-rendering is when React **updates a component** by calling its render function (or component function) again to reflect new data. It's how React keeps the UI in sync with the application state.

**What Happens During Re-render:**

```
1. State or props change
   ↓
2. React calls component's render function
   ↓
3. New Virtual DOM is created
   ↓
4. React compares new Virtual DOM with previous one (Reconciliation)
   ↓
5. React calculates minimal changes needed (Diffing)
   ↓
6. React updates only changed parts in real DOM
   ↓
7. Browser paints updated UI
```

**Initial Render vs Re-render:**

```jsx
import { useState } from 'react';

const Counter = () => {
  const [count, setCount] = useState(0);
  
  console.log('Rendering... count:', count);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
};

// Console output:
// First mount: "Rendering... count: 0"
// Click button: "Rendering... count: 1"  ← Re-render
// Click button: "Rendering... count: 2"  ← Re-render
// Click button: "Rendering... count: 3"  ← Re-render
```

**Virtual DOM and Re-rendering:**

```
State changes
     ↓
Render function creates NEW Virtual DOM
     ↓
  Virtual DOM
  (JavaScript object representing UI)
  {
    type: 'div',
    props: {
      children: [
        { type: 'p', props: { children: 'Count: 5' } },
        { type: 'button', props: { children: 'Increment' } }
      ]
    }
  }
     ↓
React compares with PREVIOUS Virtual DOM
     ↓
React finds differences ("Count: 4" → "Count: 5")
     ↓
React updates ONLY the changed text node in real DOM
     ↓
Browser re-paints only that part
```

**Example: Visualizing Re-renders**

```jsx
import { useState } from 'react';

const VisualizingReRender = () => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('John');
  
  console.log('Component re-rendered!');
  console.log('Current count:', count);
  console.log('Current name:', name);
  
  return (
    <div>
      <h1>Re-render Demo</h1>
      
      <div>
        <p>Count: {count}</p>
        <button onClick={() => setCount(count + 1)}>
          Increment Count
        </button>
      </div>
      
      <div>
        <p>Name: {name}</p>
        <button onClick={() => setName('Jane')}>
          Change Name
        </button>
      </div>
      
      <p>Rendered at: {new Date().toLocaleTimeString()}</p>
    </div>
  );
};

// Click "Increment Count" → Entire component re-renders
// Click "Change Name" → Entire component re-renders
// Even though only one piece of data changed, entire function runs again
```

**What Gets Re-executed During Re-render:**

```jsx
const MyComponent = () => {
  console.log('1. Component function runs');  // ✅ Runs on every re-render
  
  const [count, setCount] = useState(0);  // ✅ Runs, but state persists
  
  const expensiveCalculation = count * 2;  // ✅ Runs on every re-render
  console.log('2. Calculation:', expensiveCalculation);
  
  const handleClick = () => {  // ✅ Function recreated on every re-render
    console.log('Button clicked');
  };
  
  useEffect(() => {  // ❌ Doesn't run on every re-render
    console.log('3. Effect with empty deps - only on mount');
  }, []);
  
  useEffect(() => {  // ✅ Runs when count changes
    console.log('4. Effect with [count] dep - runs when count changes');
  }, [count]);
  
  console.log('5. About to return JSX');
  
  return (  // ✅ JSX evaluated on every re-render
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Click</button>
    </div>
  );
};

// On mount:
// 1, 2, 5, 3, 4

// On re-render (count changed):
// 1, 2, 5, 4
```

**Reconciliation (Diffing Algorithm):**

React's reconciliation algorithm compares new and old Virtual DOM trees to find minimal changes:

```jsx
// Previous render:
<div>
  <p>Count: 4</p>
  <button>Click</button>
</div>

// New render (after count changes):
<div>
  <p>Count: 5</p>  ← Only this text changed
  <button>Click</button>
</div>

// React's diffing finds:
// - div: same
// - p: same element, but text content changed
// - button: same

// React only updates:
// - The text node "4" → "5" in the DOM
// - Nothing else needs to change!
```

**React's Diffing Rules:**

**1. Different Element Types → Replace Entire Tree**
```jsx
// Before:
<div>
  <Counter />
</div>

// After:
<span>  // Different type!
  <Counter />
</span>

// React will:
// 1. Unmount old <div> and <Counter>
// 2. Mount new <span> and <Counter>
// Counter's state is LOST
```

**2. Same Element Type → Update Props**
```jsx
// Before:
<div className="before" title="old" />

// After:
<div className="after" title="old" />

// React will:
// - Keep the same DOM element
// - Only update className attribute
// - Leave title unchanged
```

**3. Keys for Lists → Preserve Identity**
```jsx
// Without keys - BAD
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
</ul>

// Add item at beginning:
<ul>
  <li>New Item</li>  // React thinks this WAS "Item 1" - updates text
  <li>Item 1</li>     // React thinks this WAS "Item 2" - updates text
  <li>Item 2</li>     // React thinks this is NEW - creates new element
</ul>
// Inefficient: 2 updates + 1 insertion

// With keys - GOOD
<ul>
  <li key="1">Item 1</li>
  <li key="2">Item 2</li>
</ul>

// Add item at beginning:
<ul>
  <li key="3">New Item</li>  // React knows this is NEW
  <li key="1">Item 1</li>     // React knows this is SAME
  <li key="2">Item 2</li>     // React knows this is SAME
</ul>
// Efficient: 1 insertion only
```

**Render vs Commit Phase:**

```
RENDER PHASE (Pure, can be interrupted)
  - Call component function
  - Create Virtual DOM
  - Compare with previous Virtual DOM
  - Calculate changes needed
  ↓
COMMIT PHASE (Cannot be interrupted)
  - Update real DOM
  - Run useLayoutEffect
  - Browser paints
  - Run useEffect
```

**Example: Expensive Re-renders**

```jsx
import { useState } from 'react';

const ExpensiveComponent = () => {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  console.log('Rendering...');
  
  // This runs on EVERY re-render (expensive!)
  const expensiveCalculation = () => {
    console.log('Running expensive calculation...');
    let result = 0;
    for (let i = 0; i < 1000000000; i++) {
      result += i;
    }
    return result;
  };
  
  const result = expensiveCalculation();  // Runs even when only 'text' changes!
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}  // Every keystroke re-renders!
      />
      
      <p>Calculation result: {result}</p>
    </div>
  );
};

// Problem: Typing in input causes expensive calculation to run on every keystroke!
```

**Optimized with useMemo:**

```jsx
import { useState, useMemo } from 'react';

const OptimizedComponent = () => {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  console.log('Rendering...');
  
  // Only recalculate when count changes
  const result = useMemo(() => {
    console.log('Running expensive calculation...');
    let result = 0;
    for (let i = 0; i < 1000000000; i++) {
      result += i;
    }
    return result;
  }, [count]);  // Only recalculate when count changes
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}  // No expensive calc!
      />
      
      <p>Calculation result: {result}</p>
    </div>
  );
};

// Now: Typing in input doesn't trigger expensive calculation!
```

**Parent Re-render Causes Child Re-render:**

```jsx
import { useState } from 'react';

// Child component
const ChildComponent = ({ name }) => {
  console.log('Child rendered');
  return <p>Name: {name}</p>;
};

// Parent component
const ParentComponent = () => {
  const [count, setCount] = useState(0);
  const [name] = useState('John');
  
  console.log('Parent rendered');
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      <ChildComponent name={name} />
    </div>
  );
};

// Click button:
// Console: "Parent rendered"
// Console: "Child rendered"  ← Child re-renders even though name didn't change!
```

**Optimized with React.memo:**

```jsx
import { useState, memo } from 'react';

// Memoized child - only re-renders if props change
const ChildComponent = memo(({ name }) => {
  console.log('Child rendered');
  return <p>Name: {name}</p>;
});

// Parent component
const ParentComponent = () => {
  const [count, setCount] = useState(0);
  const [name] = useState('John');
  
  console.log('Parent rendered');
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      <ChildComponent name={name} />
    </div>
  );
};

// Click button:
// Console: "Parent rendered"
// Console: (Child NOT rendered - props didn't change!)
```

**Unnecessary Re-renders Example:**

```jsx
import { useState } from 'react';

const UnnecessaryReRenders = () => {
  const [items, setItems] = useState(['Apple', 'Banana', 'Orange']);
  const [filter, setFilter] = useState('');
  
  console.log('Component rendered');
  
  // This filter runs on EVERY re-render
  const filteredItems = items.filter(item =>
    item.toLowerCase().includes(filter.toLowerCase())
  );
  
  console.log('Filtered items:', filteredItems);
  
  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter items..."
      />
      
      <ul>
        {filteredItems.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
      
      <ExpensiveComponent />  {/* Re-renders on every keystroke! */}
    </div>
  );
};

const ExpensiveComponent = () => {
  console.log('ExpensiveComponent rendered (unnecessary!)');
  // Expensive rendering logic...
  return <div>Expensive Component</div>;
};

// Problem: ExpensiveComponent re-renders even though it doesn't use filter state
```

**Optimized Version:**

```jsx
import { useState, useMemo, memo } from 'react';

const OptimizedReRenders = () => {
  const [items, setItems] = useState(['Apple', 'Banana', 'Orange']);
  const [filter, setFilter] = useState('');
  
  console.log('Component rendered');
  
  // Memoize filtered items - only recalculate when dependencies change
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item =>
      item.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);
  
  return (
    <div>
      <input
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter items..."
      />
      
      <ul>
        {filteredItems.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
      
      <ExpensiveComponent />  {/* Now memoized, won't re-render */}
    </div>
  );
};

// Memoized - only re-renders if props change
const ExpensiveComponent = memo(() => {
  console.log('ExpensiveComponent rendered');
  return <div>Expensive Component</div>;
});

// Now: ExpensiveComponent only renders once, not on every keystroke!
```

**Re-render Triggers Summary:**

```jsx
const Component = () => {
  // 1. State change → Re-render
  const [count, setCount] = useState(0);
  setCount(1);  // Triggers re-render
  
  // 2. Props change → Re-render
  // If parent passes new props, component re-renders
  
  // 3. Parent re-renders → Child re-renders
  // (unless child is memoized)
  
  // 4. Context value change → Re-render
  const value = useContext(MyContext);
  // If context value changes, component re-renders
  
  // 5. Force update (rare)
  const [, forceUpdate] = useReducer(x => x + 1, 0);
  forceUpdate();  // Triggers re-render
  
  return <div>...</div>;
};
```

**Production Example: Dashboard with Re-render Optimization**

```jsx
import { useState, useMemo, memo, useCallback } from 'react';

// Memoized chart component
const Chart = memo(({ data, type }) => {
  console.log('Chart rendered');
  
  // Expensive chart rendering logic
  return (
    <div className="chart">
      <h3>Chart ({type})</h3>
      <canvas>/* Chart visualization */</canvas>
    </div>
  );
});

// Memoized stats card
const StatsCard = memo(({ title, value }) => {
  console.log(`StatsCard rendered: ${title}`);
  return (
    <div className="stats-card">
      <h4>{title}</h4>
      <p>{value}</p>
    </div>
  );
});

const Dashboard = () => {
  const [searchQuery, setSearchQuery] = useState('');
  const [chartData, setChartData] = useState({
    sales: [100, 200, 300],
    users: [50, 75, 100]
  });
  const [stats] = useState({
    totalSales: 600,
    totalUsers: 225,
    revenue: 15000
  });
  
  console.log('Dashboard rendered');
  
  // Memoize filtered data - only recalculate when dependencies change
  const filteredData = useMemo(() => {
    console.log('Filtering data...');
    if (!searchQuery) return chartData;
    // Complex filtering logic...
    return chartData;
  }, [searchQuery, chartData]);
  
  // Memoize callback - prevents child re-renders
  const handleChartClick = useCallback((point) => {
    console.log('Chart point clicked:', point);
    // Handle click...
  }, []);  // Stable reference
  
  return (
    <div className="dashboard">
      <header>
        <h1>Dashboard</h1>
        <input
          type="text"
          value={searchQuery}
          onChange={(e) => setSearchQuery(e.target.value)}
          placeholder="Search..."
        />
      </header>
      
      {/* These won't re-render when search query changes */}
      <div className="stats">
        <StatsCard title="Total Sales" value={stats.totalSales} />
        <StatsCard title="Total Users" value={stats.totalUsers} />
        <StatsCard title="Revenue" value={`$${stats.revenue}`} />
      </div>
      
      {/* Only re-renders when filteredData changes */}
      <div className="charts">
        <Chart data={filteredData.sales} type="sales" onClick={handleChartClick} />
        <Chart data={filteredData.users} type="users" onClick={handleChartClick} />
      </div>
      
      <p>Search: "{searchQuery}"</p>
    </div>
  );
};

export default Dashboard;

// Typing in search input:
// Console: "Dashboard rendered"
// Console: "Filtering data..."
// (StatsCards and Charts don't re-render - they're memoized!)
```

**Re-render Performance Tips:**

1. **Use React.memo** for components that render often with same props
2. **Use useMemo** for expensive calculations
3. **Use useCallback** for stable function references
4. **Split components** - separate frequently updating parts from stable parts
5. **Use proper keys** in lists for efficient reconciliation
6. **Avoid inline objects/arrays** as props (creates new references)
7. **Use Context wisely** - updates trigger re-renders in all consumers

**Interview Tips:**
- Re-rendering is when React **calls render function again** to update UI
- Happens when **state changes**, **props change**, or **parent re-renders**
- React uses **Virtual DOM** and **reconciliation** to minimize actual DOM updates
- React only updates **changed parts** of the DOM, not everything
- **Render phase** (calculate changes) is separate from **Commit phase** (update DOM)
- Use **React.memo** to prevent unnecessary child re-renders
- Use **useMemo** to cache expensive calculations
- Use **useCallback** to cache function references
- Parent re-render causes **all children to re-render** by default
- Keys help React **identify which items changed** in lists for efficient updates

</details>

---

### 24. When does a component re-render?

<details>
<summary>View Answer</summary>

**When Does a Component Re-render?**

A React component re-renders when it needs to update its output. There are **5 main triggers** for re-renders:

**1. State Changes**
**2. Props Changes**
**3. Parent Re-renders**
**4. Context Changes**
**5. Force Update (rare)**

---

**1. State Changes**

When `setState` or state setter function is called, the component re-renders.

```jsx
import { useState } from 'react';

const StateExample = () => {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('John');
  
  console.log('Rendered');
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment  {/* Click = setState = Re-render */}
      </button>
      
      <p>Name: {name}</p>
      <button onClick={() => setName('Jane')}>
        Change Name  {/* Click = setState = Re-render */}
      </button>
    </div>
  );
};

// Click "Increment" → setCount called → Component re-renders
// Click "Change Name" → setName called → Component re-renders
```

**State Change Variations:**

```jsx
import { useState } from 'react';

const StateVariations = () => {
  const [count, setCount] = useState(0);
  
  const handleSameValue = () => {
    setCount(0);  // Setting to SAME value
  };
  
  const handleNewValue = () => {
    setCount(count + 1);  // Setting to DIFFERENT value
  };
  
  const handleMultipleCalls = () => {
    setCount(count + 1);  // All three use same 'count' value
    setCount(count + 1);  // React batches these
    setCount(count + 1);  // Only increments by 1!
  };
  
  const handleFunctionalUpdate = () => {
    setCount(prev => prev + 1);  // Uses previous state
    setCount(prev => prev + 1);  // Uses updated state
    setCount(prev => prev + 1);  // Increments by 3!
  };
  
  console.log('Rendered, count:', count);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleSameValue}>Same Value</button>
      <button onClick={handleNewValue}>New Value</button>
      <button onClick={handleMultipleCalls}>Multiple Calls</button>
      <button onClick={handleFunctionalUpdate}>Functional Update</button>
    </div>
  );
};

// "Same Value": May NOT re-render (React optimization)
// "New Value": Re-renders
// "Multiple Calls": Re-renders ONCE (batching)
// "Functional Update": Re-renders ONCE (batching)
```

**React Batching (Automatic Optimization):**

```jsx
const handleClick = () => {
  setCount(1);      // Queued
  setName('Jane');  // Queued
  setAge(25);       // Queued
  
  // React batches all three updates into ONE re-render
};

// Only ONE re-render happens, not three!
```

---

**2. Props Changes**

When parent passes new props, component re-renders.

```jsx
// Child Component
const ChildComponent = ({ name, age }) => {
  console.log('Child rendered with:', name, age);
  return (
    <div>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
    </div>
  );
};

// Parent Component
const ParentComponent = () => {
  const [name, setName] = useState('John');
  const [age, setAge] = useState(25);
  
  return (
    <div>
      <button onClick={() => setName('Jane')}>
        Change Name  {/* Child re-renders - name prop changed */}
      </button>
      
      <button onClick={() => setAge(26)}>
        Change Age  {/* Child re-renders - age prop changed */}
      </button>
      
      <ChildComponent name={name} age={age} />
    </div>
  );
};

// Click "Change Name" → Parent re-renders → Child receives new name prop → Child re-renders
// Click "Change Age" → Parent re-renders → Child receives new age prop → Child re-renders
```

**Prop Comparison:**

```jsx
import { memo } from 'react';

// Regular component - re-renders on any parent re-render
const RegularChild = ({ name }) => {
  console.log('Regular child rendered');
  return <p>Name: {name}</p>;
};

// Memoized component - only re-renders if props actually changed
const MemoizedChild = memo(({ name }) => {
  console.log('Memoized child rendered');
  return <p>Name: {name}</p>;
});

const Parent = () => {
  const [count, setCount] = useState(0);
  const [name] = useState('John');  // Never changes
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      <RegularChild name={name} />    {/* Re-renders every time */}
      <MemoizedChild name={name} />   {/* Doesn't re-render - props unchanged */}
    </div>
  );
};

// Click button:
// Console: "Regular child rendered" (every time)
// Console: (Memoized child NOT rendered - prop didn't change)
```

---

**3. Parent Re-renders**

**Key Rule:** When parent re-renders, ALL children re-render by default (even if props didn't change).

```jsx
import { useState } from 'react';

// Grandchild Component
const Grandchild = () => {
  console.log('Grandchild rendered');
  return <div>Grandchild</div>;
};

// Child Component
const Child = () => {
  console.log('Child rendered');
  return (
    <div>
      <p>Child</p>
      <Grandchild />
    </div>
  );
};

// Parent Component
const Parent = () => {
  const [count, setCount] = useState(0);
  
  console.log('Parent rendered');
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <Child />  {/* Has NO props */}
    </div>
  );
};

// Click button:
// Console: "Parent rendered"
// Console: "Child rendered"       ← Re-renders even though no props!
// Console: "Grandchild rendered"  ← Re-renders because parent (Child) re-rendered!

// Cascade: Parent → Child → Grandchild
```

**Preventing Unnecessary Re-renders:**

```jsx
import { useState, memo } from 'react';

// Memoized components only re-render if props change
const Grandchild = memo(() => {
  console.log('Grandchild rendered');
  return <div>Grandchild</div>;
});

const Child = memo(() => {
  console.log('Child rendered');
  return (
    <div>
      <p>Child</p>
      <Grandchild />
    </div>
  );
});

const Parent = () => {
  const [count, setCount] = useState(0);
  
  console.log('Parent rendered');
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <Child />  {/* Memoized - won't re-render! */}
    </div>
  );
};

// Click button:
// Console: "Parent rendered"
// (Child and Grandchild do NOT re-render - they're memoized!)
```

---

**4. Context Changes**

When context value changes, ALL components using that context re-render.

```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

// Component using context
const ThemedButton = () => {
  const theme = useContext(ThemeContext);
  
  console.log('ThemedButton rendered');
  
  return (
    <button style={{ background: theme.background, color: theme.color }}>
      Themed Button
    </button>
  );
};

// Another component using context
const ThemedText = () => {
  const theme = useContext(ThemeContext);
  
  console.log('ThemedText rendered');
  
  return <p style={{ color: theme.color }}>Themed Text</p>;
};

// Provider component
const App = () => {
  const [theme, setTheme] = useState({
    background: 'white',
    color: 'black'
  });
  
  const toggleTheme = () => {
    setTheme(theme.background === 'white'
      ? { background: 'black', color: 'white' }
      : { background: 'white', color: 'black' }
    );
  };
  
  return (
    <ThemeContext.Provider value={theme}>
      <div>
        <button onClick={toggleTheme}>Toggle Theme</button>
        <ThemedButton />  {/* Uses context */}
        <ThemedText />    {/* Uses context */}
      </div>
    </ThemeContext.Provider>
  );
};

// Click "Toggle Theme":
// Console: "ThemedButton rendered"  ← Context value changed
// Console: "ThemedText rendered"    ← Context value changed
```

**Context Re-render Problem:**

```jsx
const App = () => {
  const [count, setCount] = useState(0);
  const [theme] = useState({ color: 'blue' });
  
  return (
    <ThemeContext.Provider value={theme}>
      <div>
        <p>Count: {count}</p>
        <button onClick={() => setCount(count + 1)}>Increment</button>
        
        <ExpensiveComponent />  {/* Uses context */}
      </div>
    </ThemeContext.Provider>
  );
};

// Click button:
// - App re-renders
// - ExpensiveComponent re-renders (because parent re-rendered)
// - Even though theme didn't change!

// Solution: Memoize ExpensiveComponent or split context provider
```

---

**5. Force Update (Rarely Used)**

```jsx
import { useReducer } from 'react';

const ForceUpdateExample = () => {
  const [, forceUpdate] = useReducer(x => x + 1, 0);
  
  console.log('Rendered');
  
  return (
    <div>
      <p>Time: {new Date().toLocaleTimeString()}</p>
      <button onClick={forceUpdate}>
        Force Update  {/* Triggers re-render without state/props change */}
      </button>
    </div>
  );
};

// Not recommended - usually indicates a design issue
// Better to use proper state management
```

---

**What Does NOT Trigger Re-render:**

**1. Regular Variable Changes**
```jsx
const Component = () => {
  let count = 0;  // Regular variable
  
  const increment = () => {
    count++;  // This does NOT trigger re-render!
    console.log(count);  // Value changes, but UI doesn't update
  };
  
  return (
    <div>
      <p>Count: {count}</p>  {/* Always shows 0 */}
      <button onClick={increment}>Increment</button>
    </div>
  );
};

// Solution: Use useState instead
```

**2. Ref Changes**
```jsx
import { useRef } from 'react';

const Component = () => {
  const countRef = useRef(0);
  
  const increment = () => {
    countRef.current++;  // This does NOT trigger re-render!
    console.log(countRef.current);  // Value changes, but UI doesn't update
  };
  
  return (
    <div>
      <p>Count: {countRef.current}</p>  {/* Doesn't update on screen */}
      <button onClick={increment}>Increment</button>
    </div>
  );
};

// Refs are for values that don't need to trigger re-renders
```

**3. Mutating State Directly**
```jsx
const Component = () => {
  const [user, setUser] = useState({ name: 'John', age: 25 });
  
  const updateAge = () => {
    user.age = 26;  // ❌ Mutation - does NOT trigger re-render!
    // React doesn't know state changed
  };
  
  const updateAgeCorrect = () => {
    setUser({ ...user, age: 26 });  // ✅ Creates new object - triggers re-render
  };
  
  return (
    <div>
      <p>Age: {user.age}</p>
      <button onClick={updateAge}>Wrong</button>
      <button onClick={updateAgeCorrect}>Correct</button>
    </div>
  );
};
```

**4. Props with Same Reference**
```jsx
import { memo } from 'react';

const Child = memo(({ data }) => {
  console.log('Child rendered');
  return <p>{data.value}</p>;
});

const Parent = () => {
  const [count, setCount] = useState(0);
  const data = { value: 'Hello' };  // New object on every render!
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <Child data={data} />  {/* Re-renders every time - new reference! */}
    </div>
  );
};

// Solution: Move data outside component or use useMemo
const data = { value: 'Hello' };  // Outside component - stable reference

// OR

const Parent = () => {
  const data = useMemo(() => ({ value: 'Hello' }), []);  // Stable reference
  // ...
};
```

---

**Re-render Triggers Summary:**

| Trigger | Example | Re-renders Component? |
|---------|---------|----------------------|
| `setState(newValue)` | `setCount(5)` | ✅ Yes |
| `setState(sameValue)` | `setCount(5)` when count is already 5 | ⚠️ Maybe (React may skip) |
| Props change | Parent passes new prop value | ✅ Yes |
| Props same | Parent passes same prop value | ⚠️ Yes (unless memoized) |
| Parent re-renders | Parent's state changes | ✅ Yes (unless memoized) |
| Context value changes | Provider value updates | ✅ Yes (all consumers) |
| `forceUpdate()` | Manual trigger | ✅ Yes |
| Regular variable change | `count++` | ❌ No |
| Ref change | `ref.current = 5` | ❌ No |
| Direct state mutation | `state.x = 5` | ❌ No |

---

**Production Example: Optimized User List**

```jsx
import { useState, useMemo, memo, useCallback } from 'react';

// Memoized user card - only re-renders if props change
const UserCard = memo(({ user, onDelete }) => {
  console.log(`UserCard rendered: ${user.name}`);
  
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onDelete(user.id)}>Delete</button>
    </div>
  );
});

const UserList = () => {
  const [users, setUsers] = useState([
    { id: 1, name: 'John', email: 'john@example.com' },
    { id: 2, name: 'Jane', email: 'jane@example.com' },
    { id: 3, name: 'Bob', email: 'bob@example.com' }
  ]);
  const [searchQuery, setSearchQuery] = useState('');
  const [sortOrder, setSortOrder] = useState('asc');
  
  console.log('UserList rendered');
  
  // Memoize filtered users - only recalculate when dependencies change
  const filteredUsers = useMemo(() => {
    console.log('Filtering users...');
    return users.filter(user =>
      user.name.toLowerCase().includes(searchQuery.toLowerCase())
    );
  }, [users, searchQuery]);
  
  // Memoize sorted users - only recalculate when dependencies change
  const sortedUsers = useMemo(() => {
    console.log('Sorting users...');
    return [...filteredUsers].sort((a, b) => {
      if (sortOrder === 'asc') {
        return a.name.localeCompare(b.name);
      } else {
        return b.name.localeCompare(a.name);
      }
    });
  }, [filteredUsers, sortOrder]);
  
  // Memoize delete function - prevents UserCard re-renders
  const handleDelete = useCallback((id) => {
    console.log('Deleting user:', id);
    setUsers(users.filter(user => user.id !== id));
  }, [users]);
  
  return (
    <div className="user-list">
      <h1>User List</h1>
      
      <div className="controls">
        <input
          type="text"
          value={searchQuery}
          onChange={(e) => setSearchQuery(e.target.value)}
          placeholder="Search users..."
        />
        
        <button onClick={() => setSortOrder(sortOrder === 'asc' ? 'desc' : 'asc')}>
          Sort: {sortOrder}
        </button>
      </div>
      
      <div className="users">
        {sortedUsers.map(user => (
          <UserCard
            key={user.id}
            user={user}
            onDelete={handleDelete}
          />
        ))}
      </div>
      
      <p>Showing {sortedUsers.length} of {users.length} users</p>
    </div>
  );
};

export default UserList;

// Type in search:
// - UserList re-renders
// - "Filtering users..." logged
// - "Sorting users..." logged
// - UserCards don't re-render (memoized, props unchanged)

// Click sort:
// - UserList re-renders
// - "Sorting users..." logged
// - UserCards don't re-render (memoized)

// Click delete:
// - UserList re-renders
// - "Filtering users..." logged
// - "Sorting users..." logged
// - Only deleted UserCard unmounts
// - Other UserCards don't re-render
```

**When Component Re-renders - Decision Tree:**

```
Did state change?
  └─ Yes → RE-RENDER
  └─ No → Continue

Did props change?
  └─ Yes → RE-RENDER
  └─ No → Continue

Did parent re-render?
  └─ Yes → Is component memoized?
      └─ Yes → Don't re-render
      └─ No → RE-RENDER
  └─ No → Continue

Did context value change?
  └─ Yes → Does component use this context?
      └─ Yes → RE-RENDER
      └─ No → Don't re-render
  └─ No → Don't re-render
```

**Interview Tips:**
- Component re-renders when: **state changes**, **props change**, **parent re-renders**, **context changes**
- **setState** triggers re-render, even if value is same (React may optimize this)
- **Parent re-render** causes all children to re-render by default
- Use **React.memo** to prevent re-renders when props haven't changed
- **Regular variables** and **refs** don't trigger re-renders
- **Mutating state directly** doesn't trigger re-render - always create new object/array
- React **batches** multiple setState calls in event handlers into one re-render
- Use **useMemo** to cache expensive calculations
- Use **useCallback** to cache function references and prevent child re-renders
- Context changes trigger re-render in **all components using that context**

</details>

---

### 25. What is the difference between `createElement` and JSX?

<details>
<summary>View Answer</summary>

**createElement vs JSX**

JSX is **syntactic sugar** for `React.createElement()`. JSX gets compiled (transformed) into `createElement` calls by tools like Babel before the browser runs the code.

**JSX is NOT valid JavaScript** - it's a syntax extension that needs to be compiled.

---

**JSX Syntax:**

```jsx
const element = <h1 className="title">Hello, World!</h1>;
```

**Compiled to createElement:**

```js
const element = React.createElement(
  'h1',                    // Type (string for HTML, component for React)
  { className: 'title' },  // Props
  'Hello, World!'          // Children
);
```

---

**React.createElement Signature:**

```js
React.createElement(
  type,        // Element type: 'div', 'span', or Component
  props,       // Properties/attributes object (or null)
  ...children  // Child elements (text, elements, components)
)
```

---

**Simple Examples:**

**1. Basic Element**

```jsx
// JSX
const element = <div>Hello</div>;

// createElement
const element = React.createElement('div', null, 'Hello');
```

**2. Element with Props**

```jsx
// JSX
const element = <div className="container" id="main">Content</div>;

// createElement
const element = React.createElement(
  'div',
  { className: 'container', id: 'main' },
  'Content'
);
```

**3. Element with Multiple Children**

```jsx
// JSX
const element = (
  <div>
    <h1>Title</h1>
    <p>Paragraph</p>
  </div>
);

// createElement
const element = React.createElement(
  'div',
  null,
  React.createElement('h1', null, 'Title'),
  React.createElement('p', null, 'Paragraph')
);
```

**4. Element with Nested Children**

```jsx
// JSX
const element = (
  <div>
    <header>
      <h1>Welcome</h1>
    </header>
    <main>
      <p>Content</p>
    </main>
  </div>
);

// createElement
const element = React.createElement(
  'div',
  null,
  React.createElement(
    'header',
    null,
    React.createElement('h1', null, 'Welcome')
  ),
  React.createElement(
    'main',
    null,
    React.createElement('p', null, 'Content')
  )
);
```

---

**Component Examples:**

**1. Function Component**

```jsx
const Greeting = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};

// JSX
const element = <Greeting name="John" />;

// createElement
const element = React.createElement(Greeting, { name: 'John' });
```

**2. Component with Children**

```jsx
const Container = ({ children }) => {
  return <div className="container">{children}</div>;
};

// JSX
const element = (
  <Container>
    <h1>Title</h1>
    <p>Content</p>
  </Container>
);

// createElement
const element = React.createElement(
  Container,
  null,
  React.createElement('h1', null, 'Title'),
  React.createElement('p', null, 'Content')
);
```

**3. Nested Components**

```jsx
const Button = ({ onClick, children }) => (
  <button onClick={onClick}>{children}</button>
);

const Card = ({ title, children }) => (
  <div className="card">
    <h2>{title}</h2>
    {children}
  </div>
);

// JSX
const element = (
  <Card title="Welcome">
    <p>This is a card</p>
    <Button onClick={() => alert('Clicked')}>Click Me</Button>
  </Card>
);

// createElement
const element = React.createElement(
  Card,
  { title: 'Welcome' },
  React.createElement('p', null, 'This is a card'),
  React.createElement(
    Button,
    { onClick: () => alert('Clicked') },
    'Click Me'
  )
);
```

---

**What JSX Compiles To:**

**Example 1: Complete Component**

```jsx
// JSX
function Welcome() {
  return (
    <div className="welcome">
      <h1>Hello, World!</h1>
      <p>Welcome to React</p>
    </div>
  );
}

// Compiles to:
function Welcome() {
  return React.createElement(
    'div',
    { className: 'welcome' },
    React.createElement('h1', null, 'Hello, World!'),
    React.createElement('p', null, 'Welcome to React')
  );
}
```

**Example 2: With Props and Events**

```jsx
// JSX
function Button({ onClick, disabled, children }) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className="btn"
    >
      {children}
    </button>
  );
}

// Compiles to:
function Button({ onClick, disabled, children }) {
  return React.createElement(
    'button',
    {
      onClick: onClick,
      disabled: disabled,
      className: 'btn'
    },
    children
  );
}
```

**Example 3: Conditional Rendering**

```jsx
// JSX
function UserGreeting({ isLoggedIn, name }) {
  return (
    <div>
      {isLoggedIn ? (
        <h1>Welcome back, {name}!</h1>
      ) : (
        <h1>Please sign in</h1>
      )}
    </div>
  );
}

// Compiles to:
function UserGreeting({ isLoggedIn, name }) {
  return React.createElement(
    'div',
    null,
    isLoggedIn
      ? React.createElement('h1', null, 'Welcome back, ', name, '!')
      : React.createElement('h1', null, 'Please sign in')
  );
}
```

**Example 4: List Rendering**

```jsx
// JSX
function List({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// Compiles to:
function List({ items }) {
  return React.createElement(
    'ul',
    null,
    items.map(item =>
      React.createElement('li', { key: item.id }, item.name)
    )
  );
}
```

---

**createElement Return Value:**

`React.createElement()` returns a **React Element** (plain JavaScript object):

```js
// This JSX:
const element = <h1 className="title">Hello</h1>;

// Becomes:
const element = {
  type: 'h1',
  props: {
    className: 'title',
    children: 'Hello'
  },
  key: null,
  ref: null,
  $$typeof: Symbol.for('react.element')
};

// This is a plain object, not a DOM element!
// React uses this object to create actual DOM elements
```

---

**Writing React Without JSX:**

You can write React without JSX (just use `createElement` directly):

```js
import React from 'react';
import ReactDOM from 'react-dom/client';

// Without JSX
function App() {
  return React.createElement(
    'div',
    { className: 'app' },
    React.createElement('h1', null, 'Hello, World!'),
    React.createElement(
      'p',
      null,
      'This is React without JSX'
    ),
    React.createElement(
      'button',
      { onClick: () => alert('Clicked!') },
      'Click Me'
    )
  );
}

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(React.createElement(App));

// Works perfectly! But much harder to read/write
```

---

**Key Differences:**

| Aspect | JSX | createElement |
|--------|-----|---------------|
| **Syntax** | HTML-like, familiar | Function calls |
| **Readability** | Easy to read/write | Verbose, nested |
| **Compilation** | Needs Babel/TypeScript | Pure JavaScript |
| **Browser** | Not valid JS | Valid JS |
| **Nesting** | Natural indentation | Deeply nested calls |
| **Attributes** | HTML-like props | Object properties |
| **Children** | Natural syntax | Function arguments |
| **Tooling** | Needs build step | No build needed |
| **Performance** | Same (compiles to createElement) | Same |

---

**JSX Transformation:**

**Before (JSX):**
```jsx
const App = () => {
  const [count, setCount] = useState(0);
  
  return (
    <div className="app">
      <h1>Counter: {count}</h1>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
};
```

**After Babel Compilation:**
```js
const App = () => {
  const [count, setCount] = useState(0);
  
  return React.createElement(
    'div',
    { className: 'app' },
    React.createElement('h1', null, 'Counter: ', count),
    React.createElement(
      'button',
      { onClick: () => setCount(count + 1) },
      'Increment'
    )
  );
};
```

---

**JSX Special Cases:**

**1. Self-Closing Tags**

```jsx
// JSX
<img src="photo.jpg" alt="Photo" />
<input type="text" />
<br />

// createElement
React.createElement('img', { src: 'photo.jpg', alt: 'Photo' });
React.createElement('input', { type: 'text' });
React.createElement('br', null);
```

**2. JavaScript Expressions in JSX**

```jsx
// JSX
const name = 'John';
const element = <h1>Hello, {name}!</h1>;

// createElement
const name = 'John';
const element = React.createElement('h1', null, 'Hello, ', name, '!');
```

**3. Spread Attributes**

```jsx
// JSX
const props = { className: 'box', id: 'main' };
const element = <div {...props}>Content</div>;

// createElement
const props = { className: 'box', id: 'main' };
const element = React.createElement('div', props, 'Content');
```

**4. Comments in JSX**

```jsx
// JSX
const element = (
  <div>
    {/* This is a comment */}
    <h1>Title</h1>
  </div>
);

// createElement
const element = React.createElement(
  'div',
  null,
  React.createElement('h1', null, 'Title')
);
// Comments are stripped during compilation
```

---

**Modern JSX Transform (React 17+):**

React 17 introduced a new JSX transform that doesn't require importing React:

**Old Transform (React 16 and earlier):**
```jsx
import React from 'react';  // Required!

function App() {
  return <h1>Hello</h1>;
}

// Compiles to:
function App() {
  return React.createElement('h1', null, 'Hello');
}
```

**New Transform (React 17+):**
```jsx
// No React import needed!
function App() {
  return <h1>Hello</h1>;
}

// Compiles to:
import { jsx as _jsx } from 'react/jsx-runtime';

function App() {
  return _jsx('h1', { children: 'Hello' });
}
```

---

**Production Example: Complex Component**

**With JSX (Readable):**

```jsx
import { useState } from 'react';

const TodoApp = () => {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, { id: Date.now(), text: input, done: false }]);
      setInput('');
    }
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  };
  
  return (
    <div className="todo-app">
      <h1>Todo List</h1>
      
      <div className="input-section">
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="Add todo..."
        />
        <button onClick={addTodo}>Add</button>
      </div>
      
      <ul className="todo-list">
        {todos.map(todo => (
          <li key={todo.id} className={todo.done ? 'done' : ''}>
            <input
              type="checkbox"
              checked={todo.done}
              onChange={() => toggleTodo(todo.id)}
            />
            <span>{todo.text}</span>
          </li>
        ))}
      </ul>
      
      <div className="stats">
        <p>Total: {todos.length}</p>
        <p>Completed: {todos.filter(t => t.done).length}</p>
      </div>
    </div>
  );
};
```

**Without JSX (Verbose):**

```js
import { useState, createElement as h } from 'react';

const TodoApp = () => {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, { id: Date.now(), text: input, done: false }]);
      setInput('');
    }
  };
  
  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  };
  
  return h(
    'div',
    { className: 'todo-app' },
    h('h1', null, 'Todo List'),
    h(
      'div',
      { className: 'input-section' },
      h('input', {
        type: 'text',
        value: input,
        onChange: (e) => setInput(e.target.value),
        onKeyPress: (e) => e.key === 'Enter' && addTodo(),
        placeholder: 'Add todo...'
      }),
      h('button', { onClick: addTodo }, 'Add')
    ),
    h(
      'ul',
      { className: 'todo-list' },
      todos.map(todo =>
        h(
          'li',
          { key: todo.id, className: todo.done ? 'done' : '' },
          h('input', {
            type: 'checkbox',
            checked: todo.done,
            onChange: () => toggleTodo(todo.id)
          }),
          h('span', null, todo.text)
        )
      )
    ),
    h(
      'div',
      { className: 'stats' },
      h('p', null, 'Total: ', todos.length),
      h('p', null, 'Completed: ', todos.filter(t => t.done).length)
    )
  );
};

// Much harder to read and write!
```

---

**Why JSX Exists:**

1. **Readability** - HTML-like syntax is familiar and easy to understand
2. **Developer Experience** - Less verbose, easier to write
3. **Tooling** - Better IDE support (syntax highlighting, autocomplete)
4. **Maintainability** - Easier to visualize component structure
5. **Accessibility** - Clear parent-child relationships

**Why You Might Use createElement:**

1. **No Build Step** - Works directly in browser without compilation
2. **Dynamic Components** - Easier to create components programmatically
3. **Learning** - Understand what JSX compiles to
4. **Edge Cases** - Some scenarios where function calls are clearer

---

**Interview Tips:**
- JSX is **syntactic sugar** for `React.createElement()`
- JSX is **not valid JavaScript** - must be compiled by Babel/TypeScript
- JSX compiles to `createElement(type, props, ...children)` calls
- `createElement` returns a **plain JavaScript object** (React Element)
- React 17+ has **new JSX transform** - no React import needed
- JSX is more **readable** and **maintainable** than createElement
- You **can write React without JSX** using createElement directly
- JSX allows **JavaScript expressions** with curly braces `{}`
- Under the hood, JSX and createElement produce **identical output**
- Modern tooling (Vite, Create React App) handles JSX compilation automatically

</details>

---

### 26. What are React fragments and why use them?

<details>
<summary>View Answer</summary>

**React Fragments**

Fragments let you **group multiple elements without adding extra nodes to the DOM**. They solve the problem of having to wrap multiple elements in a single parent.

**The Problem:**

React components must return a **single root element**:

```jsx
// ❌ ERROR: Multiple root elements
function Component() {
  return (
    <h1>Title</h1>
    <p>Paragraph</p>
  );
}
// SyntaxError: Adjacent JSX elements must be wrapped in an enclosing tag
```

**Traditional Solution (Wrapper Div):**

```jsx
// ✅ Works, but adds unnecessary div to DOM
function Component() {
  return (
    <div>
      <h1>Title</h1>
      <p>Paragraph</p>
    </div>
  );
}

// Rendered DOM:
// <div>           ← Extra wrapper div
//   <h1>Title</h1>
//   <p>Paragraph</p>
// </div>
```

**Problem with Wrapper Divs:**

```jsx
function Table() {
  return (
    <table>
      <tbody>
        <TableRow />
      </tbody>
    </table>
  );
}

function TableRow() {
  return (
    <div>  {/* ❌ Invalid! div inside tbody */}
      <td>Cell 1</td>
      <td>Cell 2</td>
    </div>
  );
}

// Rendered (INVALID HTML):
// <table>
//   <tbody>
//     <div>      ← Invalid! Not allowed in tbody
//       <td>Cell 1</td>
//       <td>Cell 2</td>
//     </div>
//   </tbody>
// </table>
```

---

**Fragment Solution:**

**Syntax 1: Long Form**

```jsx
import { Fragment } from 'react';

function Component() {
  return (
    <Fragment>
      <h1>Title</h1>
      <p>Paragraph</p>
    </Fragment>
  );
}

// Rendered DOM (no wrapper!):
// <h1>Title</h1>
// <p>Paragraph</p>
```

**Syntax 2: Short Form (Recommended)**

```jsx
function Component() {
  return (
    <>
      <h1>Title</h1>
      <p>Paragraph</p>
    </>
  );
}

// Same result - no wrapper in DOM!
```

**Syntax 3: Array (Old Way, Not Recommended)**

```jsx
function Component() {
  return [
    <h1 key="title">Title</h1>,
    <p key="paragraph">Paragraph</p>
  ];
}

// Works, but requires keys and less readable
```

---

**Fragment Examples:**

**1. Basic Usage**

```jsx
function Greeting() {
  return (
    <>
      <h1>Hello!</h1>
      <p>Welcome to our site.</p>
    </>
  );
}

// Rendered:
// <h1>Hello!</h1>
// <p>Welcome to our site.</p>
// (No wrapper div!)
```

**2. Table Rows (Fragment Solves HTML Structure Problem)**

```jsx
function TableRow() {
  return (
    <>  {/* ✅ Fragment doesn't create DOM node */}
      <td>Cell 1</td>
      <td>Cell 2</td>
      <td>Cell 3</td>
    </>
  );
}

function Table() {
  return (
    <table>
      <tbody>
        <tr>
          <TableRow />  {/* Valid HTML! */}
        </tr>
      </tbody>
    </table>
  );
}

// Rendered (VALID HTML):
// <table>
//   <tbody>
//     <tr>
//       <td>Cell 1</td>  ← No wrapper div!
//       <td>Cell 2</td>
//       <td>Cell 3</td>
//     </tr>
//   </tbody>
// </table>
```

**3. List Items**

```jsx
function DescriptionList() {
  return (
    <dl>
      <DescriptionItem term="React" definition="A JavaScript library" />
      <DescriptionItem term="JSX" definition="Syntax extension" />
    </dl>
  );
}

function DescriptionItem({ term, definition }) {
  return (
    <>  {/* Fragment groups dt and dd */}
      <dt>{term}</dt>
      <dd>{definition}</dd>
    </>
  );
}

// Rendered:
// <dl>
//   <dt>React</dt>
//   <dd>A JavaScript library</dd>
//   <dt>JSX</dt>
//   <dd>Syntax extension</dd>
// </dl>
```

**4. Conditional Rendering**

```jsx
function UserInfo({ isLoggedIn, user }) {
  return (
    <div className="user-info">
      {isLoggedIn ? (
        <>  {/* Fragment for multiple elements */}
          <h2>Welcome, {user.name}!</h2>
          <p>Email: {user.email}</p>
          <button>Logout</button>
        </>
      ) : (
        <>  {/* Fragment for multiple elements */}
          <h2>Please log in</h2>
          <button>Login</button>
        </>
      )}
    </div>
  );
}
```

**5. Returning Multiple Elements from Function**

```jsx
function renderItems(items) {
  return (
    <>
      {items.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </>
  );
}

function App() {
  const items = [{ id: 1, name: 'Item 1' }, { id: 2, name: 'Item 2' }];
  
  return (
    <div>
      <h1>Items</h1>
      {renderItems(items)}
    </div>
  );
}
```

---

**Fragments with Keys:**

When mapping over arrays, use the **long form** `<Fragment>` to add keys:

```jsx
import { Fragment } from 'react';

function Glossary({ items }) {
  return (
    <dl>
      {items.map(item => (
        <Fragment key={item.id}>  {/* Need key for lists */}
          <dt>{item.term}</dt>
          <dd>{item.definition}</dd>
        </Fragment>
      ))}
    </dl>
  );
}

// ❌ Short form doesn't support keys:
// <> ... </>

// ✅ Long form supports key attribute:
// <Fragment key={...}> ... </Fragment>
```

**Example with Keys:**

```jsx
import { Fragment } from 'react';

function CommentList({ comments }) {
  return (
    <div className="comments">
      {comments.map(comment => (
        <Fragment key={comment.id}>
          <h3>{comment.author}</h3>
          <p>{comment.text}</p>
          <time>{comment.date}</time>
          <hr />
        </Fragment>
      ))}
    </div>
  );
}

const comments = [
  { id: 1, author: 'John', text: 'Great post!', date: '2024-01-01' },
  { id: 2, author: 'Jane', text: 'Thanks!', date: '2024-01-02' }
];

// Rendered:
// <div class="comments">
//   <h3>John</h3>
//   <p>Great post!</p>
//   <time>2024-01-01</time>
//   <hr>
//   <h3>Jane</h3>
//   <p>Thanks!</p>
//   <time>2024-01-02</time>
//   <hr>
// </div>
```

---

**When to Use Fragments:**

**1. Table Rows and Cells**
```jsx
function TableRows() {
  return (
    <>
      <tr>
        <td>Row 1</td>
      </tr>
      <tr>
        <td>Row 2</td>
      </tr>
    </>
  );
}
```

**2. List Items (dl, dt, dd)**
```jsx
function Definition({ term, definition }) {
  return (
    <>
      <dt>{term}</dt>
      <dd>{definition}</dd>
    </>
  );
}
```

**3. Flex/Grid Layouts**
```jsx
function GridItems() {
  return (
    <>
      <div className="grid-item">Item 1</div>
      <div className="grid-item">Item 2</div>
      <div className="grid-item">Item 3</div>
    </>
  );
}

// Parent:
<div className="grid">  {/* display: grid */}
  <GridItems />  {/* Don't add wrapper div - breaks grid! */}
</div>
```

**4. Avoid Unnecessary Nesting**
```jsx
// ❌ Unnecessary div
function BadComponent() {
  return (
    <div>  {/* Extra div for no reason */}
      <Header />
      <Content />
      <Footer />
    </div>
  );
}

// ✅ Use fragment
function GoodComponent() {
  return (
    <>
      <Header />
      <Content />
      <Footer />
    </>
  );
}
```

**5. Conditional Multiple Elements**
```jsx
function ConditionalContent({ showDetails }) {
  return (
    <div>
      <h1>Title</h1>
      {showDetails && (
        <>  {/* Group multiple conditional elements */}
          <p>Detail 1</p>
          <p>Detail 2</p>
          <p>Detail 3</p>
        </>
      )}
    </div>
  );
}
```

---

**Why Use Fragments:**

**1. Cleaner DOM**
```jsx
// Without Fragment:
<div id="root">
  <div class="app">
    <div>  ← Extra div
      <h1>Title</h1>
      <p>Content</p>
    </div>
  </div>
</div>

// With Fragment:
<div id="root">
  <div class="app">
    <h1>Title</h1>  ← No extra div!
    <p>Content</p>
  </div>
</div>
```

**2. Valid HTML Structure**
```jsx
// ❌ Invalid HTML
<table>
  <tbody>
    <div>  ← Not allowed!
      <td>Cell</td>
    </div>
  </tbody>
</table>

// ✅ Valid HTML
<table>
  <tbody>
    <td>Cell</td>  ← Fragment doesn't create element
  </tbody>
</table>
```

**3. CSS Layout (Flexbox/Grid)**
```jsx
// ❌ Breaks flexbox
<div style={{ display: 'flex' }}>
  <div>  ← Extra wrapper breaks flex layout
    <div>Item 1</div>
    <div>Item 2</div>
  </div>
</div>

// ✅ Works with flexbox
<div style={{ display: 'flex' }}>
  <>  ← No wrapper
    <div>Item 1</div>
    <div>Item 2</div>
  </>
</div>
```

**4. Performance (Slightly Better)**
```jsx
// Without fragments: 1000 extra divs in DOM
{items.map(item => (
  <div key={item.id}>
    <h3>{item.title}</h3>
    <p>{item.description}</p>
  </div>
))}

// With fragments: No extra elements
{items.map(item => (
  <Fragment key={item.id}>
    <h3>{item.title}</h3>
    <p>{item.description}</p>
  </Fragment>
))}
```

**5. Semantic HTML**
```jsx
// ❌ Meaningless div
<div>
  <h1>Title</h1>
  <p>Description</p>
</div>

// ✅ No meaningless wrapper
<>
  <h1>Title</h1>
  <p>Description</p>
</>
```

---

**Production Example: Product Card Grid**

```jsx
import { Fragment } from 'react';

function ProductCard({ product }) {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p className="price">${product.price}</p>
      {product.onSale && (
        <>  {/* Multiple sale elements grouped */}
          <span className="sale-badge">SALE</span>
          <p className="original-price">${product.originalPrice}</p>
          <p className="discount">{product.discount}% OFF</p>
        </>
      )}
      <button>Add to Cart</button>
    </div>
  );
}

function ProductGrid({ products }) {
  return (
    <div className="product-grid">
      {products.map(product => (
        <Fragment key={product.id}>
          <ProductCard product={product} />
          
          {/* Every 4th product, add a banner */}
          {product.id % 4 === 0 && (
            <div className="banner">Special Offer!</div>
          )}
        </Fragment>
      ))}
    </div>
  );
}

function CategorySection({ category }) {
  return (
    <>  {/* No wrapper needed */}
      <h2>{category.name}</h2>
      <p>{category.description}</p>
      <ProductGrid products={category.products} />
    </>
  );
}

function App() {
  const categories = [
    {
      id: 1,
      name: 'Electronics',
      description: 'Latest gadgets',
      products: [/* ... */]
    },
    {
      id: 2,
      name: 'Clothing',
      description: 'Fashion items',
      products: [/* ... */]
    }
  ];
  
  return (
    <div className="app">
      <header>
        <h1>Online Store</h1>
      </header>
      
      <main>
        {categories.map(category => (
          <Fragment key={category.id}>
            <CategorySection category={category} />
            <hr />  {/* Separator between categories */}
          </Fragment>
        ))}
      </main>
    </div>
  );
}
```

---

**Fragment vs Div Comparison:**

```jsx
// With Div (adds to DOM)
function WithDiv() {
  return (
    <div>  {/* Creates actual DOM element */}
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
}

// DOM:
// <div>           ← Extra element in DOM
//   <h1>Title</h1>
//   <p>Content</p>
// </div>

// With Fragment (no DOM node)
function WithFragment() {
  return (
    <>  {/* No DOM element created */}
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
}

// DOM:
// <h1>Title</h1>  ← No wrapper!
// <p>Content</p>
```

---

**Fragment Limitations:**

**1. Cannot Add CSS Classes**
```jsx
// ❌ Fragments don't support className
<Fragment className="container">  {/* Won't work */}
  <h1>Title</h1>
</Fragment>

// ✅ Use div if you need styling
<div className="container">
  <h1>Title</h1>
</div>
```

**2. Cannot Add Event Handlers**
```jsx
// ❌ Fragments don't support event handlers
<Fragment onClick={handleClick}>  {/* Won't work */}
  <h1>Title</h1>
</Fragment>

// ✅ Use div if you need events
<div onClick={handleClick}>
  <h1>Title</h1>
</div>
```

**3. Short Syntax Doesn't Support Key**
```jsx
// ❌ Short syntax can't have key
{items.map(item => (
  <> key={item.id}>  {/* Syntax error */}
    <p>{item.name}</p>
  </>
))}

// ✅ Use long form for keys
{items.map(item => (
  <Fragment key={item.id}>
    <p>{item.name}</p>
  </Fragment>
))}
```

**4. Cannot Use Refs**
```jsx
// ❌ Fragments can't have refs
const fragmentRef = useRef();
<Fragment ref={fragmentRef}>  {/* Won't work */}
  <h1>Title</h1>
</Fragment>

// ✅ Use div if you need ref
const divRef = useRef();
<div ref={divRef}>
  <h1>Title</h1>
</div>
```

---

**Interview Tips:**
- Fragments let you **group elements without adding DOM nodes**
- Syntax: `<Fragment>...</Fragment>` or `<>...</>` (short form)
- Use fragments to **avoid unnecessary wrapper divs**
- Essential for **valid HTML** (e.g., `<td>` directly in `<tbody>`)
- Important for **CSS layouts** (flexbox, grid) - extra divs break layout
- **Short form `<>`** doesn't support keys or attributes
- **Long form `<Fragment>`** supports key attribute for lists
- Fragments produce **cleaner DOM** and slightly better performance
- Use **div instead** when you need className, styles, events, or refs
- Fragments are **invisible in DOM** - only children are rendered

</details>

---

### 27. How do you apply CSS styles in React?

<details>
<summary>View Answer</summary>

**Applying CSS Styles in React**

There are **7 main ways** to style React components:

1. **Inline Styles**
2. **CSS Stylesheets**
3. **CSS Modules**
4. **Styled Components** (CSS-in-JS)
5. **Sass/SCSS**
6. **Tailwind CSS** (Utility-first)
7. **Emotion** (CSS-in-JS)

---

## 1. Inline Styles

Apply styles directly using the `style` prop with a JavaScript object.

**Basic Syntax:**

```jsx
const Component = () => {
  const buttonStyle = {
    backgroundColor: 'blue',
    color: 'white',
    padding: '10px 20px',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer'
  };
  
  return <button style={buttonStyle}>Click Me</button>;
};
```

**Inline Object:**

```jsx
const Component = () => {
  return (
    <div style={{
      backgroundColor: 'lightblue',
      padding: '20px',
      margin: '10px'
    }}>
      Content
    </div>
  );
};

// Note: Double curly braces {{}}
// Outer {} = JavaScript expression
// Inner {} = JavaScript object
```

**Key Differences from Regular CSS:**

```jsx
// CSS property names use camelCase
<div style={{
  backgroundColor: 'red',      // background-color
  fontSize: '16px',            // font-size
  marginTop: '10px',           // margin-top
  borderRadius: '5px'          // border-radius
}}>
```

**Dynamic Styles:**

```jsx
const Button = ({ primary }) => {
  return (
    <button style={{
      backgroundColor: primary ? 'blue' : 'gray',
      color: 'white',
      padding: '10px 20px'
    }}>
      Button
    </button>
  );
};
```

**Conditional Styles:**

```jsx
const Alert = ({ type, message }) => {
  const alertStyles = {
    padding: '15px',
    borderRadius: '4px',
    marginBottom: '10px',
    ...(type === 'success' && {
      backgroundColor: '#d4edda',
      color: '#155724',
      border: '1px solid #c3e6cb'
    }),
    ...(type === 'error' && {
      backgroundColor: '#f8d7da',
      color: '#721c24',
      border: '1px solid #f5c6cb'
    }),
    ...(type === 'warning' && {
      backgroundColor: '#fff3cd',
      color: '#856404',
      border: '1px solid #ffeaa7'
    })
  };
  
  return <div style={alertStyles}>{message}</div>;
};
```

**Pros:**
- Easy and quick
- Dynamic styles based on props/state
- No class name conflicts
- Scoped to component

**Cons:**
- No pseudo-classes (`:hover`, `:focus`)
- No media queries
- No CSS preprocessor features
- Can impact performance if overused
- Less separation of concerns

---

## 2. CSS Stylesheets

Import regular CSS files and use `className` prop.

**styles.css:**
```css
.button {
  background-color: blue;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.button:hover {
  background-color: darkblue;
}

.button.primary {
  background-color: green;
}

.button.disabled {
  background-color: gray;
  cursor: not-allowed;
}
```

**Component.jsx:**
```jsx
import './styles.css';

const Button = ({ primary, disabled, children }) => {
  return (
    <button 
      className={`button ${primary ? 'primary' : ''} ${disabled ? 'disabled' : ''}`}
      disabled={disabled}
    >
      {children}
    </button>
  );
};
```

**Global Styles:**

```jsx
// index.js
import './index.css';  // Global styles

// index.css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Arial', sans-serif;
  line-height: 1.6;
}
```

**Pros:**
- Familiar CSS syntax
- Supports all CSS features
- Can reuse styles
- Good for global styles

**Cons:**
- Global scope (class name conflicts)
- No automatic scoping
- Hard to track unused styles
- No dynamic styles

---

## 3. CSS Modules

CSS files where class names are **locally scoped by default**.

**Button.module.css:**
```css
.button {
  background-color: blue;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.primary {
  background-color: green;
}

.disabled {
  background-color: gray;
  cursor: not-allowed;
}

/* Compose styles */
.largeButton {
  composes: button;
  font-size: 18px;
  padding: 15px 30px;
}
```

**Button.jsx:**
```jsx
import styles from './Button.module.css';

const Button = ({ primary, disabled, children }) => {
  return (
    <button 
      className={`${styles.button} ${primary ? styles.primary : ''} ${disabled ? styles.disabled : ''}`}
      disabled={disabled}
    >
      {children}
    </button>
  );
};

export default Button;
```

**How CSS Modules Work:**

```jsx
// You write:
<button className={styles.button}>Click</button>

// Generated HTML:
<button class="Button_button__2Rx3D">Click</button>

// Unique class name prevents conflicts!
```

**Multiple Classes:**

```jsx
import styles from './Card.module.css';

const Card = ({ featured, children }) => {
  return (
    <div className={`${styles.card} ${featured ? styles.featured : ''}`}>
      {children}
    </div>
  );
};
```

**Using classnames Library:**

```jsx
import classNames from 'classnames';
import styles from './Button.module.css';

const Button = ({ primary, large, disabled, children }) => {
  return (
    <button 
      className={classNames(
        styles.button,
        { [styles.primary]: primary },
        { [styles.large]: large },
        { [styles.disabled]: disabled }
      )}
      disabled={disabled}
    >
      {children}
    </button>
  );
};
```

**Pros:**
- Locally scoped (no conflicts)
- All CSS features
- Can compose styles
- Better organization

**Cons:**
- Slightly more complex
- Need to import styles
- Class names are generated (debugging)

---

## 4. Styled Components (CSS-in-JS)

Write CSS directly in JavaScript with template literals.

**Installation:**
```bash
npm install styled-components
```

**Basic Usage:**

```jsx
import styled from 'styled-components';

const Button = styled.button`
  background-color: blue;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  
  &:hover {
    background-color: darkblue;
  }
  
  &:active {
    transform: scale(0.98);
  }
`;

const App = () => {
  return <Button>Click Me</Button>;
};
```

**Props-based Styling:**

```jsx
import styled from 'styled-components';

const Button = styled.button`
  background-color: ${props => props.primary ? 'blue' : 'gray'};
  color: white;
  padding: ${props => props.large ? '15px 30px' : '10px 20px'};
  font-size: ${props => props.large ? '18px' : '14px'};
  border: none;
  border-radius: 4px;
  cursor: ${props => props.disabled ? 'not-allowed' : 'pointer'};
  opacity: ${props => props.disabled ? 0.6 : 1};
  
  &:hover {
    background-color: ${props => props.primary ? 'darkblue' : 'darkgray'};
  }
`;

const App = () => {
  return (
    <>
      <Button>Default</Button>
      <Button primary>Primary</Button>
      <Button large>Large</Button>
      <Button primary large>Primary Large</Button>
      <Button disabled>Disabled</Button>
    </>
  );
};
```

**Extending Styles:**

```jsx
import styled from 'styled-components';

const Button = styled.button`
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
`;

const PrimaryButton = styled(Button)`
  background-color: blue;
  color: white;
`;

const DangerButton = styled(Button)`
  background-color: red;
  color: white;
`;
```

**Styled Components with Existing Components:**

```jsx
import styled from 'styled-components';
import { Link } from 'react-router-dom';

const StyledLink = styled(Link)`
  color: blue;
  text-decoration: none;
  font-weight: bold;
  
  &:hover {
    text-decoration: underline;
  }
`;

const App = () => {
  return <StyledLink to="/about">About</StyledLink>;
};
```

**Theming:**

```jsx
import styled, { ThemeProvider } from 'styled-components';

const theme = {
  primary: '#007bff',
  secondary: '#6c757d',
  success: '#28a745',
  danger: '#dc3545'
};

const Button = styled.button`
  background-color: ${props => props.theme.primary};
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
`;

const App = () => {
  return (
    <ThemeProvider theme={theme}>
      <Button>Themed Button</Button>
    </ThemeProvider>
  );
};
```

**Pros:**
- Scoped to component
- Dynamic styling with props
- Supports all CSS features
- Theming support
- No class name conflicts

**Cons:**
- Additional dependency
- Learning curve
- Slightly larger bundle size
- Runtime overhead

---

## 5. Sass/SCSS

CSS preprocessor with variables, nesting, mixins, etc.

**Installation:**
```bash
npm install sass
```

**styles.scss:**
```scss
$primary-color: #007bff;
$secondary-color: #6c757d;
$border-radius: 4px;

.button {
  background-color: $primary-color;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: $border-radius;
  cursor: pointer;
  
  &:hover {
    background-color: darken($primary-color, 10%);
  }
  
  &.large {
    padding: 15px 30px;
    font-size: 18px;
  }
  
  &.disabled {
    background-color: $secondary-color;
    cursor: not-allowed;
  }
}

.card {
  border: 1px solid #ddd;
  border-radius: $border-radius;
  padding: 20px;
  
  .card-header {
    font-size: 24px;
    margin-bottom: 10px;
  }
  
  .card-body {
    font-size: 16px;
  }
}
```

**Component.jsx:**
```jsx
import './styles.scss';

const Button = ({ large, disabled, children }) => {
  return (
    <button 
      className={`button ${large ? 'large' : ''} ${disabled ? 'disabled' : ''}`}
      disabled={disabled}
    >
      {children}
    </button>
  );
};
```

**SCSS Modules:**

**Button.module.scss:**
```scss
$primary: #007bff;

.button {
  background-color: $primary;
  color: white;
  padding: 10px 20px;
  
  &:hover {
    background-color: darken($primary, 10%);
  }
}
```

```jsx
import styles from './Button.module.scss';

const Button = ({ children }) => {
  return <button className={styles.button}>{children}</button>;
};
```

**Pros:**
- Variables, nesting, mixins
- All CSS features
- Better organization
- Can use with CSS Modules

**Cons:**
- Requires compilation
- Learning curve
- Still global scope (unless using modules)

---

## 6. Tailwind CSS (Utility-First)

Utility-first CSS framework with pre-defined classes.

**Installation:**
```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init
```

**Basic Usage:**

```jsx
const Button = ({ primary, large, disabled, children }) => {
  return (
    <button 
      className={`
        px-4 py-2 rounded font-medium
        ${primary ? 'bg-blue-500 text-white' : 'bg-gray-300 text-gray-800'}
        ${large ? 'text-lg px-6 py-3' : 'text-sm'}
        ${disabled ? 'opacity-50 cursor-not-allowed' : 'hover:bg-blue-600'}
      `}
      disabled={disabled}
    >
      {children}
    </button>
  );
};
```

**Card Example:**

```jsx
const Card = ({ title, description, image }) => {
  return (
    <div className="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-xl transition-shadow duration-300">
      <img src={image} alt={title} className="w-full h-48 object-cover" />
      <div className="p-6">
        <h3 className="text-2xl font-bold mb-2 text-gray-800">{title}</h3>
        <p className="text-gray-600 leading-relaxed">{description}</p>
        <button className="mt-4 bg-blue-500 text-white px-6 py-2 rounded hover:bg-blue-600 transition-colors">
          Read More
        </button>
      </div>
    </div>
  );
};
```

**Responsive Design:**

```jsx
const Component = () => {
  return (
    <div className="
      w-full           // Full width on mobile
      md:w-1/2         // Half width on medium screens
      lg:w-1/3         // One-third width on large screens
      p-4              // Padding on all sides
      md:p-6           // More padding on medium+
      text-sm          // Small text on mobile
      md:text-base     // Normal text on medium+
    ">
      Content
    </div>
  );
};
```

**Using clsx for Conditional Classes:**

```jsx
import clsx from 'clsx';

const Button = ({ variant, size, disabled, children }) => {
  return (
    <button
      className={clsx(
        'rounded font-medium transition-colors',
        {
          'bg-blue-500 text-white hover:bg-blue-600': variant === 'primary',
          'bg-gray-500 text-white hover:bg-gray-600': variant === 'secondary',
          'bg-red-500 text-white hover:bg-red-600': variant === 'danger',
        },
        {
          'px-3 py-1 text-sm': size === 'small',
          'px-4 py-2 text-base': size === 'medium',
          'px-6 py-3 text-lg': size === 'large',
        },
        {
          'opacity-50 cursor-not-allowed': disabled
        }
      )}
      disabled={disabled}
    >
      {children}
    </button>
  );
};
```

**Pros:**
- Fast development
- Consistent design
- Responsive by default
- No naming conflicts
- Small bundle size (with purging)

**Cons:**
- Long className strings
- Learning utility names
- HTML can look cluttered
- Harder to reuse styles

---

## 7. Emotion (CSS-in-JS)

Another popular CSS-in-JS library, similar to styled-components.

**Installation:**
```bash
npm install @emotion/react @emotion/styled
```

**Styled Components API:**

```jsx
import styled from '@emotion/styled';

const Button = styled.button`
  background-color: ${props => props.primary ? 'blue' : 'gray'};
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    opacity: 0.8;
  }
`;

const App = () => {
  return <Button primary>Click Me</Button>;
};
```

**CSS Prop API:**

```jsx
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react';

const buttonStyle = css`
  background-color: blue;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    background-color: darkblue;
  }
`;

const App = () => {
  return <button css={buttonStyle}>Click Me</button>;
};
```

---

**Production Example: Styled Card Component**

```jsx
import styled from 'styled-components';

const CardContainer = styled.div`
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: transform 0.2s, box-shadow 0.2s;
  max-width: 400px;
  
  &:hover {
    transform: translateY(-4px);
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
  }
`;

const CardImage = styled.img`
  width: 100%;
  height: 200px;
  object-fit: cover;
`;

const CardContent = styled.div`
  padding: 20px;
`;

const CardTitle = styled.h3`
  margin: 0 0 10px 0;
  font-size: 24px;
  color: #333;
`;

const CardDescription = styled.p`
  margin: 0 0 20px 0;
  font-size: 14px;
  color: #666;
  line-height: 1.6;
`;

const CardButton = styled.button`
  background-color: ${props => props.variant === 'primary' ? '#007bff' : '#6c757d'};
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: background-color 0.2s;
  
  &:hover {
    background-color: ${props => props.variant === 'primary' ? '#0056b3' : '#5a6268'};
  }
  
  &:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }
`;

const Badge = styled.span`
  display: inline-block;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
  margin-bottom: 10px;
  background-color: ${props => {
    switch (props.type) {
      case 'success': return '#d4edda';
      case 'warning': return '#fff3cd';
      case 'danger': return '#f8d7da';
      default: return '#d1ecf1';
    }
  }};
  color: ${props => {
    switch (props.type) {
      case 'success': return '#155724';
      case 'warning': return '#856404';
      case 'danger': return '#721c24';
      default: return '#0c5460';
    }
  }};
`;

const Card = ({ image, title, description, badge, badgeType, buttonText, onButtonClick }) => {
  return (
    <CardContainer>
      <CardImage src={image} alt={title} />
      <CardContent>
        {badge && <Badge type={badgeType}>{badge}</Badge>}
        <CardTitle>{title}</CardTitle>
        <CardDescription>{description}</CardDescription>
        <CardButton variant="primary" onClick={onButtonClick}>
          {buttonText}
        </CardButton>
      </CardContent>
    </CardContainer>
  );
};

export default Card;
```

---

**Comparison Table:**

| Method | Scoped | Dynamic | CSS Features | Bundle Size | Learning Curve |
|--------|--------|---------|--------------|-------------|----------------|
| **Inline Styles** | ✅ Yes | ✅ Yes | ❌ Limited | Small | Easy |
| **CSS Stylesheets** | ❌ No | ❌ No | ✅ All | Small | Easy |
| **CSS Modules** | ✅ Yes | ❌ No | ✅ All | Small | Medium |
| **Styled Components** | ✅ Yes | ✅ Yes | ✅ All | Medium | Medium |
| **Sass/SCSS** | ❌ No* | ❌ No | ✅ All+ | Small | Medium |
| **Tailwind CSS** | ✅ Yes | ⚠️ Limited | ✅ All | Small** | Medium |
| **Emotion** | ✅ Yes | ✅ Yes | ✅ All | Medium | Medium |

*Can use with CSS Modules  
**With purging unused styles

---

**Interview Tips:**
- **Inline styles** use `style` prop with JavaScript object, camelCase properties
- **CSS Modules** provide local scoping with `.module.css` extension
- **Styled Components** write CSS in JS with template literals, supports props
- **Tailwind** uses utility classes for rapid development
- Use `className` (not `class`) in JSX
- Inline styles can't use pseudo-classes or media queries
- CSS Modules prevent class name conflicts with generated unique names
- Styled Components and Emotion enable dynamic styling based on props/state
- Modern approach: **CSS Modules** for simple projects, **Styled Components/Tailwind** for complex apps
- Can combine multiple methods (e.g., global CSS + CSS Modules)

</details>

---

### 28. What is the children prop?

<details>
<summary>View Answer</summary>

**The Children Prop**

The `children` prop is a **special prop** that allows you to pass components, elements, or text **between the opening and closing tags** of a component. It enables component composition and reusable layouts.

**Basic Concept:**

```jsx
// Parent passes content between tags
<Container>
  <h1>This is the children</h1>
</Container>

// Container receives content as children prop
const Container = ({ children }) => {
  return <div className="container">{children}</div>;
};

// Rendered output:
// <div class="container">
//   <h1>This is the children</h1>
// </div>
```

---

**Basic Examples:**

**1. Simple Text Children**

```jsx
const Button = ({ children }) => {
  return <button>{children}</button>;
};

// Usage:
<Button>Click Me</Button>
// Renders: <button>Click Me</button>

<Button>Submit Form</Button>
// Renders: <button>Submit Form</button>
```

**2. Element Children**

```jsx
const Card = ({ children }) => {
  return (
    <div className="card">
      {children}
    </div>
  );
};

// Usage:
<Card>
  <h2>Card Title</h2>
  <p>Card content goes here</p>
</Card>

// Rendered:
// <div class="card">
//   <h2>Card Title</h2>
//   <p>Card content goes here</p>
// </div>
```

**3. Component Children**

```jsx
const Layout = ({ children }) => {
  return (
    <div className="layout">
      <header>Header</header>
      <main>{children}</main>
      <footer>Footer</footer>
    </div>
  );
};

// Usage:
<Layout>
  <HomePage />  {/* Component as child */}
</Layout>

// Or multiple components:
<Layout>
  <Sidebar />
  <Content />
</Layout>
```

---

**Children vs Props:**

**Using Props:**
```jsx
const Card = ({ title, content }) => {
  return (
    <div className="card">
      <h2>{title}</h2>
      <p>{content}</p>
    </div>
  );
};

// Usage:
<Card title="Hello" content="This is content" />
```

**Using Children:**
```jsx
const Card = ({ children }) => {
  return <div className="card">{children}</div>;
};

// Usage (more flexible!):
<Card>
  <h2>Hello</h2>
  <p>This is content</p>
  <button>Click</button>
  <AnyComponent />
</Card>
```

---

**Common Patterns:**

**1. Container/Wrapper Pattern**

```jsx
const Container = ({ children }) => {
  return (
    <div style={{
      maxWidth: '1200px',
      margin: '0 auto',
      padding: '20px'
    }}>
      {children}
    </div>
  );
};

// Usage:
<Container>
  <h1>Welcome</h1>
  <p>Content goes here</p>
</Container>
```

**2. Modal Pattern**

```jsx
const Modal = ({ isOpen, onClose, children }) => {
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <button className="close-button" onClick={onClose}>×</button>
        {children}  {/* Modal content */}
      </div>
    </div>
  );
};

// Usage:
<Modal isOpen={showModal} onClose={() => setShowModal(false)}>
  <h2>Modal Title</h2>
  <p>Modal content here</p>
  <button>Save</button>
</Modal>
```

**3. Card Pattern**

```jsx
const Card = ({ children }) => {
  return (
    <div style={{
      border: '1px solid #ddd',
      borderRadius: '8px',
      padding: '20px',
      boxShadow: '0 2px 4px rgba(0,0,0,0.1)'
    }}>
      {children}
    </div>
  );
};

// Usage:
<Card>
  <h3>Product Name</h3>
  <p>Product description</p>
  <span>$99.99</span>
  <button>Buy Now</button>
</Card>
```

**4. Layout Pattern**

```jsx
const PageLayout = ({ children }) => {
  return (
    <div className="page">
      <header className="header">
        <nav>Navigation</nav>
      </header>
      
      <main className="main-content">
        {children}  {/* Page-specific content */}
      </main>
      
      <footer className="footer">
        <p>&copy; 2024 Company</p>
      </footer>
    </div>
  );
};

// Usage:
<PageLayout>
  <h1>Home Page</h1>
  <p>Welcome to our site!</p>
</PageLayout>

<PageLayout>
  <h1>About Page</h1>
  <p>About us content</p>
</PageLayout>
```

**5. Button with Icon Pattern**

```jsx
const Button = ({ icon, children, onClick }) => {
  return (
    <button onClick={onClick} style={{ display: 'flex', alignItems: 'center', gap: '8px' }}>
      {icon && <span className="icon">{icon}</span>}
      {children}  {/* Button text */}
    </button>
  );
};

// Usage:
<Button icon="🔍">Search</Button>
<Button icon="❤️">Like</Button>
<Button>No Icon</Button>
```

---

**Children with Additional Props:**

```jsx
const Alert = ({ type, children }) => {
  const styles = {
    success: { backgroundColor: '#d4edda', color: '#155724' },
    error: { backgroundColor: '#f8d7da', color: '#721c24' },
    warning: { backgroundColor: '#fff3cd', color: '#856404' }
  };
  
  return (
    <div style={{ ...styles[type], padding: '15px', borderRadius: '4px' }}>
      {children}
    </div>
  );
};

// Usage:
<Alert type="success">
  <strong>Success!</strong> Your changes have been saved.
</Alert>

<Alert type="error">
  <strong>Error!</strong> Something went wrong.
</Alert>
```

---

**Multiple Children Slots (Named Slots Pattern):**

```jsx
const Card = ({ header, footer, children }) => {
  return (
    <div className="card">
      {header && <div className="card-header">{header}</div>}
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
};

// Usage:
<Card
  header={<h2>Card Title</h2>}
  footer={<button>Action</button>}
>
  <p>This is the main content</p>
</Card>
```

**Alternative Pattern with Specific Components:**

```jsx
const Dialog = ({ children }) => {
  return <div className="dialog">{children}</div>;
};

const DialogHeader = ({ children }) => {
  return <div className="dialog-header">{children}</div>;
};

const DialogBody = ({ children }) => {
  return <div className="dialog-body">{children}</div>;
};

const DialogFooter = ({ children }) => {
  return <div className="dialog-footer">{children}</div>;
};

// Usage:
<Dialog>
  <DialogHeader>
    <h2>Confirm Action</h2>
  </DialogHeader>
  <DialogBody>
    <p>Are you sure you want to continue?</p>
  </DialogBody>
  <DialogFooter>
    <button>Cancel</button>
    <button>Confirm</button>
  </DialogFooter>
</Dialog>
```

---

**Manipulating Children:**

**1. React.Children.map**

```jsx
import { Children } from 'react';

const List = ({ children }) => {
  return (
    <ul>
      {Children.map(children, (child, index) => (
        <li key={index}>{child}</li>
      ))}
    </ul>
  );
};

// Usage:
<List>
  <span>Item 1</span>
  <span>Item 2</span>
  <span>Item 3</span>
</List>

// Rendered:
// <ul>
//   <li><span>Item 1</span></li>
//   <li><span>Item 2</span></li>
//   <li><span>Item 3</span></li>
// </ul>
```

**2. React.Children.count**

```jsx
import { Children } from 'react';

const ItemCounter = ({ children }) => {
  const count = Children.count(children);
  
  return (
    <div>
      <p>Total items: {count}</p>
      {children}
    </div>
  );
};

// Usage:
<ItemCounter>
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</ItemCounter>
// Shows: "Total items: 3"
```

**3. React.Children.only**

```jsx
import { Children } from 'react';

const SingleChildWrapper = ({ children }) => {
  // Ensures only one child is passed
  const child = Children.only(children);
  
  return <div className="wrapper">{child}</div>;
};

// ✅ Works:
<SingleChildWrapper>
  <div>One child</div>
</SingleChildWrapper>

// ❌ Throws error:
<SingleChildWrapper>
  <div>Child 1</div>
  <div>Child 2</div>
</SingleChildWrapper>
```

**4. Cloning Children with Additional Props**

```jsx
import { Children, cloneElement } from 'react';

const RadioGroup = ({ name, children }) => {
  return (
    <div>
      {Children.map(children, child => {
        // Add name prop to each child
        return cloneElement(child, { name });
      })}
    </div>
  );
};

const Radio = ({ name, value, label }) => {
  return (
    <label>
      <input type="radio" name={name} value={value} />
      {label}
    </label>
  );
};

// Usage:
<RadioGroup name="options">
  <Radio value="1" label="Option 1" />  {/* name added automatically */}
  <Radio value="2" label="Option 2" />
  <Radio value="3" label="Option 3" />
</RadioGroup>
```

---

**Conditional Children:**

```jsx
const ConditionalWrapper = ({ condition, wrapper, children }) => {
  return condition ? wrapper(children) : children;
};

// Usage:
<ConditionalWrapper
  condition={isHighlighted}
  wrapper={children => <mark>{children}</mark>}
>
  This text might be highlighted
</ConditionalWrapper>

// If isHighlighted=true: <mark>This text might be highlighted</mark>
// If isHighlighted=false: This text might be highlighted
```

---

**Function as Children (Render Props Pattern):**

```jsx
const Toggle = ({ children }) => {
  const [isOn, setIsOn] = useState(false);
  
  return children({
    isOn,
    toggle: () => setIsOn(!isOn)
  });
};

// Usage:
<Toggle>
  {({ isOn, toggle }) => (
    <div>
      <p>The switch is {isOn ? 'ON' : 'OFF'}</p>
      <button onClick={toggle}>Toggle</button>
    </div>
  )}
</Toggle>
```

---

**Production Example: Tabs Component**

```jsx
import { useState, Children, cloneElement } from 'react';

const Tabs = ({ children }) => {
  const [activeTab, setActiveTab] = useState(0);
  
  return (
    <div className="tabs">
      <div className="tab-buttons">
        {Children.map(children, (child, index) => (
          <button
            key={index}
            className={activeTab === index ? 'active' : ''}
            onClick={() => setActiveTab(index)}
          >
            {child.props.label}
          </button>
        ))}
      </div>
      
      <div className="tab-content">
        {Children.toArray(children)[activeTab]}
      </div>
    </div>
  );
};

const Tab = ({ label, children }) => {
  return <div className="tab-panel">{children}</div>;
};

// Usage:
<Tabs>
  <Tab label="Profile">
    <h2>Profile Content</h2>
    <p>User profile information</p>
  </Tab>
  
  <Tab label="Settings">
    <h2>Settings Content</h2>
    <p>User settings</p>
  </Tab>
  
  <Tab label="Messages">
    <h2>Messages Content</h2>
    <p>User messages</p>
  </Tab>
</Tabs>
```

---

**Production Example: Accordion Component**

```jsx
import { useState, Children, cloneElement } from 'react';

const Accordion = ({ children, allowMultiple = false }) => {
  const [openItems, setOpenItems] = useState([]);
  
  const toggleItem = (index) => {
    if (allowMultiple) {
      setOpenItems(prev =>
        prev.includes(index)
          ? prev.filter(i => i !== index)
          : [...prev, index]
      );
    } else {
      setOpenItems(prev =>
        prev.includes(index) ? [] : [index]
      );
    }
  };
  
  return (
    <div className="accordion">
      {Children.map(children, (child, index) =>
        cloneElement(child, {
          isOpen: openItems.includes(index),
          onToggle: () => toggleItem(index)
        })
      )}
    </div>
  );
};

const AccordionItem = ({ title, isOpen, onToggle, children }) => {
  return (
    <div className="accordion-item">
      <button className="accordion-header" onClick={onToggle}>
        {title}
        <span>{isOpen ? '▲' : '▼'}</span>
      </button>
      {isOpen && (
        <div className="accordion-content">
          {children}
        </div>
      )}
    </div>
  );
};

// Usage:
<Accordion>
  <AccordionItem title="What is React?">
    <p>React is a JavaScript library for building user interfaces.</p>
  </AccordionItem>
  
  <AccordionItem title="What is JSX?">
    <p>JSX is a syntax extension for JavaScript.</p>
  </AccordionItem>
  
  <AccordionItem title="What are components?">
    <p>Components are reusable pieces of UI.</p>
  </AccordionItem>
</Accordion>
```

---

**Children Types:**

```jsx
// String
<Component>Hello</Component>  // children = "Hello"

// Number
<Component>{42}</Component>  // children = 42

// Element
<Component><div>Content</div></Component>  // children = <div>...

// Multiple elements (array)
<Component>
  <div>One</div>
  <div>Two</div>
</Component>  // children = [<div>One</div>, <div>Two</div>]

// Component
<Component><OtherComponent /></Component>  // children = <OtherComponent />

// Function
<Component>{() => <div>Rendered</div>}</Component>

// Boolean/null/undefined (not rendered)
<Component>{true}</Component>  // Nothing rendered
<Component>{false}</Component>  // Nothing rendered
<Component>{null}</Component>  // Nothing rendered
```

---

**Best Practices:**

**1. Always Accept Children When Appropriate**
```jsx
// ✅ Good: Flexible wrapper
const Box = ({ children, ...props }) => {
  return <div className="box" {...props}>{children}</div>;
};
```

**2. Provide Default Children**
```jsx
const Button = ({ children = 'Click Me', ...props }) => {
  return <button {...props}>{children}</button>;
};
```

**3. Validate Children When Necessary**
```jsx
const SingleChild = ({ children }) => {
  if (Children.count(children) > 1) {
    console.warn('SingleChild should only have one child');
  }
  return <div>{children}</div>;
};
```

**4. Document Expected Children**
```jsx
/**
 * Tabs component
 * @param {React.ReactNode} children - Must be Tab components
 */
const Tabs = ({ children }) => {
  // Implementation
};
```

---

**Interview Tips:**
- `children` is a **special prop** that contains content between component tags
- Enables **component composition** - building complex UIs from simple components
- `children` can be **any type**: string, number, element, array, function
- Use `React.Children` utilities to manipulate children: `map`, `count`, `only`, `toArray`
- Common patterns: **wrappers**, **layouts**, **modals**, **cards**
- Use `cloneElement` to add props to children dynamically
- **Function as children** pattern (render props) passes data to children
- Makes components **reusable** and **flexible**
- Named slots pattern uses separate props for different content sections
- Children prop is **automatically passed** - you don't explicitly pass it like other props

</details>

---

### 29. What is prop drilling?

<details>
<summary>View Answer</summary>

**Prop Drilling**

Prop drilling (also called "threading") is the process of passing data from a **parent component** down through **multiple levels of nested components** to reach a **deeply nested child component** that needs the data.

**The Problem:**

Intermediate components that don't need the data must still accept and pass it down, making code harder to maintain and refactor.

---

**Visual Example:**

```jsx
// Top level - has the data
function App() {
  const [user, setUser] = useState({ name: 'John', role: 'Admin' });
  
  return <Layout user={user} />;
}

// Level 1 - doesn't need user, just passes it
function Layout({ user }) {
  return (
    <div>
      <Sidebar user={user} />  {/* Pass down */}
    </div>
  );
}

// Level 2 - doesn't need user, just passes it
function Sidebar({ user }) {
  return (
    <div>
      <Navigation user={user} />  {/* Pass down */}
    </div>
  );
}

// Level 3 - doesn't need user, just passes it
function Navigation({ user }) {
  return (
    <nav>
      <UserProfile user={user} />  {/* Pass down */}
    </nav>
  );
}

// Level 4 - finally uses the data!
function UserProfile({ user }) {
  return <div>Welcome, {user.name}!</div>;
}
```

**Problems with Prop Drilling:**

1. **Maintenance Nightmare** - Changing prop names requires updating all intermediate components
2. **Tightly Coupled** - Components become dependent on props they don't use
3. **Difficult Refactoring** - Moving components requires threading props differently
4. **Code Clutter** - Intermediate components have props they don't care about
5. **Hard to Debug** - Data flow becomes unclear with many levels

---

**Realistic Example: E-commerce App**

```jsx
// ❌ BAD: Prop drilling through 5 levels

function App() {
  const [cart, setCart] = useState([]);
  const [user, setUser] = useState({ name: 'Alice', id: 123 });
  
  const addToCart = (item) => {
    setCart([...cart, item]);
  };
  
  return (
    <HomePage 
      cart={cart} 
      user={user} 
      addToCart={addToCart} 
    />
  );
}

function HomePage({ cart, user, addToCart }) {
  return (
    <div>
      <Header cart={cart} user={user} />
      <ProductSection cart={cart} user={user} addToCart={addToCart} />
    </div>
  );
}

function ProductSection({ cart, user, addToCart }) {
  return (
    <section>
      <ProductList cart={cart} user={user} addToCart={addToCart} />
    </section>
  );
}

function ProductList({ cart, user, addToCart }) {
  const products = [
    { id: 1, name: 'Laptop', price: 999 },
    { id: 2, name: 'Phone', price: 599 }
  ];
  
  return (
    <div>
      {products.map(product => (
        <ProductCard 
          key={product.id}
          product={product}
          cart={cart}
          user={user}
          addToCart={addToCart}
        />
      ))}
    </div>
  );
}

function ProductCard({ product, cart, user, addToCart }) {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <AddToCartButton 
        product={product} 
        cart={cart} 
        user={user} 
        addToCart={addToCart} 
      />
    </div>
  );
}

function AddToCartButton({ product, cart, user, addToCart }) {
  const isInCart = cart.some(item => item.id === product.id);
  
  return (
    <button 
      onClick={() => addToCart(product)}
      disabled={isInCart}
    >
      {isInCart ? 'In Cart' : 'Add to Cart'}
    </button>
  );
}

// Problem: cart, user, and addToCart are passed through 5 levels!
// HomePage, ProductSection, ProductList, ProductCard don't use these props
// They just pass them down
```

---

**Solutions to Prop Drilling:**

## 1. Context API

Share data across the component tree without passing props.

**Creating Context:**

```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const UserContext = createContext();

// Provider component
function UserProvider({ children }) {
  const [user, setUser] = useState({ name: 'John', role: 'Admin' });
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  );
}

// Custom hook for easy access
function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}

export { UserProvider, useUser };
```

**Using Context:**

```jsx
// ✅ GOOD: No prop drilling!

function App() {
  return (
    <UserProvider>
      <Layout />
    </UserProvider>
  );
}

function Layout() {
  return (
    <div>
      <Sidebar />  {/* No props! */}
    </div>
  );
}

function Sidebar() {
  return (
    <div>
      <Navigation />  {/* No props! */}
    </div>
  );
}

function Navigation() {
  return (
    <nav>
      <UserProfile />  {/* No props! */}
    </nav>
  );
}

function UserProfile() {
  const { user } = useUser();  // Get data directly!
  return <div>Welcome, {user.name}!</div>;
}
```

**E-commerce with Context:**

```jsx
import { createContext, useContext, useState } from 'react';

// Cart Context
const CartContext = createContext();

function CartProvider({ children }) {
  const [cart, setCart] = useState([]);
  
  const addToCart = (item) => {
    setCart(prev => [...prev, item]);
  };
  
  const removeFromCart = (itemId) => {
    setCart(prev => prev.filter(item => item.id !== itemId));
  };
  
  return (
    <CartContext.Provider value={{ cart, addToCart, removeFromCart }}>
      {children}
    </CartContext.Provider>
  );
}

function useCart() {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within CartProvider');
  }
  return context;
}

// Usage
function App() {
  return (
    <CartProvider>
      <HomePage />
    </CartProvider>
  );
}

function HomePage() {
  return (
    <div>
      <Header />  {/* No props! */}
      <ProductSection />
    </div>
  );
}

function ProductSection() {
  return <ProductList />;
}

function ProductList() {
  const products = [
    { id: 1, name: 'Laptop', price: 999 },
    { id: 2, name: 'Phone', price: 599 }
  ];
  
  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

function ProductCard({ product }) {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <AddToCartButton product={product} />
    </div>
  );
}

function AddToCartButton({ product }) {
  const { cart, addToCart } = useCart();  // Direct access!
  const isInCart = cart.some(item => item.id === product.id);
  
  return (
    <button 
      onClick={() => addToCart(product)}
      disabled={isInCart}
    >
      {isInCart ? 'In Cart' : 'Add to Cart'}
    </button>
  );
}

function Header() {
  const { cart } = useCart();  // Direct access!
  
  return (
    <header>
      <h1>My Store</h1>
      <div>Cart: {cart.length} items</div>
    </header>
  );
}
```

---

## 2. Component Composition

Pass components as props instead of data.

**Before (Prop Drilling):**

```jsx
function App() {
  const [user, setUser] = useState({ name: 'John' });
  
  return <Layout user={user} />;
}

function Layout({ user }) {
  return (
    <div className="layout">
      <Sidebar user={user} />
    </div>
  );
}

function Sidebar({ user }) {
  return (
    <aside>
      <UserInfo user={user} />
    </aside>
  );
}

function UserInfo({ user }) {
  return <div>{user.name}</div>;
}
```

**After (Component Composition):**

```jsx
function App() {
  const [user, setUser] = useState({ name: 'John' });
  
  return (
    <Layout 
      sidebar={<Sidebar><UserInfo user={user} /></Sidebar>}  // Pass component
    />
  );
}

function Layout({ sidebar }) {
  return (
    <div className="layout">
      {sidebar}  {/* Just render it */}
    </div>
  );
}

function Sidebar({ children }) {
  return <aside>{children}</aside>;
}

function UserInfo({ user }) {
  return <div>{user.name}</div>;
}
```

**Another Example:**

```jsx
// Instead of drilling theme prop:
function App() {
  const theme = { primaryColor: 'blue' };
  
  return (
    <Page>
      <Header>
        <Logo theme={theme} />  {/* Direct connection */}
      </Header>
      <Content>
        <Button theme={theme} />  {/* Direct connection */}
      </Content>
    </Page>
  );
}

// Page, Header, Content don't need to know about theme!
function Page({ children }) {
  return <div className="page">{children}</div>;
}

function Header({ children }) {
  return <header>{children}</header>;
}

function Content({ children }) {
  return <main>{children}</main>;
}
```

---

## 3. State Management Libraries

Use Redux, Zustand, or Jotai for global state.

**Redux Example:**

```jsx
// store.js
import { configureStore, createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: [],
  reducers: {
    addItem: (state, action) => {
      state.push(action.payload);
    },
    removeItem: (state, action) => {
      return state.filter(item => item.id !== action.payload);
    }
  }
});

export const { addItem, removeItem } = cartSlice.actions;
export const store = configureStore({ reducer: { cart: cartSlice.reducer } });
```

```jsx
// Component
import { useSelector, useDispatch } from 'react-redux';
import { addItem } from './store';

function AddToCartButton({ product }) {
  const cart = useSelector(state => state.cart);  // Access directly
  const dispatch = useDispatch();
  
  const isInCart = cart.some(item => item.id === product.id);
  
  return (
    <button 
      onClick={() => dispatch(addItem(product))}
      disabled={isInCart}
    >
      {isInCart ? 'In Cart' : 'Add to Cart'}
    </button>
  );
}
```

**Zustand Example (Simpler):**

```jsx
import create from 'zustand';

// Create store
const useCartStore = create((set) => ({
  cart: [],
  addItem: (item) => set((state) => ({ cart: [...state.cart, item] })),
  removeItem: (id) => set((state) => ({
    cart: state.cart.filter(item => item.id !== id)
  }))
}));

// Use anywhere
function AddToCartButton({ product }) {
  const { cart, addItem } = useCartStore();  // Direct access!
  const isInCart = cart.some(item => item.id === product.id);
  
  return (
    <button onClick={() => addItem(product)} disabled={isInCart}>
      {isInCart ? 'In Cart' : 'Add to Cart'}
    </button>
  );
}

function CartCount() {
  const cart = useCartStore(state => state.cart);  // Subscribe to cart only
  return <span>Cart: {cart.length}</span>;
}
```

---

## 4. Render Props Pattern

```jsx
function DataProvider({ render }) {
  const [data, setData] = useState({ user: 'John' });
  
  return render(data);  // Pass data to render function
}

// Usage
function App() {
  return (
    <Layout>
      <Sidebar>
        <DataProvider render={(data) => <UserInfo user={data.user} />} />
      </Sidebar>
    </Layout>
  );
}

// Layout and Sidebar don't need data props
```

---

**Production Example: Theme System**

```jsx
import { createContext, useContext, useState } from 'react';

// Theme Context
const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const themeStyles = {
    light: {
      background: '#ffffff',
      text: '#000000',
      primary: '#007bff'
    },
    dark: {
      background: '#1a1a1a',
      text: '#ffffff',
      primary: '#4da6ff'
    }
  };
  
  return (
    <ThemeContext.Provider value={{
      theme,
      styles: themeStyles[theme],
      toggleTheme
    }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// App
function App() {
  return (
    <ThemeProvider>
      <Layout />
    </ThemeProvider>
  );
}

// Deep nested component - no prop drilling!
function Layout() {
  return (
    <div>
      <Header />
      <Content />
      <Footer />
    </div>
  );
}

function Header() {
  return (
    <header>
      <Navigation />
    </header>
  );
}

function Navigation() {
  return (
    <nav>
      <ThemeToggleButton />  {/* Deeply nested */}
    </nav>
  );
}

function ThemeToggleButton() {
  const { theme, toggleTheme } = useTheme();  // Direct access!
  
  return (
    <button onClick={toggleTheme}>
      Switch to {theme === 'light' ? 'Dark' : 'Light'} Mode
    </button>
  );
}

function Content() {
  const { styles } = useTheme();  // Direct access!
  
  return (
    <main style={{ 
      backgroundColor: styles.background, 
      color: styles.text 
    }}>
      <Article />
    </main>
  );
}

function Article() {
  const { styles } = useTheme();  // Direct access!
  
  return (
    <article>
      <h1 style={{ color: styles.primary }}>Article Title</h1>
      <p>Article content</p>
    </article>
  );
}

function Footer() {
  const { styles } = useTheme();  // Direct access!
  
  return (
    <footer style={{ 
      backgroundColor: styles.background, 
      color: styles.text 
    }}>
      &copy; 2024 Company
    </footer>
  );
}
```

---

**When to Use Each Solution:**

| Solution | Best For | Pros | Cons |
|----------|----------|------|------|
| **Context API** | Shared state (theme, auth, settings) | Built-in, simple API | Can cause re-renders, not for frequent updates |
| **Component Composition** | Layout components | No extra dependencies | Only works for specific patterns |
| **Redux/Zustand** | Complex global state | Powerful, devtools, middleware | More boilerplate (Redux), learning curve |
| **Render Props** | Reusable logic | Flexible | Can be harder to read |

---

**Best Practices:**

**1. Not All Drilling is Bad**

```jsx
// ✅ OK: Just 1-2 levels
function Parent() {
  const [count, setCount] = useState(0);
  return <Child count={count} setCount={setCount} />;
}

function Child({ count, setCount }) {
  return <GrandChild count={count} setCount={setCount} />;
}

// This is fine - only 2 levels
```

**2. Use Context for Truly Global Data**

```jsx
// ✅ Good use cases:
// - Theme/Dark mode
// - User authentication
// - Language/i18n
// - App configuration

// ❌ Bad use cases:
// - Form state (use local state)
// - Frequently changing data (can cause performance issues)
```

**3. Combine Solutions**

```jsx
// Context for global state
// Props for component-specific data
// Composition for layout components

function App() {
  return (
    <ThemeProvider>
      <AuthProvider>
        <Layout>
          <ProductList />  {/* Gets products via props */}
        </Layout>
      </AuthProvider>
    </ThemeProvider>
  );
}
```

---

**Anti-patterns:**

**❌ Don't Overuse Context**

```jsx
// Bad: Too many contexts
<ThemeContext>
  <UserContext>
    <CartContext>
      <NotificationContext>
        <ModalContext>
          <App />
        </ModalContext>
      </NotificationContext>
    </CartContext>
  </UserContext>
</ThemeContext>

// Better: Combine related contexts
<AppProvider>  {/* Combines theme, user, cart */}
  <App />
</AppProvider>
```

**❌ Don't Put Everything in Context**

```jsx
// Bad: Form state in context
const FormContext = createContext();

function Form() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  return (
    <FormContext.Provider value={{ email, setEmail, password, setPassword }}>
      <FormInputs />
    </FormContext.Provider>
  );
}

// Good: Keep form state local
function Form() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  return <FormInputs email={email} password={password} onChange={...} />;
}
```

---

**Interview Tips:**
- **Prop drilling** = passing props through multiple component levels to reach deeply nested child
- Problem: intermediate components become **tightly coupled** and harder to maintain
- Solutions: **Context API** (most common), **component composition**, **state management** (Redux/Zustand), **render props**
- **Context API** best for global/shared state (theme, auth, settings)
- Not all drilling is bad - **1-2 levels is acceptable**
- Avoid overusing Context - can cause **unnecessary re-renders**
- Component composition uses `children` prop to avoid drilling
- Modern approach: Context for global state, props for local data, composition for layouts
- **Custom hooks** (like `useUser`, `useCart`) make Context easier to use
- Context causes all consumers to re-render when value changes (optimization needed)

</details>

---

### 30. How do you conditionally apply CSS classes in React?

<details>
<summary>View Answer</summary>

**Conditional CSS Classes in React**

There are **several ways** to apply CSS classes conditionally based on props, state, or other conditions.

---

## 1. Template Literals (String Interpolation)

Use JavaScript template literals with conditional logic.

**Basic Conditional:**

```jsx
const Button = ({ isPrimary }) => {
  return (
    <button className={`button ${isPrimary ? 'primary' : 'secondary'}`}>
      Click Me
    </button>
  );
};

// isPrimary=true  → "button primary"
// isPrimary=false → "button secondary"
```

**Multiple Conditions:**

```jsx
const Button = ({ isPrimary, isLarge, isDisabled }) => {
  return (
    <button 
      className={`
        button 
        ${isPrimary ? 'primary' : 'secondary'} 
        ${isLarge ? 'large' : 'small'}
        ${isDisabled ? 'disabled' : ''}
      `}
    >
      Click Me
    </button>
  );
};

// Result: "button primary large disabled" (with extra spaces)
```

**Cleaning Up Spaces:**

```jsx
const Button = ({ isPrimary, isLarge, isDisabled }) => {
  const classes = [
    'button',
    isPrimary ? 'primary' : 'secondary',
    isLarge && 'large',
    isDisabled && 'disabled'
  ].filter(Boolean).join(' ');
  
  return <button className={classes}>Click Me</button>;
};

// Result: "button primary large disabled" (clean)
```

---

## 2. Conditional (Ternary) Operator

**Simple True/False:**

```jsx
const Alert = ({ isError }) => {
  return (
    <div className={isError ? 'alert-error' : 'alert-success'}>
      Message
    </div>
  );
};
```

**Add Class or Empty String:**

```jsx
const Card = ({ isFeatured }) => {
  return (
    <div className={`card ${isFeatured ? 'featured' : ''}`}>
      Content
    </div>
  );
};

// isFeatured=true  → "card featured"
// isFeatured=false → "card "
```

**Nested Ternary (Not Recommended):**

```jsx
// ❌ Avoid: Hard to read
const Badge = ({ status }) => {
  return (
    <span className={
      status === 'success' ? 'badge-success' :
      status === 'warning' ? 'badge-warning' :
      status === 'error' ? 'badge-error' :
      'badge-default'
    }>
      {status}
    </span>
  );
};
```

---

## 3. Logical AND (&&) Operator

Add a class only if condition is true.

**Single Condition:**

```jsx
const Button = ({ isActive }) => {
  return (
    <button className={`button ${isActive && 'active'}`}>
      Click Me
    </button>
  );
};

// isActive=true  → "button active"
// isActive=false → "button false" ⚠️ (adds "false" string!)
```

**Safe Version:**

```jsx
const Button = ({ isActive }) => {
  return (
    <button className={`button ${isActive ? 'active' : ''}`}>
      Click Me
    </button>
  );
};

// isActive=true  → "button active"
// isActive=false → "button "
```

**Multiple Conditions:**

```jsx
const Card = ({ isFeatured, isNew, hasDiscount }) => {
  return (
    <div className={`
      card
      ${isFeatured && 'featured'}
      ${isNew && 'new'}
      ${hasDiscount && 'discount'}
    `}>
      Content
    </div>
  );
};
```

---

## 4. Array Methods (filter/join)

Build array of classes and join them.

**Basic Pattern:**

```jsx
const Button = ({ isPrimary, isLarge, isDisabled }) => {
  const classes = [
    'button',
    isPrimary && 'primary',
    isLarge && 'large',
    isDisabled && 'disabled'
  ].filter(Boolean).join(' ');
  
  return <button className={classes}>Click Me</button>;
};

// filter(Boolean) removes false, null, undefined, 0, "", NaN
```

**Ternary in Array:**

```jsx
const Alert = ({ type, isDismissible }) => {
  const classes = [
    'alert',
    type === 'success' ? 'alert-success' : 'alert-error',
    isDismissible && 'dismissible'
  ].filter(Boolean).join(' ');
  
  return <div className={classes}>Alert message</div>;
};
```

---

## 5. Object Mapping

Map string values to class names.

**Status Mapping:**

```jsx
const Badge = ({ status }) => {
  const statusClasses = {
    success: 'badge-success',
    warning: 'badge-warning',
    error: 'badge-error',
    info: 'badge-info'
  };
  
  return (
    <span className={`badge ${statusClasses[status] || 'badge-default'}`}>
      {status}
    </span>
  );
};

// status="success" → "badge badge-success"
// status="invalid" → "badge badge-default"
```

**Size Mapping:**

```jsx
const Button = ({ size = 'medium' }) => {
  const sizeClasses = {
    small: 'btn-sm',
    medium: 'btn-md',
    large: 'btn-lg'
  };
  
  return (
    <button className={`button ${sizeClasses[size]}`}>
      Click Me
    </button>
  );
};
```

---

## 6. classnames Library (Most Popular)

Dedicated library for conditional classes.

**Installation:**
```bash
npm install classnames
```

**Basic Usage:**

```jsx
import classNames from 'classnames';

const Button = ({ isPrimary, isLarge, isDisabled }) => {
  return (
    <button className={classNames(
      'button',
      { 'primary': isPrimary },
      { 'large': isLarge },
      { 'disabled': isDisabled }
    )}>
      Click Me
    </button>
  );
};

// isPrimary=true, isLarge=false, isDisabled=true
// Result: "button primary disabled"
```

**Shorter Syntax:**

```jsx
import classNames from 'classnames';

const Button = ({ isPrimary, isLarge, isDisabled }) => {
  return (
    <button className={classNames('button', {
      primary: isPrimary,
      large: isLarge,
      disabled: isDisabled
    })}>
      Click Me
    </button>
  );
};
```

**Mixed Styles:**

```jsx
import classNames from 'classnames';

const Alert = ({ type, isDismissible, className }) => {
  return (
    <div className={classNames(
      'alert',
      `alert-${type}`,           // String interpolation
      { dismissible: isDismissible },  // Conditional
      className                   // External class
    )}>
      Alert message
    </div>
  );
};
```

**Arrays:**

```jsx
import classNames from 'classnames';

const Component = () => {
  return (
    <div className={classNames(['foo', 'bar', 'baz'])}>
      Content
    </div>
  );
};
// Result: "foo bar baz"
```

**Complex Example:**

```jsx
import classNames from 'classnames';

const Card = ({ variant, size, isActive, isDisabled, hasHover, className }) => {
  return (
    <div className={classNames(
      'card',
      `card-${variant}`,     // 'card-primary', 'card-secondary'
      `card-${size}`,        // 'card-sm', 'card-lg'
      {
        'card-active': isActive,
        'card-disabled': isDisabled,
        'card-hoverable': hasHover && !isDisabled
      },
      className              // Allow external classes
    )}>
      Card content
    </div>
  );
};

// Usage:
<Card 
  variant="primary" 
  size="lg" 
  isActive 
  hasHover 
  className="custom-card"
/>
// Result: "card card-primary card-lg card-active card-hoverable custom-card"
```

---

## 7. clsx Library (Lighter Alternative)

Similar to classnames but smaller bundle size.

**Installation:**
```bash
npm install clsx
```

**Usage (Same API):**

```jsx
import clsx from 'clsx';

const Button = ({ isPrimary, isLarge, isDisabled }) => {
  return (
    <button className={clsx('button', {
      primary: isPrimary,
      large: isLarge,
      disabled: isDisabled
    })}>
      Click Me
    </button>
  );
};
```

---

## 8. CSS Modules

Local scoping with conditional classes.

**Button.module.css:**
```css
.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
}

.primary {
  background-color: blue;
  color: white;
}

.secondary {
  background-color: gray;
  color: white;
}

.large {
  font-size: 18px;
  padding: 15px 30px;
}

.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
```

**Button.jsx:**
```jsx
import styles from './Button.module.css';
import classNames from 'classnames';

const Button = ({ isPrimary, isLarge, isDisabled, children }) => {
  return (
    <button className={classNames(
      styles.button,
      { [styles.primary]: isPrimary },
      { [styles.secondary]: !isPrimary },
      { [styles.large]: isLarge },
      { [styles.disabled]: isDisabled }
    )}>
      {children}
    </button>
  );
};
```

**Without classnames:**

```jsx
import styles from './Button.module.css';

const Button = ({ isPrimary, isLarge, isDisabled, children }) => {
  const classes = [
    styles.button,
    isPrimary ? styles.primary : styles.secondary,
    isLarge && styles.large,
    isDisabled && styles.disabled
  ].filter(Boolean).join(' ');
  
  return <button className={classes}>{children}</button>;
};
```

---

## 9. Styled Components (CSS-in-JS)

Dynamic styling with props.

```jsx
import styled from 'styled-components';

const Button = styled.button`
  padding: ${props => props.large ? '15px 30px' : '10px 20px'};
  font-size: ${props => props.large ? '18px' : '14px'};
  background-color: ${props => {
    if (props.primary) return '#007bff';
    if (props.danger) return '#dc3545';
    return '#6c757d';
  }};
  color: white;
  border: none;
  border-radius: 4px;
  opacity: ${props => props.disabled ? 0.5 : 1};
  cursor: ${props => props.disabled ? 'not-allowed' : 'pointer'};
  
  &:hover {
    opacity: ${props => props.disabled ? 0.5 : 0.8};
  }
`;

// Usage:
<Button primary large>Primary Large</Button>
<Button danger>Danger</Button>
<Button disabled>Disabled</Button>
```

---

**Production Example: Dynamic Card Component**

```jsx
import classNames from 'classnames';
import './Card.css';

const Card = ({
  variant = 'default',     // 'default', 'primary', 'success', 'danger'
  size = 'medium',         // 'small', 'medium', 'large'
  isHoverable = false,
  isElevated = false,
  isOutlined = false,
  isFeatured = false,
  isLoading = false,
  isDisabled = false,
  className,
  children
}) => {
  const cardClasses = classNames(
    'card',
    // Variant classes
    `card-${variant}`,
    // Size classes
    `card-${size}`,
    // State classes
    {
      'card-hoverable': isHoverable && !isDisabled,
      'card-elevated': isElevated,
      'card-outlined': isOutlined,
      'card-featured': isFeatured,
      'card-loading': isLoading,
      'card-disabled': isDisabled
    },
    // Custom classes
    className
  );
  
  return (
    <div className={cardClasses}>
      {isLoading && (
        <div className="card-loader">
          <span>Loading...</span>
        </div>
      )}
      <div className={classNames('card-content', {
        'card-content-blur': isLoading
      })}>
        {children}
      </div>
    </div>
  );
};

// Usage:
function App() {
  return (
    <div>
      <Card variant="primary" size="large" isHoverable isElevated>
        <h3>Featured Product</h3>
        <p>This is a featured card</p>
      </Card>
      
      <Card variant="success" isOutlined>
        <h3>Success Message</h3>
        <p>Operation completed successfully</p>
      </Card>
      
      <Card variant="danger" isDisabled>
        <h3>Disabled Card</h3>
        <p>This card is disabled</p>
      </Card>
      
      <Card isLoading>
        <h3>Loading Card</h3>
        <p>Content is loading...</p>
      </Card>
    </div>
  );
}

export default Card;
```

**Card.css:**
```css
/* Base */
.card {
  background: white;
  border-radius: 8px;
  padding: 20px;
  transition: all 0.3s ease;
}

/* Variants */
.card-default { border-left: 4px solid #ccc; }
.card-primary { border-left: 4px solid #007bff; }
.card-success { border-left: 4px solid #28a745; }
.card-danger { border-left: 4px solid #dc3545; }

/* Sizes */
.card-small { padding: 10px; font-size: 14px; }
.card-medium { padding: 20px; font-size: 16px; }
.card-large { padding: 30px; font-size: 18px; }

/* States */
.card-hoverable:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  cursor: pointer;
}

.card-elevated {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.card-outlined {
  border: 2px solid #ddd;
}

.card-featured {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.card-disabled {
  opacity: 0.5;
  pointer-events: none;
  cursor: not-allowed;
}

.card-loading {
  position: relative;
}

.card-content-blur {
  filter: blur(2px);
  opacity: 0.5;
}

.card-loader {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 10;
}
```

---

**Production Example: Toggle Button**

```jsx
import { useState } from 'react';
import classNames from 'classnames';
import './ToggleButton.css';

const ToggleButton = ({ 
  defaultChecked = false, 
  size = 'medium',
  disabled = false,
  onChange 
}) => {
  const [isChecked, setIsChecked] = useState(defaultChecked);
  
  const handleToggle = () => {
    if (disabled) return;
    setIsChecked(!isChecked);
    onChange?.(!isChecked);
  };
  
  return (
    <button
      className={classNames('toggle', `toggle-${size}`, {
        'toggle-checked': isChecked,
        'toggle-disabled': disabled
      })}
      onClick={handleToggle}
      disabled={disabled}
      role="switch"
      aria-checked={isChecked}
    >
      <span className={classNames('toggle-slider', {
        'toggle-slider-checked': isChecked
      })} />
    </button>
  );
};

export default ToggleButton;
```

---

**Comparison Table:**

| Method | Pros | Cons | Best For |
|--------|------|------|----------|
| **Template Literals** | No dependencies | Can get messy | Simple conditions |
| **Ternary Operator** | Concise | Hard to read with multiple | 2-3 conditions |
| **Logical AND** | Short syntax | Can add "false" to className | Single conditions |
| **Array Filter/Join** | Clean, readable | More code | Multiple conditions |
| **Object Mapping** | Clear mapping | Limited flexibility | Status/type mapping |
| **classnames** | Most popular, clean API | Extra dependency (2.7kb) | Complex conditionals |
| **clsx** | Smallest bundle (0.5kb) | Less features | Performance-critical |
| **CSS Modules** | Scoped styles | Need build setup | Large projects |
| **Styled Components** | Dynamic, scoped | Runtime overhead | CSS-in-JS projects |

---

**Best Practices:**

**1. Keep Base Classes Separate**

```jsx
// ✅ Good
<button className={classNames('button', { 'button-primary': isPrimary })}>

// ❌ Bad: Mixing base and conditional
<button className={classNames({ 'button': true, 'button-primary': isPrimary })}>
```

**2. Use Helper Functions for Complex Logic**

```jsx
const getButtonClasses = ({ variant, size, isDisabled, isLoading }) => {
  return classNames(
    'button',
    `button-${variant}`,
    `button-${size}`,
    {
      'button-disabled': isDisabled,
      'button-loading': isLoading
    }
  );
};

const Button = (props) => {
  return <button className={getButtonClasses(props)}>{props.children}</button>;
};
```

**3. Allow External Classes**

```jsx
// ✅ Good: Accept custom className
const Card = ({ className, ...props }) => {
  return (
    <div className={classNames('card', className)} {...props}>
      Content
    </div>
  );
};

// Usage:
<Card className="my-custom-class" />
```

---

**Interview Tips:**
- Use `className` (not `class`) in React
- **Template literals** are simplest for basic conditionals
- **classnames/clsx** libraries best for complex conditions
- Avoid logical AND (`&&`) for conditionals - can add "false" to className
- Use **object mapping** for status/type-based classes
- **CSS Modules** provide automatic scoping with `[name].module.css`
- **Styled Components** enable props-based dynamic styling
- Always filter falsy values when building class arrays
- `classNames()` ignores `false`, `null`, `undefined` automatically
- **clsx** is lighter (0.5kb) than classnames (2.7kb)
- Keep base classes separate from conditional classes for clarity
- Allow external `className` prop for component flexibility
- Use helper functions for complex class logic
- Modern approach: **classnames/clsx** + **CSS Modules** or **Tailwind CSS**

</details>
