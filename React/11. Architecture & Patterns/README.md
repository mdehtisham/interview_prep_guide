# Architecture & Patterns

> Advanced / Senior Level (3-5 years)

---

## Questions

101. What is the Component Composition pattern?
102. What are Render Props and when do you use them?
103. What are Higher-Order Components (HOC)?
104. What is the difference between HOC and custom hooks?
105. What is the Compound Components pattern?
106. What is the Container/Presentational pattern?
107. What is the Provider pattern?
108. What are Controlled vs Uncontrolled patterns?
109. What is the Observer pattern in React?
110. What is Dependency Injection in React?

---

## Detailed Answers

### 101. What is the Component Composition pattern?

<details>
<summary>View Answer</summary>

**Component Composition** is a design pattern where you build complex components by combining simpler, reusable components together, rather than using inheritance or prop drilling.

#### What is Component Composition?

Composition allows you to:
- **Combine components** like building blocks
- **Pass components as props** (including children)
- **Create flexible, reusable** component hierarchies
- **Avoid prop drilling** by passing components directly
- **Implement inversion of control**

---

#### Basic Examples

**1. Using `children` prop:**
```tsx
// Simple container composition
interface CardProps {
  children: React.ReactNode;
}

const Card: React.FC<CardProps> = ({ children }) => {
  return (
    <div className="card">
      {children}
    </div>
  );
};

// Usage
function App() {
  return (
    <Card>
      <h2>Card Title</h2>
      <p>Card content here</p>
    </Card>
  );
}
```

**2. Multiple composition slots:**
```tsx
interface LayoutProps {
  header: React.ReactNode;
  sidebar: React.ReactNode;
  content: React.ReactNode;
  footer: React.ReactNode;
}

const Layout: React.FC<LayoutProps> = ({ header, sidebar, content, footer }) => {
  return (
    <div className="layout">
      <header>{header}</header>
      <div className="main">
        <aside>{sidebar}</aside>
        <main>{content}</main>
      </div>
      <footer>{footer}</footer>
    </div>
  );
};

// Usage
function App() {
  return (
    <Layout
      header={<Header />}
      sidebar={<Sidebar />}
      content={<MainContent />}
      footer={<Footer />}
    />
  );
}
```

---

#### Advanced Composition Patterns

**1. Specialization (Creating specific versions):**
```tsx
// Generic Dialog
interface DialogProps {
  title: string;
  children: React.ReactNode;
  onClose: () => void;
}

const Dialog: React.FC<DialogProps> = ({ title, children, onClose }) => {
  return (
    <div className="dialog">
      <div className="dialog-header">
        <h2>{title}</h2>
        <button onClick={onClose}>×</button>
      </div>
      <div className="dialog-content">
        {children}
      </div>
    </div>
  );
};

// Specialized Dialog
interface ConfirmDialogProps {
  message: string;
  onConfirm: () => void;
  onCancel: () => void;
}

const ConfirmDialog: React.FC<ConfirmDialogProps> = ({ 
  message, 
  onConfirm, 
  onCancel 
}) => {
  return (
    <Dialog title="Confirm Action" onClose={onCancel}>
      <p>{message}</p>
      <div className="dialog-actions">
        <button onClick={onCancel}>Cancel</button>
        <button onClick={onConfirm}>Confirm</button>
      </div>
    </Dialog>
  );
};
```

**2. Containment (Sidebar with slots):**
```tsx
interface SidebarProps {
  children: React.ReactNode;
}

const Sidebar: React.FC<SidebarProps> = ({ children }) => {
  return (
    <aside className="sidebar">
      {children}
    </aside>
  );
};

const SidebarHeader: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return <div className="sidebar-header">{children}</div>;
};

const SidebarContent: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return <div className="sidebar-content">{children}</div>;
};

const SidebarFooter: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return <div className="sidebar-footer">{children}</div>;
};

// Attach sub-components
Sidebar.Header = SidebarHeader;
Sidebar.Content = SidebarContent;
Sidebar.Footer = SidebarFooter;

// Usage
function App() {
  return (
    <Sidebar>
      <Sidebar.Header>
        <h2>Menu</h2>
      </Sidebar.Header>
      <Sidebar.Content>
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
      </Sidebar.Content>
      <Sidebar.Footer>
        <p>© 2024</p>
      </Sidebar.Footer>
    </Sidebar>
  );
}
```

---

#### Real-World Example: Form Builder

```tsx
// Base form components
interface FormProps {
  onSubmit: (e: React.FormEvent) => void;
  children: React.ReactNode;
}

const Form: React.FC<FormProps> = ({ onSubmit, children }) => {
  return (
    <form onSubmit={onSubmit} className="form">
      {children}
    </form>
  );
};

interface FormSectionProps {
  title: string;
  children: React.ReactNode;
}

const FormSection: React.FC<FormSectionProps> = ({ title, children }) => {
  return (
    <div className="form-section">
      <h3>{title}</h3>
      {children}
    </div>
  );
};

interface FormFieldProps {
  label: string;
  children: React.ReactNode;
  error?: string;
}

const FormField: React.FC<FormFieldProps> = ({ label, children, error }) => {
  return (
    <div className="form-field">
      <label>{label}</label>
      {children}
      {error && <span className="error">{error}</span>}
    </div>
  );
};

const FormActions: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  return <div className="form-actions">{children}</div>;
};

// Attach sub-components
Form.Section = FormSection;
Form.Field = FormField;
Form.Actions = FormActions;

// Usage - Compose a complex form
function RegistrationForm() {
  const [errors, setErrors] = useState<Record<string, string>>({});

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    // Handle form submission
  };

  return (
    <Form onSubmit={handleSubmit}>
      <Form.Section title="Personal Information">
        <Form.Field label="First Name" error={errors.firstName}>
          <input type="text" name="firstName" />
        </Form.Field>
        <Form.Field label="Last Name" error={errors.lastName}>
          <input type="text" name="lastName" />
        </Form.Field>
        <Form.Field label="Email" error={errors.email}>
          <input type="email" name="email" />
        </Form.Field>
      </Form.Section>

      <Form.Section title="Account Details">
        <Form.Field label="Username" error={errors.username}>
          <input type="text" name="username" />
        </Form.Field>
        <Form.Field label="Password" error={errors.password}>
          <input type="password" name="password" />
        </Form.Field>
      </Form.Section>

      <Form.Actions>
        <button type="button">Cancel</button>
        <button type="submit">Register</button>
      </Form.Actions>
    </Form>
  );
}
```

---

#### Composition vs Inheritance

| Aspect | Composition | Inheritance |
|--------|-------------|-------------|
| **Flexibility** | High - mix and match components | Limited - fixed hierarchy |
| **Reusability** | Excellent - components stay independent | Poor - tightly coupled |
| **React Philosophy** | ✅ Recommended | ❌ Not recommended |
| **Type Safety** | Easy with TypeScript | Complex with generics |
| **Testing** | Easy - test components separately | Hard - must test hierarchy |
| **Learning Curve** | Gentle | Steep |

---

#### Benefits of Composition

**1. Flexibility:**
```tsx
// Can easily swap implementations
<Card>
  <SimpleHeader title="Hello" />
</Card>

// Or use a different header
<Card>
  <ComplexHeader 
    title="Hello" 
    subtitle="World"
    actions={<button>Edit</button>}
  />
</Card>
```

**2. Avoiding Prop Drilling:**
```tsx
// Instead of passing props through many levels
<Page user={user}>
  <Header user={user}>
    <UserMenu user={user} />
  </Header>
</Page>

// Compose directly
<Page>
  <Header>
    <UserMenu user={user} />
  </Header>
</Page>
```

**3. Code Reusability:**
```tsx
// Reuse the same Dialog with different content
<Dialog title="Success">
  <SuccessIcon />
  <p>Operation completed!</p>
</Dialog>

<Dialog title="Error">
  <ErrorIcon />
  <p>Something went wrong!</p>
  <ErrorDetails />
</Dialog>
```

---

#### Common Patterns

**1. Slot Pattern:**
```tsx
interface PageProps {
  hero?: React.ReactNode;
  content: React.ReactNode;
  sidebar?: React.ReactNode;
}

const Page: React.FC<PageProps> = ({ hero, content, sidebar }) => {
  return (
    <div className="page">
      {hero && <div className="hero">{hero}</div>}
      <div className="main">
        <div className="content">{content}</div>
        {sidebar && <div className="sidebar">{sidebar}</div>}
      </div>
    </div>
  );
};
```

**2. Clone and Enhance Pattern:**
```tsx
import { Children, cloneElement, isValidElement } from 'react';

interface TabsProps {
  activeTab: string;
  children: React.ReactNode;
}

const Tabs: React.FC<TabsProps> = ({ activeTab, children }) => {
  return (
    <div className="tabs">
      {Children.map(children, (child) => {
        if (isValidElement(child)) {
          // Inject props into children
          return cloneElement(child, {
            isActive: child.props.id === activeTab,
          });
        }
        return child;
      })}
    </div>
  );
};
```

---

#### Best Practices

1. **Use `children` for simple composition**
   ```tsx
   <Container>{content}</Container>
   ```

2. **Use named props for multiple slots**
   ```tsx
   <Layout header={<Header />} content={<Content />} />
   ```

3. **Create namespaced sub-components**
   ```tsx
   Card.Header, Card.Body, Card.Footer
   ```

4. **Keep components small and focused**
   - Each component should do one thing well

5. **Use TypeScript for type safety**
   ```tsx
   interface Props {
     children: React.ReactNode;
     header?: React.ReactNode;
   }
   ```

6. **Document component composition patterns**
   - Show usage examples in comments/docs

---

#### Interview Tips

✅ **Key Points:**
- Composition is React's way of achieving code reuse
- React recommends composition over inheritance
- Use `children` prop for containment
- Use named props for multiple slots
- Components should be composable building blocks

✅ **When to Mention:**
- Discussing component design patterns
- Explaining React philosophy
- Comparing React to class-based frameworks
- Talking about code reusability
- Discussing component libraries

✅ **Common Follow-ups:**
- "How is this different from inheritance?"
- "What are the benefits?"
- "Show me a real-world example"
- "How does this work with TypeScript?"

</details>

---

### 102. What are Render Props and when do you use them?

<details>
<summary>View Answer</summary>

**Render Props** is a pattern where a component receives a function as a prop, and that function returns React elements. This allows you to share code and logic between components without using HOCs or inheritance.

#### What are Render Props?

The pattern involves:
- **Passing a function** that returns JSX as a prop
- **The function receives data/methods** from the component
- **Allows logic reuse** without wrapper components
- **Provides inversion of control** to the consumer

---

#### Basic Example

```tsx
interface MousePosition {
  x: number;
  y: number;
}

interface MouseTrackerProps {
  render: (position: MousePosition) => React.ReactNode;
}

class MouseTracker extends React.Component<MouseTrackerProps, MousePosition> {
  state: MousePosition = { x: 0, y: 0 };

  handleMouseMove = (event: MouseEvent) => {
    this.setState({
      x: event.clientX,
      y: event.clientY,
    });
  };

  componentDidMount() {
    window.addEventListener('mousemove', this.handleMouseMove);
  }

  componentWillUnmount() {
    window.removeEventListener('mousemove', this.handleMouseMove);
  }

  render() {
    return (
      <div>
        {this.props.render(this.state)}
      </div>
    );
  }
}

// Usage
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <h1>The mouse position is ({x}, {y})</h1>
      )}
    />
  );
}
```

---

#### Using `children` as Render Prop

```tsx
interface MousePosition {
  x: number;
  y: number;
}

interface MouseProps {
  children: (position: MousePosition) => React.ReactNode;
}

const Mouse: React.FC<MouseProps> = ({ children }) => {
  const [position, setPosition] = useState<MousePosition>({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (event: MouseEvent) => {
      setPosition({ x: event.clientX, y: event.clientY });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return <>{children(position)}</>;
};

// Usage - more natural with children
function App() {
  return (
    <Mouse>
      {({ x, y }) => (
        <div>
          <h1>Mouse Position</h1>
          <p>X: {x}, Y: {y}</p>
        </div>
      )}
    </Mouse>
  );
}
```

---

#### Real-World Example: Data Fetching

```tsx
interface DataFetcherProps<T> {
  url: string;
  children: (data: {
    data: T | null;
    loading: boolean;
    error: Error | null;
    refetch: () => void;
  }) => React.ReactNode;
}

function DataFetcher<T>({ url, children }: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error('Failed to fetch');
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return <>{children({ data, loading, error, refetch: fetchData })}</>;
}

// Usage
interface User {
  id: number;
  name: string;
  email: string;
}

function UserProfile() {
  return (
    <DataFetcher<User> url="/api/user/123">
      {({ data, loading, error, refetch }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error.message}</div>;
        if (!data) return <div>No data</div>;

        return (
          <div>
            <h1>{data.name}</h1>
            <p>{data.email}</p>
            <button onClick={refetch}>Refresh</button>
          </div>
        );
      }}
    </DataFetcher>
  );
}
```

---

#### Multiple Render Props

```tsx
interface WindowSizeProps {
  renderMobile?: (size: { width: number; height: number }) => React.ReactNode;
  renderDesktop?: (size: { width: number; height: number }) => React.ReactNode;
}

const WindowSize: React.FC<WindowSizeProps> = ({ 
  renderMobile, 
  renderDesktop 
}) => {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  const isMobile = size.width < 768;

  if (isMobile && renderMobile) {
    return <>{renderMobile(size)}</>;
  }

  if (!isMobile && renderDesktop) {
    return <>{renderDesktop(size)}</>;
  }

  return null;
};

// Usage
function App() {
  return (
    <WindowSize
      renderMobile={(size) => (
        <div>
          <h1>Mobile View</h1>
          <p>Width: {size.width}px</p>
        </div>
      )}
      renderDesktop={(size) => (
        <div>
          <h1>Desktop View</h1>
          <p>Screen: {size.width} × {size.height}</p>
        </div>
      )}
    />
  );
}
```

---

#### Advanced Pattern: Toggle Component

```tsx
interface ToggleProps {
  children: (state: {
    on: boolean;
    toggle: () => void;
    setOn: (value: boolean) => void;
  }) => React.ReactNode;
  defaultOn?: boolean;
}

const Toggle: React.FC<ToggleProps> = ({ children, defaultOn = false }) => {
  const [on, setOn] = useState(defaultOn);

  const toggle = useCallback(() => {
    setOn((prev) => !prev);
  }, []);

  return <>{children({ on, toggle, setOn })}</>;
};

// Usage - Different UI implementations
function App() {
  return (
    <div>
      {/* Switch implementation */}
      <Toggle>
        {({ on, toggle }) => (
          <label>
            <input type="checkbox" checked={on} onChange={toggle} />
            Switch is {on ? 'ON' : 'OFF'}
          </label>
        )}
      </Toggle>

      {/* Button implementation */}
      <Toggle defaultOn={true}>
        {({ on, toggle }) => (
          <button onClick={toggle}>
            {on ? '🌙 Dark Mode' : '☀️ Light Mode'}
          </button>
        )}
      </Toggle>

      {/* Conditional rendering */}
      <Toggle>
        {({ on, toggle, setOn }) => (
          <div>
            <button onClick={toggle}>Toggle</button>
            <button onClick={() => setOn(true)}>Show</button>
            <button onClick={() => setOn(false)}>Hide</button>
            {on && <div>Content is visible!</div>}
          </div>
        )}
      </Toggle>
    </div>
  );
}
```

---

#### Real-World: Form Field with Validation

