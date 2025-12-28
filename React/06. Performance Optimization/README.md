# Performance Optimization

> Intermediate / Mid-Level (1-3 years)

---

## Questions

51. What is React.memo and when should you use it?
52. What is the difference between useMemo and React.memo?
53. How do you prevent unnecessary re-renders?
54. What is code splitting in React?
55. What is lazy loading and how do you implement it with React.lazy()?
56. What is Suspense component?
57. How do you optimize large lists in React?
58. What is virtualization and which libraries support it?
59. What are React DevTools Profiler and how do you use it?
60. What is debouncing and throttling in React context?

---

## Detailed Answers

### 51. What is React.memo and when should you use it?

<details>
<summary>View Answer</summary>

**React.memo**

`React.memo` is a **higher-order component (HOC)** that **memoizes** a component. It prevents re-renders when props haven't changed, similar to `PureComponent` for class components but for **functional components**.

---

## How React.memo Works

**Without React.memo:**
```jsx
function Child({ name, age }) {
  console.log('Child rendered');
  return (
    <div>
      {name} is {age} years old
    </div>
  );
}

function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child name="John" age={25} />
      {/* Child re-renders every time Parent re-renders, even though props didn't change */}
    </div>
  );
}
```

**With React.memo:**
```jsx
import { memo } from 'react';

const Child = memo(function Child({ name, age }) {
  console.log('Child rendered');
  return (
    <div>
      {name} is {age} years old
    </div>
  );
});

function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child name="John" age={25} />
      {/* Child will NOT re-render - props haven't changed */}
    </div>
  );
}
```

---

## Basic Syntax

```jsx
import { memo } from 'react';

// Wrap component with memo
const MyComponent = memo(function MyComponent({ prop1, prop2 }) {
  return <div>{prop1} - {prop2}</div>;
});

// Or with arrow function
const MyComponent = memo(({ prop1, prop2 }) => {
  return <div>{prop1} - {prop2}</div>;
});

// Or wrap existing component
function MyComponent({ prop1, prop2 }) {
  return <div>{prop1} - {prop2}</div>;
}

export default memo(MyComponent);
```

---

## Shallow Comparison (Default)

**React.memo does shallow comparison of props:**

```jsx
const User = memo(function User({ user }) {
  console.log('User rendered');
  return <div>{user.name}</div>;
});

function App() {
  const [count, setCount] = useState(0);
  const user = { name: 'John', age: 25 };  // New object every render!
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <User user={user} />  {/* Will re-render because user is a new object */}
    </div>
  );
}
```

**Solution: Memoize the object:**

```jsx
import { useMemo } from 'react';

function App() {
  const [count, setCount] = useState(0);
  
  // Memoize object
  const user = useMemo(() => ({ name: 'John', age: 25 }), []);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <User user={user} />  {/* Won't re-render - same object reference */}
    </div>
  );
}
```

---

## Custom Comparison Function

**Second argument: custom comparison function**

```jsx
const User = memo(
  function User({ user }) {
    return <div>{user.name} - {user.age}</div>;
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    // Return false if props are different (re-render)
    return (
      prevProps.user.name === nextProps.user.name &&
      prevProps.user.age === nextProps.user.age
    );
  }
);

// Now even with different object references, won't re-render if name and age are same
function App() {
  const [count, setCount] = useState(0);
  const user = { name: 'John', age: 25 };  // New object, but same values
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <User user={user} />  {/* Won't re-render - custom comparison checks values */}
    </div>
  );
}
```

**Note:** This is opposite of `shouldComponentUpdate`
- Return `true` = props are equal = **skip re-render**
- Return `false` = props are different = **re-render**

---

## Common Pitfalls

### 1. Inline Functions

```jsx
const Button = memo(function Button({ onClick, label }) {
  console.log('Button rendered');
  return <button onClick={onClick}>{label}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      {/* ❌ Bad: New function every render */}
      <Button onClick={() => console.log('clicked')} label="Click me" />
      
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
}

// Button re-renders every time because onClick is a new function
```

**Solution: useCallback**

```jsx
import { useCallback } from 'react';

function Parent() {
  const [count, setCount] = useState(0);
  
  // ✅ Good: Memoized function
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  
  return (
    <div>
      <Button onClick={handleClick} label="Click me" />  {/* Won't re-render */}
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
}
```

---

### 2. Inline Objects

```jsx
const User = memo(function User({ user }) {
  return <div>{user.name}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      {/* ❌ Bad: New object every render */}
      <User user={{ name: 'John', age: 25 }} />
      
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
}
```

**Solution: useMemo or move outside**

```jsx
// Option 1: Move outside component
const userData = { name: 'John', age: 25 };

function Parent() {
  return <User user={userData} />;
}

// Option 2: useMemo
function Parent() {
  const userData = useMemo(() => ({ name: 'John', age: 25 }), []);
  return <User user={userData} />;
}
```

---

### 3. Inline Arrays

```jsx
const List = memo(function List({ items }) {
  return (
    <ul>
      {items.map(item => <li key={item}>{item}</li>)}
    </ul>
  );
});

function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      {/* ❌ Bad: New array every render */}
      <List items={['a', 'b', 'c']} />
      
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
    </div>
  );
}
```

**Solution:**

```jsx
// Option 1: Move outside
const ITEMS = ['a', 'b', 'c'];

function Parent() {
  return <List items={ITEMS} />;
}

// Option 2: useMemo
function Parent() {
  const items = useMemo(() => ['a', 'b', 'c'], []);
  return <List items={items} />;
}
```

---

## When to Use React.memo

### ✅ Good Use Cases

**1. Pure components with complex rendering**

```jsx
const ComplexChart = memo(function ComplexChart({ data }) {
  // Heavy calculations or expensive rendering
  const processedData = processChartData(data);  // Expensive
  
  return <canvas>{/* Complex chart rendering */}</canvas>;
});

// Only re-renders when data actually changes
```

**2. Components that render frequently with same props**

```jsx
const MenuItem = memo(function MenuItem({ label, icon, onClick }) {
  return (
    <div className="menu-item" onClick={onClick}>
      <Icon name={icon} />
      <span>{label}</span>
    </div>
  );
});

// Menu items don't need to re-render when other items are clicked
```

**3. Large lists with individual items**

```jsx
const TodoItem = memo(function TodoItem({ todo, onToggle, onDelete }) {
  return (
    <li>
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

// When one todo updates, others don't re-render
```

**4. Components in the middle of component tree**

```jsx
const Sidebar = memo(function Sidebar({ user }) {
  return (
    <aside>
      <UserProfile user={user} />
      <Navigation />
      <Settings />
    </aside>
  );
});

// Prevents re-rendering entire sidebar when parent re-renders
```

---

### ❌ Don't Use React.memo When

**1. Props change frequently**

```jsx
// ❌ Bad: Counter changes on every click
const Counter = memo(function Counter({ count }) {
  return <div>{count}</div>;
});

// No benefit - props change every render anyway
```

**2. Simple components**

```jsx
// ❌ Bad: Overhead not worth it
const Title = memo(function Title({ text }) {
  return <h1>{text}</h1>;
});

// Re-rendering <h1> is already very fast
```

**3. Component uses context that changes frequently**

```jsx
// ❌ Bad: Context changes trigger re-render anyway
const Component = memo(function Component({ prop }) {
  const theme = useContext(ThemeContext);  // Changes frequently
  return <div>{prop} - {theme}</div>;
});

// memo is useless - context changes force re-render
```

**4. Premature optimization**

```jsx
// ❌ Bad: Optimizing without measuring
const MyComponent = memo(function MyComponent(props) {
  return <div>...</div>;
});

// Only optimize if you have performance issues
```

---

## Production Example: Dashboard

```jsx
import { memo, useState, useCallback, useMemo } from 'react';

// Expensive component - heavy rendering
const Chart = memo(function Chart({ data, type }) {
  console.log('Chart rendered');
  
  // Expensive calculation
  const processedData = useMemo(() => {
    return data.map(item => ({
      ...item,
      value: item.value * 1.1
    }));
  }, [data]);
  
  return (
    <div className="chart">
      {/* Complex chart rendering */}
      <canvas>Chart: {type}</canvas>
      {processedData.map(item => (
        <div key={item.id}>{item.value}</div>
      ))}
    </div>
  );
});

// Simple component - cheap to render
const StatCard = memo(function StatCard({ title, value, icon }) {
  console.log('StatCard rendered:', title);
  return (
    <div className="stat-card">
      <div className="icon">{icon}</div>
      <div className="title">{title}</div>
      <div className="value">{value}</div>
    </div>
  );
});

// List item component
const ActivityItem = memo(function ActivityItem({ activity, onView }) {
  console.log('ActivityItem rendered:', activity.id);
  return (
    <div className="activity-item">
      <span>{activity.description}</span>
      <button onClick={() => onView(activity.id)}>View</button>
    </div>
  );
});

function Dashboard() {
  const [selectedChart, setSelectedChart] = useState('sales');
  const [refreshKey, setRefreshKey] = useState(0);
  
  // Static data - won't change
  const statsData = useMemo(() => ([
    { title: 'Revenue', value: '$10,000', icon: '💰' },
    { title: 'Users', value: '1,234', icon: '👥' },
    { title: 'Orders', value: '567', icon: '📦' }
  ]), []);
  
  // Chart data - changes when refreshed
  const chartData = useMemo(() => ([
    { id: 1, value: 100 },
    { id: 2, value: 200 },
    { id: 3, value: 300 }
  ]), [refreshKey]);
  
  // Activities - static
  const activities = useMemo(() => ([
    { id: 1, description: 'User signed up' },
    { id: 2, description: 'Order placed' },
    { id: 3, description: 'Payment received' }
  ]), []);
  
  // Memoized callbacks
  const handleViewActivity = useCallback((id) => {
    console.log('View activity:', id);
  }, []);
  
  const handleRefresh = useCallback(() => {
    setRefreshKey(prev => prev + 1);
  }, []);
  
  return (
    <div className="dashboard">
      <h1>Dashboard</h1>
      
      {/* Stats cards - won't re-render unless data changes */}
      <div className="stats">
        {statsData.map(stat => (
          <StatCard
            key={stat.title}
            title={stat.title}
            value={stat.value}
            icon={stat.icon}
          />
        ))}
      </div>
      
      {/* Chart selector */}
      <div className="controls">
        <button onClick={() => setSelectedChart('sales')}>Sales</button>
        <button onClick={() => setSelectedChart('users')}>Users</button>
        <button onClick={handleRefresh}>Refresh Data</button>
      </div>
      
      {/* Chart - only re-renders when data or type changes */}
      <Chart data={chartData} type={selectedChart} />
      
      {/* Activity list - items only re-render when clicked */}
      <div className="activities">
        <h2>Recent Activities</h2>
        {activities.map(activity => (
          <ActivityItem
            key={activity.id}
            activity={activity}
            onView={handleViewActivity}
          />
        ))}
      </div>
    </div>
  );
}

export default Dashboard;
```

**Result:**
- Stats cards: Don't re-render when chart selection changes
- Chart: Only re-renders when data or type changes
- Activity items: Only clicked item re-renders

---

## Measuring Impact

**Use React DevTools Profiler:**

```jsx
import { Profiler } from 'react';

function App() {
  return (
    <Profiler id="Dashboard" onRender={onRender}>
      <Dashboard />
    </Profiler>
  );
}

function onRender(id, phase, actualDuration, baseDuration, startTime, commitTime) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
}
```

**Before memo:**
```
Dashboard (update) took 45ms
- Chart: 20ms
- Stats: 15ms
- Activities: 10ms
```

**After memo:**
```
Dashboard (update) took 15ms
- Chart: 0ms (skipped)
- Stats: 0ms (skipped)
- Activities: 15ms (updated)
```

---

## React.memo vs PureComponent

**PureComponent (Class Components):**
```jsx
import { PureComponent } from 'react';

class MyComponent extends PureComponent {
  render() {
    return <div>{this.props.name}</div>;
  }
}
```

**React.memo (Functional Components):**
```jsx
import { memo } from 'react';

const MyComponent = memo(function MyComponent({ name }) {
  return <div>{name}</div>;
});
```

**Both do shallow comparison of props**

---

## Best Practices

**1. Profile before optimizing**
```jsx
// Don't wrap everything in memo without reason
// Use React DevTools Profiler to identify slow components
```

**2. Combine with useCallback and useMemo**
```jsx
const Child = memo(function Child({ onClick, data }) {
  return <div onClick={onClick}>{data.text}</div>;
});

function Parent() {
  const onClick = useCallback(() => {}, []);     // Memoize callback
  const data = useMemo(() => ({ text: 'Hi' }), []); // Memoize object
  
  return <Child onClick={onClick} data={data} />;
}
```

**3. Use custom comparison for complex props**
```jsx
const Component = memo(
  ({ user }) => <div>{user.name}</div>,
  (prev, next) => prev.user.id === next.user.id
);
```

**4. Don't optimize too early**
```jsx
// ❌ Bad: Optimizing everything
const A = memo(() => <div>A</div>);
const B = memo(() => <div>B</div>);
const C = memo(() => <div>C</div>);

// ✅ Good: Optimize only slow components
const ExpensiveChart = memo(() => <Chart />);
```

---

**Interview Tips:**
- `React.memo` is a **HOC** for functional components
- Prevents re-renders when **props haven't changed**
- Does **shallow comparison** by default
- Similar to `PureComponent` for class components
- Takes **two arguments**: component and optional comparison function
- Custom comparison returns **true to skip** re-render
- Doesn't work with **context** or **state** changes
- Use with **useCallback** for function props
- Use with **useMemo** for object/array props
- Don't optimize **simple components**
- Don't optimize **frequently changing** components
- **Profile first** - don't optimize prematurely
- Useful for **expensive rendering**
- Useful for **large lists**
- Useful for **components in middle of tree**
- Comparison function is opposite of `shouldComponentUpdate`
- React.memo is **shallow** - doesn't deep compare objects
- Can cause bugs if props are **mutated** instead of replaced
- Modern pattern: optimize **strategically**, not everywhere

</details>

---

### 52. What is the difference between useMemo and React.memo?

<details>
<summary>View Answer</summary>

**useMemo vs React.memo**

Both are used for **memoization** (caching), but they work at different levels:

- **React.memo**: Memoizes entire **component**
- **useMemo**: Memoizes a **value** (result of a calculation)

---

## Quick Comparison

| Feature | React.memo | useMemo |
|---------|------------|----------|
| **What it memoizes** | Entire component | A computed value |
| **Where used** | Wraps component definition | Inside component body |
| **Prevents** | Component re-render | Recalculation of value |
| **Syntax** | HOC (wraps component) | Hook (inside component) |
| **Compares** | Props | Dependencies |
| **Returns** | Memoized component | Memoized value |

---

## React.memo (Component-Level)

**Purpose:** Prevent component from re-rendering if props haven't changed

```jsx
import { memo } from 'react';

// Memoizes the ENTIRE component
const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
  console.log('Component rendered');
  
  // This entire component won't re-render if data prop is the same
  return (
    <div>
      {data.items.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
});

function Parent() {
  const [count, setCount] = useState(0);
  const data = { items: [{ id: 1, name: 'Item 1' }] };
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ExpensiveComponent data={data} />
      {/* Component re-renders because data is a new object each time */}
    </div>
  );
}
```

---

## useMemo (Value-Level)

**Purpose:** Avoid recalculating expensive values on every render

```jsx
import { useMemo } from 'react';

function ExpensiveComponent({ items }) {
  console.log('Component rendered');
  
  // Memoizes the RESULT of the calculation
  const sortedItems = useMemo(() => {
    console.log('Sorting items...');
    return [...items].sort((a, b) => a.value - b.value);
  }, [items]);  // Only recalculates when items change
  
  return (
    <div>
      {sortedItems.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}

function Parent() {
  const [count, setCount] = useState(0);
  const items = [{ id: 1, name: 'B', value: 2 }, { id: 2, name: 'A', value: 1 }];
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ExpensiveComponent items={items} />
      {/* Component re-renders, but sorting only happens when items change */}
    </div>
  );
}
```

---

## Key Differences

### 1. What They Memoize

**React.memo - Entire Component:**
```jsx
// Prevents entire component from rendering
const Child = memo(function Child({ name }) {
  return <div>{name}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child name="John" />  {/* Child doesn't re-render */}
    </div>
  );
}
```

**useMemo - A Value:**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([1, 2, 3]);
  
  // Memoizes the doubled array
  const doubledItems = useMemo(() => {
    console.log('Doubling...');
    return items.map(item => item * 2);
  }, [items]);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      {/* Component re-renders, but doubling only happens when items change */}
      {doubledItems.join(', ')}
    </div>
  );
}
```

---

### 2. Where They're Used

**React.memo - Outside Component:**
```jsx
import { memo } from 'react';

// Wraps component definition
const MyComponent = memo(function MyComponent(props) {
  return <div>{props.text}</div>;
});

// Or
function MyComponent(props) {
  return <div>{props.text}</div>;
}

export default memo(MyComponent);
```

**useMemo - Inside Component:**
```jsx
import { useMemo } from 'react';

function MyComponent({ items }) {
  // Inside component body
  const total = useMemo(() => {
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]);
  
  return <div>Total: {total}</div>;
}
```

---

### 3. When They Run

**React.memo - Before Render:**
```jsx
const Child = memo(function Child({ name }) {
  console.log('Rendering child');  // Only logs if props changed
  return <div>{name}</div>;
});

// React.memo checks: "Did props change?"
// Yes → Render component
// No → Skip render
```

**useMemo - During Render:**
```jsx
function Component({ items }) {
  console.log('Rendering component');  // Always logs
  
  const sorted = useMemo(() => {
    console.log('Sorting');  // Only logs if items changed
    return [...items].sort();
  }, [items]);
  
  return <div>{sorted.join(', ')}</div>;
}