```tsx
interface ValidationResult {
  isValid: boolean;
  errors: string[];
}

interface ValidatedFieldProps {
  name: string;
  validators: Array<(value: string) => string | null>;
  children: (state: {
    value: string;
    onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
    onBlur: () => void;
    validation: ValidationResult;
    touched: boolean;
  }) => React.ReactNode;
}

const ValidatedField: React.FC<ValidatedFieldProps> = ({ 
  name, 
  validators, 
  children 
}) => {
  const [value, setValue] = useState('');
  const [touched, setTouched] = useState(false);

  const validation = useMemo<ValidationResult>(() => {
    const errors = validators
      .map((validator) => validator(value))
      .filter((error): error is string => error !== null);

    return {
      isValid: errors.length === 0,
      errors,
    };
  }, [value, validators]);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };

  const handleBlur = () => {
    setTouched(true);
  };

  return (
    <>
      {children({
        value,
        onChange: handleChange,
        onBlur: handleBlur,
        validation,
        touched,
      })}
    </>
  );
};

// Usage
function RegistrationForm() {
  const emailValidators = [
    (value: string) => !value ? 'Email is required' : null,
    (value: string) => !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) 
      ? 'Invalid email format' 
      : null,
  ];

  return (
    <form>
      <ValidatedField name="email" validators={emailValidators}>
        {({ value, onChange, onBlur, validation, touched }) => (
          <div className="form-field">
            <label>Email</label>
            <input
              type="email"
              value={value}
              onChange={onChange}
              onBlur={onBlur}
              className={touched && !validation.isValid ? 'error' : ''}
            />
            {touched && !validation.isValid && (
              <div className="errors">
                {validation.errors.map((error, i) => (
                  <div key={i} className="error-message">
                    {error}
                  </div>
                ))}
              </div>
            )}
            {touched && validation.isValid && (
              <div className="success">✓ Valid email</div>
            )}
          </div>
        )}
      </ValidatedField>
    </form>
  );
}
```

---

#### When to Use Render Props

**✅ Good Use Cases:**

1. **Sharing stateful logic** between components
   ```tsx
   <WindowSize>{(size) => <UI size={size} />}</WindowSize>
   ```

2. **Cross-cutting concerns** (auth, theming, etc.)
   ```tsx
   <Auth>{(user) => <Dashboard user={user} />}</Auth>
   ```

3. **Flexible rendering** based on state
   ```tsx
   <DataFetcher>{({ data, loading }) => /* custom UI */}</DataFetcher>
   ```

4. **Component logic reuse** with different UIs
   ```tsx
   <Toggle>{({ on }) => /* any toggle UI */}</Toggle>
   ```

**❌ Avoid When:**

1. **Simple prop passing** (use regular props)
2. **Component composition** is sufficient (use children)
3. **Custom hooks** would be cleaner (modern React)
4. **Performance critical** (function creates new reference)

---

#### Render Props vs Other Patterns

| Pattern | Pros | Cons | When to Use |
|---------|------|------|-------------|
| **Render Props** | Flexible rendering, explicit data flow | Callback hell, wrapper nesting | Complex logic reuse with flexible UI |
| **HOC** | Compose multiple enhancements | Props collision, wrapper hell | Adding cross-cutting behavior |
| **Custom Hooks** | Clean, composable, no nesting | Only works with hooks | Modern React, logic reuse |
| **Component Composition** | Simple, intuitive | Less dynamic | Fixed component structure |

---

#### Migration to Custom Hooks

Render Props pattern has largely been **replaced by custom hooks** in modern React:

**Old (Render Props):**
```tsx
<Mouse>
  {({ x, y }) => (
    <p>Mouse at {x}, {y}</p>
  )}
</Mouse>
```

**New (Custom Hook):**
```tsx
function useMouse() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return position;
}

// Usage - Much cleaner!
function App() {
  const { x, y } = useMouse();
  return <p>Mouse at {x}, {y}</p>;
}
```

---

#### Performance Considerations

**Problem: Inline Functions**
```tsx
// Creates new function on every render
<DataFetcher url="/api/data">
  {({ data }) => <div>{data}</div>}
</DataFetcher>
```

**Solution 1: Extract Function**
```tsx
const renderData = ({ data }: { data: string }) => <div>{data}</div>;

<DataFetcher url="/api/data">
  {renderData}
</DataFetcher>
```

**Solution 2: Use useCallback**
```tsx
const renderData = useCallback(({ data }: { data: string }) => {
  return <div>{data}</div>;
}, []);

<DataFetcher url="/api/data">
  {renderData}
</DataFetcher>
```

---

#### Best Practices

1. **Use descriptive prop names**
   ```tsx
   // Good
   <List renderItem={(item) => <Item {...item} />} />
   
   // Less clear
   <List render={(item) => <Item {...item} />} />
   ```

2. **Provide type safety**
   ```tsx
   interface Props<T> {
     children: (data: T) => React.ReactNode;
   }
   ```

3. **Consider using children prop**
   ```tsx
   // More natural
   <Toggle>{(on) => <Button active={on} />}</Toggle>
   ```

4. **Avoid deep nesting**
   ```tsx
   // Bad - pyramid of doom
   <A>{(a) => <B>{(b) => <C>{(c) => ...}</C>}</B>}</A>
   
   // Better - use custom hooks instead
   ```

5. **Document the render function signature**
   ```tsx
   /**
    * @param render Function that receives { data, loading, error }
    */
   ```

---

#### Interview Tips

✅ **Key Points:**
- Render props is a pattern for code reuse
- Pass a function that returns JSX as a prop
- Provides flexible rendering based on component state
- Largely replaced by custom hooks in modern React
- Good for complex logic with flexible UI

✅ **When to Mention:**
- Discussing code reuse patterns
- Comparing with HOCs and hooks
- Legacy codebases
- Component library design
- Inversion of control patterns

✅ **Common Follow-ups:**
- "How is this different from HOC?"
- "Why use hooks instead?"
- "What are the performance implications?"
- "Show me a real-world example"
- "How does TypeScript work with this?"

✅ **Be Ready to Discuss:**
- Migration path to custom hooks
- When render props are still useful
- Performance optimization techniques
- Comparison with other patterns

</details>

---

### 103. What are Higher-Order Components (HOC)?

<details>
<summary>View Answer</summary>

**Higher-Order Component (HOC)** is a pattern where a function takes a component as an argument and returns a new component with additional props, state, or behavior. It's a way to reuse component logic.

#### What is a HOC?

A HOC is:
- **A function** that takes a component and returns a new component
- **Not a component itself** - it's a function
- **For code reuse** - share logic between components
- **Pure function** - doesn't modify the original component
- **Wraps components** - creates a wrapper component

```tsx
const EnhancedComponent = higherOrderComponent(OriginalComponent);
```

---

#### Basic Example

```tsx
import React from 'react';

// HOC that adds a loading prop
function withLoading<P extends object>(
  Component: React.ComponentType<P>
) {
  return function WithLoadingComponent(
    props: P & { isLoading: boolean }
  ) {
    const { isLoading, ...restProps } = props;

    if (isLoading) {
      return <div>Loading...</div>;
    }

    return <Component {...(restProps as P)} />;
  };
}

// Original component
interface UserProps {
  name: string;
  email: string;
}

const User: React.FC<UserProps> = ({ name, email }) => {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
};

// Enhanced component
const UserWithLoading = withLoading(User);

// Usage
function App() {
  const [loading, setLoading] = useState(true);

  return (
    <UserWithLoading 
      name="John Doe" 
      email="john@example.com" 
      isLoading={loading}
    />
  );
}
```

---

#### Common HOC Patterns

**1. Props Injection:**
```tsx
// Inject user authentication data
interface WithAuthProps {
  user: User | null;
  isAuthenticated: boolean;
}

function withAuth<P extends object>(
  Component: React.ComponentType<P & WithAuthProps>
) {
  return function WithAuthComponent(props: P) {
    const [user, setUser] = useState<User | null>(null);
    const [isAuthenticated, setIsAuthenticated] = useState(false);

    useEffect(() => {
      // Check authentication
      const authUser = getCurrentUser();
      setUser(authUser);
      setIsAuthenticated(!!authUser);
    }, []);

    return (
      <Component 
        {...props} 
        user={user} 
        isAuthenticated={isAuthenticated}
      />
    );
  };
}

// Usage
interface DashboardProps {
  title: string;
}

const Dashboard: React.FC<DashboardProps & WithAuthProps> = ({ 
  title, 
  user, 
  isAuthenticated 
}) => {
  if (!isAuthenticated) {
    return <div>Please log in</div>;
  }

  return (
    <div>
      <h1>{title}</h1>
      <p>Welcome, {user?.name}</p>
    </div>
  );
};

const DashboardWithAuth = withAuth(Dashboard);
```

**2. Conditional Rendering:**
```tsx
interface WithVisibilityProps {
  visible: boolean;
}

function withVisibility<P extends object>(
  Component: React.ComponentType<P>
) {
  return function WithVisibilityComponent(
    props: P & WithVisibilityProps
  ) {
    const { visible, ...restProps } = props;

    if (!visible) {
      return null;
    }

    return <Component {...(restProps as P)} />;
  };
}

// Usage
const Message: React.FC<{ text: string }> = ({ text }) => {
  return <div className="message">{text}</div>;
};

const ConditionalMessage = withVisibility(Message);

// Use it
<ConditionalMessage text="Hello" visible={true} />
```

**3. State Management:**
```tsx
interface WithToggleProps {
  isOn: boolean;
  toggle: () => void;
}

function withToggle<P extends object>(
  Component: React.ComponentType<P & WithToggleProps>
) {
  return function WithToggleComponent(props: P) {
    const [isOn, setIsOn] = useState(false);

    const toggle = useCallback(() => {
      setIsOn((prev) => !prev);
    }, []);

    return <Component {...props} isOn={isOn} toggle={toggle} />;
  };
}

// Usage
interface SwitchProps {
  label: string;
}

const Switch: React.FC<SwitchProps & WithToggleProps> = ({ 
  label, 
  isOn, 
  toggle 
}) => {
  return (
    <div>
      <label>
        {label}
        <input type="checkbox" checked={isOn} onChange={toggle} />
      </label>
    </div>
  );
};

const SwitchWithToggle = withToggle(Switch);
```

---

#### Real-World Example: Data Fetching HOC

```tsx
interface WithDataProps<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

interface WithDataConfig {
  url: string;
  loadOnMount?: boolean;
}

function withData<T, P extends object>(
  config: WithDataConfig
) {
  return function (Component: React.ComponentType<P & WithDataProps<T>>) {
    return function WithDataComponent(props: P) {
      const [data, setData] = useState<T | null>(null);
      const [loading, setLoading] = useState(false);
      const [error, setError] = useState<Error | null>(null);

      const fetchData = useCallback(async () => {
        setLoading(true);
        setError(null);
        try {
          const response = await fetch(config.url);
          if (!response.ok) throw new Error('Failed to fetch');
          const result = await response.json();
          setData(result);
        } catch (err) {
          setError(err as Error);
        } finally {
          setLoading(false);
        }
      }, []);

      useEffect(() => {
        if (config.loadOnMount !== false) {
          fetchData();
        }
      }, [fetchData]);

      return (
        <Component
          {...props}
          data={data}
          loading={loading}
          error={error}
          refetch={fetchData}
        />
      );
    };
  };
}

// Usage
interface User {
  id: number;
  name: string;
  email: string;
}

interface UserListProps {
  title: string;
}

const UserList: React.FC<UserListProps & WithDataProps<User[]>> = ({
  title,
  data,
  loading,
  error,
  refetch,
}) => {
  if (loading) return <div>Loading users...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!data) return <div>No data</div>;

  return (
    <div>
      <h1>{title}</h1>
      <button onClick={refetch}>Refresh</button>
      <ul>
        {data.map((user) => (
          <li key={user.id}>
            {user.name} - {user.email}
          </li>
        ))}
      </ul>
    </div>
  );
};

const UserListWithData = withData<User[], UserListProps>({
  url: '/api/users',
  loadOnMount: true,
})(UserList);
```

---

#### Composing Multiple HOCs

```tsx
// Multiple HOCs
const withAuth = (Component) => { /* ... */ };
const withLoading = (Component) => { /* ... */ };
const withError = (Component) => { /* ... */ };

// Manual composition (nested)
const Enhanced = withAuth(withLoading(withError(MyComponent)));

// Better: Use compose utility
import { compose } from 'redux'; // or create your own

const enhance = compose(
  withAuth,
  withLoading,
  withError
);

const EnhancedComponent = enhance(MyComponent);

// Or using lodash/flowRight
import { flowRight } from 'lodash';

const enhance = flowRight(
  withAuth,
  withLoading,
  withError
);
```

**Custom compose function:**
```tsx
function compose<T>(...fns: Array<(arg: T) => T>) {
  return (initialArg: T) => {
    return fns.reduceRight((acc, fn) => fn(acc), initialArg);
  };
}
```

---

#### Advanced Pattern: Configuration HOC

```tsx
interface WithTrackingConfig {
  eventName: string;
  trackOnMount?: boolean;
}

function withTracking<P extends object>(
  config: WithTrackingConfig
) {
  return function (Component: React.ComponentType<P>) {
    return function WithTrackingComponent(props: P) {
      useEffect(() => {
        if (config.trackOnMount !== false) {
          analytics.track(config.eventName, { props });
        }
      }, []);

      const handleClick = useCallback(() => {
        analytics.track(`${config.eventName}_click`, { props });
      }, [props]);

      return (
        <div onClick={handleClick}>
          <Component {...props} />
        </div>
      );
    };
  };
}

// Usage
const ProductCard: React.FC<{ name: string; price: number }> = ({ 
  name, 
  price 
}) => {
  return (
    <div>
      <h3>{name}</h3>
      <p>${price}</p>
    </div>
  );
};

const TrackedProductCard = withTracking({
  eventName: 'product_viewed',
  trackOnMount: true,
})(ProductCard);
```

---

#### HOC Best Practices

**1. Don't Mutate the Original Component:**
```tsx
// ❌ Bad - mutates original
function withSomething(Component) {
  Component.prototype.componentDidUpdate = function() {
    // ...
  };
  return Component;
}

// ✅ Good - returns new component
function withSomething(Component) {
  return function WithSomethingComponent(props) {
    return <Component {...props} />;
  };
}
```

**2. Pass Unrelated Props Through:**
```tsx
function withData(Component) {
  return function WithDataComponent(props) {
    const { data, loading } = useData();
    
    // Pass all props through
    return <Component {...props} data={data} loading={loading} />;
  };
}
```

**3. Maximize Composability:**
```tsx
// ✅ Good - returns function that takes component
function withAuth(Component) {
  return function WithAuthComponent(props) {
    // ...
  };
}

// Can be composed
const enhanced = compose(withAuth, withData, withTracking);
```

**4. Use Display Names for Debugging:**
```tsx
function withData(Component) {
  function WithDataComponent(props) {
    // ...
  }

  // Set display name
  WithDataComponent.displayName = `WithData(${getDisplayName(Component)})`;

  return WithDataComponent;
}

function getDisplayName(Component) {
  return Component.displayName || Component.name || 'Component';
}
```

**5. Copy Static Methods:**
```tsx
import hoistNonReactStatics from 'hoist-non-react-statics';

function withSomething(Component) {
  function WithSomethingComponent(props) {
    return <Component {...props} />;
  }

  // Copy static methods
  hoistNonReactStatics(WithSomethingComponent, Component);

  return WithSomethingComponent;
}
```

---

#### Common Pitfalls

**1. Don't Use HOCs Inside render:**
```tsx
// ❌ Bad - creates new component every render
function MyComponent() {
  const EnhancedComponent = withData(SomeComponent);
  return <EnhancedComponent />;
}

// ✅ Good - create once outside
const EnhancedComponent = withData(SomeComponent);

function MyComponent() {
  return <EnhancedComponent />;
}
```

**2. Refs Aren't Passed Through:**
```tsx
// Problem: ref is not forwarded
function withData(Component) {
  return function WithDataComponent(props) {
    return <Component {...props} />;
  };
}

// Solution: Use forwardRef
function withData(Component) {
  function WithDataComponent(props, ref) {
    return <Component {...props} ref={ref} />;
  }

  return React.forwardRef(WithDataComponent);
}
```

---

#### TypeScript with HOCs

```tsx
import React from 'react';

// Generic HOC with proper typing
interface WithLoadingProps {
  isLoading: boolean;
}

function withLoading<P extends object>(
  Component: React.ComponentType<P>
): React.FC<P & WithLoadingProps> {
  return function WithLoadingComponent({ isLoading, ...props }: WithLoadingProps & P) {
    if (isLoading) {
      return <div>Loading...</div>;
    }

    return <Component {...(props as P)} />;
  };
}

// Usage with TypeScript
interface UserProps {
  name: string;
  email: string;
}

const User: React.FC<UserProps> = ({ name, email }) => (
  <div>
    <h2>{name}</h2>
    <p>{email}</p>
  </div>
);

const UserWithLoading = withLoading(User);

// Type-safe usage
<UserWithLoading 
  name="John" 
  email="john@example.com" 
  isLoading={false}
/>
```

---

#### When to Use HOCs

**✅ Good Use Cases:**

1. **Cross-cutting concerns** (auth, logging, analytics)
2. **Legacy codebases** (before hooks)
3. **Third-party library integration**
4. **Wrapping class components** (can't use hooks)

**❌ Prefer Hooks Instead:**

1. **Logic reuse** (use custom hooks)
2. **State management** (use hooks)
3. **Side effects** (use useEffect)
4. **Modern React apps** (hooks are cleaner)

---

#### Interview Tips

✅ **Key Points:**
- HOC is a function that takes a component and returns a new component
- Used for code reuse and cross-cutting concerns
- Pure function - doesn't modify original component
- Can compose multiple HOCs together
- Largely replaced by hooks in modern React

✅ **When to Mention:**
- Discussing code reuse patterns
- Legacy React patterns
- Class component patterns
- Comparing with hooks and render props
- Component composition strategies

✅ **Common Follow-ups:**
- "How is this different from hooks?"
- "What are the disadvantages?"
- "How do you compose multiple HOCs?"
- "How do you handle TypeScript?"
- "When would you still use HOCs?"

✅ **Be Ready to Discuss:**
- Migration path to hooks
- Props collision problems
- Ref forwarding issues
- Display name conventions
- Performance implications

</details>

---

### 104. What is the difference between HOC and custom hooks?

<details>
<summary>View Answer</summary>

**Custom Hooks** and **Higher-Order Components (HOCs)** both solve the same problem - code reuse - but in different ways. Hooks are the modern, preferred approach in React.

---

#### Quick Comparison

| Aspect | HOC | Custom Hooks |
|--------|-----|-------------|
| **What it is** | Function that wraps a component | Function that uses React hooks |
| **Returns** | New component | State/functions |
| **Syntax** | `const Enhanced = withData(Component)` | `const { data } = useData()` |
| **Component wrapper** | Yes - adds wrapper | No wrapper |
| **Props** | Injects props | Returns values directly |
| **Composability** | Complex (nesting) | Simple (call multiple hooks) |
| **TypeScript** | Complex | Simple |
| **Debugging** | Harder (wrapper hell) | Easier (no wrappers) |
| **React DevTools** | Shows wrappers | Clean component tree |
| **Performance** | Extra component layer | No extra components |
| **Modern React** | Legacy pattern | ✅ Recommended |

---

#### Same Logic - Different Approaches

**Example: Mouse Position Tracking**

**HOC Approach:**
```tsx
// HOC implementation
interface WithMouseProps {
  mousePosition: { x: number; y: number };
}

function withMouse<P extends object>(
  Component: React.ComponentType<P & WithMouseProps>
) {
  return function WithMouseComponent(props: P) {
    const [position, setPosition] = useState({ x: 0, y: 0 });

    useEffect(() => {
      const handleMouseMove = (e: MouseEvent) => {
        setPosition({ x: e.clientX, y: e.clientY });
      };
      window.addEventListener('mousemove', handleMouseMove);
      return () => window.removeEventListener('mousemove', handleMouseMove);
    }, []);

    return <Component {...props} mousePosition={position} />;
  };
}

// Usage with HOC
interface DisplayProps {
  title: string;
}

const Display: React.FC<DisplayProps & WithMouseProps> = ({ 
  title, 
  mousePosition 
}) => {
  return (
    <div>
      <h1>{title}</h1>
      <p>Mouse: {mousePosition.x}, {mousePosition.y}</p>
    </div>
  );
};

const DisplayWithMouse = withMouse(Display);

// Component tree in DevTools:
// <WithMouseComponent>
//   <Display />
// </WithMouseComponent>
```

**Custom Hook Approach:**
```tsx
// Custom hook implementation
function useMouse() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return position;
}

// Usage with hook - Much cleaner!
interface DisplayProps {
  title: string;
}

const Display: React.FC<DisplayProps> = ({ title }) => {
  const mousePosition = useMouse();

  return (
    <div>
      <h1>{title}</h1>
      <p>Mouse: {mousePosition.x}, {mousePosition.y}</p>
    </div>
  );
};

// Component tree in DevTools:
// <Display />
// (No wrappers!)
```

---

#### Key Differences

**1. Component Wrapper**

**HOC:**
```tsx
// Creates wrapper component
const EnhancedComponent = withData(MyComponent);

// Renders as:
// <WithDataComponent>
//   <MyComponent />
// </WithDataComponent>
```

**Hook:**
```tsx
// No wrapper - just uses the hook
function MyComponent() {
  const data = useData();
  return <div>{data}</div>;
}

// Renders as:
// <MyComponent />
```

---

**2. Composability**

**HOC - Nested/Nested:**
```tsx
// Multiple HOCs create deep nesting
const Enhanced = withAuth(
  withData(
    withLoading(
      withError(
        MyComponent
      )
    )
  )
);

// Component tree:
// <WithAuthComponent>
//   <WithDataComponent>
//     <WithLoadingComponent>
//       <WithErrorComponent>
//         <MyComponent />
//       </WithErrorComponent>
//     </WithLoadingComponent>
//   </WithDataComponent>
// </WithAuthComponent>

// Or using compose (still complex)
const enhance = compose(
  withAuth,
  withData,
  withLoading,
  withError
);
const Enhanced = enhance(MyComponent);
```

**Hooks - Simple and Flat:**
```tsx
// Multiple hooks are just function calls
function MyComponent() {
  const auth = useAuth();
  const data = useData();
  const loading = useLoading();
  const error = useError();

  // Use them
  return <div>...</div>;
}

// Component tree:
// <MyComponent />
// (Clean and flat!)
```

---

**3. Props vs Direct Access**

**HOC - Props Injection:**
```tsx
// HOC injects props
const MyComponent: React.FC<{ 
  injectedData: string;  // From HOC
  ownProp: string;       // Own prop
}> = ({ injectedData, ownProp }) => {
  return <div>{injectedData} {ownProp}</div>;
};

const Enhanced = withData(MyComponent);

// Props collision possible!
interface MyProps {
  data: string; // Conflicts with HOC's 'data' prop
}
```

**Hooks - Direct Access:**
```tsx
// Hook returns values directly
function MyComponent({ ownProp }: { ownProp: string }) {
  const data = useData();  // No prop collision
  return <div>{data} {ownProp}</div>;
}

// No prop name conflicts!
```

---

#### Real-World Comparison: Data Fetching

**HOC Implementation:**
```tsx
interface WithDataProps<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function withData<T, P extends object>(url: string) {
  return function (Component: React.ComponentType<P & WithDataProps<T>>) {
    return function WithDataComponent(props: P) {
      const [data, setData] = useState<T | null>(null);
      const [loading, setLoading] = useState(true);
      const [error, setError] = useState<Error | null>(null);

      useEffect(() => {
        fetch(url)
          .then((res) => res.json())
          .then(setData)
          .catch(setError)
          .finally(() => setLoading(false));
      }, []);

      return (
        <Component 
          {...props} 
          data={data} 
          loading={loading} 
          error={error}
        />
      );
    };
  };
}

// Usage - Complex types
interface User {
  id: number;
  name: string;
}

interface UserListProps {
  title: string;
}

const UserList: React.FC<UserListProps & WithDataProps<User[]>> = ({
  title,
  data,
  loading,
  error,
}) => {
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <div>
      <h1>{title}</h1>
      {data?.map((user) => <div key={user.id}>{user.name}</div>)}
    </div>
  );
};

const UserListWithData = withData<User[], UserListProps>('/api/users')(UserList);
```

**Custom Hook Implementation:**
```tsx
function useData<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  return { data, loading, error };
}

// Usage - Much simpler!
interface User {
  id: number;
  name: string;
}

interface UserListProps {
  title: string;
}

const UserList: React.FC<UserListProps> = ({ title }) => {
  const { data, loading, error } = useData<User[]>('/api/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h1>{title}</h1>
      {data?.map((user) => <div key={user.id}>{user.name}</div>)}
    </div>
  );
};

// No wrapping needed!
```

---

#### Multiple Enhancements Comparison

**HOC - Wrapper Hell:**
```tsx
// Need multiple enhancements
const MyComponent = () => <div>Content</div>;

const Enhanced = withAuth(
  withData(
    withLoading(
      withTheme(
        withRouter(
          MyComponent
        )
      )
    )
  )
);

// DevTools shows:
// <WithAuthComponent>
//   <WithDataComponent>
//     <WithLoadingComponent>
//       <WithThemeComponent>
//         <WithRouterComponent>
//           <MyComponent />

// Props can collide!
// TypeScript becomes nightmare
```

**Hooks - Clean and Simple:**
```tsx
// Multiple hooks - just call them
const MyComponent = () => {
  const auth = useAuth();
  const data = useData();
  const loading = useLoading();
  const theme = useTheme();
  const router = useRouter();

  return <div>Content</div>;
};

// DevTools shows:
// <MyComponent />

// No prop collisions
// TypeScript is simple
// Easy to read and maintain
```

---

#### Advantages and Disadvantages

**HOC Advantages:**
- ✅ Works with class components
- ✅ Can wrap third-party components
- ✅ Established pattern (legacy codebases)
- ✅ Can enhance components without modifying them

**HOC Disadvantages:**
- ❌ Wrapper components (extra layers)
- ❌ Props collision possible
- ❌ Complex TypeScript types
- ❌ Hard to debug (wrapper hell)
- ❌ Difficult to compose many HOCs
- ❌ Must be applied outside component
- ❌ Not compatible with hooks

**Custom Hooks Advantages:**
- ✅ No wrapper components
- ✅ No props collision
- ✅ Simple TypeScript types
- ✅ Easy to debug
- ✅ Easy to compose (call multiple hooks)
- ✅ Use anywhere in component
- ✅ Works with all React features
- ✅ Modern React pattern

**Custom Hooks Disadvantages:**
- ❌ Only works with function components
- ❌ Must follow hooks rules
- ❌ Can't wrap components without rendering

---

#### When to Use Each

**Use HOC When:**
1. Working with **class components**
2. Working with **legacy codebases**
3. Need to **wrap third-party components**
4. Need to **enhance without modifying** component

```tsx
// Wrapping a third-party component
import ThirdPartyComponent from 'some-library';

const Enhanced = withTracking(ThirdPartyComponent);
```

**Use Custom Hooks When:**
1. Building **modern React apps** ✅
2. Working with **function components** ✅
3. Need **clean component trees** ✅
4. Want **simple TypeScript** ✅
5. Need **better debugging** ✅

```tsx
// Modern approach
function MyComponent() {
  const data = useData();
  return <div>{data}</div>;
}
```

---

#### Migration Example

**Before (HOC):**
```tsx
// Old HOC pattern
interface WithAuthProps {
  user: User | null;
  isAuthenticated: boolean;
  login: () => void;
  logout: () => void;
}

function withAuth<P extends object>(
  Component: React.ComponentType<P & WithAuthProps>
) {
  return function WithAuthComponent(props: P) {
    const [user, setUser] = useState<User | null>(null);
    // ... auth logic
    return <Component {...props} user={user} /* ... */ />;
  };
}

const Dashboard: React.FC<DashboardProps & WithAuthProps> = ({
  user,
  isAuthenticated,
  // ... other props
}) => {
  // ...
};

const DashboardWithAuth = withAuth(Dashboard);
```

**After (Custom Hook):**
```tsx
// Modern hook pattern
function useAuth() {
  const [user, setUser] = useState<User | null>(null);
  // ... same auth logic
  return {
    user,
    isAuthenticated: !!user,
    login: () => { /* ... */ },
    logout: () => { /* ... */ },
  };
}

const Dashboard: React.FC<DashboardProps> = (props) => {
  const { user, isAuthenticated, login, logout } = useAuth();
  // ...
};

// No wrapping needed!
```

---

#### Best Practices

**For HOCs (Legacy Code):**
1. Use `displayName` for debugging
2. Forward refs with `React.forwardRef`
3. Hoist static methods
4. Don't create HOCs inside render
5. Pass unrelated props through

**For Custom Hooks (Modern):**
1. Start name with `use`
2. Return object or array
3. Follow hooks rules
4. Keep hooks focused and single-purpose
5. Document return values

---

#### Interview Tips

✅ **Key Points:**
- Both solve code reuse problem
- HOCs wrap components, hooks don't
- Hooks are modern, preferred approach
- HOCs create wrapper components (performance/debugging issues)
- Hooks are simpler, cleaner, more composable

✅ **When to Mention:**
- Discussing React evolution
- Code reuse patterns
- Modern vs legacy React
- Component composition
- Why hooks were introduced

✅ **Common Follow-ups:**
- "Why did React introduce hooks?"
- "When would you still use HOCs?"
- "How do you migrate from HOC to hooks?"
- "What are the performance implications?"
- "Can you use both together?"

✅ **Perfect Answer Structure:**
1. Both solve code reuse
2. HOCs are older pattern
3. Hooks are modern approach
4. Show example of both
5. Explain why hooks are better
6. Mention when HOCs still useful (legacy/class components)

</details>

---

### 105. What is the Compound Components pattern?

<details>
<summary>View Answer</summary>

**Compound Components** is a pattern where multiple components work together as a single unit, sharing implicit state without prop drilling. Think of it like HTML's `<select>` and `<option>` elements working together.

#### What are Compound Components?

This pattern involves:
- **Multiple related components** that work together
- **Shared implicit state** (no prop drilling)
- **Flexible composition** - arrange components freely
- **Parent manages state** - children consume it
- **Clear component relationships**

---

#### Simple Example: Tabs

```tsx
// Using compound components
function App() {
  return (
    <Tabs defaultValue="tab1">
      <TabList>
        <Tab value="tab1">Tab 1</Tab>
        <Tab value="tab2">Tab 2</Tab>
        <Tab value="tab3">Tab 3</Tab>
      </TabList>
      
      <TabPanel value="tab1">
        <h2>Content 1</h2>
      </TabPanel>
      <TabPanel value="tab2">
        <h2>Content 2</h2>
      </TabPanel>
      <TabPanel value="tab3">
        <h2>Content 3</h2>
      </TabPanel>
    </Tabs>
  );
}
```

Notice:
- `Tab` and `TabPanel` don't receive state as props
- State is shared implicitly through context
- You can arrange components flexibly
- Clear semantic structure

---

#### Implementation: Basic Counter

```tsx
import { createContext, useContext, useState } from 'react';

// 1. Create context for shared state
interface CounterContextValue {
  count: number;
  increment: () => void;
  decrement: () => void;
}

const CounterContext = createContext<CounterContextValue | null>(null);

// 2. Parent component - manages state
interface CounterProps {
  children: React.ReactNode;
  initialCount?: number;
}

function Counter({ children, initialCount = 0 }: CounterProps) {
  const [count, setCount] = useState(initialCount);

  const increment = () => setCount((c) => c + 1);
  const decrement = () => setCount((c) => c - 1);

  return (
    <CounterContext.Provider value={{ count, increment, decrement }}>
      <div className="counter">{children}</div>
    </CounterContext.Provider>
  );
}

// 3. Hook to access context
function useCounterContext() {
  const context = useContext(CounterContext);
  if (!context) {
    throw new Error('Counter compound components must be used within Counter');
  }
  return context;
}

// 4. Child components - consume state
Counter.Display = function CounterDisplay() {
  const { count } = useCounterContext();
  return <div className="counter-display">Count: {count}</div>;
};

Counter.Increment = function CounterIncrement() {
  const { increment } = useCounterContext();
  return <button onClick={increment}>+</button>;
};

Counter.Decrement = function CounterDecrement() {
  const { decrement } = useCounterContext();
  return <button onClick={decrement}>-</button>;
};

// Usage - Flexible arrangement!
function App() {
  return (
    <Counter initialCount={0}>
      <Counter.Display />
      <Counter.Increment />
      <Counter.Decrement />
    </Counter>
  );
}

// Or different arrangement
function App2() {
  return (
    <Counter initialCount={10}>
      <Counter.Increment />
      <Counter.Increment />
      <Counter.Display />
      <Counter.Decrement />
    </Counter>
  );
}
```

---

#### Real-World Example: Accordion

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// Context
interface AccordionContextValue {
  openItems: Set<string>;
  toggle: (id: string) => void;
  allowMultiple: boolean;
}

const AccordionContext = createContext<AccordionContextValue | null>(null);

// Parent component
interface AccordionProps {
  children: ReactNode;
  allowMultiple?: boolean;
  defaultOpen?: string[];
}

function Accordion({ 
  children, 
  allowMultiple = false,
  defaultOpen = []
}: AccordionProps) {
  const [openItems, setOpenItems] = useState<Set<string>>(
    new Set(defaultOpen)
  );

  const toggle = (id: string) => {
    setOpenItems((prev) => {
      const next = new Set(prev);
      
      if (next.has(id)) {
        next.delete(id);
      } else {
        if (!allowMultiple) {
          next.clear();
        }
        next.add(id);
      }
      
      return next;
    });
  };

  return (
    <AccordionContext.Provider value={{ openItems, toggle, allowMultiple }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

function useAccordion() {
  const context = useContext(AccordionContext);
  if (!context) {
    throw new Error('Accordion components must be within Accordion');
  }
  return context;
}

// Item component
interface AccordionItemProps {
  id: string;
  children: ReactNode;
}

Accordion.Item = function AccordionItem({ id, children }: AccordionItemProps) {
  return (
    <div className="accordion-item" data-id={id}>
      {children}
    </div>
  );
};

// Trigger component
interface AccordionTriggerProps {
  id: string;
  children: ReactNode;
}

Accordion.Trigger = function AccordionTrigger({ 
  id, 
  children 
}: AccordionTriggerProps) {
  const { openItems, toggle } = useAccordion();
  const isOpen = openItems.has(id);

  return (
    <button 
      className="accordion-trigger"
      onClick={() => toggle(id)}
      aria-expanded={isOpen}
    >
      {children}
      <span>{isOpen ? '▼' : '▶'}</span>
    </button>
  );
};

// Content component
interface AccordionContentProps {
  id: string;
  children: ReactNode;
}

Accordion.Content = function AccordionContent({ 
  id, 
  children 
}: AccordionContentProps) {
  const { openItems } = useAccordion();
  const isOpen = openItems.has(id);

  if (!isOpen) return null;

  return (
    <div className="accordion-content">
      {children}
    </div>
  );
};

// Usage
function App() {
  return (
    <Accordion allowMultiple={false} defaultOpen={['item1']}>
      <Accordion.Item id="item1">
        <Accordion.Trigger id="item1">
          What is React?
        </Accordion.Trigger>
        <Accordion.Content id="item1">
          React is a JavaScript library for building user interfaces.
        </Accordion.Content>
      </Accordion.Item>

      <Accordion.Item id="item2">
        <Accordion.Trigger id="item2">
          What are Compound Components?
        </Accordion.Trigger>
        <Accordion.Content id="item2">
          A pattern where components work together sharing implicit state.
        </Accordion.Content>
      </Accordion.Item>

      <Accordion.Item id="item3">
        <Accordion.Trigger id="item3">
          Why use this pattern?
        </Accordion.Trigger>
        <Accordion.Content id="item3">
          Flexible, composable, and avoids prop drilling.
        </Accordion.Content>
      </Accordion.Item>
    </Accordion>
  );
}
```

---

#### Advanced Pattern: Dropdown Menu

```tsx
import { createContext, useContext, useState, useRef, useEffect } from 'react';

// Context
interface DropdownContextValue {
  isOpen: boolean;
  toggle: () => void;
  close: () => void;
}

const DropdownContext = createContext<DropdownContextValue | null>(null);

// Parent
interface DropdownProps {
  children: React.ReactNode;
}

function Dropdown({ children }: DropdownProps) {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);

  const toggle = () => setIsOpen((prev) => !prev);
  const close = () => setIsOpen(false);

  // Close on outside click
  useEffect(() => {
    function handleClickOutside(event: MouseEvent) {
      if (
        dropdownRef.current &&
        !dropdownRef.current.contains(event.target as Node)
      ) {
        close();
      }
    }

    if (isOpen) {
      document.addEventListener('mousedown', handleClickOutside);
      return () => document.removeEventListener('mousedown', handleClickOutside);
    }
  }, [isOpen]);

  return (
    <DropdownContext.Provider value={{ isOpen, toggle, close }}>
      <div ref={dropdownRef} className="dropdown">
        {children}
      </div>
    </DropdownContext.Provider>
  );
}

function useDropdown() {
  const context = useContext(DropdownContext);
  if (!context) {
    throw new Error('Dropdown components must be within Dropdown');
  }
  return context;
}

// Trigger
Dropdown.Trigger = function DropdownTrigger({ 
  children 
}: { 
  children: React.ReactNode 
}) {
  const { toggle } = useDropdown();
  return (
    <button onClick={toggle} className="dropdown-trigger">
      {children}
    </button>
  );
};

// Menu
Dropdown.Menu = function DropdownMenu({ 
  children 
}: { 
  children: React.ReactNode 
}) {
  const { isOpen } = useDropdown();
  if (!isOpen) return null;

  return (
    <div className="dropdown-menu">
      {children}
    </div>
  );
};

// Item
interface DropdownItemProps {
  onClick?: () => void;
  children: React.ReactNode;
}

Dropdown.Item = function DropdownItem({ 
  onClick, 
  children 
}: DropdownItemProps) {
  const { close } = useDropdown();

  const handleClick = () => {
    onClick?.();
    close();
  };

  return (
    <button onClick={handleClick} className="dropdown-item">
      {children}
    </button>
  );
};

// Divider
Dropdown.Divider = function DropdownDivider() {
  return <hr className="dropdown-divider" />;
};

// Usage
function UserMenu() {
  return (
    <Dropdown>
      <Dropdown.Trigger>
        User Menu ▼
      </Dropdown.Trigger>
      
      <Dropdown.Menu>
        <Dropdown.Item onClick={() => console.log('Profile')}>
          👤 Profile
        </Dropdown.Item>
        <Dropdown.Item onClick={() => console.log('Settings')}>
          ⚙️ Settings
        </Dropdown.Item>
        <Dropdown.Divider />
        <Dropdown.Item onClick={() => console.log('Logout')}>
          🚪 Logout
        </Dropdown.Item>
      </Dropdown.Menu>
    </Dropdown>
  );
}
```

---

#### Benefits of Compound Components

**1. Flexibility:**
```tsx
// Can arrange children however you want
<Tabs>
  <TabPanel value="a">Content A</TabPanel>  {/* Panel first */}
  <TabList>
    <Tab value="a">Tab A</Tab>
  </TabList>
</Tabs>

// Or traditional order
<Tabs>
  <TabList>
    <Tab value="a">Tab A</Tab>
  </TabList>
  <TabPanel value="a">Content A</TabPanel>
</Tabs>
```

**2. No Prop Drilling:**
```tsx
// Without compound components
<Accordion>
  <AccordionItem 
    isOpen={openItems.includes('1')}
    onToggle={() => toggle('1')}
  >
    <AccordionHeader 
      isOpen={openItems.includes('1')}
      onToggle={() => toggle('1')}
    />
    <AccordionContent isOpen={openItems.includes('1')} />
  </AccordionItem>
</Accordion>

// With compound components - clean!
<Accordion>
  <Accordion.Item id="1">
    <Accordion.Trigger id="1" />
    <Accordion.Content id="1" />
  </Accordion.Item>
</Accordion>
```

**3. Semantic and Clear:**
```tsx
// Clear what each part does
<Modal>
  <Modal.Header>Title</Modal.Header>
  <Modal.Body>Content</Modal.Body>
  <Modal.Footer>Actions</Modal.Footer>
</Modal>
```

**4. Composable:**
```tsx
// Easy to add/remove parts
<Card>
  {showImage && <Card.Image src="..." />}
  <Card.Title>Title</Card.Title>
  <Card.Description>Text</Card.Description>
  {showActions && <Card.Actions><button>Click</button></Card.Actions>}
</Card>
```

---

#### Common Use Cases

**1. Navigation Menus**
```tsx
<Menu>
  <Menu.Item icon="🏠">Home</Menu.Item>
  <Menu.Item icon="📝">Posts</Menu.Item>
  <Menu.Divider />
  <Menu.Item icon="⚙️">Settings</Menu.Item>
</Menu>
```

**2. Forms**
```tsx
<Form onSubmit={handleSubmit}>
  <Form.Field>
    <Form.Label>Email</Form.Label>
    <Form.Input type="email" />
    <Form.Error>Invalid email</Form.Error>
  </Form.Field>
  <Form.Actions>
    <Form.Submit>Submit</Form.Submit>
  </Form.Actions>
</Form>
```

**3. Data Display**
```tsx
<Table>
  <Table.Header>
    <Table.Row>
      <Table.HeaderCell>Name</Table.HeaderCell>
      <Table.HeaderCell>Email</Table.HeaderCell>
    </Table.Row>
  </Table.Header>
  <Table.Body>
    <Table.Row>
      <Table.Cell>John</Table.Cell>
      <Table.Cell>john@example.com</Table.Cell>
    </Table.Row>
  </Table.Body>
</Table>
```

---

#### Best Practices

**1. Use Context for State Sharing:**
```tsx
// Always use Context to share state
const ParentContext = createContext(null);

function Parent({ children }) {
  const [state, setState] = useState();
  return (
    <ParentContext.Provider value={{ state, setState }}>
      {children}
    </ParentContext.Provider>
  );
}
```

**2. Create Custom Hook:**
```tsx
// Provide a hook to access context
function useParent() {
  const context = useContext(ParentContext);
  if (!context) {
    throw new Error('Must be used within Parent');
  }
  return context;
}
```

**3. Namespace Sub-components:**
```tsx
// Attach sub-components to parent
Parent.Child = function Child() { /* ... */ };
Parent.Header = function Header() { /* ... */ };
Parent.Footer = function Footer() { /* ... */ };
```

**4. Validate Component Usage:**
```tsx
// Throw helpful errors
function useAccordion() {
  const context = useContext(AccordionContext);
  if (!context) {
    throw new Error(
      'Accordion.Item, Accordion.Trigger, and Accordion.Content ' +
      'must be used within an Accordion component'
    );
  }
  return context;
}
```

**5. TypeScript Support:**
```tsx
// Provide proper types
interface TabsProps {
  defaultValue?: string;
  children: React.ReactNode;
}

interface TabProps {
  value: string;
  children: React.ReactNode;
  disabled?: boolean;
}
```

---

#### Comparison with Other Patterns

| Pattern | Structure | State Sharing | Flexibility | Complexity |
|---------|-----------|---------------|-------------|------------|
| **Compound Components** | Parent + multiple children | Context | High | Medium |
| **Render Props** | Single component | Function prop | Medium | Medium |
| **Props** | Single component | Props | Low | Low |
| **HOC** | Wrapper | Props injection | Low | High |

---

#### Interview Tips

✅ **Key Points:**
- Multiple components work together as a unit
- Share state implicitly via Context
- Flexible composition - arrange components freely
- Avoids prop drilling
- Similar to HTML elements like `<select>` + `<option>`

✅ **When to Mention:**
- Discussing component design patterns
- Building reusable component libraries
- Explaining advanced React patterns
- UI component architecture
- API design for components

✅ **Common Follow-ups:**
- "How do components share state?"
- "Give me a real-world example"
- "What are the benefits?"
- "When should you use this pattern?"
- "How is this different from props?"

✅ **Perfect Answer Structure:**
1. Define: Components working together
2. How: Use Context for state sharing
3. Example: Show simple implementation
4. Benefits: Flexibility, no prop drilling
5. Use cases: Tabs, Accordion, Dropdown, etc.
6. Compare: vs regular props/render props

✅ **Show You Know:**
- Mention Context API usage
- Show namespace pattern (Parent.Child)
- Discuss error handling
- Compare to HTML elements
- Mention popular libraries (Radix UI, Reach UI)

</details>

---

### 106. What is the Container/Presentational pattern?

<details>
<summary>View Answer</summary>

**Container/Presentational** (also called Smart/Dumb or Stateful/Stateless) is a pattern that separates components into two categories: containers that handle logic and data, and presentational components that handle UI rendering.

#### What is Container/Presentational Pattern?

This pattern divides components into:

**Presentational Components (Dumb):**
- Focus on **how things look**
- Receive data via **props**
- **No state** (or only UI state)
- **No business logic**
- Highly **reusable**
- Example: Button, Card, List

**Container Components (Smart):**
- Focus on **how things work**
- Manage **state and side effects**
- Handle **data fetching**
- Contain **business logic**
- Pass data to presentational components
- Example: UserListContainer, DashboardContainer

---

#### Basic Example

**Presentational Component (Dumb):**
```tsx
// UserList.tsx - Presentational
interface User {
  id: number;
  name: string;
  email: string;
}

interface UserListProps {
  users: User[];
  onUserClick: (user: User) => void;
}

// Only focuses on rendering
const UserList: React.FC<UserListProps> = ({ users, onUserClick }) => {
  return (
    <div className="user-list">
      {users.map((user) => (
        <div 
          key={user.id} 
          className="user-card"
          onClick={() => onUserClick(user)}
        >
          <h3>{user.name}</h3>
          <p>{user.email}</p>
        </div>
      ))}
    </div>
  );
};

export default UserList;
```

**Container Component (Smart):**
```tsx
// UserListContainer.tsx - Container
import { useState, useEffect } from 'react';
import UserList from './UserList';

interface User {
  id: number;
  name: string;
  email: string;
}

// Handles logic, data fetching, state
const UserListContainer: React.FC = () => {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // Data fetching
  useEffect(() => {
    fetch('/api/users')
      .then((res) => res.json())
      .then((data) => {
        setUsers(data);
        setLoading(false);
      })
      .catch((err) => {
        setError(err.message);
        setLoading(false);
      });
  }, []);

  // Business logic
  const handleUserClick = (user: User) => {
    console.log('User clicked:', user);
    // Navigate, show modal, etc.
  };

  // Handle loading and error states
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  // Pass data to presentational component
  return <UserList users={users} onUserClick={handleUserClick} />;
};

export default UserListContainer;
```

---

#### Real-World Example: Product Dashboard

**Presentational Components:**

```tsx
// ProductCard.tsx - Pure presentation
interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
}

interface ProductCardProps {
  product: Product;
  onAddToCart: (product: Product) => void;
  onViewDetails: (product: Product) => void;
}

const ProductCard: React.FC<ProductCardProps> = ({ 
  product, 
  onAddToCart,
  onViewDetails 
}) => {
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p className="price">${product.price}</p>
      <div className="actions">
        <button onClick={() => onAddToCart(product)}>
          Add to Cart
        </button>
        <button onClick={() => onViewDetails(product)}>
          Details
        </button>
      </div>
    </div>
  );
};

// ProductGrid.tsx - Pure presentation
interface ProductGridProps {
  products: Product[];
  onAddToCart: (product: Product) => void;
  onViewDetails: (product: Product) => void;
}

const ProductGrid: React.FC<ProductGridProps> = ({ 
  products,
  onAddToCart,
  onViewDetails 
}) => {
  return (
    <div className="product-grid">
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={onAddToCart}
          onViewDetails={onViewDetails}
        />
      ))}
    </div>
  );
};