// Component always renders
// useMemo checks: "Did dependencies change?"
// Yes → Recalculate value
// No → Return cached value
```

---

## Using Both Together

**Common pattern: React.memo + useMemo**

```jsx
import { memo, useMemo } from 'react';

// React.memo prevents component re-render
const DataTable = memo(function DataTable({ data, sortBy }) {
  console.log('DataTable rendered');
  
  // useMemo prevents re-sorting on every render
  const sortedData = useMemo(() => {
    console.log('Sorting data...');
    return [...data].sort((a, b) => a[sortBy] - b[sortBy]);
  }, [data, sortBy]);
  
  return (
    <table>
      <tbody>
        {sortedData.map(row => (
          <tr key={row.id}>
            <td>{row.name}</td>
            <td>{row.value}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
});

function App() {
  const [count, setCount] = useState(0);
  
  // useMemo creates stable reference for props
  const data = useMemo(() => [
    { id: 1, name: 'Item 1', value: 100 },
    { id: 2, name: 'Item 2', value: 200 }
  ], []);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <DataTable data={data} sortBy="value" />
      {/* DataTable won't re-render because props haven't changed */}
      {/* If it did render, sorting would be skipped because data/sortBy haven't changed */}
    </div>
  );
}
```

---

## Complete Example: Product List

```jsx
import { useState, useMemo, memo, useCallback } from 'react';

// Child component with React.memo
const ProductItem = memo(function ProductItem({ product, onAddToCart }) {
  console.log('ProductItem rendered:', product.id);
  
  return (
    <div className="product-item">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product)}>Add to Cart</button>
    </div>
  );
});

// Parent component
function ProductList() {
  const [products] = useState([
    { id: 1, name: 'Laptop', price: 999, category: 'Electronics' },
    { id: 2, name: 'Mouse', price: 29, category: 'Electronics' },
    { id: 3, name: 'Book', price: 19, category: 'Books' },
    { id: 4, name: 'Pen', price: 2, category: 'Stationery' }
  ]);
  
  const [filter, setFilter] = useState('all');
  const [sortBy, setSortBy] = useState('name');
  const [cart, setCart] = useState([]);
  
  // useMemo: Filter products
  const filteredProducts = useMemo(() => {
    console.log('Filtering products...');
    if (filter === 'all') return products;
    return products.filter(p => p.category === filter);
  }, [products, filter]);
  
  // useMemo: Sort products
  const sortedProducts = useMemo(() => {
    console.log('Sorting products...');
    return [...filteredProducts].sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      if (sortBy === 'price') return a.price - b.price;
      return 0;
    });
  }, [filteredProducts, sortBy]);
  
  // useMemo: Calculate total
  const cartTotal = useMemo(() => {
    console.log('Calculating cart total...');
    return cart.reduce((sum, item) => sum + item.price, 0);
  }, [cart]);
  
  // useCallback: Stable function reference for React.memo
  const handleAddToCart = useCallback((product) => {
    setCart(prev => [...prev, product]);
  }, []);
  
  return (
    <div>
      <div className="controls">
        <select value={filter} onChange={(e) => setFilter(e.target.value)}>
          <option value="all">All Categories</option>
          <option value="Electronics">Electronics</option>
          <option value="Books">Books</option>
          <option value="Stationery">Stationery</option>
        </select>
        
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="name">Sort by Name</option>
          <option value="price">Sort by Price</option>
        </select>
      </div>
      
      <div className="cart-info">
        Cart Total: ${cartTotal} ({cart.length} items)
      </div>
      
      <div className="product-list">
        {sortedProducts.map(product => (
          <ProductItem
            key={product.id}
            product={product}
            onAddToCart={handleAddToCart}
          />
        ))}
      </div>
    </div>
  );
}

export default ProductList;
```

**What's optimized:**
1. **React.memo** on `ProductItem` - items don't re-render when other items change
2. **useMemo** for filtering - only filters when products or filter changes
3. **useMemo** for sorting - only sorts when filtered list or sortBy changes
4. **useMemo** for cart total - only calculates when cart changes
5. **useCallback** for handleAddToCart - stable reference for React.memo

---

## When to Use Each

### Use React.memo When:

**✅ Component is expensive to render**
```jsx
const ComplexChart = memo(function ComplexChart({ data }) {
  // Heavy rendering
  return <canvas>{/* Complex visualization */}</canvas>;
});
```

**✅ Component renders with same props often**
```jsx
const Header = memo(function Header({ title, logo }) {
  return <header>{title}</header>;
});
// Header doesn't need to re-render when page content changes
```

**✅ Component is part of large list**
```jsx
const ListItem = memo(function ListItem({ item }) {
  return <li>{item.name}</li>;
});
// Individual items don't re-render when other items change
```

---

### Use useMemo When:

**✅ Calculation is expensive**
```jsx
function Component({ data }) {
  const processed = useMemo(() => {
    // Expensive operation
    return data.map(item => complexCalculation(item));
  }, [data]);
  
  return <div>{processed}</div>;
}
```

**✅ Creating objects/arrays for props**
```jsx
function Parent() {
  // Stable reference for child's React.memo
  const config = useMemo(() => ({
    theme: 'dark',
    lang: 'en'
  }), []);
  
  return <Child config={config} />;
}
```

**✅ Value used in dependency array**
```jsx
function Component({ items }) {
  const ids = useMemo(() => items.map(item => item.id), [items]);
  
  useEffect(() => {
    fetchData(ids);  // Effect only runs when ids actually change
  }, [ids]);
}
```

---

## Common Mistakes

### ❌ Using React.memo without stable props

```jsx
const Child = memo(function Child({ data, onClick }) {
  return <div onClick={onClick}>{data.name}</div>;
});

function Parent() {
  return (
    <Child
      data={{ name: 'John' }}  // ❌ New object every render
      onClick={() => {}}        // ❌ New function every render
    />
  );
}

// React.memo is useless here - props always different
```

**✅ Fix with useMemo and useCallback:**

```jsx
function Parent() {
  const data = useMemo(() => ({ name: 'John' }), []);
  const onClick = useCallback(() => {}, []);
  
  return <Child data={data} onClick={onClick} />;
}
```

---

### ❌ Using useMemo for cheap calculations

```jsx
function Component({ a, b }) {
  // ❌ Overkill - simple addition is fast
  const sum = useMemo(() => a + b, [a, b]);
  
  return <div>{sum}</div>;
}
```

**✅ Just calculate directly:**

```jsx
function Component({ a, b }) {
  const sum = a + b;  // Simple, no memoization needed
  return <div>{sum}</div>;
}
```

---

### ❌ Forgetting dependencies

```jsx
function Component({ items, multiplier }) {
  // ❌ Missing multiplier in dependencies
  const doubled = useMemo(() => {
    return items.map(item => item * multiplier);
  }, [items]);  // Bug: doesn't update when multiplier changes
  
  return <div>{doubled.join(', ')}</div>;
}
```

**✅ Include all dependencies:**

```jsx
const doubled = useMemo(() => {
  return items.map(item => item * multiplier);
}, [items, multiplier]);  // ✅ Correct
```

---

## Performance Comparison

**Scenario: Parent re-renders 100 times**

**Without optimization:**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const data = expensiveCalculation();  // Runs 100 times
  
  return <Child data={data} />;  // Child renders 100 times
}

// Total: 100 calculations + 100 renders = 200 operations
```

**With useMemo only:**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const data = useMemo(() => expensiveCalculation(), []);  // Runs 1 time
  
  return <Child data={data} />;  // Child renders 100 times
}