// FilterBar.tsx - Pure presentation
interface FilterBarProps {
  categories: string[];
  selectedCategory: string;
  onCategoryChange: (category: string) => void;
  sortBy: string;
  onSortChange: (sort: string) => void;
}

const FilterBar: React.FC<FilterBarProps> = ({
  categories,
  selectedCategory,
  onCategoryChange,
  sortBy,
  onSortChange,
}) => {
  return (
    <div className="filter-bar">
      <select 
        value={selectedCategory} 
        onChange={(e) => onCategoryChange(e.target.value)}
      >
        <option value="all">All Categories</option>
        {categories.map((cat) => (
          <option key={cat} value={cat}>{cat}</option>
        ))}
      </select>

      <select 
        value={sortBy} 
        onChange={(e) => onSortChange(e.target.value)}
      >
        <option value="name">Name</option>
        <option value="price-asc">Price: Low to High</option>
        <option value="price-desc">Price: High to Low</option>
      </select>
    </div>
  );
};
```

**Container Component:**

```tsx
// ProductDashboardContainer.tsx - Smart container
import { useState, useEffect, useMemo } from 'react';
import ProductGrid from './ProductGrid';
import FilterBar from './FilterBar';

interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
  category: string;
}

const ProductDashboardContainer: React.FC = () => {
  // State management
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(true);
  const [selectedCategory, setSelectedCategory] = useState('all');
  const [sortBy, setSortBy] = useState('name');

  // Data fetching
  useEffect(() => {
    fetch('/api/products')
      .then((res) => res.json())
      .then((data) => {
        setProducts(data);
        setLoading(false);
      })
      .catch((err) => {
        console.error(err);
        setLoading(false);
      });
  }, []);

  // Business logic - filtering and sorting
  const filteredAndSortedProducts = useMemo(() => {
    let result = [...products];

    // Filter by category
    if (selectedCategory !== 'all') {
      result = result.filter((p) => p.category === selectedCategory);
    }

    // Sort
    result.sort((a, b) => {
      switch (sortBy) {
        case 'name':
          return a.name.localeCompare(b.name);
        case 'price-asc':
          return a.price - b.price;
        case 'price-desc':
          return b.price - a.price;
        default:
          return 0;
      }
    });

    return result;
  }, [products, selectedCategory, sortBy]);

  // Extract unique categories
  const categories = useMemo(() => {
    return Array.from(new Set(products.map((p) => p.category)));
  }, [products]);

  // Business logic - handlers
  const handleAddToCart = (product: Product) => {
    console.log('Adding to cart:', product);
    // Call API, update cart state, show notification, etc.
  };

  const handleViewDetails = (product: Product) => {
    console.log('View details:', product);
    // Navigate to product page
  };

  if (loading) return <div>Loading products...</div>;

  return (
    <div className="product-dashboard">
      <h1>Product Catalog</h1>
      
      <FilterBar
        categories={categories}
        selectedCategory={selectedCategory}
        onCategoryChange={setSelectedCategory}
        sortBy={sortBy}
        onSortChange={setSortBy}
      />

      <ProductGrid
        products={filteredAndSortedProducts}
        onAddToCart={handleAddToCart}
        onViewDetails={handleViewDetails}
      />
    </div>
  );
};

export default ProductDashboardContainer;
```

---

#### Benefits of This Pattern

**1. Reusability:**
```tsx
// Presentational component can be reused anywhere
<UserList 
  users={adminUsers} 
  onUserClick={handleAdminClick}
/>

<UserList 
  users={regularUsers} 
  onUserClick={handleUserClick}
/>

<UserList 
  users={searchResults} 
  onUserClick={handleSearchClick}
/>
```

**2. Testability:**
```tsx
// Presentational - Easy to test (just props)
import { render, fireEvent } from '@testing-library/react';

test('UserList renders users and handles click', () => {
  const mockUsers = [
    { id: 1, name: 'John', email: 'john@test.com' },
  ];
  const mockClick = jest.fn();

  const { getByText } = render(
    <UserList users={mockUsers} onUserClick={mockClick} />
  );

  fireEvent.click(getByText('John'));
  expect(mockClick).toHaveBeenCalledWith(mockUsers[0]);
});

// Container - Test logic separately
test('UserListContainer fetches and displays users', async () => {
  // Mock fetch
  global.fetch = jest.fn(() =>
    Promise.resolve({
      json: () => Promise.resolve([{ id: 1, name: 'John' }]),
    })
  );

  const { findByText } = render(<UserListContainer />);
  expect(await findByText('John')).toBeInTheDocument();
});
```

**3. Separation of Concerns:**
```tsx
// Designer can work on presentational components
// Developer can work on container logic
// No conflicts!
```

**4. Easy Styling:**
```tsx
// Presentational components are style-focused
const Button: React.FC<ButtonProps> = ({ children, onClick, variant }) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
};
```

---

#### Comparison Table

| Aspect | Presentational | Container |
|--------|----------------|----------|
| **Purpose** | How things look | How things work |
| **State** | No (or only UI state) | Yes |
| **Data** | From props | From hooks/API |
| **Behavior** | Callbacks from props | Business logic |
| **Dependencies** | None (pure) | APIs, services, state |
| **Reusable** | Highly reusable | Less reusable |
| **Testing** | Easy (props in/out) | More complex |
| **Examples** | Button, Card, List | DashboardContainer |

---

#### Modern React Approach

With hooks, the distinction is less strict, but the **concept still applies**:

**Before Hooks (Strict Separation):**
```tsx
// Container - class component
class UserListContainer extends React.Component {
  state = { users: [] };
  
  componentDidMount() {
    fetch('/api/users').then(/* ... */);
  }
  
  render() {
    return <UserList users={this.state.users} />;
  }
}

// Presentational - function component
const UserList = ({ users }) => (
  <div>{users.map(/* ... */)}</div>
);
```

**With Hooks (Flexible):**
```tsx
// Can use custom hooks to extract logic
function useUsers() {
  const [users, setUsers] = useState([]);
  
  useEffect(() => {
    fetch('/api/users')
      .then((res) => res.json())
      .then(setUsers);
  }, []);
  
  return users;
}

// Component can use the hook
const UserList: React.FC = () => {
  const users = useUsers();
  
  return (
    <div>
      {users.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
};

// Or keep strict separation
const UserListContainer: React.FC = () => {
  const users = useUsers();
  return <UserListPresentation users={users} />;
};
```

---

#### When to Use This Pattern

**✅ Good Use Cases:**

1. **Building component libraries**
   - Presentational components for UI library
   - Containers for app-specific logic

2. **Large applications**
   - Clear separation helps team collaboration
   - Designers work on presentational
   - Developers work on containers

3. **High reusability needed**
   - Same UI with different data sources

4. **Complex business logic**
   - Separate logic from UI

**❌ When Not Necessary:**

1. **Simple components** (overkill)
2. **Small applications** (unnecessary complexity)
3. **Components used only once** (no reusability benefit)
4. **Modern apps with hooks** (can use custom hooks instead)

---

#### Evolution: Custom Hooks Replace Containers

**Old Pattern (Container):**
```tsx
const UserListContainer = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUsers().then(setUsers).finally(() => setLoading(false));
  }, []);
  
  return <UserList users={users} loading={loading} />;
};
```

**Modern Pattern (Custom Hook):**
```tsx
// Extract logic to custom hook
function useUsers() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetchUsers().then(setUsers).finally(() => setLoading(false));
  }, []);
  
  return { users, loading };
}

// Component uses the hook
const UserList = () => {
  const { users, loading } = useUsers();
  
  if (loading) return <div>Loading...</div>;
  
  return (
    <div>
      {users.map((user) => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
};
```

---

#### Best Practices

**1. Keep Presentational Components Pure:**
```tsx
// ✅ Good - pure, predictable
const Button = ({ label, onClick }) => (
  <button onClick={onClick}>{label}</button>
);

// ❌ Bad - has side effects
const Button = ({ label }) => {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    fetch('/api/track').then(/* ... */);
  }, []);
  
  return <button>{label}</button>;
};
```

**2. Use PropTypes or TypeScript:**
```tsx
interface UserListProps {
  users: User[];
  onUserClick: (user: User) => void;
  loading?: boolean;
}

const UserList: React.FC<UserListProps> = ({ /* ... */ }) => {
  // ...
};
```

**3. Name Components Clearly:**
```tsx
// Container
UserListContainer, ProductDashboardContainer

// Presentational  
UserList, ProductCard, Button

// Or use suffixes
UserListView, UserListPresentation
```

**4. Don't Over-Separate:**
```tsx
// ❌ Overkill for simple component
const ButtonContainer = () => {
  const handleClick = () => console.log('clicked');
  return <Button onClick={handleClick} />;
};

// ✅ Just make it one component
const Button = () => {
  const handleClick = () => console.log('clicked');
  return <button onClick={handleClick}>Click</button>;
};
```

---

#### Interview Tips

✅ **Key Points:**
- Separates concerns: presentation vs logic
- Presentational = how it looks (props, no state, reusable)
- Container = how it works (state, data fetching, logic)
- Benefits: reusability, testability, separation of concerns
- Modern approach: custom hooks can replace containers

✅ **When to Mention:**
- Discussing component architecture
- Code organization strategies
- Reusable component design
- Testing strategies
- Team collaboration patterns

✅ **Common Follow-ups:**
- "How has this pattern changed with hooks?"
- "When should you use it?"
- "What are the alternatives?"
- "How do you test each type?"
- "Show me a real example"

✅ **Perfect Answer Structure:**
1. Define both types clearly
2. Show simple example of each
3. Explain benefits (reusability, testing)
4. Mention modern approach with hooks
5. When to use vs when not to use

</details>

---

### 107. What is the Provider pattern?

<details>
<summary>View Answer</summary>

**Provider Pattern** is a design pattern that uses React Context API to make data available to multiple components without prop drilling. A provider component wraps the component tree and provides shared state/functions to all descendants.

#### What is the Provider Pattern?

The pattern consists of:
- **Context** - creates a shared data container
- **Provider** - component that supplies the data
- **Consumer/Hook** - components that access the data
- **No prop drilling** - data available anywhere in tree

```tsx
<Provider value={data}>
  <App>
    <Child>
      <GrandChild />  {/* Can access data */}
    </Child>
  </App>
</Provider>
```

---

#### Basic Example

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// 1. Create Context
interface ThemeContextValue {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

// 2. Create Provider Component
interface ThemeProviderProps {
  children: ReactNode;
}

export const ThemeProvider: React.FC<ThemeProviderProps> = ({ children }) => {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

// 3. Create Custom Hook for Easy Access
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

// 4. Usage in App
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Content />
      <Footer />
    </ThemeProvider>
  );
}

// 5. Consuming Components
function Header() {
  const { theme, toggleTheme } = useTheme();

  return (
    <header className={theme}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}

function Content() {
  const { theme } = useTheme();

  return (
    <main className={theme}>
      <p>Current theme: {theme}</p>
    </main>
  );
}
```

---

#### Real-World Example: Authentication Provider

```tsx
import { createContext, useContext, useState, useEffect, ReactNode } from 'react';

// Types
interface User {
  id: string;
  name: string;
  email: string;
  role: string;
}

interface AuthContextValue {
  user: User | null;
  loading: boolean;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  signup: (name: string, email: string, password: string) => Promise<void>;
}

// Create Context
const AuthContext = createContext<AuthContextValue | undefined>(undefined);

// Provider Component
interface AuthProviderProps {
  children: ReactNode;
}

export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  // Check if user is logged in on mount
  useEffect(() => {
    const checkAuth = async () => {
      try {
        const token = localStorage.getItem('token');
        if (token) {
          const response = await fetch('/api/auth/me', {
            headers: { Authorization: `Bearer ${token}` },
          });
          if (response.ok) {
            const userData = await response.json();
            setUser(userData);
          }
        }
      } catch (error) {
        console.error('Auth check failed:', error);
      } finally {
        setLoading(false);
      }
    };

    checkAuth();
  }, []);

  // Login function
  const login = async (email: string, password: string) => {
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });

      if (!response.ok) throw new Error('Login failed');

      const { user: userData, token } = await response.json();
      localStorage.setItem('token', token);
      setUser(userData);
    } catch (error) {
      console.error('Login error:', error);
      throw error;
    }
  };

  // Logout function
  const logout = async () => {
    try {
      await fetch('/api/auth/logout', { method: 'POST' });
      localStorage.removeItem('token');
      setUser(null);
    } catch (error) {
      console.error('Logout error:', error);
    }
  };

  // Signup function
  const signup = async (name: string, email: string, password: string) => {
    try {
      const response = await fetch('/api/auth/signup', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ name, email, password }),
      });

      if (!response.ok) throw new Error('Signup failed');

      const { user: userData, token } = await response.json();
      localStorage.setItem('token', token);
      setUser(userData);
    } catch (error) {
      console.error('Signup error:', error);
      throw error;
    }
  };

  const value: AuthContextValue = {
    user,
    loading,
    isAuthenticated: !!user,
    login,
    logout,
    signup,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};

// Custom Hook
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};

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

// Protected Route Component
function ProtectedRoute({ children }: { children: ReactNode }) {
  const { isAuthenticated, loading } = useAuth();

  if (loading) return <div>Loading...</div>;
  if (!isAuthenticated) return <Navigate to="/login" />;

  return <>{children}</>;
}

// Login Component
function LoginPage() {
  const { login } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await login(email, password);
      // Redirect to dashboard
    } catch (error) {
      alert('Login failed');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="email" 
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
      />
      <input 
        type="password" 
        value={password} 
        onChange={(e) => setPassword(e.target.value)} 
      />
      <button type="submit">Login</button>
    </form>
  );
}

// Dashboard Component
function Dashboard() {
  const { user, logout } = useAuth();

  return (
    <div>
      <h1>Welcome, {user?.name}!</h1>
      <p>Email: {user?.email}</p>
      <p>Role: {user?.role}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

---

#### Multiple Providers Pattern

```tsx
// Combine multiple providers
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <LanguageProvider>
          <NotificationProvider>
            <Router>
              <Routes>
                {/* Your app routes */}
              </Routes>
            </Router>
          </NotificationProvider>
        </LanguageProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

// Better: Create a composed provider
interface AppProvidersProps {
  children: ReactNode;
}

const AppProviders: React.FC<AppProvidersProps> = ({ children }) => {
  return (
    <AuthProvider>
      <ThemeProvider>
        <LanguageProvider>
          <NotificationProvider>
            {children}
          </NotificationProvider>
        </LanguageProvider>
      </ThemeProvider>
    </AuthProvider>
  );
};

// Usage
function App() {
  return (
    <AppProviders>
      <Router>
        <Routes>{/* routes */}</Routes>
      </Router>
    </AppProviders>
  );
}
```

---

#### Advanced Example: Shopping Cart Provider

```tsx
import { createContext, useContext, useReducer, ReactNode } from 'react';

interface Product {
  id: string;
  name: string;
  price: number;
}

interface CartItem extends Product {
  quantity: number;
}

interface CartState {
  items: CartItem[];
  total: number;
}

type CartAction =
  | { type: 'ADD_ITEM'; payload: Product }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'CLEAR_CART' };

interface CartContextValue extends CartState {
  addItem: (product: Product) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
}

const CartContext = createContext<CartContextValue | undefined>(undefined);

// Reducer
function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(
        (item) => item.id === action.payload.id
      );

      let newItems: CartItem[];
      if (existingItem) {
        newItems = state.items.map((item) =>
          item.id === action.payload.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      } else {
        newItems = [...state.items, { ...action.payload, quantity: 1 }];
      }

      const total = newItems.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0
      );

      return { items: newItems, total };
    }

    case 'REMOVE_ITEM': {
      const newItems = state.items.filter((item) => item.id !== action.payload);
      const total = newItems.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0
      );
      return { items: newItems, total };
    }

    case 'UPDATE_QUANTITY': {
      const newItems = state.items.map((item) =>
        item.id === action.payload.id
          ? { ...item, quantity: action.payload.quantity }
          : item
      );
      const total = newItems.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0
      );
      return { items: newItems, total };
    }

    case 'CLEAR_CART':
      return { items: [], total: 0 };

    default:
      return state;
  }
}