// Total: 1 calculation + 100 renders = 101 operations
```

**With React.memo only:**
```jsx
const Child = memo(function Child({ data }) {
  return <div>{data}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const data = expensiveCalculation();  // Runs 100 times
  
  return <Child data={data} />;  // Child renders 1 time
}

// Total: 100 calculations + 1 render = 101 operations
```

**With both:**
```jsx
const Child = memo(function Child({ data }) {
  return <div>{data}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const data = useMemo(() => expensiveCalculation(), []);  // Runs 1 time
  
  return <Child data={data} />;  // Child renders 1 time
}

// Total: 1 calculation + 1 render = 2 operations
```

---

**Interview Tips:**
- **React.memo** memoizes **component**, **useMemo** memoizes **value**
- React.memo is **HOC**, useMemo is **hook**
- React.memo prevents **re-render**, useMemo prevents **recalculation**
- React.memo compares **props**, useMemo compares **dependencies**
- React.memo used **outside** component, useMemo used **inside**
- Use **both together** for maximum optimization
- React.memo without stable props is **useless**
- Use **useCallback** for function props with React.memo
- Use **useMemo** for object/array props with React.memo
- Don't memoize **everything** - has overhead
- Profile with **React DevTools** before optimizing
- React.memo good for **expensive components**
- useMemo good for **expensive calculations**
- Both use **shallow comparison**
- **Functional components** only - not for classes
- React.memo like **PureComponent** for functions
- useMemo has **dependency array** like useEffect
- Common mistake: **forgetting dependencies** in useMemo
- Common mistake: React.memo with **inline props**
- Modern React: optimize **strategically**, not everywhere

</details>

---

### 53. How do you prevent unnecessary re-renders?

<details>
<summary>View Answer</summary>

**Preventing Unnecessary Re-renders**

Unnecessary re-renders occur when components re-render even though their output hasn't changed. This wastes CPU and can cause performance issues.

---

## Understanding Re-renders

**What triggers a re-render?**
1. **State change** (useState, useReducer)
2. **Props change**
3. **Parent re-renders** (even if props same)
4. **Context change** (useContext)
5. **Force update** (forceUpdate in classes)

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child />  {/* Re-renders even though it has no props */}
    </div>
  );
}

function Child() {
  console.log('Child rendered');  // Logs on every Parent re-render
  return <div>I'm a child</div>;
}
```

---

## 1. React.memo (Memoize Components)

**Prevent component from re-rendering if props haven't changed**

```jsx
import { memo, useState } from 'react';

// Without memo - re-renders every time
function Child({ name }) {
  console.log('Child rendered');
  return <div>Hello {name}</div>;
}

// With memo - only re-renders when name changes
const ChildMemo = memo(function Child({ name }) {
  console.log('Child rendered');
  return <div>Hello {name}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ChildMemo name="John" />  {/* Doesn't re-render */}
    </div>
  );
}
```

---

## 2. useCallback (Memoize Functions)

**Prevent creating new function references**

```jsx
import { useState, useCallback, memo } from 'react';

const Button = memo(function Button({ onClick, label }) {
  console.log('Button rendered:', label);
  return <button onClick={onClick}>{label}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  // ❌ Bad: New function every render
  const handleClick = () => {
    console.log('Clicked');
  };
  
  // ✅ Good: Same function reference
  const handleClickMemo = useCallback(() => {
    console.log('Clicked');
  }, []);
  
  return (
    <div>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      
      <Button onClick={handleClick} label="Bad" />  {/* Re-renders */}
      <Button onClick={handleClickMemo} label="Good" />  {/* Doesn't re-render */}
    </div>
  );
}
```

---

## 3. useMemo (Memoize Values)

**Prevent creating new object/array references**

```jsx
import { useState, useMemo, memo } from 'react';

const List = memo(function List({ items }) {
  console.log('List rendered');
  return (
    <ul>
      {items.map(item => <li key={item}>{item}</li>)}
    </ul>
  );
});

function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ Bad: New array every render
  const items = ['a', 'b', 'c'];
  
  // ✅ Good: Same array reference
  const itemsMemo = useMemo(() => ['a', 'b', 'c'], []);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      
      <List items={items} />  {/* Re-renders */}
      <List items={itemsMemo} />  {/* Doesn't re-render */}
    </div>
  );
}
```

---

## 4. Split State (Colocate State)

**Move state closer to where it's used**

```jsx
// ❌ Bad: State in parent causes everything to re-render
function Parent() {
  const [name, setName] = useState('');
  const [count, setCount] = useState(0);
  
  return (
    <div>
      {/* Name input causes counter to re-render */}
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <Counter count={count} setCount={setCount} />
      <ExpensiveChart />  {/* Re-renders when name changes */}
    </div>
  );
}

// ✅ Good: Split into separate components
function NameInput() {
  const [name, setName] = useState('');
  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}

function CounterSection() {
  const [count, setCount] = useState(0);
  return <Counter count={count} setCount={setCount} />;
}

function Parent() {
  return (
    <div>
      <NameInput />  {/* Name changes don't affect others */}
      <CounterSection />
      <ExpensiveChart />  {/* Doesn't re-render */}
    </div>
  );
}
```

---

## 5. Move State Down

**Only parent of changing components needs state**

```jsx
// ❌ Bad: Sidebar re-renders when content changes
function App() {
  const [content, setContent] = useState('home');
  
  return (
    <div>
      <Sidebar />  {/* Re-renders unnecessarily */}
      <Content page={content} onNavigate={setContent} />
    </div>
  );
}

// ✅ Good: State only in Content area
function App() {
  return (
    <div>
      <Sidebar />  {/* Never re-renders */}
      <ContentArea />
    </div>
  );
}

function ContentArea() {
  const [content, setContent] = useState('home');
  return <Content page={content} onNavigate={setContent} />;
}
```

---

## 6. Lift Content Up (Component Composition)

**Pass children as props - they won't re-render**

```jsx
// ❌ Bad: ExpensiveComponent re-renders when count changes
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ExpensiveComponent />  {/* Re-renders */}
    </div>
  );
}

// ✅ Good: ExpensiveComponent passed as children
function Wrapper({ children }) {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      {children}  {/* Doesn't re-render - same object reference */}
    </div>
  );
}

function App() {
  return (
    <Wrapper>
      <ExpensiveComponent />  {/* Created in App, not in Wrapper */}
    </Wrapper>
  );
}
```

---

## 7. Context Optimization

**Split context to prevent unnecessary re-renders**

```jsx
// ❌ Bad: All consumers re-render when any value changes
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  const value = { user, setUser, theme, setTheme };
  
  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// ✅ Good: Split contexts
const UserContext = createContext();
const ThemeContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  return (
    <UserContext.Provider value={user}>
      <ThemeContext.Provider value={theme}>
        {children}
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}

// Now components only re-render when their specific context changes
function UserProfile() {
  const user = useContext(UserContext);  // Only re-renders when user changes
  return <div>{user?.name}</div>;
}

function ThemedButton() {
  const theme = useContext(ThemeContext);  // Only re-renders when theme changes
  return <button className={theme}>Click</button>;
}
```

---

## 8. Separate State and Dispatch

**For reducers, split state and dispatch contexts**

```jsx
import { createContext, useReducer, useContext } from 'react';

const StateContext = createContext();
const DispatchContext = createContext();

function AppProvider({ children }) {
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
function AddTodoButton() {
  const dispatch = useContext(DispatchContext);  // No state = no re-render
  
  return (
    <button onClick={() => dispatch({ type: 'ADD_TODO', text: 'New' })}>
      Add Todo
    </button>
  );
}

// Components using state re-render when state changes
function TodoList() {
  const state = useContext(StateContext);  // Re-renders when state changes
  const dispatch = useContext(DispatchContext);
  
  return (
    <ul>
      {state.todos.map(todo => <li key={todo.id}>{todo.text}</li>)}
    </ul>
  );
}
```

---

## 9. Use Keys Correctly

**Keys help React identify which items changed**

```jsx
// ❌ Bad: Index as key - causes re-renders when list changes
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map((todo, index) => (
        <li key={index}>{todo.text}</li>  // Bad!
      ))}
    </ul>
  );
}

// ✅ Good: Unique ID as key
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>  // Good!
      ))}
    </ul>
  );
}
```

---

## 10. Avoid Inline Styles

**Inline styles create new objects every render**

```jsx
// ❌ Bad: New style object every render
function Component() {
  return <div style={{ color: 'red', fontSize: '16px' }}>Text</div>;
}

// ✅ Good: CSS classes
function Component() {
  return <div className="text-red">Text</div>;
}

// ✅ Good: Static style object
const style = { color: 'red', fontSize: '16px' };

function Component() {
  return <div style={style}>Text</div>;
}

// ✅ Good: useMemo for dynamic styles
function Component({ isActive }) {
  const style = useMemo(() => ({
    color: isActive ? 'red' : 'blue',
    fontSize: '16px'
  }), [isActive]);
  
  return <div style={style}>Text</div>;
}
```

---

## 11. Debounce/Throttle

**Limit how often expensive operations run**

```jsx
import { useState, useCallback, useEffect } from 'react';

// Debounce: Wait for user to stop typing
function SearchInput() {
  const [input, setInput] = useState('');
  const [debouncedValue, setDebouncedValue] = useState('');
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(input);
    }, 500);  // Wait 500ms after last keystroke
    
    return () => clearTimeout(timer);
  }, [input]);
  
  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <ExpensiveSearchResults query={debouncedValue} />
    </div>
  );
}

// Throttle: Limit to once per interval
function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  
  useEffect(() => {
    let lastRun = 0;
    
    const handleScroll = () => {
      const now = Date.now();
      if (now - lastRun >= 100) {  // Max once per 100ms
        setScrollY(window.scrollY);
        lastRun = now;
      }
    };
    
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return <div>Scroll: {scrollY}px</div>;
}
```

---

## 12. Virtualization (Large Lists)

**Only render visible items**

```jsx
import { FixedSizeList } from 'react-window';

// ❌ Bad: Render 10,000 items
function LargeList({ items }) {
  return (
    <div>
      {items.map(item => <div key={item.id}>{item.name}</div>)}
    </div>
  );
}

// ✅ Good: Only render ~20 visible items
function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>{items[index].name}</div>
  );
  
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={35}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

---

## Complete Example: Optimized Dashboard

```jsx
import { useState, useMemo, useCallback, memo } from 'react';

// 1. React.memo for components
const StatCard = memo(function StatCard({ title, value, icon }) {
  console.log('StatCard rendered:', title);
  return (
    <div className="stat-card">
      <div className="icon">{icon}</div>
      <div className="title">{title}</div>
      <div className="value">{value}</div>
    </div>
  );
});

const Chart = memo(function Chart({ data }) {
  console.log('Chart rendered');
  
  // Expensive calculation
  const processedData = useMemo(() => {
    console.log('Processing chart data...');
    return data.map(item => ({ ...item, value: item.value * 1.1 }));
  }, [data]);
  
  return <div className="chart">Chart with {processedData.length} points</div>;
});

// 2. Split state into separate components
function SearchBar() {
  const [query, setQuery] = useState('');
  console.log('SearchBar rendered');
  
  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  );
}

function FilterPanel() {
  const [category, setCategory] = useState('all');
  console.log('FilterPanel rendered');
  
  return (
    <select value={category} onChange={(e) => setCategory(e.target.value)}>
      <option value="all">All</option>
      <option value="active">Active</option>
      <option value="inactive">Inactive</option>
    </select>
  );
}

// Main component
function Dashboard() {
  const [refreshKey, setRefreshKey] = useState(0);
  
  // 3. useMemo for stable references
  const stats = useMemo(() => [
    { title: 'Revenue', value: '$10,000', icon: '💰' },
    { title: 'Users', value: '1,234', icon: '👥' },
    { title: 'Orders', value: '567', icon: '📦' }
  ], []);
  
  const chartData = useMemo(() => [
    { id: 1, value: 100 },
    { id: 2, value: 200 },
    { id: 3, value: 300 }
  ], [refreshKey]);
  
  // 4. useCallback for stable function references
  const handleRefresh = useCallback(() => {
    setRefreshKey(prev => prev + 1);
  }, []);
  
  return (
    <div className="dashboard">
      <h1>Dashboard</h1>
      
      {/* 5. Separate components with their own state */}
      <div className="toolbar">
        <SearchBar />  {/* SearchBar state doesn't affect others */}
        <FilterPanel />  {/* FilterPanel state doesn't affect others */}
        <button onClick={handleRefresh}>Refresh</button>
      </div>
      
      {/* 6. Memoized components with stable props */}
      <div className="stats">
        {stats.map(stat => (
          <StatCard key={stat.title} {...stat} />
        ))}
      </div>
      
      <Chart data={chartData} />
    </div>
  );
}

export default Dashboard;
```

**Optimizations applied:**
1. ✅ React.memo on StatCard and Chart
2. ✅ Split state (SearchBar, FilterPanel separate)
3. ✅ useMemo for stats and chartData
4. ✅ useCallback for handleRefresh
5. ✅ Each component manages own state
6. ✅ No inline objects/functions in JSX

---

## Debugging Re-renders

**1. Use console.log**
```jsx
function Component() {
  console.log('Component rendered');
  return <div>Content</div>;
}
```

**2. Use React DevTools Profiler**
- Open React DevTools
- Go to Profiler tab
- Click record
- Interact with app
- See which components rendered and why

**3. Use why-did-you-render library**
```bash
npm install @welldone-software/why-did-you-render
```

```jsx
import whyDidYouRender from '@welldone-software/why-did-you-render';

if (process.env.NODE_ENV === 'development') {
  whyDidYouRender(React, {
    trackAllPureComponents: true,
  });
}
```

**4. Custom hook to track renders**
```jsx
function useRenderCount(componentName) {
  const renderCount = useRef(0);
  
  useEffect(() => {
    renderCount.current++;
    console.log(`${componentName} rendered ${renderCount.current} times`);
  });
}

function MyComponent() {
  useRenderCount('MyComponent');
  return <div>Content</div>;
}
```

---

## Common Mistakes

**1. Inline functions in props**
```jsx
// ❌ Bad
<Button onClick={() => console.log('clicked')} />

// ✅ Good
const handleClick = useCallback(() => console.log('clicked'), []);
<Button onClick={handleClick} />
```

**2. Inline objects in props**
```jsx
// ❌ Bad
<Component style={{ color: 'red' }} />

// ✅ Good
const style = useMemo(() => ({ color: 'red' }), []);
<Component style={style} />
```

**3. Using index as key**
```jsx
// ❌ Bad
{items.map((item, i) => <div key={i}>{item}</div>)}

// ✅ Good
{items.map(item => <div key={item.id}>{item}</div>)}
```

**4. Not splitting state**
```jsx
// ❌ Bad: All state in parent
function Parent() {
  const [a, setA] = useState();
  const [b, setB] = useState();
  const [c, setC] = useState();
  // Everything re-renders when any state changes
}

// ✅ Good: Split into components
function ParentA() {
  const [a, setA] = useState();
}
function ParentB() {
  const [b, setB] = useState();
}
```

---

## Quick Reference

| Problem | Solution |
|---------|----------|
| Component re-renders with same props | `React.memo` |
| Function prop changes every render | `useCallback` |
| Object/array prop changes every render | `useMemo` |
| Parent state affects unrelated children | Split state into separate components |
| Context causes all consumers to re-render | Split context by concern |
| Large list renders slowly | Virtualization (react-window) |
| Expensive calculation on every render | `useMemo` |
| Input causes entire form to re-render | Move input to separate component |

---

**Interview Tips:**
- Re-renders happen when **state**, **props**, or **context** changes
- **Parent re-renders** cause child re-renders by default
- Use **React.memo** to prevent component re-renders
- Use **useCallback** to memoize functions
- Use **useMemo** to memoize values (objects/arrays)
- **Split state** - move it closer to where it's used
- **Lift content up** - pass as children to avoid re-renders
- **Split context** - separate concerns to reduce re-renders
- **Separate state and dispatch** contexts
- Use **correct keys** (not index) in lists
- Avoid **inline styles** - use CSS classes
- **Debounce** inputs, **throttle** scroll handlers
- Use **virtualization** for large lists (react-window)
- **Profile first** with React DevTools before optimizing
- Don't optimize **everything** - has overhead
- Most common issue: **inline functions/objects** in props
- **Component composition** > memoization when possible
- Modern React is already **quite fast** - optimize when needed
- Premature optimization is **root of all evil**

</details>

---

### 54. What is code splitting in React?

<details>
<summary>View Answer</summary>

**Code Splitting**

Code splitting is a technique to **split your application into smaller bundles** that can be loaded on demand, rather than loading everything upfront. This improves initial load time.

**Problem:** Large JavaScript bundle = slow initial load
**Solution:** Split into smaller chunks, load only what's needed

---

## The Problem: Large Bundle

**Without code splitting:**
```
┌───────────────────────────────────────────┐
│  app.bundle.js (5 MB)                      │
│  - Home page                               │
│  - About page                              │
│  - Dashboard (only for logged in users)    │
│  - Admin panel (only for admins)           │
│  - Profile editor                          │
│  - Settings                                │
│  - ... everything else ...                 │
└───────────────────────────────────────────┘
         ↓
User downloads 5 MB just to see home page!
```

**With code splitting:**
```
┌───────────────────────────────────────────┐
│  main.bundle.js (200 KB)                   │  ← Initial load
│  - App shell                               │
│  - Router                                  │
│  - Home page                               │
└───────────────────────────────────────────┘

┌───────────────────────────────────────────┐
│  dashboard.chunk.js (500 KB)               │  ← Loaded when user
└───────────────────────────────────────────┘     navigates to
                                                   dashboard
┌───────────────────────────────────────────┐
│  admin.chunk.js (300 KB)                   │  ← Loaded only
└───────────────────────────────────────────┘     for admins

User downloads 200 KB initially, more as needed!
```

---

## 1. Dynamic Import

**Standard import (static):**
```jsx
// Bundled immediately
import Dashboard from './Dashboard';

function App() {
  return <Dashboard />;
}
```

**Dynamic import (code splitting):**
```jsx
// Loaded on demand
const Dashboard = () => {
  return import('./Dashboard');  // Returns a Promise
};

// Later...
Dashboard().then(module => {
  const DashboardComponent = module.default;
  // Use component
});
```

---

## 2. React.lazy()

**Basic usage:**

```jsx
import { lazy, Suspense } from 'react';

// Static import - bundled immediately
import Home from './Home';

// Dynamic import - loaded on demand
const Dashboard = lazy(() => import('./Dashboard'));
const Profile = lazy(() => import('./Profile'));
const Settings = lazy(() => import('./Settings'));

function App() {
  return (
    <div>
      <Home />  {/* Always available */}
      
      {/* Lazy loaded components wrapped in Suspense */}
      <Suspense fallback={<div>Loading Dashboard...</div>}>
        <Dashboard />
      </Suspense>
      
      <Suspense fallback={<div>Loading Profile...</div>}>
        <Profile />
      </Suspense>
    </div>
  );
}
```

---

## 3. Route-Based Code Splitting

**Most common pattern: Split by route**

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

// Static imports (always needed)
import Header from './Header';
import Footer from './Footer';

// Lazy load routes
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));
const Settings = lazy(() => import('./pages/Settings'));
const Admin = lazy(() => import('./pages/Admin'));

function App() {
  return (
    <BrowserRouter>
      <Header />
      
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/dashboard">Dashboard</Link>
      </nav>
      
      <Suspense fallback={<div className="loading">Loading page...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/admin" element={<Admin />} />
        </Routes>
      </Suspense>
      
      <Footer />
    </BrowserRouter>
  );
}

export default App;
```

**Result:**
- Initial load: `main.js` + `Home.js` only
- Navigate to /about: Download `About.js`
- Navigate to /dashboard: Download `Dashboard.js`
- etc.

---

## 4. Component-Based Code Splitting

**Split large components:**

```jsx
import { lazy, Suspense, useState } from 'react';

// Heavy components loaded on demand
const HeavyChart = lazy(() => import('./HeavyChart'));
const HeavyTable = lazy(() => import('./HeavyTable'));
const VideoPlayer = lazy(() => import('./VideoPlayer'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  const [showTable, setShowTable] = useState(false);
  
  return (
    <div>
      <h1>Dashboard</h1>
      
      <button onClick={() => setShowChart(!showChart)}>Toggle Chart</button>
      <button onClick={() => setShowTable(!showTable)}>Toggle Table</button>
      
      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart />
        </Suspense>
      )}
      
      {showTable && (
        <Suspense fallback={<div>Loading table...</div>}>
          <HeavyTable />
        </Suspense>
      )}
    </div>
  );
}
```

---

## 5. Modal/Dialog Code Splitting

**Load modals only when opened:**

```jsx
import { lazy, Suspense, useState } from 'react';

const LoginModal = lazy(() => import('./LoginModal'));
const SignupModal = lazy(() => import('./SignupModal'));
const SettingsModal = lazy(() => import('./SettingsModal'));

function App() {
  const [showLogin, setShowLogin] = useState(false);
  const [showSignup, setShowSignup] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowLogin(true)}>Login</button>
      <button onClick={() => setShowSignup(true)}>Sign Up</button>
      
      {/* Only load modal code when opened */}
      {showLogin && (
        <Suspense fallback={<div>Loading...</div>}>
          <LoginModal onClose={() => setShowLogin(false)} />
        </Suspense>
      )}
      
      {showSignup && (
        <Suspense fallback={<div>Loading...</div>}>
          <SignupModal onClose={() => setShowSignup(false)} />
        </Suspense>
      )}
    </div>
  );
}
```

---

## 6. Library Code Splitting

**Split heavy third-party libraries:**

```jsx
import { lazy, Suspense, useState } from 'react';

// Heavy library - load on demand
const MarkdownEditor = lazy(() => import('./MarkdownEditor'));  // Contains react-markdown

function App() {
  const [showEditor, setShowEditor] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowEditor(true)}>Write Article</button>
      
      {showEditor && (
        <Suspense fallback={<div>Loading editor...</div>}>
          <MarkdownEditor />
        </Suspense>
      )}
    </div>
  );
}
```

```jsx
// MarkdownEditor.jsx
import ReactMarkdown from 'react-markdown';  // Heavy library
import 'highlight.js/styles/github.css';  // Syntax highlighting

function MarkdownEditor() {
  const [content, setContent] = useState('');
  
  return (
    <div>
      <textarea value={content} onChange={(e) => setContent(e.target.value)} />
      <ReactMarkdown>{content}</ReactMarkdown>
    </div>
  );
}

export default MarkdownEditor;
```

---

## 7. Named Exports with lazy()

**Default export (works):**
```jsx
// Component.jsx
export default function Component() {
  return <div>Component</div>;
}

// App.jsx
const Component = lazy(() => import('./Component'));  // ✅ Works
```

**Named export (needs extra step):**
```jsx
// Component.jsx
export function Component() {  // Named export
  return <div>Component</div>;
}

// App.jsx - ❌ Doesn't work
const Component = lazy(() => import('./Component'));

// App.jsx - ✅ Works
const Component = lazy(() => 
  import('./Component').then(module => ({ default: module.Component }))
);
```

---

## 8. Error Boundaries with Code Splitting

**Handle loading errors:**

```jsx
import { Component } from 'react';

class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error loading component:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Failed to load component</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try Again
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
function App() {
  const Dashboard = lazy(() => import('./Dashboard'));
  
  return (
    <ErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <Dashboard />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

## 9. Preloading

**Preload chunks before they're needed:**

```jsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./Dashboard'));

function App() {
  // Preload on hover
  const handleMouseEnter = () => {
    import('./Dashboard');  // Starts loading immediately
  };
  
  return (
    <div>
      <Link
        to="/dashboard"
        onMouseEnter={handleMouseEnter}  // Preload on hover
      >
        Dashboard
      </Link>
      
      <Suspense fallback={<div>Loading...</div>}>
        <Dashboard />
      </Suspense>
    </div>
  );
}
```

**Preload on mount:**
```jsx
import { useEffect } from 'react';

function Home() {
  useEffect(() => {
    // Preload dashboard in background
    import('./Dashboard');
  }, []);
  
  return <div>Home Page</div>;
}
```

---

## 10. webpack Magic Comments

**Control chunk names and loading:**

```jsx
// Name the chunk
const Dashboard = lazy(() => 
  import(
    /* webpackChunkName: "dashboard" */
    './Dashboard'
  )
);
// Creates: dashboard.chunk.js

// Prefetch (download during idle time)
const Profile = lazy(() => 
  import(
    /* webpackPrefetch: true */
    './Profile'
  )
);

// Preload (download immediately)
const Settings = lazy(() => 
  import(
    /* webpackPreload: true */
    './Settings'
  )
);
```

---

## Complete Example: E-commerce App

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import Header from './Header';  // Always needed
import Footer from './Footer';  // Always needed

// Lazy load pages
const Home = lazy(() => 
  import(/* webpackChunkName: "home" */ './pages/Home')
);

const Products = lazy(() => 
  import(/* webpackChunkName: "products" */ './pages/Products')
);

const ProductDetail = lazy(() => 
  import(/* webpackChunkName: "product-detail" */ './pages/ProductDetail')
);

const Cart = lazy(() => 
  import(/* webpackChunkName: "cart" */ './pages/Cart')
);

const Checkout = lazy(() => 
  import(/* webpackChunkName: "checkout" */ './pages/Checkout')
);

// Admin section - only for admins
const AdminDashboard = lazy(() => 
  import(/* webpackChunkName: "admin" */ './pages/admin/Dashboard')
);

const AdminProducts = lazy(() => 
  import(/* webpackChunkName: "admin" */ './pages/admin/Products')
);

const AdminOrders = lazy(() => 
  import(/* webpackChunkName: "admin" */ './pages/admin/Orders')
);

// Loading component
function LoadingSpinner() {
  return (
    <div className="loading-spinner">
      <div className="spinner"></div>
      <p>Loading...</p>
    </div>
  );
}

function App() {
  return (
    <BrowserRouter>
      <div className="app">
        <Header />
        
        <main>
          <Suspense fallback={<LoadingSpinner />}>
            <Routes>
              <Route path="/" element={<Home />} />
              <Route path="/products" element={<Products />} />
              <Route path="/products/:id" element={<ProductDetail />} />
              <Route path="/cart" element={<Cart />} />
              <Route path="/checkout" element={<Checkout />} />
              
              {/* Admin routes - separate bundle */}
              <Route path="/admin" element={<AdminDashboard />} />
              <Route path="/admin/products" element={<AdminProducts />} />
              <Route path="/admin/orders" element={<AdminOrders />} />
            </Routes>
          </Suspense>
        </main>
        
        <Footer />
      </div>
    </BrowserRouter>
  );
}

export default App;
```

**Bundle output:**
```
main.js                - 150 KB  (App shell, Header, Footer)
home.chunk.js          - 20 KB
products.chunk.js      - 50 KB
product-detail.chunk.js- 30 KB
cart.chunk.js          - 40 KB
checkout.chunk.js      - 60 KB
admin.chunk.js         - 100 KB  (All admin pages together)
```

---

## Benefits of Code Splitting

**1. Faster Initial Load**
```
Without splitting: 5 MB download = 10 seconds
With splitting: 200 KB download = 2 seconds
```

**2. Better Caching**
- Main bundle rarely changes
- Page bundles change independently
- Browser can cache each chunk separately

**3. Pay for What You Use**
- Regular users: Don't download admin code
- Mobile users: Don't download desktop features
- Free tier users: Don't download premium features

**4. Improved Perceived Performance**
- App shell loads fast
- Progressive enhancement
- User sees something quickly

---

## Best Practices

**1. Split by route**
```jsx
// ✅ Good: Each route is a separate bundle
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
```

**2. Split large dependencies**
```jsx
// ✅ Good: Heavy chart library only when needed
const ChartPage = lazy(() => import('./ChartPage'));  // Contains Chart.js
```

**3. Group related code**
```jsx
// ✅ Good: All admin pages in one chunk
const Admin = lazy(() => import('./admin'));  // Admin index that imports all admin pages
```

**4. Don't split too much**
```jsx
// ❌ Bad: Too many small chunks = too many requests
const Button = lazy(() => import('./Button'));
const Input = lazy(() => import('./Input'));
const Label = lazy(() => import('./Label'));

// ✅ Good: Keep common components in main bundle
import Button from './Button';
import Input from './Input';
import Label from './Label';
```

**5. Always use Suspense**
```jsx
// ❌ Bad: No fallback
const Dashboard = lazy(() => import('./Dashboard'));
<Dashboard />  // Error!

// ✅ Good: With Suspense
<Suspense fallback={<Loading />}>
  <Dashboard />
</Suspense>
```

---

## Measuring Impact

**Before code splitting:**
```
Initial bundle size: 2.5 MB
Load time: 8 seconds (3G)
Time to Interactive: 10 seconds
```

**After code splitting:**
```
Initial bundle size: 300 KB
Load time: 2 seconds (3G)
Time to Interactive: 3 seconds
```

**Use webpack-bundle-analyzer:**
```bash
npm install --save-dev webpack-bundle-analyzer
```

```js
// webpack.config.js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin()
  ]
};
```

---

**Interview Tips:**
- Code splitting = **breaking bundle into smaller chunks**
- Improves **initial load time**
- Use **React.lazy()** for components
- Always wrap with **Suspense** for fallback
- Most common: **route-based** code splitting
- Also split **large components** (charts, editors)
- Split **modals/dialogs** (load when opened)
- Split **admin sections** (not for all users)
- Can **preload** chunks on hover/mount
- **Dynamic import()** returns Promise
- React.lazy() only works with **default exports**
- Named exports need **.then()** transformation
- Use **Error Boundaries** to handle load failures
- webpack magic comments control **chunk names**
- **webpackPrefetch** = download during idle time
- **webpackPreload** = download immediately
- Don't split **too much** - balance needed
- Split by **route** is most effective
- **Group related code** in same chunk
- Keep **common components** in main bundle
- Use **webpack-bundle-analyzer** to visualize
- Modern React apps should **always use** code splitting
- Reduces bundle from **MB to KB**
- Better **caching** - chunks change independently

</details>

---

### 55. What is lazy loading and how do you implement it with React.lazy()?

<details>
<summary>View Answer</summary>

**Lazy Loading**

Lazy loading is a technique to **defer loading of components until they're actually needed**. Instead of loading all components upfront, you load them on-demand, reducing initial bundle size and improving load time.

**Key Concept:** Load what you need, when you need it.

---

## The Problem

**Without lazy loading:**
```jsx
import Home from './Home';        // 50 KB
import About from './About';      // 30 KB
import Dashboard from './Dashboard';  // 500 KB
import Admin from './Admin';      // 200 KB

function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
      <Route path="/dashboard" element={<Dashboard />} />
      <Route path="/admin" element={<Admin />} />
    </Routes>
  );
}

// User visits homepage
// Downloads: 50 + 30 + 500 + 200 = 780 KB
// But only uses: 50 KB (Home)
// Wasted: 730 KB
```

**With lazy loading:**
```jsx
import { lazy } from 'react';

// Only Home is loaded initially
import Home from './Home';  // 50 KB

// These load on-demand
const About = lazy(() => import('./About'));         // 30 KB
const Dashboard = lazy(() => import('./Dashboard')); // 500 KB
const Admin = lazy(() => import('./Admin'));         // 200 KB

// User visits homepage
// Downloads: 50 KB
// Navigates to /dashboard
// Downloads: 500 KB more
// Total so far: 550 KB (only what's needed)
```

---

## React.lazy() Syntax

**Basic usage:**

```jsx
import { lazy, Suspense } from 'react';

// Dynamic import with React.lazy()
const MyComponent = lazy(() => import('./MyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <MyComponent />
    </Suspense>
  );
}
```

**Key points:**
1. `lazy()` takes a function that returns `import()`
2. `import()` returns a Promise
3. Must wrap with `<Suspense>` for fallback
4. Works only with **default exports**

---

## 1. Basic Example

```jsx
import { lazy, Suspense, useState } from 'react';

// Regular import (always loaded)
import Header from './Header';

// Lazy import (loaded on demand)
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  const [show, setShow] = useState(false);
  
  return (
    <div>
      <Header />
      
      <button onClick={() => setShow(!show)}>Toggle Heavy Component</button>
      
      {show && (
        <Suspense fallback={<div>Loading heavy component...</div>}>
          <HeavyComponent />
        </Suspense>
      )}
    </div>
  );
}
```

**What happens:**
1. App loads → Header loads immediately
2. User clicks button → `HeavyComponent` starts downloading
3. Shows "Loading heavy component..." during download
4. Once loaded → Renders HeavyComponent
5. Next time button clicked → Component already cached, renders immediately

---

## 2. Route-Based Lazy Loading

**Most common pattern:**

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

// Eager loading (always needed)
import Navbar from './Navbar';

// Lazy loading (per route)
const Home = lazy(() => import('./pages/Home'));
const Products = lazy(() => import('./pages/Products'));
const ProductDetail = lazy(() => import('./pages/ProductDetail'));
const Cart = lazy(() => import('./pages/Cart'));
const Checkout = lazy(() => import('./pages/Checkout'));
const Profile = lazy(() => import('./pages/Profile'));

// Loading spinner component
function PageLoader() {
  return (
    <div className="page-loader">
      <div className="spinner"></div>
      <p>Loading page...</p>
    </div>
  );
}

function App() {
  return (
    <BrowserRouter>
      <Navbar />
      
      <main>
        <Suspense fallback={<PageLoader />}>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/products" element={<Products />} />
            <Route path="/products/:id" element={<ProductDetail />} />
            <Route path="/cart" element={<Cart />} />
            <Route path="/checkout" element={<Checkout />} />
            <Route path="/profile" element={<Profile />} />
          </Routes>
        </Suspense>
      </main>
    </BrowserRouter>
  );
}

export default App;
```

---

## 3. Conditional Lazy Loading

**Load based on conditions:**

```jsx
import { lazy, Suspense, useState } from 'react';

// Lazy load components
const AdminPanel = lazy(() => import('./AdminPanel'));
const UserDashboard = lazy(() => import('./UserDashboard'));

function App() {
  const [user] = useState({ role: 'admin', name: 'John' });
  
  return (
    <div>
      <h1>Welcome {user.name}</h1>
      
      <Suspense fallback={<div>Loading dashboard...</div>}>
        {user.role === 'admin' ? (
          <AdminPanel />  {/* Only admins download this */}
        ) : (
          <UserDashboard />  {/* Only regular users download this */}
        )}
      </Suspense>
    </div>
  );
}
```

---

## 4. Modal/Dialog Lazy Loading

**Load modals only when opened:**

```jsx
import { lazy, Suspense, useState } from 'react';

const LoginModal = lazy(() => import('./LoginModal'));
const SignupModal = lazy(() => import('./SignupModal'));
const SettingsModal = lazy(() => import('./SettingsModal'));

function App() {
  const [activeModal, setActiveModal] = useState(null);
  
  return (
    <div>
      <nav>
        <button onClick={() => setActiveModal('login')}>Login</button>
        <button onClick={() => setActiveModal('signup')}>Sign Up</button>
        <button onClick={() => setActiveModal('settings')}>Settings</button>
      </nav>
      
      <Suspense fallback={<div>Loading...</div>}>
        {activeModal === 'login' && (
          <LoginModal onClose={() => setActiveModal(null)} />
        )}
        {activeModal === 'signup' && (
          <SignupModal onClose={() => setActiveModal(null)} />
        )}
        {activeModal === 'settings' && (
          <SettingsModal onClose={() => setActiveModal(null)} />
        )}
      </Suspense>
    </div>
  );
}
```

---

## 5. Tab-Based Lazy Loading

**Load tab content on demand:**

```jsx
import { lazy, Suspense, useState } from 'react';

const Overview = lazy(() => import('./tabs/Overview'));
const Analytics = lazy(() => import('./tabs/Analytics'));
const Reports = lazy(() => import('./tabs/Reports'));
const Settings = lazy(() => import('./tabs/Settings'));

function Dashboard() {
  const [activeTab, setActiveTab] = useState('overview');
  
  return (
    <div>
      <div className="tabs">
        <button onClick={() => setActiveTab('overview')}>Overview</button>
        <button onClick={() => setActiveTab('analytics')}>Analytics</button>
        <button onClick={() => setActiveTab('reports')}>Reports</button>
        <button onClick={() => setActiveTab('settings')}>Settings</button>
      </div>
      
      <div className="tab-content">
        <Suspense fallback={<div>Loading tab...</div>}>
          {activeTab === 'overview' && <Overview />}
          {activeTab === 'analytics' && <Analytics />}
          {activeTab === 'reports' && <Reports />}
          {activeTab === 'settings' && <Settings />}
        </Suspense>
      </div>
    </div>
  );
}
```

---

## 6. Named Exports with lazy()

**Default export (works directly):**

```jsx
// Component.jsx
export default function Component() {
  return <div>Component</div>;
}

// App.jsx
const Component = lazy(() => import('./Component'));  // ✅ Works
```

**Named export (needs transformation):**

```jsx
// Component.jsx
export function Component() {  // Named export
  return <div>Component</div>;
}

// App.jsx - ❌ Doesn't work
const Component = lazy(() => import('./Component'));

// App.jsx - ✅ Works
const Component = lazy(() => 
  import('./Component').then(module => ({
    default: module.Component
  }))
);

// Or create a wrapper file
// ComponentWrapper.jsx
export { Component as default } from './Component';

// App.jsx
const Component = lazy(() => import('./ComponentWrapper'));
```

---

## 7. Multiple Components from One File

**Load multiple components together:**

```jsx
// components/Forms.jsx
export function LoginForm() {
  return <form>Login Form</form>;
}

export function SignupForm() {
  return <form>Signup Form</form>;
}

export function PasswordResetForm() {
  return <form>Password Reset Form</form>;
}

// App.jsx - Load all forms together
const LoginForm = lazy(() => 
  import('./components/Forms').then(module => ({
    default: module.LoginForm
  }))
);

const SignupForm = lazy(() => 
  import('./components/Forms').then(module => ({
    default: module.SignupForm
  }))
);

// First form loads entire file
// Second form uses cached file - instant
```

---

## 8. Preloading

**Start loading before component is needed:**

```jsx
import { lazy, Suspense } from 'react';
import { Link } from 'react-router-dom';

const Dashboard = lazy(() => import('./Dashboard'));

function HomePage() {
  // Preload on hover
  const preloadDashboard = () => {
    import('./Dashboard');  // Starts loading immediately
  };
  
  // Preload on mount (after a delay)
  useEffect(() => {
    const timer = setTimeout(() => {
      import('./Dashboard');  // Preload in background
    }, 2000);  // After 2 seconds
    
    return () => clearTimeout(timer);
  }, []);
  
  return (
    <div>
      <h1>Home Page</h1>
      
      <Link 
        to="/dashboard"
        onMouseEnter={preloadDashboard}  // Preload on hover
      >
        Go to Dashboard
      </Link>
    </div>
  );
}
```

---

## 9. Error Handling

**Handle loading errors:**

```jsx
import { Component, lazy, Suspense } from 'react';

// Error Boundary
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Lazy loading error:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error">
          <h2>Failed to load component</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
const Dashboard = lazy(() => import('./Dashboard'));

function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <Dashboard />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

## 10. Custom Lazy Wrapper

**Add retry logic:**

```jsx
function lazyWithRetry(componentImport, retries = 3, interval = 1000) {
  return lazy(() => {
    return new Promise((resolve, reject) => {
      const attemptImport = (retriesLeft) => {
        componentImport()
          .then(resolve)
          .catch((error) => {
            if (retriesLeft === 0) {
              reject(error);
            } else {
              console.log(`Retry loading... (${retriesLeft} attempts left)`);
              setTimeout(() => {
                attemptImport(retriesLeft - 1);
              }, interval);
            }
          });
      };
      
      attemptImport(retries);
    });
  });
}

// Usage
const Dashboard = lazyWithRetry(
  () => import('./Dashboard'),
  3,    // 3 retries
  1000  // 1 second between retries
);

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Dashboard />
    </Suspense>
  );
}
```

---

## 11. Nested Suspense

**Different loading states for different parts:**

```jsx
import { lazy, Suspense } from 'react';

const Sidebar = lazy(() => import('./Sidebar'));
const MainContent = lazy(() => import('./MainContent'));
const Comments = lazy(() => import('./Comments'));

function Page() {
  return (
    <div className="page">
      {/* Sidebar loads independently */}
      <Suspense fallback={<div className="sidebar-skeleton">Loading sidebar...</div>}>
        <Sidebar />
      </Suspense>
      
      <div className="main">
        {/* Main content loads independently */}
        <Suspense fallback={<div className="content-skeleton">Loading content...</div>}>
          <MainContent />
          
          {/* Comments load independently */}
          <Suspense fallback={<div className="comments-skeleton">Loading comments...</div>}>
            <Comments />
          </Suspense>
        </Suspense>
      </div>
    </div>
  );
}
```

---

## Complete Example: E-commerce App

```jsx
import { lazy, Suspense, useState, useEffect } from 'react';
import { BrowserRouter, Routes, Route, Link, useLocation } from 'react-router-dom';

// Always loaded (small, essential)
import Navbar from './components/Navbar';
import Footer from './components/Footer';
import './App.css';

// Lazy loaded pages
const Home = lazy(() => import('./pages/Home'));
const Products = lazy(() => import('./pages/Products'));
const ProductDetail = lazy(() => import('./pages/ProductDetail'));
const Cart = lazy(() => import('./pages/Cart'));
const Checkout = lazy(() => import('./pages/Checkout'));
const Profile = lazy(() => import('./pages/Profile'));
const Orders = lazy(() => import('./pages/Orders'));
const Wishlist = lazy(() => import('./pages/Wishlist'));

// Admin section (only for admins)
const AdminDashboard = lazy(() => import('./pages/admin/Dashboard'));
const AdminProducts = lazy(() => import('./pages/admin/Products'));
const AdminOrders = lazy(() => import('./pages/admin/Orders'));

// Loading components
function PageLoader() {
  return (
    <div className="page-loader">
      <div className="spinner"></div>
      <p>Loading page...</p>
    </div>
  );
}

function ComponentLoader() {
  return (
    <div className="component-loader">
      <div className="spinner-small"></div>
    </div>
  );
}

// Preload common pages
function PreloadManager() {
  const location = useLocation();
  
  useEffect(() => {
    // Preload cart and profile after initial load
    const timer = setTimeout(() => {
      import('./pages/Cart');
      import('./pages/Profile');
    }, 2000);
    
    return () => clearTimeout(timer);
  }, []);
  
  // Preload based on current page
  useEffect(() => {
    if (location.pathname === '/products') {
      // User looking at products, likely to view details
      import('./pages/ProductDetail');
    }
    if (location.pathname === '/cart') {
      // User in cart, likely to checkout
      import('./pages/Checkout');
    }
  }, [location.pathname]);
  
  return null;
}

function App() {
  const [user] = useState({ role: 'customer', name: 'John' });
  
  return (
    <BrowserRouter>
      <div className="app">
        <Navbar user={user} />
        <PreloadManager />
        
        <main className="main-content">
          <Suspense fallback={<PageLoader />}>
            <Routes>
              {/* Public routes */}
              <Route path="/" element={<Home />} />
              <Route path="/products" element={<Products />} />
              <Route path="/products/:id" element={<ProductDetail />} />
              
              {/* User routes */}
              <Route path="/cart" element={<Cart />} />
              <Route path="/checkout" element={<Checkout />} />
              <Route path="/profile" element={<Profile />} />
              <Route path="/orders" element={<Orders />} />
              <Route path="/wishlist" element={<Wishlist />} />
              
              {/* Admin routes - only loaded for admins */}
              {user.role === 'admin' && (
                <>
                  <Route path="/admin" element={<AdminDashboard />} />
                  <Route path="/admin/products" element={<AdminProducts />} />
                  <Route path="/admin/orders" element={<AdminOrders />} />
                </>
              )}
            </Routes>
          </Suspense>
        </main>
        
        <Footer />
      </div>
    </BrowserRouter>
  );
}

export default App;
```

**Benefits:**
- Home page: ~150 KB (instead of 2 MB)
- Products page: Loads only when visited
- Admin pages: Never loaded for regular users
- Cart/Profile: Preloaded in background
- Checkout: Preloaded when user is in cart

---

## Best Practices

**1. Split by route first**
```jsx
// ✅ Good: Each route is separate
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
```

**2. Keep critical components eager**
```jsx
// ✅ Good: Navigation always available
import Navbar from './Navbar';
import Footer from './Footer';

// ❌ Bad: Navigation delayed
const Navbar = lazy(() => import('./Navbar'));
```

**3. Group related code**
```jsx
// ✅ Good: All admin code in one chunk
const Admin = lazy(() => import('./admin'));

// ❌ Bad: Too many small chunks
const AdminDashboard = lazy(() => import('./AdminDashboard'));
const AdminUsers = lazy(() => import('./AdminUsers'));
const AdminSettings = lazy(() => import('./AdminSettings'));
```

**4. Always provide fallback**
```jsx
// ✅ Good: User sees loading state
<Suspense fallback={<Loading />}>
  <Component />
</Suspense>

// ❌ Bad: No feedback
<Suspense fallback={null}>
  <Component />
</Suspense>
```

**5. Preload predictable navigation**
```jsx
// ✅ Good: Preload on hover
<Link
  to="/dashboard"
  onMouseEnter={() => import('./Dashboard')}
>
  Dashboard
</Link>
```

---

## Common Issues

**1. Forgetting Suspense**
```jsx
// ❌ Error: Component suspended while rendering
const Dashboard = lazy(() => import('./Dashboard'));
<Dashboard />

// ✅ Fixed: Wrap with Suspense
<Suspense fallback={<Loading />}>
  <Dashboard />
</Suspense>
```

**2. Named exports**
```jsx
// ❌ Doesn't work
export function Component() {}
const Component = lazy(() => import('./Component'));

// ✅ Works
export default function Component() {}
const Component = lazy(() => import('./Component'));
```

**3. Dynamic paths**
```jsx
// ❌ Doesn't work: Dynamic path
const Component = lazy(() => import(`./${pageName}`));

// ✅ Works: Static path
const Component = lazy(() => import('./Component'));
```

---

**Interview Tips:**
- Lazy loading = **deferring component load** until needed
- Use **React.lazy()** with dynamic **import()**
- **Must wrap** with Suspense for fallback
- Most common: **route-based** lazy loading
- Reduces **initial bundle size** significantly
- Components load **on demand** (when rendered)
- **Dynamic import()** returns Promise
- Only works with **default exports**
- Named exports need **.then()** transformation
- **Suspense fallback** shows while loading
- Can have **nested Suspense** for granular control
- **Preload** components before needed (hover, idle time)
- Use **Error Boundary** for load failures
- **First load**: Downloads and renders
- **Second load**: Uses cache (instant)
- Don't lazy load **critical** components (navbar)
- Do lazy load **routes**, **modals**, **admin sections**
- **Code splitting** + lazy loading = smaller bundles
- webpack creates **separate chunks** automatically
- Can **retry** failed imports with custom wrapper
- Modern pattern: lazy load by **default**, eager load **exceptions**
- Improves **Time to Interactive** (TTI)
- Better **perceived performance**
- **React.lazy()** is built-in, no extra library

</details>

---

### 56. What is Suspense component?

<details>
<summary>View Answer</summary>

**Suspense**

Suspense is a React component that lets you **"wait" for code to load** and declaratively specify a loading state (like a spinner) while waiting.

**Purpose:** Show fallback UI while child components are loading

```jsx
import { Suspense } from 'react';

<Suspense fallback={<Loading />}>
  <SomeComponent />  {/* If this suspends, show fallback */}
</Suspense>
```

---

## Basic Usage

**Without Suspense (doesn't work):**
```jsx
import { lazy } from 'react';

const Dashboard = lazy(() => import('./Dashboard'));

function App() {
  return <Dashboard />;  // ❌ Error: Component suspended while rendering
}
```

**With Suspense (works):**
```jsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./Dashboard'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Dashboard />  {/* ✅ Works */}
    </Suspense>
  );
}
```

---

## How Suspense Works

**Flow:**

```
1. React tries to render child component
   ↓
2. Component "suspends" (throws a Promise)
   ↓
3. React catches the Promise
   ↓
4. Shows fallback UI
   ↓
5. Waits for Promise to resolve
   ↓
6. Re-renders with actual component
   ↓
7. Shows component content
```

**Visual:**

```jsx
<Suspense fallback={<Spinner />}>
  <LazyComponent />
</Suspense>

// Timeline:
// t=0ms:   Start rendering
// t=1ms:   LazyComponent suspends
// t=1ms:   Show <Spinner />
// t=500ms: LazyComponent.js downloaded
// t=500ms: Hide <Spinner />
// t=500ms: Show <LazyComponent />
```

---

## 1. Basic Suspense

```jsx
import { lazy, Suspense } from 'react';

const Profile = lazy(() => import('./Profile'));

function App() {
  return (
    <div>
      <h1>My App</h1>
      
      <Suspense fallback={<div>Loading profile...</div>}>
        <Profile />
      </Suspense>
    </div>
  );
}
```

---

## 2. Multiple Components

**Same Suspense boundary:**

```jsx
import { lazy, Suspense } from 'react';

const Header = lazy(() => import('./Header'));
const Sidebar = lazy(() => import('./Sidebar'));
const Content = lazy(() => import('./Content'));

function App() {
  return (
    <Suspense fallback={<div>Loading page...</div>}>
      <Header />   {/* All three must load */}
      <Sidebar />  {/* before any are shown */}
      <Content />
    </Suspense>
  );
}

// Shows fallback until ALL components are loaded
// Then shows all at once
```

---

## 3. Nested Suspense

**Independent loading states:**

```jsx
import { lazy, Suspense } from 'react';

const Header = lazy(() => import('./Header'));
const Sidebar = lazy(() => import('./Sidebar'));
const Content = lazy(() => import('./Content'));

function App() {
  return (
    <div>
      {/* Header loads independently */}
      <Suspense fallback={<div>Loading header...</div>}>
        <Header />
      </Suspense>
      
      <div className="main">
        {/* Sidebar loads independently */}
        <Suspense fallback={<div>Loading sidebar...</div>}>
          <Sidebar />
        </Suspense>
        
        {/* Content loads independently */}
        <Suspense fallback={<div>Loading content...</div>}>
          <Content />
        </Suspense>
      </div>
    </div>
  );
}

// Each section shows its own loading state
// Parts load independently - progressive rendering
```

---

## 4. Suspense with Routes

**Route-based code splitting:**

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load pages
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Profile = lazy(() => import('./pages/Profile'));

// Loading component
function PageLoader() {
  return (
    <div className="page-loader">
      <div className="spinner"></div>
      <p>Loading page...</p>
    </div>
  );
}

function App() {
  return (
    <BrowserRouter>
      <nav>
        <a href="/">Home</a>
        <a href="/about">About</a>
        <a href="/dashboard">Dashboard</a>
      </nav>
      
      <main>
        <Suspense fallback={<PageLoader />}>
          <Routes>
            <Route path="/" element={<Home />} />
            <Route path="/about" element={<About />} />
            <Route path="/dashboard" element={<Dashboard />} />
            <Route path="/profile" element={<Profile />} />
          </Routes>
        </Suspense>
      </main>
    </BrowserRouter>
  );
}
```

---

## 5. Custom Fallback Components

**Simple spinner:**

```jsx
function Spinner() {
  return (
    <div className="spinner">
      <div className="spinner-circle"></div>
    </div>
  );
}

<Suspense fallback={<Spinner />}>
  <Component />
</Suspense>
```

**Skeleton screen:**

```jsx
function ProfileSkeleton() {
  return (
    <div className="profile-skeleton">
      <div className="skeleton-avatar"></div>
      <div className="skeleton-line"></div>
      <div className="skeleton-line"></div>
      <div className="skeleton-line short"></div>
    </div>
  );
}

<Suspense fallback={<ProfileSkeleton />}>
  <Profile />
</Suspense>
```

**Progress indicator:**

```jsx
function LoadingProgress() {
  const [progress, setProgress] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setProgress(prev => Math.min(prev + 10, 90));
    }, 100);
    
    return () => clearInterval(interval);
  }, []);
  
  return (
    <div className="loading-progress">
      <div className="progress-bar" style={{ width: `${progress}%` }} />
      <p>Loading... {progress}%</p>
    </div>
  );
}

<Suspense fallback={<LoadingProgress />}>
  <HeavyComponent />
</Suspense>
```

---

## 6. Conditional Suspense

**Show Suspense only when needed:**

```jsx
import { lazy, Suspense, useState } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <h1>Dashboard</h1>
      
      <button onClick={() => setShowChart(!showChart)}>
        {showChart ? 'Hide' : 'Show'} Chart
      </button>
      
      {/* Suspense only renders when showChart is true */}
      {showChart && (
        <Suspense fallback={<div>Loading chart...</div>}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

---

## 7. Suspense with Data Fetching (Experimental)

**⚠️ Experimental - Not stable yet**

```jsx
import { Suspense } from 'react';

// Suspense-enabled data fetching (future feature)
function ProfilePage({ userId }) {
  const user = useUserData(userId);  // This suspends while fetching
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

function App() {
  return (
    <Suspense fallback={<div>Loading user...</div>}>
      <ProfilePage userId={123} />
    </Suspense>
  );
}

// Note: useUserData must use Suspense-compatible API
// Most common: React Query with suspense: true
```

**With React Query (Suspense mode):**

```jsx
import { Suspense } from 'react';
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  // suspense: true makes this suspend
  const { data } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(r => r.json()),
    suspense: true  // Enable Suspense
  });
  
  return <div>{data.name}</div>;
}

function App() {
  return (
    <Suspense fallback={<div>Loading user...</div>}>
      <UserProfile userId={123} />
    </Suspense>
  );
}
```

---

## 8. Error Boundaries with Suspense

**Handle errors during loading:**

```jsx
import { Component, lazy, Suspense } from 'react';

// Error Boundary
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    console.error('Error:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return (
        <div className="error">
          <h2>Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Usage
const Dashboard = lazy(() => import('./Dashboard'));

function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <Dashboard />
      </Suspense>
    </ErrorBoundary>
  );
}
```

---

## 9. Suspense List (Experimental)

**⚠️ Experimental - Control reveal order**

```jsx
import { Suspense, SuspenseList } from 'react';

const Post1 = lazy(() => import('./Post1'));
const Post2 = lazy(() => import('./Post2'));
const Post3 = lazy(() => import('./Post3'));

function Feed() {
  return (
    <SuspenseList revealOrder="forwards">  {/* Show in order */}
      <Suspense fallback={<Skeleton />}>
        <Post1 />
      </Suspense>
      
      <Suspense fallback={<Skeleton />}>
        <Post2 />
      </Suspense>
      
      <Suspense fallback={<Skeleton />}>
        <Post3 />
      </Suspense>
    </SuspenseList>
  );
}

// revealOrder options:
// "forwards" - Show in order, wait for previous
// "backwards" - Show in reverse order
// "together" - Wait for all, show all at once
```

---

## Complete Example: Dashboard App

```jsx
import { lazy, Suspense, useState, useEffect } from 'react';
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import ErrorBoundary from './ErrorBoundary';

// Static imports (always needed)
import Navbar from './components/Navbar';
import Sidebar from './components/Sidebar';
import './App.css';

// Lazy imports (on demand)
const Overview = lazy(() => import('./pages/Overview'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Reports = lazy(() => import('./pages/Reports'));
const Settings = lazy(() => import('./pages/Settings'));
const UserManagement = lazy(() => import('./pages/UserManagement'));

// Loading components
function PageSkeleton() {
  return (
    <div className="page-skeleton">
      <div className="skeleton-header"></div>
      <div className="skeleton-content">
        <div className="skeleton-line"></div>
        <div className="skeleton-line"></div>
        <div className="skeleton-line short"></div>
      </div>
    </div>
  );
}

function ComponentSkeleton() {
  return (
    <div className="component-skeleton">
      <div className="skeleton-box"></div>
    </div>
  );
}

function LoadingSpinner() {
  return (
    <div className="spinner-container">
      <div className="spinner"></div>
      <p>Loading...</p>
    </div>
  );
}

function App() {
  return (
    <BrowserRouter>
      <ErrorBoundary>
        <div className="app">
          <Navbar />
          
          <div className="layout">
            <Sidebar />
            
            <main className="main-content">
              {/* Page-level Suspense */}
              <Suspense fallback={<PageSkeleton />}>
                <Routes>
                  <Route path="/" element={<Overview />} />
                  <Route path="/analytics" element={<Analytics />} />
                  <Route path="/reports" element={<Reports />} />
                  <Route path="/settings" element={<Settings />} />
                  <Route path="/users" element={<UserManagement />} />
                </Routes>
              </Suspense>
            </main>
          </div>
        </div>
      </ErrorBoundary>
    </BrowserRouter>
  );
}

// Page component with nested Suspense
function AnalyticsPage() {
  const ChartComponent = lazy(() => import('./components/Chart'));
  const TableComponent = lazy(() => import('./components/Table'));
  const MapComponent = lazy(() => import('./components/Map'));
  
  return (
    <div className="analytics-page">
      <h1>Analytics</h1>
      
      {/* Each section loads independently */}
      <section>
        <h2>Sales Chart</h2>
        <Suspense fallback={<ComponentSkeleton />}>
          <ChartComponent />
        </Suspense>
      </section>
      
      <section>
        <h2>Data Table</h2>
        <Suspense fallback={<ComponentSkeleton />}>
          <TableComponent />
        </Suspense>
      </section>
      
      <section>
        <h2>Regional Map</h2>
        <Suspense fallback={<ComponentSkeleton />}>
          <MapComponent />
        </Suspense>
      </section>
    </div>
  );
}

export default App;
```

---

## Best Practices

**1. Always provide fallback**
```jsx
// ✅ Good: User sees something
<Suspense fallback={<Loading />}>
  <Component />
</Suspense>

// ❌ Bad: Blank screen
<Suspense fallback={null}>
  <Component />
</Suspense>
```

**2. Match fallback to content**
```jsx
// ✅ Good: Skeleton matches actual layout
<Suspense fallback={<ProfileSkeleton />}>
  <Profile />
</Suspense>

// ❌ Bad: Generic spinner for complex layout
<Suspense fallback={<Spinner />}>
  <Profile />  {/* Complex layout */}
</Suspense>
```

**3. Use nested Suspense for progressive loading**
```jsx
// ✅ Good: Parts load independently
<Suspense fallback={<HeaderSkeleton />}>
  <Header />
</Suspense>
<Suspense fallback={<ContentSkeleton />}>
  <Content />
</Suspense>

// ❌ Bad: Wait for everything
<Suspense fallback={<PageSkeleton />}>
  <Header />
  <Content />
</Suspense>
```

**4. Wrap with Error Boundary**
```jsx
// ✅ Good: Handle errors gracefully
<ErrorBoundary>
  <Suspense fallback={<Loading />}>
    <Component />
  </Suspense>
</ErrorBoundary>

// ❌ Bad: No error handling
<Suspense fallback={<Loading />}>
  <Component />
</Suspense>
```

**5. Keep fallback lightweight**
```jsx
// ✅ Good: Simple, fast fallback
<Suspense fallback={<div>Loading...</div>}>
  <Component />
</Suspense>

// ❌ Bad: Heavy fallback defeats purpose
<Suspense fallback={<ComplexAnimatedLoader />}>
  <Component />
</Suspense>
```

---

## Common Patterns

**Pattern 1: Page-level Suspense**
```jsx
<Suspense fallback={<PageLoader />}>
  <Routes>
    <Route path="/" element={<Home />} />
    <Route path="/about" element={<About />} />
  </Routes>
</Suspense>
```

**Pattern 2: Section-level Suspense**
```jsx
<Suspense fallback={<Skeleton />}>
  <Sidebar />
</Suspense>
<Suspense fallback={<Skeleton />}>
  <Content />
</Suspense>
```

**Pattern 3: Component-level Suspense**
```jsx
<Suspense fallback={<ChartSkeleton />}>
  <Chart />
</Suspense>
```

**Pattern 4: Modal Suspense**
```jsx
{showModal && (
  <Suspense fallback={<ModalSkeleton />}>
    <Modal />
  </Suspense>
)}
```

---

## Current Limitations

**1. Only works with lazy() and data fetching**
- Code splitting: ✅ Stable
- Data fetching: ⚠️ Experimental

**2. Server-Side Rendering**
- Requires React 18+ for streaming SSR
- Older versions don't support SSR with Suspense

**3. Error recovery**
- Need Error Boundary for error handling
- Suspense itself doesn't catch errors

---

**Interview Tips:**
- Suspense = component for **declarative loading states**
- Shows **fallback UI** while child components load
- **Required** for React.lazy() components
- `fallback` prop accepts any **React element**
- Can be **nested** for granular control
- Multiple components under one Suspense = **wait for all**
- Nested Suspense = **independent loading states**
- Works with **code splitting** (stable)
- Works with **data fetching** (experimental)
- Catches **thrown Promises** from children
- **Progressive rendering** with nested Suspense
- Use **Error Boundary** for error handling
- Fallback should be **lightweight**
- **Skeleton screens** better than spinners
- Common pattern: **route-level** Suspense
- Also use for **modals**, **tabs**, **sections**
- React 18+: **Streaming SSR** support
- SuspenseList controls **reveal order** (experimental)
- **Not for asynchronous operations** inside components
- Only for **components that suspend**
- Modern React: Suspense is **essential pattern**
- Future: **Primary way** to handle loading states
- Improves **perceived performance**
- Better **user experience** than blank screens

</details>

---

### 57. How do you optimize large lists in React?

<details>
<summary>View Answer</summary>

**Optimizing Large Lists**

Rendering large lists (1000s of items) can cause performance issues. React has to create, update, and manage DOM nodes for every item, which becomes slow with large datasets.

**Problem:** Rendering 10,000 list items = 10,000 DOM nodes = slow
**Solution:** Use various optimization techniques

---

## The Problem

**Unoptimized large list:**

```jsx
function ProductList({ products }) {
  // Rendering 10,000 products
  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Performance issues:
// - Initial render: 2-5 seconds
// - Scrolling: Laggy
// - Memory: High (10,000 DOM nodes)
// - Updates: Slow (React has to check 10,000 items)
```

---

## 1. Use Stable Keys

**Most important: Correct key prop**

```jsx
// ❌ Bad: Index as key
{products.map((product, index) => (
  <ProductCard key={index} product={product} />
))}

// When list changes:
// - React can't identify which items changed
// - Re-renders entire list
// - Very slow

// ✅ Good: Unique ID as key
{products.map(product => (
  <ProductCard key={product.id} product={product} />
))}

// When list changes:
// - React identifies exactly which items changed
// - Only re-renders changed items
// - Much faster
```

---

## 2. Memoize List Items

**Prevent unnecessary re-renders:**

```jsx
import { memo } from 'react';

// Without memo
function ProductCard({ product, onAddToCart }) {
  console.log('ProductCard rendered:', product.id);
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product)}>Add to Cart</button>
    </div>
  );
}

// With memo - only re-renders when product or onAddToCart changes
const ProductCard = memo(function ProductCard({ product, onAddToCart }) {
  console.log('ProductCard rendered:', product.id);
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product)}>Add to Cart</button>
    </div>
  );
});

function ProductList({ products }) {
  const [cart, setCart] = useState([]);
  
  // Stable callback with useCallback
  const handleAddToCart = useCallback((product) => {
    setCart(prev => [...prev, product]);
  }, []);
  
  return (
    <div>
      {products.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={handleAddToCart}
        />
      ))}
    </div>
  );
}

// Result: When one item is added to cart, other items don't re-render
```

---

## 3. Virtualization (Window/Viewport Rendering)

**Only render visible items:**

```jsx
import { FixedSizeList } from 'react-window';

function ProductList({ products }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      <ProductCard product={products[index]} />
    </div>
  );
  
  return (
    <FixedSizeList
      height={600}        // Viewport height
      itemCount={products.length}  // Total items: 10,000
      itemSize={100}      // Each item height
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}

// Only renders ~10-20 items at a time (visible + buffer)
// Other 9,980+ items not in DOM
// Smooth scrolling through 10,000+ items
```

---

## 4. Pagination

**Load data in chunks:**

```jsx
import { useState } from 'react';

function ProductList({ products }) {
  const [page, setPage] = useState(1);
  const itemsPerPage = 50;
  
  const startIndex = (page - 1) * itemsPerPage;
  const endIndex = startIndex + itemsPerPage;
  const currentProducts = products.slice(startIndex, endIndex);
  const totalPages = Math.ceil(products.length / itemsPerPage);
  
  return (
    <div>
      {/* Only render 50 items */}
      {currentProducts.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
      
      <div className="pagination">
        <button
          onClick={() => setPage(page - 1)}
          disabled={page === 1}
        >
          Previous
        </button>
        
        <span>Page {page} of {totalPages}</span>
        
        <button
          onClick={() => setPage(page + 1)}
          disabled={page === totalPages}
        >
          Next
        </button>
      </div>
    </div>
  );
}

// Pros: Simple, easy to implement
// Cons: User has to click through pages
```

---

## 5. Infinite Scroll

**Load more on scroll:**

```jsx
import { useState, useEffect, useRef } from 'react';

function ProductList({ products }) {
  const [displayedProducts, setDisplayedProducts] = useState([]);
  const [hasMore, setHasMore] = useState(true);
  const loaderRef = useRef(null);
  
  // Load initial batch
  useEffect(() => {
    setDisplayedProducts(products.slice(0, 50));
  }, [products]);
  
  // Load more on scroll
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting && hasMore) {
          // Load 50 more items
          const nextBatch = products.slice(
            displayedProducts.length,
            displayedProducts.length + 50
          );
          
          if (nextBatch.length > 0) {
            setDisplayedProducts(prev => [...prev, ...nextBatch]);
          } else {
            setHasMore(false);
          }
        }
      },
      { threshold: 1.0 }
    );
    
    if (loaderRef.current) {
      observer.observe(loaderRef.current);
    }
    
    return () => observer.disconnect();
  }, [displayedProducts, hasMore, products]);
  
  return (
    <div>
      {displayedProducts.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
      
      {hasMore && (
        <div ref={loaderRef} className="loader">
          Loading more...
        </div>
      )}
    </div>
  );
}

// Pros: Better UX than pagination
// Cons: DOM grows over time (combine with virtualization)
```

---

## 6. Debounce Filtering/Searching

**Avoid re-rendering on every keystroke:**

```jsx
import { useState, useMemo, useEffect } from 'react';

function ProductList({ products }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [debouncedSearch, setDebouncedSearch] = useState('');
  
  // Debounce search
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedSearch(searchTerm);
    }, 300);  // Wait 300ms after user stops typing
    
    return () => clearTimeout(timer);
  }, [searchTerm]);
  
  // Filter only when debounced value changes
  const filteredProducts = useMemo(() => {
    return products.filter(product =>
      product.name.toLowerCase().includes(debouncedSearch.toLowerCase())
    );
  }, [products, debouncedSearch]);
  
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search products..."
      />
      
      {filteredProducts.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Without debounce: Filter on every keystroke = 10,000 item filter per key
// With debounce: Filter 300ms after user stops = Much less work
```

---

## 7. useMemo for Expensive Calculations

**Cache filtered/sorted results:**

```jsx
import { useState, useMemo } from 'react';

function ProductList({ products }) {
  const [sortBy, setSortBy] = useState('name');
  const [filterCategory, setFilterCategory] = useState('all');
  
  // Memoize filtering
  const filteredProducts = useMemo(() => {
    console.log('Filtering...');
    if (filterCategory === 'all') return products;
    return products.filter(p => p.category === filterCategory);
  }, [products, filterCategory]);
  
  // Memoize sorting
  const sortedProducts = useMemo(() => {
    console.log('Sorting...');
    return [...filteredProducts].sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      if (sortBy === 'price') return a.price - b.price;
      return 0;
    });
  }, [filteredProducts, sortBy]);
  
  return (
    <div>
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
        <option value="name">Sort by Name</option>
        <option value="price">Sort by Price</option>
      </select>
      
      <select value={filterCategory} onChange={(e) => setFilterCategory(e.target.value)}>
        <option value="all">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
      
      {sortedProducts.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// Only re-filters/re-sorts when dependencies change
```

---

## 8. Lazy Loading Images

**Don't load all images at once:**

```jsx
import { useState, useEffect, useRef } from 'react';

function LazyImage({ src, alt }) {
  const [imageSrc, setImageSrc] = useState('');
  const imgRef = useRef(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          setImageSrc(src);
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );
    
    if (imgRef.current) {
      observer.observe(imgRef.current);
    }
    
    return () => observer.disconnect();
  }, [src]);
  
  return (
    <img
      ref={imgRef}
      src={imageSrc || 'placeholder.jpg'}
      alt={alt}
      loading="lazy"  // Native lazy loading
    />
  );
}

function ProductCard({ product }) {
  return (
    <div className="product-card">
      <LazyImage src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>${product.price}</p>
    </div>
  );
}

// Only loads images when they're about to be visible
```

---

## 9. Split into Smaller Components

**Reduce what re-renders:**

```jsx
import { memo, useState } from 'react';

// ❌ Bad: Entire list re-renders when any item changes
function ProductList({ products }) {
  return (
    <div>
      {products.map(product => {
        const [liked, setLiked] = useState(false);
        
        return (
          <div key={product.id}>
            <h3>{product.name}</h3>
            <button onClick={() => setLiked(!liked)}>
              {liked ? '♥' : '♡'}
            </button>
          </div>
        );
      })}
    </div>
  );
}

// ✅ Good: Each item manages own state
const ProductCard = memo(function ProductCard({ product }) {
  const [liked, setLiked] = useState(false);
  
  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={() => setLiked(!liked)}>
        {liked ? '♥' : '♡'}
      </button>
    </div>
  );
});

function ProductList({ products }) {
  return (
    <div>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// When one item is liked, only that item re-renders
```

---

## 10. Use CSS Grid/Flexbox Layout

**Browser-native layout is faster:**

```jsx
// ✅ Good: CSS Grid
function ProductList({ products }) {
  return (
    <div className="product-grid">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}

// CSS
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

// Browser handles layout - very fast
// No JavaScript calculations needed
```

---

## Complete Example: Optimized Product List

```jsx
import { useState, useMemo, useCallback, memo } from 'react';
import { FixedSizeGrid } from 'react-window';

// Memoized product card
const ProductCard = memo(function ProductCard({ product, onAddToCart, style }) {
  return (
    <div style={style} className="product-card">
      <img src={product.image} alt={product.name} loading="lazy" />
      <h3>{product.name}</h3>
      <p className="price">${product.price}</p>
      <p className="category">{product.category}</p>
      <button onClick={() => onAddToCart(product)}>Add to Cart</button>
    </div>
  );
});

function ProductList({ products }) {
  const [searchTerm, setSearchTerm] = useState('');
  const [category, setCategory] = useState('all');
  const [sortBy, setSortBy] = useState('name');
  const [cart, setCart] = useState([]);
  
  // Stable callback
  const handleAddToCart = useCallback((product) => {
    setCart(prev => [...prev, product]);
  }, []);
  
  // Filter products
  const filteredProducts = useMemo(() => {
    return products.filter(product => {
      const matchesSearch = product.name.toLowerCase().includes(searchTerm.toLowerCase());
      const matchesCategory = category === 'all' || product.category === category;
      return matchesSearch && matchesCategory;
    });
  }, [products, searchTerm, category]);
  
  // Sort products
  const sortedProducts = useMemo(() => {
    return [...filteredProducts].sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      if (sortBy === 'price-asc') return a.price - b.price;
      if (sortBy === 'price-desc') return b.price - a.price;
      return 0;
    });
  }, [filteredProducts, sortBy]);
  
  // Virtualized grid
  const Cell = ({ columnIndex, rowIndex, style }) => {
    const index = rowIndex * 4 + columnIndex;  // 4 columns
    if (index >= sortedProducts.length) return null;
    
    const product = sortedProducts[index];
    return (
      <ProductCard
        product={product}
        onAddToCart={handleAddToCart}
        style={style}
      />
    );
  };
  
  return (
    <div className="product-list">
      {/* Filters */}
      <div className="filters">
        <input
          type="text"
          value={searchTerm}
          onChange={(e) => setSearchTerm(e.target.value)}
          placeholder="Search products..."
        />
        
        <select value={category} onChange={(e) => setCategory(e.target.value)}>
          <option value="all">All Categories</option>
          <option value="electronics">Electronics</option>
          <option value="clothing">Clothing</option>
          <option value="books">Books</option>
        </select>
        
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="name">Sort by Name</option>
          <option value="price-asc">Price: Low to High</option>
          <option value="price-desc">Price: High to Low</option>
        </select>
        
        <div className="cart-info">
          Cart: {cart.length} items
        </div>
      </div>
      
      {/* Virtualized grid */}
      <FixedSizeGrid
        columnCount={4}
        columnWidth={280}
        height={800}
        rowCount={Math.ceil(sortedProducts.length / 4)}
        rowHeight={350}
        width={1200}
      >
        {Cell}
      </FixedSizeGrid>
      
      <div className="results-count">
        Showing {sortedProducts.length} of {products.length} products
      </div>
    </div>
  );
}

export default ProductList;
```

**Optimizations applied:**
1. ✅ React.memo on ProductCard
2. ✅ useCallback for handleAddToCart
3. ✅ useMemo for filtering and sorting
4. ✅ Virtualization with react-window (only renders visible items)
5. ✅ Stable keys (product.id)
6. ✅ Lazy loading images

**Performance:**
- 10,000 products in array
- Only ~40 products rendered in DOM
- Smooth 60fps scrolling
- Instant filtering/sorting

---

## Performance Comparison

**Unoptimized (10,000 items):**
```
Initial render:  5000ms
Scroll FPS:      15-20fps (laggy)
Memory:          ~500MB
Search typing:   200ms per keystroke
DOM nodes:       10,000+
```

**Optimized with virtualization:**
```
Initial render:  150ms
Scroll FPS:      60fps (smooth)
Memory:          ~50MB
Search typing:   < 16ms per keystroke
DOM nodes:       ~50
```

---

## Best Practices Summary

| Technique | When to Use | Benefit |
|-----------|-------------|----------|
| Stable keys | Always | React identifies items correctly |
| React.memo | List items | Prevents unnecessary re-renders |
| Virtualization | 500+ items | Only renders visible items |
| Pagination | 100+ items | Simple, works everywhere |
| Infinite scroll | Feeds, social | Better UX than pagination |
| Debouncing | Search/filter | Reduces computation |
| useMemo | Expensive ops | Caches results |
| Lazy images | Image-heavy | Faster initial load |
| Split components | Complex items | Isolates state changes |

---

**Interview Tips:**
- Large lists = **performance challenge** in React
- Always use **unique, stable keys** (not index)
- **React.memo** list items to prevent re-renders
- **useCallback** for callbacks passed to list items
- **Virtualization** = only render visible items
- **react-window** most popular virtualization library
- **Pagination** simplest approach (50-100 items per page)
- **Infinite scroll** better UX, combine with virtualization
- **Debounce** search/filter inputs (300-500ms)
- **useMemo** for filtering and sorting
- **Lazy load** images with Intersection Observer
- Split into **smaller components** to isolate state
- Use **CSS Grid/Flexbox** for layout (browser-native)
- **10,000+ items** definitely need virtualization
- **500-1000 items** consider virtualization
- **< 100 items** usually fine without optimization
- Measure with **React DevTools Profiler**
- Common mistake: **index as key** in dynamic lists
- Virtualization trade-off: **No native search** (Ctrl+F)
- Modern solution: **Virtualization + pagination**
- Always **profile before** optimizing
- Premature optimization = **waste of time**

</details>

---

### 58. What is virtualization and which libraries support it?

<details>
<summary>View Answer</summary>

**Virtualization (Windowing)**

Virtualization is a technique where you **only render items visible in the viewport**, not the entire list. As you scroll, items are dynamically rendered and removed, keeping DOM size constant.

**Concept:** Why render 10,000 items when user can only see 20?

---

## How Virtualization Works

**Traditional rendering (no virtualization):**
```
┌────────────────────────────────────────┐
│ Viewport (what user sees)             │
│  Item 1  ← Rendered                    │
│  Item 2  ← Rendered                    │
│  Item 3  ← Rendered                    │
│  ...                                   │
│  Item 20 ← Rendered                    │
└────────────────────────────────────────┘
  Item 21 ← Rendered (below viewport)
  Item 22 ← Rendered
  ...
  Item 10000 ← Rendered
  
Total DOM nodes: 10,000
```

**With virtualization:**
```
┌────────────────────────────────────────┐
│ Viewport (what user sees)             │
│  Item 1  ← Rendered                    │
│  Item 2  ← Rendered                    │
│  Item 3  ← Rendered                    │
│  ...                                   │
│  Item 20 ← Rendered                    │
└────────────────────────────────────────┘
  <virtual space for items 21-10000>
  (Not in DOM, just empty space)
  
Total DOM nodes: ~30 (20 visible + 10 buffer)
```

**As you scroll:**
- Items entering viewport: Rendered
- Items leaving viewport: Removed from DOM
- Illusion of scrolling through 10,000 items
- Actually only ~30 DOM nodes

---

## Libraries for Virtualization

### 1. react-window (Most Popular)

**Best for: Most use cases, modern apps**

**Install:**
```bash
npm install react-window
```

**Features:**
- ✅ Small bundle size (~3KB)
- ✅ Simple API
- ✅ Fixed and variable sizes
- ✅ Vertical and horizontal scrolling
- ✅ Grid support
- ✅ Great performance
- ❌ Less features than react-virtualized

**Fixed Size List:**
```jsx
import { FixedSizeList } from 'react-window';

const items = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`);

function App() {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index]}
    </div>
  );
  
  return (
    <FixedSizeList
      height={600}         // Container height
      itemCount={10000}    // Total items
      itemSize={50}        // Each item height (px)
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

**Variable Size List:**
```jsx
import { VariableSizeList } from 'react-window';

function App() {
  const items = [
    { id: 1, text: 'Short' },
    { id: 2, text: 'Medium length text here' },
    { id: 3, text: 'Very long text that takes multiple lines...' }
  ];
  
  // Calculate height for each item
  const getItemSize = (index) => {
    const textLength = items[index].text.length;
    return textLength > 50 ? 100 : 50;
  };
  
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].text}
    </div>
  );
  
  return (
    <VariableSizeList
      height={600}
      itemCount={items.length}
      itemSize={getItemSize}  // Function that returns height
      width="100%"
    >
      {Row}
    </VariableSizeList>
  );
}
```

**Grid:**
```jsx
import { FixedSizeGrid } from 'react-window';

function App() {
  const Cell = ({ columnIndex, rowIndex, style }) => (
    <div style={style}>
      Row {rowIndex}, Col {columnIndex}
    </div>
  );
  
  return (
    <FixedSizeGrid
      columnCount={10}      // 10 columns
      columnWidth={100}     // Each column width
      height={600}
      rowCount={1000}       // 1000 rows
      rowHeight={50}        // Each row height
      width={1000}
    >
      {Cell}
    </FixedSizeGrid>
  );
}
```

---

### 2. react-virtualized (Feature-Rich)

**Best for: Complex requirements, need advanced features**

**Install:**
```bash
npm install react-virtualized
```

**Features:**
- ✅ More features than react-window
- ✅ AutoSizer (auto-calculate dimensions)
- ✅ InfiniteLoader (infinite scrolling)
- ✅ WindowScroller (use window as scroller)
- ✅ CellMeasurer (auto-measure heights)
- ✅ MultiGrid (frozen rows/columns)
- ❌ Larger bundle size (~30KB)
- ❌ More complex API

**Basic List:**
```jsx
import { List } from 'react-virtualized';

function App() {
  const items = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`);
  
  const rowRenderer = ({ key, index, style }) => (
    <div key={key} style={style}>
      {items[index]}
    </div>
  );
  
  return (
    <List
      width={800}
      height={600}
      rowCount={items.length}
      rowHeight={50}
      rowRenderer={rowRenderer}
    />
  );
}
```

**AutoSizer (auto-calculate dimensions):**
```jsx
import { AutoSizer, List } from 'react-virtualized';