// Provider
export const CartProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [state, dispatch] = useReducer(cartReducer, { items: [], total: 0 });

  const addItem = (product: Product) => {
    dispatch({ type: 'ADD_ITEM', payload: product });
  };

  const removeItem = (id: string) => {
    dispatch({ type: 'REMOVE_ITEM', payload: id });
  };

  const updateQuantity = (id: string, quantity: number) => {
    dispatch({ type: 'UPDATE_QUANTITY', payload: { id, quantity } });
  };

  const clearCart = () => {
    dispatch({ type: 'CLEAR_CART' });
  };

  const value: CartContextValue = {
    ...state,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
  };

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
};

// Hook
export const useCart = () => {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within CartProvider');
  }
  return context;
};

// Usage
function ProductList() {
  const { addItem } = useCart();
  const products = [/* ... */];

  return (
    <div>
      {products.map((product) => (
        <div key={product.id}>
          <h3>{product.name}</h3>
          <p>${product.price}</p>
          <button onClick={() => addItem(product)}>Add to Cart</button>
        </div>
      ))}
    </div>
  );
}

function CartSummary() {
  const { items, total, removeItem, clearCart } = useCart();

  return (
    <div>
      <h2>Cart ({items.length} items)</h2>
      {items.map((item) => (
        <div key={item.id}>
          <span>{item.name} x {item.quantity}</span>
          <span>${item.price * item.quantity}</span>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
      <div>
        <strong>Total: ${total.toFixed(2)}</strong>
      </div>
      <button onClick={clearCart}>Clear Cart</button>
    </div>
  );
}
```

---

#### Provider Pattern Benefits

**1. No Prop Drilling:**
```tsx
// Without Provider - prop drilling
<App>
  <Header user={user} />
  <Content user={user}>
    <Sidebar user={user}>
      <UserMenu user={user} />  {/* 4 levels deep! */}
    </Sidebar>
  </Content>
</App>

// With Provider - no drilling
<AuthProvider>
  <App>
    <Header />
    <Content>
      <Sidebar>
        <UserMenu />  {/* Access via useAuth() */}
      </Sidebar>
    </Content>
  </App>
</AuthProvider>
```

**2. Global State Management:**
```tsx
// Any component can access and modify state
function AnyComponent() {
  const { theme, toggleTheme } = useTheme();
  const { user, logout } = useAuth();
  const { items, addItem } = useCart();
  
  // Use shared state
}
```

**3. Separation of Concerns:**
```tsx
// State logic in provider
// Components focus on UI
```

**4. Easy Testing:**
```tsx
// Wrap component with test provider
import { render } from '@testing-library/react';

test('component uses theme', () => {
  const { getByText } = render(
    <ThemeProvider>
      <MyComponent />
    </ThemeProvider>
  );
  // ...
});
```

---

#### Best Practices

**1. Always Create Custom Hook:**
```tsx
// ✅ Good - custom hook with validation
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
};

// ❌ Bad - direct useContext usage
function MyComponent() {
  const theme = useContext(ThemeContext); // No validation!
}
```

**2. Define TypeScript Types:**
```tsx
interface AuthContextValue {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextValue | undefined>(undefined);
```

**3. Initialize with undefined, Not null:**
```tsx
// ✅ Good
const AuthContext = createContext<AuthContextValue | undefined>(undefined);

// ❌ Bad - null can be valid state
const AuthContext = createContext<AuthContextValue | null>(null);
```

**4. Split Large Providers:**
```tsx
// ❌ Bad - one giant provider
<AppProvider>  {/* user, theme, cart, settings, etc. */}

// ✅ Good - multiple focused providers
<AuthProvider>
  <ThemeProvider>
    <CartProvider>
      {/* ... */}
    </CartProvider>
  </ThemeProvider>
</AuthProvider>
```

**5. Memoize Context Value:**
```tsx
// Prevent unnecessary re-renders
export const ThemeProvider: React.FC<Props> = ({ children }) => {
  const [theme, setTheme] = useState('light');

  const value = useMemo(
    () => ({
      theme,
      toggleTheme: () => setTheme((prev) => prev === 'light' ? 'dark' : 'light'),
    }),
    [theme]
  );

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
};
```

---

#### Common Use Cases

1. **Authentication** - user data, login/logout
2. **Theming** - dark/light mode, colors
3. **Localization** - language, translations
4. **Shopping Cart** - items, total, checkout
5. **Notifications** - toast messages, alerts
6. **Modal Management** - open/close modals
7. **Form State** - shared form data
8. **WebSocket** - real-time connections

---

#### Provider vs Other State Management

| Solution | Best For | Complexity |
|----------|----------|------------|
| **Context Provider** | App-wide state, simple-medium apps | Low |
| **Redux** | Large apps, complex state, time-travel debugging | High |
| **Zustand** | Simple global state, modern alternative | Low |
| **Recoil** | Derived state, atoms pattern | Medium |
| **MobX** | Reactive state, OOP style | Medium |

---

#### Interview Tips

✅ **Key Points:**
- Provider pattern uses React Context API
- Avoids prop drilling
- Provider supplies data, consumers access it
- Use custom hook for type-safe access
- Good for global/shared state

✅ **When to Mention:**
- Discussing state management
- Avoiding prop drilling
- Global state solutions
- Context API usage
- Component communication

✅ **Common Follow-ups:**
- "How does it work internally?"
- "What are the performance implications?"
- "When to use Provider vs Redux?"
- "How do you test components with providers?"
- "What are the best practices?"

✅ **Perfect Answer Structure:**
1. Define: Provider supplies data to component tree
2. How: Create context, provider, custom hook
3. Show example (theme or auth)
4. Benefits: no prop drilling, global state
5. When to use: shared state, cross-cutting concerns
6. Mention alternatives: Redux, Zustand

</details>

---

### 108. What are Controlled vs Uncontrolled patterns?

<details>
<summary>View Answer</summary>

**Controlled** and **Uncontrolled** components are two different approaches to managing form inputs and component state in React.

#### Quick Comparison

| Aspect | Controlled | Uncontrolled |
|--------|------------|-------------|
| **State** | React state | DOM state |
| **Value** | `value={state}` | No value prop |
| **Updates** | `onChange` handler | Direct DOM access |
| **Source of truth** | React | DOM |
| **Access value** | From state | Using ref |
| **Validation** | Real-time | On submit |
| **React way** | ✅ Recommended | Legacy/special cases |

---

#### Controlled Components

**Definition:** React state is the **single source of truth**. Input value is controlled by React state.

**Basic Example:**
```tsx
import { useState } from 'react';

function ControlledInput() {
  const [value, setValue] = useState('');

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };

  return (
    <div>
      <input 
        type="text" 
        value={value}  {/* Controlled by state */}
        onChange={handleChange}  {/* Update state on change */}
      />
      <p>Current value: {value}</p>
    </div>
  );
}
```

**How it works:**
1. User types in input
2. `onChange` handler fires
3. State updates with new value
4. Component re-renders
5. Input displays new value from state

---

#### Uncontrolled Components

**Definition:** DOM is the **source of truth**. Access value using refs.

**Basic Example:**
```tsx
import { useRef } from 'react';

function UncontrolledInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleSubmit = () => {
    // Access value from DOM
    const value = inputRef.current?.value;
    console.log('Submitted:', value);
  };

  return (
    <div>
      <input 
        type="text" 
        ref={inputRef}  {/* Access DOM directly */}
        defaultValue=""  {/* Initial value only */}
      />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

**How it works:**
1. User types in input
2. DOM updates directly (no React)
3. Access value via ref when needed
4. No re-renders on typing

---

#### Side-by-Side Comparison

**Controlled Form:**
```tsx
interface FormData {
  username: string;
  email: string;
  password: string;
}

function ControlledForm() {
  const [formData, setFormData] = useState<FormData>({
    username: '',
    email: '',
    password: '',
  });
  const [errors, setErrors] = useState<Partial<FormData>>({});

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value } = e.target;
    setFormData((prev) => ({ ...prev, [name]: value }));

    // Real-time validation
    if (name === 'email' && !value.includes('@')) {
      setErrors((prev) => ({ ...prev, email: 'Invalid email' }));
    } else {
      setErrors((prev) => ({ ...prev, [name]: undefined }));
    }
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('Submitted:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username</label>
        <input
          type="text"
          name="username"
          value={formData.username}  {/* Controlled */}
          onChange={handleChange}
        />
        {errors.username && <span>{errors.username}</span>}
      </div>

      <div>
        <label>Email</label>
        <input
          type="email"
          name="email"
          value={formData.email}  {/* Controlled */}
          onChange={handleChange}
        />
        {errors.email && <span>{errors.email}</span>}
      </div>

      <div>
        <label>Password</label>
        <input
          type="password"
          name="password"
          value={formData.password}  {/* Controlled */}
          onChange={handleChange}
        />
        {errors.password && <span>{errors.password}</span>}
      </div>

      <button type="submit">Submit</button>
    </form>
  );
}
```

**Uncontrolled Form:**
```tsx
function UncontrolledForm() {
  const usernameRef = useRef<HTMLInputElement>(null);
  const emailRef = useRef<HTMLInputElement>(null);
  const passwordRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    // Access values from DOM
    const formData = {
      username: usernameRef.current?.value || '',
      email: emailRef.current?.value || '',
      password: passwordRef.current?.value || '',
    };

    console.log('Submitted:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>Username</label>
        <input
          type="text"
          name="username"
          ref={usernameRef}  {/* Uncontrolled */}
          defaultValue=""
        />
      </div>

      <div>
        <label>Email</label>
        <input
          type="email"
          name="email"
          ref={emailRef}  {/* Uncontrolled */}
          defaultValue=""
        />
      </div>

      <div>
        <label>Password</label>
        <input
          type="password"
          name="password"
          ref={passwordRef}  {/* Uncontrolled */}
          defaultValue=""
        />
      </div>

      <button type="submit">Submit</button>
    </form>
  );
}
```

---

#### Real-World Example: Search Input

**Controlled (with debouncing):**
```tsx
import { useState, useEffect } from 'react';

function ControlledSearch() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<string[]>([]);

  // Debounced search
  useEffect(() => {
    const timer = setTimeout(() => {
      if (query) {
        fetch(`/api/search?q=${query}`)
          .then((res) => res.json())
          .then(setResults);
      }
    }, 300);

    return () => clearTimeout(timer);
  }, [query]);

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results.map((result, i) => (
          <li key={i}>{result}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Uncontrolled (simple search):**
```tsx
function UncontrolledSearch() {
  const searchRef = useRef<HTMLInputElement>(null);
  const [results, setResults] = useState<string[]>([]);

  const handleSearch = () => {
    const query = searchRef.current?.value || '';
    
    fetch(`/api/search?q=${query}`)
      .then((res) => res.json())
      .then(setResults);
  };

  return (
    <div>
      <input
        type="text"
        ref={searchRef}
        placeholder="Search..."
      />
      <button onClick={handleSearch}>Search</button>
      <ul>
        {results.map((result, i) => (
          <li key={i}>{result}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

#### Controlled Component Benefits

**1. Real-time Validation:**
```tsx
const [email, setEmail] = useState('');
const [error, setError] = useState('');

const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  const value = e.target.value;
  setEmail(value);

  // Validate immediately
  if (!value.includes('@')) {
    setError('Invalid email format');
  } else {
    setError('');
  }
};

return (
  <div>
    <input value={email} onChange={handleChange} />
    {error && <span className="error">{error}</span>}
  </div>
);
```

**2. Conditional Formatting:**
```tsx
const [phone, setPhone] = useState('');

const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  let value = e.target.value.replace(/\D/g, ''); // Remove non-digits
  
  // Format: (123) 456-7890
  if (value.length > 6) {
    value = `(${value.slice(0, 3)}) ${value.slice(3, 6)}-${value.slice(6, 10)}`;
  } else if (value.length > 3) {
    value = `(${value.slice(0, 3)}) ${value.slice(3)}`;
  }
  
  setPhone(value);
};

return <input value={phone} onChange={handleChange} />;
```

**3. Dependent Fields:**
```tsx
const [country, setCountry] = useState('US');
const [state, setState] = useState('');

const states = country === 'US' ? US_STATES : CANADA_PROVINCES;

return (
  <>
    <select value={country} onChange={(e) => setCountry(e.target.value)}>
      <option value="US">United States</option>
      <option value="CA">Canada</option>
    </select>

    <select value={state} onChange={(e) => setState(e.target.value)}>
      {states.map((s) => (
        <option key={s} value={s}>{s}</option>
      ))}
    </select>
  </>
);
```

**4. Instant Feedback:**
```tsx
const [password, setPassword] = useState('');
const strength = calculatePasswordStrength(password);

return (
  <div>
    <input 
      type="password" 
      value={password} 
      onChange={(e) => setPassword(e.target.value)} 
    />
    <div className={`strength strength-${strength}`}>
      Password strength: {strength}
    </div>
  </div>
);
```

---

#### Uncontrolled Component Benefits

**1. Simpler Code (for simple forms):**
```tsx
function SimpleForm() {
  const nameRef = useRef<HTMLInputElement>(null);

  const handleSubmit = () => {
    console.log(nameRef.current?.value);
  };

  return (
    <>
      <input ref={nameRef} defaultValue="" />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
}
```

**2. Better Performance (no re-renders):**
```tsx
// Uncontrolled - no re-renders on typing
function UncontrolledTextarea() {
  const textRef = useRef<HTMLTextAreaElement>(null);

  return (
    <textarea 
      ref={textRef} 
      defaultValue="Type here..." 
    />
  );
}

// Controlled - re-renders on every keystroke
function ControlledTextarea() {
  const [text, setText] = useState('');

  return (
    <textarea 
      value={text} 
      onChange={(e) => setText(e.target.value)} 
    />
  );
}
```

**3. Integration with Non-React Code:**
```tsx
// Integrating with jQuery plugin
function JQueryDatePicker() {
  const dateRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    if (dateRef.current) {
      // jQuery plugin takes control
      $(dateRef.current).datepicker();
    }
  }, []);

  return <input ref={dateRef} />;
}
```

**4. File Inputs (always uncontrolled):**
```tsx
function FileUpload() {
  const fileRef = useRef<HTMLInputElement>(null);

  const handleUpload = () => {
    const files = fileRef.current?.files;
    if (files && files.length > 0) {
      const file = files[0];
      console.log('Uploading:', file.name);
      // Upload file
    }
  };

  return (
    <div>
      {/* File inputs are ALWAYS uncontrolled */}
      <input type="file" ref={fileRef} />
      <button onClick={handleUpload}>Upload</button>
    </div>
  );
}
```

---

#### Hybrid Approach

Sometimes you need both:

```tsx
function HybridForm() {
  // Controlled - needs validation
  const [email, setEmail] = useState('');
  const [emailError, setEmailError] = useState('');

  // Uncontrolled - simple field
  const nameRef = useRef<HTMLInputElement>(null);
  const phoneRef = useRef<HTMLInputElement>(null);

  const handleEmailChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value;
    setEmail(value);

    if (!value.includes('@')) {
      setEmailError('Invalid email');
    } else {
      setEmailError('');
    }
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    const formData = {
      name: nameRef.current?.value || '',
      email: email,
      phone: phoneRef.current?.value || '',
    };

    console.log('Submitted:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* Uncontrolled */}
      <input ref={nameRef} placeholder="Name" />

      {/* Controlled - needs validation */}
      <input 
        value={email} 
        onChange={handleEmailChange} 
        placeholder="Email" 
      />
      {emailError && <span>{emailError}</span>}

      {/* Uncontrolled */}
      <input ref={phoneRef} placeholder="Phone" />

      <button type="submit">Submit</button>
    </form>
  );
}
```

---

#### When to Use Each

**Use Controlled When:**

✅ Need real-time validation
✅ Need to format input (phone, credit card)
✅ Conditional logic based on input
✅ Need to disable submit until valid
✅ Multi-step forms with navigation
✅ Instant search/autocomplete
✅ Password strength indicators
✅ Character counters
✅ **Most React forms** (recommended)

**Use Uncontrolled When:**

✅ Simple forms with no validation
✅ File uploads (required)
✅ Integration with non-React libraries
✅ Performance-critical large text areas
✅ Legacy code migration
✅ Quick prototypes

---

#### Common Patterns

**1. Controlled with useReducer:**
```tsx
type FormState = {
  username: string;
  email: string;
};

type FormAction =
  | { type: 'UPDATE_FIELD'; field: keyof FormState; value: string }
  | { type: 'RESET' };

function formReducer(state: FormState, action: FormAction): FormState {
  switch (action.type) {
    case 'UPDATE_FIELD':
      return { ...state, [action.field]: action.value };
    case 'RESET':
      return { username: '', email: '' };
    default:
      return state;
  }
}

function ControlledFormWithReducer() {
  const [state, dispatch] = useReducer(formReducer, {
    username: '',
    email: '',
  });

  const handleChange = (field: keyof FormState) => (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    dispatch({ type: 'UPDATE_FIELD', field, value: e.target.value });
  };

  return (
    <form>
      <input value={state.username} onChange={handleChange('username')} />
      <input value={state.email} onChange={handleChange('email')} />
    </form>
  );
}
```

**2. Uncontrolled with FormData:**
```tsx
function UncontrolledFormData() {
  const formRef = useRef<HTMLFormElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    // Use native FormData API
    const formData = new FormData(formRef.current!);
    const data = Object.fromEntries(formData.entries());

    console.log('Submitted:', data);
  };

  return (
    <form ref={formRef} onSubmit={handleSubmit}>
      <input name="username" defaultValue="" />
      <input name="email" defaultValue="" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

#### Best Practices

**1. Prefer Controlled Components:**
```tsx
// ✅ Recommended - controlled
const [value, setValue] = useState('');
<input value={value} onChange={(e) => setValue(e.target.value)} />
```

**2. Use defaultValue for Uncontrolled:**
```tsx
// ✅ Good - defaultValue
<input ref={ref} defaultValue="initial" />

// ❌ Bad - value without onChange
<input value="initial" />  // Warning: read-only
```

**3. Don't Mix Controlled and Uncontrolled:**
```tsx
// ❌ Bad - switching between controlled/uncontrolled
const [value, setValue] = useState<string | undefined>(undefined);
<input value={value} onChange={(e) => setValue(e.target.value)} />

// ✅ Good - always controlled
const [value, setValue] = useState('');
<input value={value} onChange={(e) => setValue(e.target.value)} />
```

**4. Type Refs Properly:**
```tsx
const inputRef = useRef<HTMLInputElement>(null);
const textareaRef = useRef<HTMLTextAreaElement>(null);
const selectRef = useRef<HTMLSelectElement>(null);
```

---

#### Interview Tips

✅ **Key Points:**
- Controlled: React state is source of truth
- Uncontrolled: DOM is source of truth
- Controlled: use `value` + `onChange`
- Uncontrolled: use `ref` + `defaultValue`
- Controlled is recommended for most cases
- File inputs are always uncontrolled

✅ **When to Mention:**
- Discussing form handling
- State management in forms
- Performance optimization
- Integration with third-party libraries

✅ **Common Follow-ups:**
- "Which one should you use?"
- "What are the trade-offs?"
- "When would you use uncontrolled?"
- "How do file inputs work?"
- "Can you mix both?"

✅ **Perfect Answer Structure:**
1. Define both clearly
2. Show side-by-side example
3. Explain trade-offs
4. Controlled is preferred (mention why)
5. When to use uncontrolled (file inputs, legacy)

</details>

---

### 109. What is the Observer pattern in React?

<details>
<summary>View Answer</summary>

**Observer Pattern** is a behavioral design pattern where an object (subject) maintains a list of dependents (observers) and notifies them of state changes. In React, this pattern is used for event handling, subscriptions, and reactive state management.

#### What is the Observer Pattern?

The pattern consists of:
- **Subject** - object that holds state and notifies observers
- **Observers** - objects that subscribe to changes
- **Subscribe** - observers register to receive updates
- **Notify** - subject alerts all observers when state changes
- **Unsubscribe** - observers can stop receiving updates

```
Subject → notify → Observer 1
                 → Observer 2
                 → Observer 3
```

---

#### Basic Implementation

```tsx
// Subject (Observable)
class Observable<T> {
  private observers: Array<(data: T) => void> = [];
  private data: T;

  constructor(initialData: T) {
    this.data = initialData;
  }

  // Subscribe to changes
  subscribe(observer: (data: T) => void) {
    this.observers.push(observer);
    
    // Return unsubscribe function
    return () => {
      this.observers = this.observers.filter((obs) => obs !== observer);
    };
  }

  // Notify all observers
  notify() {
    this.observers.forEach((observer) => observer(this.data));
  }

  // Update data and notify
  setValue(newData: T) {
    this.data = newData;
    this.notify();
  }

  getValue() {
    return this.data;
  }
}

// Usage
const counter = new Observable(0);

// Observer 1
const unsubscribe1 = counter.subscribe((value) => {
  console.log('Observer 1:', value);
});

// Observer 2
const unsubscribe2 = counter.subscribe((value) => {
  console.log('Observer 2:', value);
});

counter.setValue(1);  // Both observers notified
counter.setValue(2);  // Both observers notified

unsubscribe1();       // Observer 1 stops listening
counter.setValue(3);  // Only Observer 2 notified
```

---

#### React Hook for Observer Pattern

```tsx
import { useEffect, useState } from 'react';

// Observable class (same as above)
class Observable<T> {
  private observers: Array<(data: T) => void> = [];
  private data: T;

  constructor(initialData: T) {
    this.data = initialData;
  }

  subscribe(observer: (data: T) => void) {
    this.observers.push(observer);
    return () => {
      this.observers = this.observers.filter((obs) => obs !== observer);
    };
  }

  notify() {
    this.observers.forEach((observer) => observer(this.data));
  }

  setValue(newData: T) {
    this.data = newData;
    this.notify();
  }

  getValue() {
    return this.data;
  }
}

// Custom hook to use observable
function useObservable<T>(observable: Observable<T>): T {
  const [value, setValue] = useState(observable.getValue());

  useEffect(() => {
    // Subscribe to changes
    const unsubscribe = observable.subscribe((newValue) => {
      setValue(newValue);
    });

    // Cleanup subscription
    return unsubscribe;
  }, [observable]);

  return value;
}

// Create observable outside components
const userStore = new Observable({
  name: 'John Doe',
  email: 'john@example.com',
});

// Component 1 - observes user
function UserProfile() {
  const user = useObservable(userStore);

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Component 2 - also observes user
function UserHeader() {
  const user = useObservable(userStore);

  return <div>Welcome, {user.name}!</div>;
}

// Component 3 - modifies user
function UserSettings() {
  const user = useObservable(userStore);

  const handleUpdate = () => {
    userStore.setValue({
      name: 'Jane Smith',
      email: 'jane@example.com',
    });
    // All observers (UserProfile, UserHeader) will update!
  };

  return (
    <div>
      <p>Current: {user.name}</p>
      <button onClick={handleUpdate}>Update User</button>
    </div>
  );
}
```

---

#### Real-World Example: Event Emitter

```tsx
import { useEffect, useState } from 'react';

// Event Emitter (Observer Pattern)
class EventEmitter {
  private events: Map<string, Array<(data: any) => void>> = new Map();

  on(event: string, callback: (data: any) => void) {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(callback);

    // Return unsubscribe function
    return () => {
      const callbacks = this.events.get(event);
      if (callbacks) {
        this.events.set(
          event,
          callbacks.filter((cb) => cb !== callback)
        );
      }
    };
  }

  emit(event: string, data?: any) {
    const callbacks = this.events.get(event);
    if (callbacks) {
      callbacks.forEach((callback) => callback(data));
    }
  }

  off(event: string, callback: (data: any) => void) {
    const callbacks = this.events.get(event);
    if (callbacks) {
      this.events.set(
        event,
        callbacks.filter((cb) => cb !== callback)
      );
    }
  }
}

// Global event bus
const eventBus = new EventEmitter();

// Custom hook to listen to events
function useEvent<T = any>(
  event: string,
  handler: (data: T) => void
) {
  useEffect(() => {
    const unsubscribe = eventBus.on(event, handler);
    return unsubscribe;
  }, [event, handler]);
}

// Component 1 - Emits events
function NotificationButton() {
  const handleClick = () => {
    eventBus.emit('notification', {
      type: 'success',
      message: 'Action completed!',
    });
  };

  return <button onClick={handleClick}>Show Notification</button>;
}

// Component 2 - Listens to events
function NotificationDisplay() {
  const [notifications, setNotifications] = useState<any[]>([]);

  useEvent('notification', (data) => {
    setNotifications((prev) => [...prev, data]);
    
    // Auto-remove after 3 seconds
    setTimeout(() => {
      setNotifications((prev) => prev.slice(1));
    }, 3000);
  });

  return (
    <div className="notification-container">
      {notifications.map((notif, i) => (
        <div key={i} className={`notification ${notif.type}`}>
          {notif.message}
        </div>
      ))}
    </div>
  );
}

// Component 3 - Also listens
function ActivityLog() {
  const [logs, setLogs] = useState<string[]>([]);

  useEvent('notification', (data) => {
    setLogs((prev) => [...prev, `${new Date().toISOString()}: ${data.message}`]);
  });

  return (
    <div>
      <h3>Activity Log</h3>
      <ul>
        {logs.map((log, i) => (
          <li key={i}>{log}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

#### React's Built-in Observer Pattern

**1. Context API (Observer Pattern):**
```tsx
// Provider is the Subject
// Consumers are Observers

const ThemeContext = createContext('light');

// Subject (Provider)
function App() {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={theme}>
      {/* All consumers will be notified when theme changes */}
      <Header />
      <Content />
      <Footer />
    </ThemeContext.Provider>
  );
}

// Observers (Consumers)
function Header() {
  const theme = useContext(ThemeContext);  // Observing theme
  return <header className={theme}>Header</header>;
}

function Content() {
  const theme = useContext(ThemeContext);  // Observing theme
  return <main className={theme}>Content</main>;
}
```

**2. useState/useReducer (Observer Pattern):**
```tsx
// State is the Subject
// Components using the state are Observers

function Counter() {
  const [count, setCount] = useState(0);  // Subject

  // When count changes, component re-renders (notified)
  return (
    <div>
      <p>Count: {count}</p>  {/* Observer */}
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

**3. useEffect (Observer Pattern):**
```tsx
function Component() {
  const [data, setData] = useState(null);

  // Effect observes 'data'
  useEffect(() => {
    console.log('Data changed:', data);
  }, [data]);  // Observer: triggers when data changes

  return <div>...</div>;
}
```

---

#### Real-World: WebSocket Observer

```tsx
import { useEffect, useState } from 'react';

// WebSocket Subject
class WebSocketService {
  private socket: WebSocket | null = null;
  private observers: Array<(data: any) => void> = [];

  connect(url: string) {
    this.socket = new WebSocket(url);

    this.socket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      this.notify(data);
    };

    this.socket.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }

  subscribe(observer: (data: any) => void) {
    this.observers.push(observer);
    
    return () => {
      this.observers = this.observers.filter((obs) => obs !== observer);
    };
  }

  notify(data: any) {
    this.observers.forEach((observer) => observer(data));
  }

  send(data: any) {
    if (this.socket && this.socket.readyState === WebSocket.OPEN) {
      this.socket.send(JSON.stringify(data));
    }
  }

  disconnect() {
    if (this.socket) {
      this.socket.close();
    }
  }
}

// Global WebSocket instance
const wsService = new WebSocketService();
wsService.connect('ws://localhost:8080');

// Custom hook
function useWebSocket<T = any>() {
  const [messages, setMessages] = useState<T[]>([]);

  useEffect(() => {
    const unsubscribe = wsService.subscribe((data: T) => {
      setMessages((prev) => [...prev, data]);
    });

    return unsubscribe;
  }, []);

  return { messages, send: wsService.send.bind(wsService) };
}

// Component 1 - Chat Messages
function ChatMessages() {
  const { messages } = useWebSocket<{ user: string; text: string }>();

  return (
    <div>
      {messages.map((msg, i) => (
        <div key={i}>
          <strong>{msg.user}:</strong> {msg.text}
        </div>
      ))}
    </div>
  );
}

// Component 2 - Online Users
function OnlineUsers() {
  const { messages } = useWebSocket<{ type: string; users: string[] }>();
  const onlineUsers = messages
    .filter((msg) => msg.type === 'user-list')
    .flatMap((msg) => msg.users);

  return (
    <div>
      <h3>Online Users</h3>
      <ul>
        {onlineUsers.map((user, i) => (
          <li key={i}>{user}</li>
        ))}
      </ul>
    </div>
  );
}

// Component 3 - Send Message
function ChatInput() {
  const { send } = useWebSocket();
  const [text, setText] = useState('');

  const handleSend = () => {
    send({ type: 'message', text });
    setText('');
  };

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={handleSend}>Send</button>
    </div>
  );
}
```

---

#### Observer Pattern with RxJS

```tsx
import { BehaviorSubject } from 'rxjs';
import { useEffect, useState } from 'react';

// Create observable with RxJS
const userSubject = new BehaviorSubject({
  name: 'John Doe',
  email: 'john@example.com',
});

// Custom hook for RxJS observable
function useObservable<T>(subject: BehaviorSubject<T>): T {
  const [value, setValue] = useState(subject.getValue());

  useEffect(() => {
    const subscription = subject.subscribe((newValue) => {
      setValue(newValue);
    });

    return () => subscription.unsubscribe();
  }, [subject]);

  return value;
}

// Component using RxJS observable
function UserProfile() {
  const user = useObservable(userSubject);

  const handleUpdate = () => {
    userSubject.next({
      name: 'Jane Smith',
      email: 'jane@example.com',
    });
  };

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={handleUpdate}>Update</button>
    </div>
  );
}
```

---

#### Benefits of Observer Pattern

**1. Loose Coupling:**
```tsx
// Components don't need to know about each other
eventBus.emit('user-updated', userData);

// Any component can listen
useEvent('user-updated', (data) => {
  console.log('User updated:', data);
});
```

**2. Multiple Observers:**
```tsx
// Multiple components can observe the same data
const store = new Observable(initialData);

// Component A
const dataA = useObservable(store);

// Component B
const dataB = useObservable(store);

// Component C
const dataC = useObservable(store);

// All update when store changes!
```

**3. Reactive Updates:**
```tsx
// Automatic updates when data changes
store.setValue(newData);  // All observers notified automatically
```

---

#### When to Use Observer Pattern in React

**✅ Good Use Cases:**

1. **Real-time updates** (WebSocket, SSE)
2. **Global events** (notifications, analytics)
3. **Cross-component communication**
4. **State synchronization** across components
5. **Pub/sub systems**
6. **Live data feeds**

**❌ When Not to Use:**

1. **Simple parent-child communication** (use props)
2. **Local component state** (use useState)
3. **When Context API is sufficient**
4. **Over-engineering simple apps**

---

#### Best Practices

**1. Always Unsubscribe:**
```tsx
useEffect(() => {
  const unsubscribe = observable.subscribe(handler);
  return unsubscribe;  // Cleanup!
}, []);
```

**2. Use Custom Hooks:**
```tsx
// Encapsulate subscription logic
function useObservable<T>(observable: Observable<T>) {
  const [value, setValue] = useState(observable.getValue());
  
  useEffect(() => {
    const unsubscribe = observable.subscribe(setValue);
    return unsubscribe;
  }, [observable]);
  
  return value;
}
```

**3. Type Safety:**
```tsx
class Observable<T> {
  private observers: Array<(data: T) => void> = [];
  // TypeScript ensures type safety
}
```

**4. Error Handling:**
```tsx
class Observable<T> {
  notify(data: T) {
    this.observers.forEach((observer) => {
      try {
        observer(data);
      } catch (error) {
        console.error('Observer error:', error);
      }
    });
  }
}
```

---

#### Interview Tips

✅ **Key Points:**
- Observer pattern: subject notifies observers of changes
- Subject maintains list of observers
- Observers subscribe/unsubscribe
- When state changes, all observers notified
- React uses this internally (Context, useState, useEffect)

✅ **When to Mention:**
- Discussing state management patterns
- Event-driven architectures
- Real-time features
- Component communication
- Reactive programming

✅ **Common Follow-ups:**
- "How is this used in React?"
- "Give me a real-world example"
- "What are the benefits?"
- "How does Context API use this?"
- "When would you implement this?"

✅ **Perfect Answer Structure:**
1. Define: Subject notifies observers of changes
2. Components: Subject, Observers, Subscribe, Notify
3. React examples: Context API, useState, useEffect
4. Show custom implementation
5. Real-world use: WebSocket, event bus
6. When to use: real-time, global events

</details>

---

### 110. What is Dependency Injection in React?

<details>
<summary>View Answer</summary>

**Dependency Injection (DI)** is a design pattern where dependencies are provided to a component from the outside rather than being created inside the component. In React, this is achieved through props, Context API, or custom hooks.

#### What is Dependency Injection?

Dependency Injection means:
- **Dependencies are passed in** (not created internally)
- **Loose coupling** between components
- **Easy testing** (inject mock dependencies)
- **Flexibility** - swap implementations easily
- **Inversion of Control** - component doesn't control its dependencies

```tsx
// ❌ Without DI - tight coupling
function UserProfile() {
  const api = new UserAPI();  // Creates dependency internally
  const user = api.getUser();
  return <div>{user.name}</div>;
}

// ✅ With DI - loose coupling
function UserProfile({ api }) {  // Dependency injected via props
  const user = api.getUser();
  return <div>{user.name}</div>;
}
```

---

#### Basic Example: Props Injection

```tsx
// Service/Dependency
class UserService {
  async getUser(id: string) {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }

  async updateUser(id: string, data: any) {
    const response = await fetch(`/api/users/${id}`, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
    return response.json();
  }
}

// Component with DI via props
interface UserProfileProps {
  userId: string;
  userService: UserService;  // Dependency injected
}

function UserProfile({ userId, userService }: UserProfileProps) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    userService.getUser(userId).then(setUser);
  }, [userId, userService]);

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Usage - inject the service
const userService = new UserService();

function App() {
  return <UserProfile userId="123" userService={userService} />;
}
```

---

#### DI with Context API

```tsx
import { createContext, useContext, ReactNode } from 'react';

// 1. Define services
class UserService {
  async getUser(id: string) {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }
}

class AuthService {
  async login(email: string, password: string) {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    });
    return response.json();
  }
}

class ApiService {
  async request(url: string, options?: RequestInit) {
    const response = await fetch(url, options);
    return response.json();
  }
}

// 2. Create service container
interface Services {
  userService: UserService;
  authService: AuthService;
  apiService: ApiService;
}

const ServicesContext = createContext<Services | undefined>(undefined);

// 3. Provider component
interface ServicesProviderProps {
  children: ReactNode;
  services: Services;
}

export const ServicesProvider: React.FC<ServicesProviderProps> = ({ 
  children, 
  services 
}) => {
  return (
    <ServicesContext.Provider value={services}>
      {children}
    </ServicesContext.Provider>
  );
};

// 4. Custom hook to access services
export const useServices = () => {
  const context = useContext(ServicesContext);
  if (!context) {
    throw new Error('useServices must be used within ServicesProvider');
  }
  return context;
};

// Hook for specific service
export const useUserService = () => useServices().userService;
export const useAuthService = () => useServices().authService;
export const useApiService = () => useServices().apiService;

// 5. Usage in App
function App() {
  const services: Services = {
    userService: new UserService(),
    authService: new AuthService(),
    apiService: new ApiService(),
  };

  return (
    <ServicesProvider services={services}>
      <UserProfile userId="123" />
      <LoginForm />
    </ServicesProvider>
  );
}

// 6. Components use injected services
function UserProfile({ userId }: { userId: string }) {
  const userService = useUserService();  // DI via Context
  const [user, setUser] = useState(null);

  useEffect(() => {
    userService.getUser(userId).then(setUser);
  }, [userId, userService]);

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

function LoginForm() {
  const authService = useAuthService();  // DI via Context
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = async () => {
    try {
      await authService.login(email, password);
      alert('Login successful!');
    } catch (error) {
      alert('Login failed!');
    }
  };

  return (
    <form onSubmit={(e) => { e.preventDefault(); handleLogin(); }}>
      <input 
        type="email" 
        value={email} 
        onChange={(e) => setEmail(e.target.value)} 
      />
      <input 
        type="password" 
        value={password} 
        onChange={(e) => setPassword(e.target.value)} 
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

#### Real-World Example: Testing with DI

```tsx
// API Service
interface IUserService {
  getUser(id: string): Promise<User>;
  updateUser(id: string, data: Partial<User>): Promise<User>;
}

class UserService implements IUserService {
  async getUser(id: string): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    return response.json();
  }

  async updateUser(id: string, data: Partial<User>): Promise<User> {
    const response = await fetch(`/api/users/${id}`, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
    return response.json();
  }
}

// Mock service for testing
class MockUserService implements IUserService {
  async getUser(id: string): Promise<User> {
    return {
      id,
      name: 'Test User',
      email: 'test@example.com',
    };
  }

  async updateUser(id: string, data: Partial<User>): Promise<User> {
    return {
      id,
      name: data.name || 'Test User',
      email: data.email || 'test@example.com',
    };
  }
}

// Component using DI
interface UserProfileProps {
  userId: string;
  userService: IUserService;  // Interface, not concrete class
}

function UserProfile({ userId, userService }: UserProfileProps) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    userService.getUser(userId)
      .then(setUser)
      .finally(() => setLoading(false));
  }, [userId, userService]);

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Production usage
function App() {
  const userService = new UserService();  // Real service
  return <UserProfile userId="123" userService={userService} />;
}

// Test usage
import { render, screen, waitFor } from '@testing-library/react';

test('displays user data', async () => {
  const mockService = new MockUserService();  // Mock service
  
  render(<UserProfile userId="123" userService={mockService} />);

  await waitFor(() => {
    expect(screen.getByText('Test User')).toBeInTheDocument();
  });
});
```

---

#### Advanced: Custom Hook DI

```tsx
// Create custom hooks that abstract dependencies

// Hook that provides data fetching
function useDataFetcher() {
  const fetch = async (url: string) => {
    const response = await window.fetch(url);
    return response.json();
  };
  
  return { fetch };
}

// Component uses the hook
function UserList() {
  const { fetch } = useDataFetcher();  // DI via custom hook
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetch('/api/users').then(setUsers);
  }, [fetch]);

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// For testing, create mock hook
function useMockDataFetcher() {
  const fetch = async (url: string) => {
    return [
      { id: 1, name: 'Test User 1' },
      { id: 2, name: 'Test User 2' },
    ];
  };
  
  return { fetch };
}
```

---

#### DI with Higher-Order Components

```tsx
// HOC for DI
interface WithUserServiceProps {
  userService: UserService;
}

function withUserService<P extends WithUserServiceProps>(
  Component: React.ComponentType<P>
) {
  return function WithUserServiceComponent(
    props: Omit<P, 'userService'>
  ) {
    const userService = useUserService();  // Inject service
    
    return <Component {...(props as P)} userService={userService} />;
  };
}

// Component expecting injected service
interface UserProfileProps extends WithUserServiceProps {
  userId: string;
}

function UserProfile({ userId, userService }: UserProfileProps) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    userService.getUser(userId).then(setUser);
  }, [userId, userService]);

  return <div>{user?.name}</div>;
}

// Enhanced component with DI
const UserProfileWithService = withUserService(UserProfile);

// Usage - no need to pass userService prop
<UserProfileWithService userId="123" />
```

---

#### DI Container Pattern

```tsx
// Service container
class ServiceContainer {
  private services: Map<string, any> = new Map();

  register<T>(name: string, service: T): void {
    this.services.set(name, service);
  }

  get<T>(name: string): T {
    const service = this.services.get(name);
    if (!service) {
      throw new Error(`Service ${name} not registered`);
    }
    return service;
  }

  has(name: string): boolean {
    return this.services.has(name);
  }
}

// Create container
const container = new ServiceContainer();

// Register services
container.register('userService', new UserService());
container.register('authService', new AuthService());
container.register('apiService', new ApiService());

// Context for container
const ContainerContext = createContext<ServiceContainer | undefined>(undefined);

export const ContainerProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  return (
    <ContainerContext.Provider value={container}>
      {children}
    </ContainerContext.Provider>
  );
};

// Hook to get service from container
export function useService<T>(name: string): T {
  const container = useContext(ContainerContext);
  if (!container) {
    throw new Error('useService must be used within ContainerProvider');
  }
  return container.get<T>(name);
}

// Usage
function UserProfile({ userId }: { userId: string }) {
  const userService = useService<UserService>('userService');
  const [user, setUser] = useState(null);

  useEffect(() => {
    userService.getUser(userId).then(setUser);
  }, [userId, userService]);

  return <div>{user?.name}</div>;
}
```

---

#### Benefits of Dependency Injection

**1. Easy Testing:**
```tsx
// ✅ With DI - easy to test with mocks
function UserProfile({ userService }) {
  // ...
}

// Test
test('renders user', () => {
  const mockService = {
    getUser: jest.fn().mockResolvedValue({ name: 'Test' }),
  };
  render(<UserProfile userService={mockService} />);
});

// ❌ Without DI - hard to test
function UserProfile() {
  const service = new UserService();  // Can't mock!
  // ...
}
```

**2. Loose Coupling:**
```tsx
// Component doesn't depend on concrete implementation
interface IApiService {
  get(url: string): Promise<any>;
}

function DataDisplay({ apiService }: { apiService: IApiService }) {
  // Works with any implementation of IApiService
}

// Can use different implementations
<DataDisplay apiService={new RestApiService()} />
<DataDisplay apiService={new GraphQLApiService()} />
<DataDisplay apiService={new MockApiService()} />
```

**3. Configuration Flexibility:**
```tsx
// Different configs for different environments
const services = {
  apiService: process.env.NODE_ENV === 'development'
    ? new MockApiService()
    : new RealApiService(),
};

<ServicesProvider services={services}>
  <App />
</ServicesProvider>
```

**4. Reusability:**
```tsx
// Same component works with different services
<UserList userService={adminUserService} />
<UserList userService={publicUserService} />
```

---

#### Common DI Patterns in React

**1. Props (Simple DI):**
```tsx
<Component service={service} />
```

**2. Context API (Global DI):**
```tsx
<ServiceProvider>
  <Component />  {/* Access via useService() */}
</ServiceProvider>
```

**3. Custom Hooks (Abstracted DI):**
```tsx
function useUserData() {
  const service = useUserService();
  return service.getData();
}
```

**4. HOC (Legacy DI):**
```tsx
const Enhanced = withService(Component);
```

---

#### Real-World: API Client DI

```tsx
// API Client interface
interface IApiClient {
  get<T>(url: string): Promise<T>;
  post<T>(url: string, data: any): Promise<T>;
  put<T>(url: string, data: any): Promise<T>;
  delete<T>(url: string): Promise<T>;
}

// Fetch implementation
class FetchApiClient implements IApiClient {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  async get<T>(url: string): Promise<T> {
    const response = await fetch(`${this.baseUrl}${url}`);
    return response.json();
  }

  async post<T>(url: string, data: any): Promise<T> {
    const response = await fetch(`${this.baseUrl}${url}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    return response.json();
  }

  async put<T>(url: string, data: any): Promise<T> {
    const response = await fetch(`${this.baseUrl}${url}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    return response.json();
  }

  async delete<T>(url: string): Promise<T> {
    const response = await fetch(`${this.baseUrl}${url}`, {
      method: 'DELETE',
    });
    return response.json();
  }
}

// Axios implementation
class AxiosApiClient implements IApiClient {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  async get<T>(url: string): Promise<T> {
    const response = await axios.get(`${this.baseUrl}${url}`);
    return response.data;
  }

  async post<T>(url: string, data: any): Promise<T> {
    const response = await axios.post(`${this.baseUrl}${url}`, data);
    return response.data;
  }

  async put<T>(url: string, data: any): Promise<T> {
    const response = await axios.put(`${this.baseUrl}${url}`, data);
    return response.data;
  }

  async delete<T>(url: string): Promise<T> {
    const response = await axios.delete(`${this.baseUrl}${url}`);
    return response.data;
  }
}

// Context setup
const ApiClientContext = createContext<IApiClient | undefined>(undefined);

export const ApiClientProvider: React.FC<{ 
  client: IApiClient; 
  children: ReactNode 
}> = ({ client, children }) => {
  return (
    <ApiClientContext.Provider value={client}>
      {children}
    </ApiClientContext.Provider>
  );
};

export const useApiClient = () => {
  const context = useContext(ApiClientContext);
  if (!context) {
    throw new Error('useApiClient must be used within ApiClientProvider');
  }
  return context;
};

// App setup
function App() {
  // Switch implementation easily!
  const apiClient = new FetchApiClient('https://api.example.com');
  // const apiClient = new AxiosApiClient('https://api.example.com');

  return (
    <ApiClientProvider client={apiClient}>
      <UserList />
      <ProductList />
    </ApiClientProvider>
  );
}

// Components use injected client
function UserList() {
  const apiClient = useApiClient();
  const [users, setUsers] = useState([]);

  useEffect(() => {
    apiClient.get<User[]>('/users').then(setUsers);
  }, [apiClient]);

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

#### Best Practices

**1. Use Interfaces/Types:**
```tsx
// ✅ Good - depend on interface
interface IUserService {
  getUser(id: string): Promise<User>;
}

function Component({ userService }: { userService: IUserService }) {
  // ...
}

// ❌ Bad - depend on concrete class
function Component({ userService }: { userService: UserService }) {
  // ...
}
```

**2. Single Responsibility:**
```tsx
// ✅ Good - focused services
class UserService {
  getUser() { /* ... */ }
  updateUser() { /* ... */ }
}

class AuthService {
  login() { /* ... */ }
  logout() { /* ... */ }
}

// ❌ Bad - god service
class ApiService {
  getUser() { /* ... */ }
  login() { /* ... */ }
  getProducts() { /* ... */ }
  checkout() { /* ... */ }
  // ... everything!
}
```

**3. Immutable Services:**
```tsx
// Services should be stateless or read-only
class UserService {
  constructor(private apiClient: IApiClient) {}
  
  getUser(id: string) {
    return this.apiClient.get(`/users/${id}`);
  }
}
```

**4. Provide Defaults:**
```tsx
// Provide default implementation
const ApiClientContext = createContext<IApiClient>(
  new FetchApiClient('https://api.example.com')
);
```

---

#### When to Use DI in React

**✅ Use DI When:**

1. **Need testability** - inject mocks in tests
2. **Multiple implementations** - swap services easily
3. **Cross-cutting concerns** - logging, analytics, API
4. **Complex dependencies** - services depend on other services
5. **Configuration** - different behavior per environment

**❌ Don't Use DI When:**

1. **Simple components** - overkill for basic UI
2. **One-time use** - no reusability needed
3. **UI-only logic** - no external dependencies
4. **Over-engineering** - adds unnecessary complexity

---

#### Interview Tips

✅ **Key Points:**
- DI = dependencies passed from outside, not created internally
- Benefits: testability, loose coupling, flexibility
- React DI methods: props, Context API, custom hooks, HOCs
- Use interfaces for flexibility
- Makes testing easier (inject mocks)

✅ **When to Mention:**
- Discussing testing strategies
- Component architecture
- Service layer design
- Code maintainability
- SOLID principles

✅ **Common Follow-ups:**
- "How do you implement DI in React?"
- "What are the benefits?"
- "How does it help with testing?"
- "Give a real-world example"
- "What's the difference between DI and props?"

✅ **Perfect Answer Structure:**
1. Define: Dependencies injected from outside
2. Why: Testability, flexibility, loose coupling
3. How in React: Props, Context, hooks
4. Example: Service injection with Context
5. Benefits: Easy to mock, swap implementations
6. When to use: Complex apps, testing, multiple configs

</details>