function App() {
  return (
    <AutoSizer>
      {({ height, width }) => (
        <List
          width={width}      // Auto-calculated
          height={height}    // Auto-calculated
          rowCount={10000}
          rowHeight={50}
          rowRenderer={rowRenderer}
        />
      )}
    </AutoSizer>
  );
}
```

**InfiniteLoader (load more on scroll):**
```jsx
import { InfiniteLoader, List } from 'react-virtualized';
import { useState } from 'react';

function App() {
  const [items, setItems] = useState(Array.from({ length: 100 }));
  const [hasMore, setHasMore] = useState(true);
  
  const loadMore = ({ startIndex, stopIndex }) => {
    return new Promise((resolve) => {
      setTimeout(() => {
        const newItems = Array.from({ length: 100 });
        setItems(prev => [...prev, ...newItems]);
        if (items.length > 1000) setHasMore(false);
        resolve();
      }, 1000);
    });
  };
  
  const isRowLoaded = ({ index }) => {
    return index < items.length;
  };
  
  return (
    <InfiniteLoader
      isRowLoaded={isRowLoaded}
      loadMoreRows={loadMore}
      rowCount={hasMore ? items.length + 100 : items.length}
    >
      {({ onRowsRendered, registerChild }) => (
        <List
          ref={registerChild}
          onRowsRendered={onRowsRendered}
          rowCount={items.length}
          rowHeight={50}
          width={800}
          height={600}
          rowRenderer={rowRenderer}
        />
      )}
    </InfiniteLoader>
  );
}
```

---

### 3. @tanstack/react-virtual (Modern)

**Best for: TypeScript, headless UI, flexibility**

**Install:**
```bash
npm install @tanstack/react-virtual
```

**Features:**
- ✅ Headless (no styling)
- ✅ Framework agnostic core
- ✅ TypeScript first
- ✅ Flexible API
- ✅ Horizontal/vertical/grid
- ✅ Dynamic sizing
- ✅ Active development

**Basic usage:**
```jsx
import { useVirtualizer } from '@tanstack/react-virtual';
import { useRef } from 'react';

function App() {
  const parentRef = useRef(null);
  
  const items = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`);
  
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,  // Estimated item height
  });
  
  return (
    <div
      ref={parentRef}
      style={{ height: '600px', overflow: 'auto' }}
    >
      <div
        style={{
          height: `${virtualizer.getTotalSize()}px`,
          position: 'relative',
        }}
      >
        {virtualizer.getVirtualItems().map((virtualRow) => (
          <div
            key={virtualRow.index}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualRow.size}px`,
              transform: `translateY(${virtualRow.start}px)`,
            }}
          >
            {items[virtualRow.index]}
          </div>
        ))}
      </div>
    </div>
  );
}
```

**Dynamic sizing:**
```jsx
const virtualizer = useVirtualizer({
  count: items.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 50,
  // Measure actual size after render
  measureElement: (el) => el?.getBoundingClientRect().height,
});
```

---

### 4. react-virtuoso (Simplified)

**Best for: Simple API, handles complexity internally**

**Install:**
```bash
npm install react-virtuoso
```

**Features:**
- ✅ Very simple API
- ✅ Auto-calculates item heights
- ✅ Grouped items
- ✅ Infinite scroll built-in
- ✅ Sticky headers
- ✅ Great for lists
- ❌ Less control than react-window

**Basic usage:**
```jsx
import { Virtuoso } from 'react-virtuoso';

function App() {
  const items = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`);
  
  return (
    <Virtuoso
      style={{ height: '600px' }}
      data={items}
      itemContent={(index, item) => (
        <div>
          {item}
        </div>
      )}
    />
  );
}
```

**With infinite scroll:**
```jsx
import { Virtuoso } from 'react-virtuoso';
import { useState } from 'react';

function App() {
  const [items, setItems] = useState(Array.from({ length: 100 }));
  
  const loadMore = () => {
    setTimeout(() => {
      setItems(prev => [
        ...prev,
        ...Array.from({ length: 100 }, (_, i) => `Item ${prev.length + i + 1}`)
      ]);
    }, 500);
  };
  
  return (
    <Virtuoso
      style={{ height: '600px' }}
      data={items}
      endReached={loadMore}
      itemContent={(index, item) => <div>{item}</div>}
    />
  );
}
```

**Grouped list:**
```jsx
import { GroupedVirtuoso } from 'react-virtuoso';

function App() {
  const groupCounts = [10, 20, 15];  // 3 groups
  
  return (
    <GroupedVirtuoso
      style={{ height: '600px' }}
      groupCounts={groupCounts}
      groupContent={(index) => (
        <div style={{ background: '#eee', padding: '10px' }}>
          Group {index + 1}
        </div>
      )}
      itemContent={(index) => (
        <div style={{ padding: '10px' }}>
          Item {index + 1}
        </div>
      )}
    />
  );
}
```

---

## Library Comparison

| Feature | react-window | react-virtualized | @tanstack/react-virtual | react-virtuoso |
|---------|--------------|-------------------|------------------------|----------------|
| **Bundle Size** | ~3KB | ~30KB | ~5KB | ~15KB |
| **API Complexity** | Simple | Complex | Medium | Very Simple |
| **Fixed Size** | ✅ | ✅ | ✅ | ✅ |
| **Variable Size** | ✅ | ✅ | ✅ | ✅ (auto) |
| **Grid** | ✅ | ✅ | ✅ | ❌ |
| **AutoSizer** | ❌ | ✅ | Built-in | Built-in |
| **Infinite Scroll** | ❌ | ✅ | ✅ | ✅ |
| **Horizontal** | ✅ | ✅ | ✅ | ✅ |
| **TypeScript** | ✅ | ✅ | ✅✅ | ✅ |
| **Active Dev** | Stable | Maintenance | ✅✅ | ✅ |
| **Learning Curve** | Easy | Hard | Medium | Very Easy |

---

## When to Use Which Library

**react-window:**
- ✅ Most use cases
- ✅ Need small bundle size
- ✅ Simple requirements
- ✅ Good performance
- ✅ **Recommended for most apps**

**react-virtualized:**
- ✅ Need advanced features (AutoSizer, InfiniteLoader)
- ✅ Complex table/grid requirements
- ✅ Already using it (maintenance)
- ❌ Larger bundle size acceptable

**@tanstack/react-virtual:**
- ✅ TypeScript project
- ✅ Need flexibility/control
- ✅ Headless UI (custom styling)
- ✅ Modern stack
- ✅ **Best for new TypeScript projects**

**react-virtuoso:**
- ✅ Want simplest API
- ✅ Auto-sizing items
- ✅ Grouped lists
- ✅ Infinite scroll out of the box
- ✅ **Best for quick implementation**

---

## Complete Example: Product Grid with react-window

```jsx
import { FixedSizeGrid } from 'react-window';
import { useState, useMemo } from 'react';

function ProductGrid({ products }) {
  const [searchTerm, setSearchTerm] = useState('');
  
  // Filter products
  const filteredProducts = useMemo(() => {
    return products.filter(product =>
      product.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }, [products, searchTerm]);
  
  const columnCount = 4;
  const rowCount = Math.ceil(filteredProducts.length / columnCount);
  
  const Cell = ({ columnIndex, rowIndex, style }) => {
    const index = rowIndex * columnCount + columnIndex;
    
    if (index >= filteredProducts.length) {
      return null;
    }
    
    const product = filteredProducts[index];
    
    return (
      <div style={style}>
        <div className="product-card">
          <img src={product.image} alt={product.name} />
          <h3>{product.name}</h3>
          <p>${product.price}</p>
          <button>Add to Cart</button>
        </div>
      </div>
    );
  };
  
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search products..."
        style={{ marginBottom: '20px', padding: '10px', width: '100%' }}
      />
      
      <FixedSizeGrid
        columnCount={columnCount}
        columnWidth={280}
        height={800}
        rowCount={rowCount}
        rowHeight={350}
        width={1200}
      >
        {Cell}
      </FixedSizeGrid>
      
      <p>Showing {filteredProducts.length} products</p>
    </div>
  );
}

export default ProductGrid;
```

---

## Performance Benefits

**Without virtualization (10,000 items):**
```
DOM nodes:        10,000
Memory usage:     ~500MB
Initial render:   5000ms
Scroll FPS:       15-20fps
```

**With virtualization:**
```
DOM nodes:        ~40
Memory usage:     ~50MB
Initial render:   150ms
Scroll FPS:       60fps
```

**90% reduction** in memory and render time!

---

## Trade-offs

**Pros:**
- ✅ Massive performance improvement
- ✅ Handles millions of items
- ✅ Constant memory usage
- ✅ Smooth 60fps scrolling

**Cons:**
- ❌ Browser Find (Ctrl+F) doesn't work
- ❌ Accessibility challenges
- ❌ Can't use CSS selectors like `:nth-child`
- ❌ Items must be uniform or measurable
- ❌ Extra complexity

---

**Interview Tips:**
- Virtualization = **only render visible items**
- Also called **windowing**
- Keeps DOM size **constant** regardless of data size
- **react-window** most popular (~3KB)
- **react-virtualized** more features (~30KB)
- **@tanstack/react-virtual** modern, TypeScript-first
- **react-virtuoso** simplest API
- Use when **500+ items** in list
- **Must-have** for 1000+ items
- Renders ~20-40 items regardless of total
- **Dynamic rendering** as you scroll
- Items entering viewport = rendered
- Items leaving viewport = removed from DOM
- Massive **performance benefit** (90% faster)
- Trade-off: **Ctrl+F doesn't work** on entire list
- Trade-off: **Accessibility** more complex
- FixedSizeList = all items **same height**
- VariableSizeList = items **different heights**
- FixedSizeGrid = **2D grid** virtualization
- Can combine with **infinite scroll**
- **AutoSizer** auto-calculates container dimensions
- Modern approach: **@tanstack/react-virtual**
- Easiest: **react-virtuoso** (auto-sizing)
- Most apps: **react-window** (simple, small)

</details>

---

### 59. What are React DevTools Profiler and how do you use it?

<details>
<summary>View Answer</summary>

**React DevTools Profiler**

The React DevTools Profiler is a **performance profiling tool** that helps you **identify performance bottlenecks** in your React applications. It records component render times and shows you which components are slow.

**Purpose:** Find and fix performance problems

---

## Installation

**Browser Extension:**
- Chrome: [React Developer Tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi)
- Firefox: [React Developer Tools](https://addons.mozilla.org/en-US/firefox/addon/react-devtools/)
- Edge: Available in Chrome Web Store

**Standalone:**
```bash
npm install -g react-devtools
react-devtools
```

---

## Opening the Profiler

1. Open **React DevTools** in browser (F12)
2. Click **"Profiler"** tab (next to Components tab)
3. Click **record button** (red circle)
4. Interact with your app
5. Click **stop button**
6. View performance data

---

## Understanding the Profiler UI

**Main views:**

### 1. Flamegraph

```
     App (15ms)                    ← Root component
    /    |    \
  Header Sidebar Content (10ms)   ← Child components
  (2ms)  (3ms)    |
                  |
               ProductList (8ms)   ← Slow component!
                  |
               ProductCard (6ms)   ← Individual items
```

**Reading:**
- **Width** = render time (wider = slower)
- **Color** = performance (yellow = fast, orange/red = slow)
- **Height** = component hierarchy
- Click component to see details

### 2. Ranked Chart

Lists components by render time:
```
1. ProductList      - 8ms  ← Slowest
2. Content          - 10ms
3. ProductCard      - 6ms
4. Sidebar          - 3ms
5. Header           - 2ms  ← Fastest
```

### 3. Component Chart

Shows individual component over time:
```
ProductCard renders:
Render 1: 5ms
Render 2: 6ms  ← Getting slower
Render 3: 8ms
Render 4: 10ms ← Problem!
```

---

## What the Profiler Shows

**For each render:**
- **Render duration** - How long component took
- **Why it rendered** - Props change, state change, parent render, context change
- **Commit duration** - How long React took to update DOM
- **Which props changed** - Specific props that caused re-render

---

## Step-by-Step: Using Profiler

**Example: Finding slow components**

### Step 1: Record a Session

```jsx
// Your app
function App() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <ProductList />  {/* Suspect this is slow */}
    </div>
  );
}
```

1. Open Profiler
2. Click record
3. Click button 5 times
4. Stop recording

### Step 2: Analyze Flamegraph

**What you see:**
```
Commit 1:          Commit 2:          Commit 3:
  App (20ms)         App (18ms)         App (22ms)
    |                  |                  |
  ProductList       ProductList       ProductList
  (18ms)            (16ms)            (20ms)  ← Consistently slow!
```

**Finding:** ProductList takes 15-20ms every render

### Step 3: Click Component for Details

**Details panel shows:**
```
ProductList
  Render duration: 18ms
  Rendered because: parent rendered
  Props: products (didn't change)
  
Problem: Re-rendering even though props didn't change!
```

### Step 4: Fix the Issue

```jsx
import { memo } from 'react';

// Before: Re-renders every time
function ProductList({ products }) {
  return (
    <div>
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
}

// After: Only re-renders when products change
const ProductList = memo(function ProductList({ products }) {
  return (
    <div>
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  );
});
```

### Step 5: Record Again and Compare

**After optimization:**
```
Commit 1:          Commit 2:          Commit 3:
  App (3ms)          App (2ms)          App (3ms)
    |                  |                  |
  ProductList       ProductList       ProductList
  (0ms - skipped)   (0ms - skipped)   (0ms - skipped)
```

**Result:** 18ms → 0ms = **100% faster!**

---

## Reading "Why Did This Render?"

Profiler shows exact reason for re-render:

**1. "Parent component rendered"**
```
Child rendered because: Parent component rendered

Fix: Use React.memo
```

**2. "Hook X changed"**
```
Component rendered because: Hook 1 changed (useState)

Expected - state change should trigger re-render
```

**3. "Props changed"**
```
Component rendered because: Props changed
  - name: 'John' → 'Jane'
  - age: 25 → 26

Expected - props changed
```

**4. "The parent component rendered"**
```
Component rendered because: The parent component rendered

If props didn't change, consider React.memo
```

**5. "Context changed"**
```
Component rendered because: Context changed

Expected - component uses context
```

---

## Profiler API (Programmatic)

**Use Profiler component in code:**

```jsx
import { Profiler } from 'react';

function onRenderCallback(
  id,                   // Profiler id
  phase,                // "mount" or "update"
  actualDuration,       // Time to render
  baseDuration,         // Estimated time without memoization
  startTime,            // When React started rendering
  commitTime,           // When React committed update
  interactions          // Set of interactions being traced
) {
  console.log(`${id} (${phase}) took ${actualDuration}ms`);
  
  if (actualDuration > 10) {
    console.warn(`Slow render: ${id} took ${actualDuration}ms`);
  }
}

function App() {
  return (
    <Profiler id="App" onRender={onRenderCallback}>
      <Header />
      <Profiler id="ProductList" onRender={onRenderCallback}>
        <ProductList />
      </Profiler>
      <Footer />
    </Profiler>
  );
}
```

**Output:**
```
App (mount) took 45ms
ProductList (mount) took 20ms
App (update) took 15ms
ProductList (update) took 8ms
```

**Send to analytics:**
```jsx
function onRenderCallback(id, phase, actualDuration) {
  // Send to analytics service
  analytics.track('React Render', {
    component: id,
    phase: phase,
    duration: actualDuration
  });
  
  // Alert if slow
  if (actualDuration > 100) {
    console.error(`SLOW: ${id} took ${actualDuration}ms`);
  }
}
```

---

## Common Issues and How to Find Them

### Issue 1: Unnecessary Re-renders

**Symptom in Profiler:**
```
Component rendered because: Parent component rendered
Props: (all unchanged)
```

**Fix:**
```jsx
const Component = memo(function Component(props) {
  // Component code
});
```

---

### Issue 2: New Function Every Render

**Symptom in Profiler:**
```
ChildComponent rendered because: Props changed
  - onClick: function() {} → function() {}  ← New function
```

**Fix:**
```jsx
const onClick = useCallback(() => {
  // Handler code
}, []);
```

---

### Issue 3: New Object Every Render

**Symptom in Profiler:**
```
ChildComponent rendered because: Props changed
  - config: {a: 1} → {a: 1}  ← New object (same values)
```

**Fix:**
```jsx
const config = useMemo(() => ({ a: 1 }), []);
```

---

### Issue 4: Expensive Calculation

**Symptom in Profiler:**
```
Component render duration: 150ms  ← Very slow!
```

**Investigate:**
```jsx
function Component({ data }) {
  console.time('calculation');
  const result = expensiveCalculation(data);  // Takes 150ms
  console.timeEnd('calculation');
  
  return <div>{result}</div>;
}
```

**Fix:**
```jsx
function Component({ data }) {
  const result = useMemo(() => {
    return expensiveCalculation(data);
  }, [data]);  // Only recalculate when data changes
  
  return <div>{result}</div>;
}
```

---

## Complete Example: Debugging Performance

```jsx
import { useState, memo, useCallback, useMemo, Profiler } from 'react';

// Profiler callback
function onRender(id, phase, actualDuration) {
  if (actualDuration > 5) {
    console.warn(`⚠️ ${id} took ${actualDuration.toFixed(2)}ms`);
  } else {
    console.log(`✓ ${id} took ${actualDuration.toFixed(2)}ms`);
  }
}

// Slow component (before optimization)
function SlowProductCard({ product, onAddToCart }) {
  // Simulate slow render
  const now = Date.now();
  while (Date.now() - now < 10) {} // Block for 10ms
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product)}>Add to Cart</button>
    </div>
  );
}

// Optimized component
const ProductCard = memo(function ProductCard({ product, onAddToCart }) {
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product)}>Add to Cart</button>
    </div>
  );
});

function ProductList({ products }) {
  const [cart, setCart] = useState([]);
  
  // Optimized callback
  const handleAddToCart = useCallback((product) => {
    setCart(prev => [...prev, product]);
  }, []);
  
  return (
    <Profiler id="ProductList" onRender={onRender}>
      <div>
        <h2>Products</h2>
        <p>Cart: {cart.length} items</p>
        
        <div className="product-grid">
          {products.map(product => (
            <Profiler key={product.id} id={`Product-${product.id}`} onRender={onRender}>
              <ProductCard
                product={product}
                onAddToCart={handleAddToCart}
              />
            </Profiler>
          ))}
        </div>
      </div>
    </Profiler>
  );
}

function App() {
  const [count, setCount] = useState(0);
  
  const products = useMemo(() => [
    { id: 1, name: 'Laptop', price: 999 },
    { id: 2, name: 'Mouse', price: 29 },
    { id: 3, name: 'Keyboard', price: 79 }
  ], []);
  
  return (
    <Profiler id="App" onRender={onRender}>
      <div>
        <h1>My Store</h1>
        <button onClick={() => setCount(count + 1)}>Clicks: {count}</button>
        
        <ProductList products={products} />
      </div>
    </Profiler>
  );
}

export default App;
```

**Console output (after clicking button):**
```
✓ App took 0.5ms
✓ ProductList took 0.2ms
✓ Product-1 took 0.0ms (skipped)
✓ Product-2 took 0.0ms (skipped)
✓ Product-3 took 0.0ms (skipped)
```

---

## Best Practices

**1. Profile in production mode**
```bash
npm run build
serve -s build
```
Development mode is slower (extra checks)

**2. Record specific interactions**
- Don't record everything
- Focus on slow interactions
- Test one feature at a time

**3. Look for patterns**
- Components rendering on every keystroke
- Large lists without virtualization
- Expensive calculations without useMemo

**4. Set performance budgets**
```jsx
function onRender(id, phase, actualDuration) {
  const budget = {
    'App': 50,
    'ProductList': 20,
    'ProductCard': 5
  };
  
  if (actualDuration > budget[id]) {
    console.error(`❌ ${id} exceeded budget: ${actualDuration}ms > ${budget[id]}ms`);
  }
}
```

**5. Compare before/after**
- Record before optimization
- Make changes
- Record after optimization
- Compare results

---

## Profiler Settings

**In DevTools:**

1. **Record why each component rendered**
   - Shows exact reason for re-render
   - Shows which props/state changed

2. **Hide commits below X ms**
   - Filter out fast renders
   - Focus on slow ones

3. **Flamegraph / Ranked / Timeline**
   - Switch between views
   - Each useful for different analysis

---

## Interpreting Results

**Good performance:**
```
Commit duration: < 16ms (60fps)
Most components: < 5ms
No red/orange bars in flamegraph
```

**Needs optimization:**
```
Commit duration: > 16ms (< 60fps)
Some components: > 10ms
Red/orange bars in flamegraph
```

**Critical issues:**
```
Commit duration: > 100ms
Some components: > 50ms
Many red bars, janky scrolling
```

---

## Common Patterns to Spot

**Pattern 1: Cascading re-renders**
```
App renders →
  Header renders →
    Nav renders →
      NavItem renders (x10)
        ↑
  Everything re-renders!
```
Fix: Add React.memo

**Pattern 2: List without keys**
```
Commit 1: Render all items (100ms)
Commit 2: Render all items (100ms)
Commit 3: Render all items (100ms)
```
Fix: Use unique keys

**Pattern 3: Context causing re-renders**
```
All components using context render together
```
Fix: Split context

---

## Real-World Debugging Session

**Problem:** App is slow when typing in search box

**Step 1: Profile the interaction**
- Type "laptop" in search box
- Profiler shows 5 commits (one per letter)

**Step 2: Analyze flamegraph**
```
Commit 1 (l):      Commit 2 (la):     Commit 3 (lap):
  App (50ms)         App (52ms)         App (48ms)
    SearchBox          SearchBox          SearchBox
    (1ms)              (1ms)              (1ms)
    ProductList        ProductList        ProductList
    (45ms!)            (48ms!)            (44ms!)
```

**Finding:** ProductList re-renders on every keystroke (45-50ms each)

**Step 3: Check why**
```
ProductList rendered because: Props changed
  - searchTerm: 'l' → 'la'
```

**Step 4: Fix with debouncing**
```jsx
const [searchTerm, setSearchTerm] = useState('');
const [debouncedTerm, setDebouncedTerm] = useState('');

useEffect(() => {
  const timer = setTimeout(() => {
    setDebouncedTerm(searchTerm);
  }, 300);
  return () => clearTimeout(timer);
}, [searchTerm]);

// Use debouncedTerm for filtering
```

**Step 5: Profile again**
```
Type "laptop" (5 letters):
  5 SearchBox renders (1ms each = 5ms total)
  1 ProductList render after 300ms pause (45ms)
  
Total: 50ms (vs 250ms before)
80% faster!
```

---

**Interview Tips:**
- React DevTools Profiler = **performance analysis tool**
- Records **component render times**
- Shows **why components rendered**
- Available as **browser extension**
- Two main views: **Flamegraph** and **Ranked**
- Flamegraph width = **render duration**
- Color coding: **yellow=fast, red=slow**
- Shows **exact props** that changed
- "Parent component rendered" = common issue
- Fix with **React.memo**, **useCallback**, **useMemo**
- Can use **Profiler component** programmatically
- **onRender** callback receives timing data
- Profile in **production mode** for accuracy
- Development mode has **extra checks** (slower)
- Look for **patterns** not individual renders
- Set **performance budgets** (< 16ms per frame)
- Compare **before/after** optimizations
- "Record why each component rendered" setting very useful
- **Commit duration** should be < 16ms for 60fps
- Identifies **unnecessary re-renders**
- Shows **expensive calculations**
- Helps find **missing memoization**
- Essential tool for **React performance**
- Use with **React.memo**, **useCallback**, **useMemo**
- Real problems: **cascading re-renders**, **large lists**, **context**

</details>

---

### 60. What is debouncing and throttling in React context?

<details>
<summary>View Answer</summary>

**Debouncing and Throttling**

Both are techniques to **limit how often a function executes**. They're essential for optimizing event handlers that fire frequently (typing, scrolling, resizing).

**Key Difference:**
- **Debouncing:** Wait until user **stops** doing something
- **Throttling:** Execute at **regular intervals** while user is doing something

---

## Debouncing

**Concept:** Wait X milliseconds after the last event before executing

**Visual:**
```
User types: H e l l o
            | | | | |
            v v v v v
         (reset timer each time)
                     |
              (wait 300ms)
                     |
                     v
              Execute function!
```

**Use cases:**
- Search input (wait for user to finish typing)
- Form validation (validate after user stops typing)
- Auto-save (save after user stops editing)
- Window resize (recalculate layout after resize finishes)

---

## Throttling

**Concept:** Execute at most once every X milliseconds

**Visual:**
```
User scrolls continuously:
  Scroll event: |||||||||||||||||||||||||||||
                |
  Execute:      ^       ^       ^       ^
              (every 100ms while scrolling)
```

**Use cases:**
- Scroll events (update position indicator)
- Mouse move tracking
- Window resize (update during resize)
- Game controls
- API rate limiting

---

## Debouncing in React

### Basic Implementation

```jsx
import { useState, useEffect } from 'react';

function SearchInput() {
  const [input, setInput] = useState('');
  const [debouncedValue, setDebouncedValue] = useState('');
  
  // Debounce logic
  useEffect(() => {
    // Set timer
    const timer = setTimeout(() => {
      setDebouncedValue(input);
    }, 500);  // Wait 500ms after last keystroke
    
    // Cleanup: clear timer if input changes
    return () => clearTimeout(timer);
  }, [input]);  // Re-run when input changes
  
  // Search with debounced value
  useEffect(() => {
    if (debouncedValue) {
      console.log('Searching for:', debouncedValue);
      // API call here
    }
  }, [debouncedValue]);
  
  return (
    <div>
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Search..."
      />
      <p>Searching for: {debouncedValue}</p>
    </div>
  );
}
```

**How it works:**
1. User types "hello"
2. Timer resets on each keystroke
3. 500ms after last keystroke ("o")
4. Debounced value updates
5. Search executes

**Result:**
```
User types: h e l l o (5 keystrokes)
Input updates: 5 times (instant feedback)
Search executes: 1 time (after 500ms pause)
```

---

### Custom Debounce Hook

```jsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay) {
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
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // API call
      fetch(`/api/search?q=${debouncedSearchTerm}`)
        .then(res => res.json())
        .then(data => console.log(data));
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input
      type="text"
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

---

### Debounced Callback

```jsx
import { useCallback, useRef } from 'react';

function useDebouncedCallback(callback, delay) {
  const timeoutRef = useRef(null);
  
  return useCallback((...args) => {
    // Clear existing timer
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    // Set new timer
    timeoutRef.current = setTimeout(() => {
      callback(...args);
    }, delay);
  }, [callback, delay]);
}

// Usage
function AutoSave({ content }) {
  const debouncedSave = useDebouncedCallback((content) => {
    console.log('Saving:', content);
    // API call to save
  }, 1000);
  
  return (
    <textarea
      onChange={(e) => debouncedSave(e.target.value)}
      placeholder="Type something..."
    />
  );
}
```

---

## Throttling in React

### Basic Implementation

```jsx
import { useState, useEffect, useRef } from 'react';

function ScrollTracker() {
  const [scrollY, setScrollY] = useState(0);
  const lastRun = useRef(Date.now());
  
  useEffect(() => {
    const handleScroll = () => {
      const now = Date.now();
      
      // Only execute if 100ms passed since last run
      if (now - lastRun.current >= 100) {
        setScrollY(window.scrollY);
        lastRun.current = now;
      }
    };
    
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return <div className="scroll-indicator">Scroll: {scrollY}px</div>;
}
```

**How it works:**
1. User scrolls continuously
2. Scroll event fires hundreds of times
3. Function checks: Has 100ms passed?
4. Yes → Execute and update lastRun
5. No → Skip execution

**Result:**
```
Scroll events: 100 per second
Updates: 10 per second (max once per 100ms)
90% reduction!
```

---

### Custom Throttle Hook

```jsx
import { useCallback, useRef } from 'react';

function useThrottle(callback, delay) {
  const lastRun = useRef(Date.now());
  const timeoutRef = useRef(null);
  
  return useCallback((...args) => {
    const now = Date.now();
    
    if (now - lastRun.current >= delay) {
      // Enough time passed - execute immediately
      callback(...args);
      lastRun.current = now;
    } else {
      // Not enough time - schedule for later
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
      
      timeoutRef.current = setTimeout(() => {
        callback(...args);
        lastRun.current = Date.now();
      }, delay - (now - lastRun.current));
    }
  }, [callback, delay]);
}

// Usage
function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  const throttledUpdate = useThrottle((e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  }, 100);
  
  return (
    <div
      onMouseMove={throttledUpdate}
      style={{ width: '100vw', height: '100vh' }}
    >
      Mouse: {position.x}, {position.y}
    </div>
  );
}
```

---

## Comparison

**Debouncing:**
```jsx
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);
  
  useEffect(() => {
    // Only executes 500ms AFTER user stops typing
    if (debouncedQuery) {
      searchAPI(debouncedQuery);
    }
  }, [debouncedQuery]);
  
  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}

// User types "react": r e a c t
// Timer: reset reset reset reset reset ... (500ms) ... execute!
// API calls: 1 (after user stops)
```

**Throttling:**
```jsx
function ScrollPosition() {
  const [scrollY, setScrollY] = useState(0);
  
  useEffect(() => {
    const handleScroll = throttle(() => {
      // Executes at most every 100ms WHILE scrolling
      setScrollY(window.scrollY);
    }, 100);
    
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);
  
  return <div>Scroll: {scrollY}px</div>;
}

// User scrolls continuously for 1 second
// Scroll events: ~100
// Updates: 10 (every 100ms)
```

---

## Using Lodash

**Install:**
```bash
npm install lodash
```

**Debounce:**
```jsx
import { useState, useCallback } from 'react';
import debounce from 'lodash/debounce';

function SearchInput() {
  const [results, setResults] = useState([]);
  
  // Create debounced function
  const debouncedSearch = useCallback(
    debounce((query) => {
      fetch(`/api/search?q=${query}`)
        .then(res => res.json())
        .then(data => setResults(data));
    }, 500),
    []
  );
  
  return (
    <input
      onChange={(e) => debouncedSearch(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

**Throttle:**
```jsx
import { useState, useCallback, useEffect } from 'react';
import throttle from 'lodash/throttle';

function ScrollProgress() {
  const [progress, setProgress] = useState(0);
  
  const throttledScroll = useCallback(
    throttle(() => {
      const winScroll = window.scrollY;
      const height = document.documentElement.scrollHeight - window.innerHeight;
      const scrolled = (winScroll / height) * 100;
      setProgress(scrolled);
    }, 100),
    []
  );
  
  useEffect(() => {
    window.addEventListener('scroll', throttledScroll);
    return () => window.removeEventListener('scroll', throttledScroll);
  }, [throttledScroll]);
  
  return (
    <div className="progress-bar" style={{ width: `${progress}%` }} />
  );
}
```

---

## Real-World Examples

### Example 1: Search with Debounce

```jsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

function ProductSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const [products, setProducts] = useState([]);
  const [loading, setLoading] = useState(false);
  
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      setLoading(true);
      
      fetch(`/api/products?search=${debouncedSearchTerm}`)
        .then(res => res.json())
        .then(data => {
          setProducts(data);
          setLoading(false);
        });
    } else {
      setProducts([]);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        placeholder="Search products..."
      />
      
      {loading && <div>Searching...</div>}
      
      <div className="results">
        {products.map(product => (
          <div key={product.id}>{product.name}</div>
        ))}
      </div>
      
      {searchTerm && !loading && products.length === 0 && (
        <div>No products found</div>
      )}
    </div>
  );
}
```

**Benefits:**
- User types "laptop" (6 letters)
- Without debounce: 6 API calls
- With debounce: 1 API call
- **83% reduction** in API calls

---

### Example 2: Auto-save with Debounce

```jsx
import { useState, useEffect, useRef } from 'react';

function useDebouncedCallback(callback, delay) {
  const timeoutRef = useRef(null);
  
  return (...args) => {
    if (timeoutRef.current) clearTimeout(timeoutRef.current);
    timeoutRef.current = setTimeout(() => callback(...args), delay);
  };
}

function TextEditor() {
  const [content, setContent] = useState('');
  const [saveStatus, setSaveStatus] = useState('saved');
  
  const saveContent = async (text) => {
    setSaveStatus('saving...');
    
    // Simulate API call
    await new Promise(resolve => setTimeout(resolve, 500));
    console.log('Saved:', text);
    
    setSaveStatus('saved');
  };
  
  const debouncedSave = useDebouncedCallback(saveContent, 1000);
  
  const handleChange = (e) => {
    const newContent = e.target.value;
    setContent(newContent);
    setSaveStatus('typing...');
    debouncedSave(newContent);
  };
  
  return (
    <div>
      <div className="status">{saveStatus}</div>
      <textarea
        value={content}
        onChange={handleChange}
        placeholder="Start typing..."
        rows={10}
        cols={50}
      />
    </div>
  );
}
```

---

### Example 3: Infinite Scroll with Throttle

```jsx
import { useState, useEffect, useCallback, useRef } from 'react';

function useThrottle(callback, delay) {
  const lastRun = useRef(Date.now());
  
  return useCallback((...args) => {
    const now = Date.now();
    if (now - lastRun.current >= delay) {
      callback(...args);
      lastRun.current = now;
    }
  }, [callback, delay]);
}

function InfiniteScrollList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  
  const loadMore = useCallback(() => {
    if (loading) return;
    
    setLoading(true);
    
    // Simulate API call
    setTimeout(() => {
      const newItems = Array.from({ length: 20 }, (_, i) => 
        `Item ${(page - 1) * 20 + i + 1}`
      );
      setItems(prev => [...prev, ...newItems]);
      setPage(prev => prev + 1);
      setLoading(false);
    }, 500);
  }, [page, loading]);
  
  const throttledLoadMore = useThrottle(loadMore, 1000);
  
  useEffect(() => {
    loadMore(); // Initial load
  }, []);
  
  useEffect(() => {
    const handleScroll = () => {
      const bottom = Math.ceil(window.innerHeight + window.scrollY) >= 
                     document.documentElement.scrollHeight;
      
      if (bottom) {
        throttledLoadMore();
      }
    };
    
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, [throttledLoadMore]);
  
  return (
    <div>
      {items.map((item, index) => (
        <div key={index} style={{ padding: '20px', border: '1px solid #ddd' }}>
          {item}
        </div>
      ))}
      
      {loading && <div>Loading more...</div>}
    </div>
  );
}
```

---

## Performance Impact

**Without debouncing/throttling:**
```
Search input (typing "react"):
  Keystrokes: 5
  API calls: 5
  Total time: 250ms (50ms per call)
  Network: High load
```

**With debouncing:**
```
Search input (typing "react"):
  Keystrokes: 5
  API calls: 1 (after 500ms pause)
  Total time: 50ms
  Network: 80% reduction
```

**Scroll tracking (1 second of scrolling):**

Without throttling:
```
Scroll events: ~100
State updates: 100
Re-renders: 100
FPS: 30-40 (laggy)
```

With throttling (100ms):
```
Scroll events: ~100
State updates: 10 (max once per 100ms)
Re-renders: 10
FPS: 60 (smooth)
```

---

## Common Patterns

| Use Case | Technique | Delay |
|----------|-----------|-------|
| Search input | Debounce | 300-500ms |
| Auto-save | Debounce | 1000-2000ms |
| Form validation | Debounce | 500ms |
| Window resize | Debounce | 150-300ms |
| Scroll position | Throttle | 50-100ms |
| Mouse move | Throttle | 50-100ms |
| Button click spam | Debounce | 300ms |
| Infinite scroll | Throttle | 200-500ms |
| API rate limiting | Throttle | 1000ms |

---

**Interview Tips:**
- **Debouncing** = wait until user **stops**
- **Throttling** = execute at **regular intervals**
- Debounce for **search inputs** (300-500ms)
- Throttle for **scroll events** (100ms)
- Both **reduce function calls**
- Debounce **delays** execution
- Throttle **limits rate** of execution
- Use **useEffect** with **setTimeout** for debounce
- Use **useRef** to track **last execution time** for throttle
- **Custom hooks** for reusability
- **Lodash** has built-in debounce/throttle
- Debounce example: **search after typing stops**
- Throttle example: **update scroll position every 100ms**
- Improves **performance** significantly
- Reduces **API calls** (debounce)
- Reduces **re-renders** (throttle)
- Critical for **high-frequency events**
- Search without debounce = **bad UX** (too many requests)
- Scroll without throttle = **janky** (too many updates)
- Common mistake: **not cleaning up timers**
- Use **cleanup function** in useEffect
- Debounce **delay** depends on use case (300-2000ms)
- Throttle **interval** usually 50-200ms
- Both are **optimization techniques**
- Essential for **production React apps**

</details>
