# Testing

> Advanced / Senior Level (3-5 years)

---

## Questions

131. What is React Testing Library and why use it over Enzyme?
132. How do you test React components?
133. What is the difference between render and screen in Testing Library?
134. How do you test hooks with React Testing Library?
135. What is Jest and how does it work with React?
136. How do you mock API calls in tests?
137. What is the difference between unit and integration tests?
138. How do you test user interactions?
139. What is Vitest and how does it compare to Jest?
140. How do you achieve high test coverage?

---

## Detailed Answers

### 131. What is React Testing Library and why use it over Enzyme?

<details>
<summary>View Answer</summary>

**React Testing Library (RTL)** is a **lightweight testing library** that encourages **testing components from the user's perspective**. It's now the **industry standard** for React testing, replacing Enzyme.

---

#### **1. Core Philosophy**

```typescript
// React Testing Library's Guiding Principle:
"The more your tests resemble the way your software is used, 
the more confidence they can give you."

const philosophyComparison = {
  enzyme: {
    approach: 'Implementation details',
    focus: 'Component internals',
    tests: 'Test state, props, lifecycle',
    example: 'wrapper.state().count',
  },
  
  rtl: {
    approach: 'User behavior',
    focus: 'What user sees and does',
    tests: 'Test rendered output and interactions',
    example: 'screen.getByText("Count: 5")',
  },
};
```

---

#### **2. Key Differences**

**Enzyme Approach:**
```typescript
// ❌ Enzyme - Testing implementation details
import { shallow } from 'enzyme';

function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// Enzyme test
it('increments counter', () => {
  const wrapper = shallow(<Counter />);
  
  // Testing implementation details (state)
  expect(wrapper.state('count')).toBe(0);
  
  // Shallow rendering (doesn't render children)
  wrapper.find('button').simulate('click');
  
  expect(wrapper.state('count')).toBe(1);
});

// Problems:
// - Tests break when refactoring (e.g., moving state to useReducer)
// - Doesn't test what user sees
// - Brittle tests
// - False confidence
```

**React Testing Library Approach:**
```typescript
// ✅ RTL - Testing user behavior
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// Same Counter component
function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// RTL test
it('increments counter when button clicked', async () => {
  const user = userEvent.setup();
  render(<Counter />);
  
  // Test what user sees
  expect(screen.getByText('Count: 0')).toBeInTheDocument();
  
  // Simulate real user interaction
  await user.click(screen.getByRole('button', { name: /increment/i }));
  
  // Verify what user sees after interaction
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});

// Benefits:
// - Tests don't break when refactoring implementation
// - Tests what user actually experiences
// - More confident tests
// - Encourages accessible components (roles, labels)
```

---

#### **3. Feature Comparison**

```typescript
const featureComparison = {
  rendering: {
    enzyme: 'Shallow, mount, render',
    rtl: 'Full DOM rendering only',
    winner: 'RTL - More realistic',
  },
  
  queries: {
    enzyme: 'CSS selectors, component instances',
    rtl: 'Accessible queries (role, label, text)',
    winner: 'RTL - Encourages accessibility',
  },
  
  assertions: {
    enzyme: 'State, props, internal methods',
    rtl: 'Rendered output only',
    winner: 'RTL - Tests user experience',
  },
  
  userInteractions: {
    enzyme: 'simulate() (not realistic)',
    rtl: 'userEvent (realistic browser events)',
    winner: 'RTL - More accurate',
  },
  
  maintenance: {
    enzyme: 'Requires adapter updates for React versions',
    rtl: 'Works with any React version',
    winner: 'RTL - Less maintenance',
  },
  
  communitySupport: {
    enzyme: 'Maintenance mode (no longer actively developed)',
    rtl: 'Active development, officially recommended',
    winner: 'RTL - Future-proof',
  },
};
```

---

#### **4. Why Use React Testing Library**

**1. Tests User Behavior, Not Implementation**

```typescript
// Example: Refactoring doesn't break tests

// Version 1: useState
function SearchBox() {
  const [query, setQuery] = useState('');
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}

// Version 2: Refactored to useReducer
function SearchBox() {
  const [state, dispatch] = useReducer(reducer, { query: '' });
  return (
    <input 
      value={state.query} 
      onChange={e => dispatch({ type: 'SET_QUERY', payload: e.target.value })}
    />
  );
}

// ✅ RTL test works for both versions!
it('updates search query', async () => {
  const user = userEvent.setup();
  render(<SearchBox />);
  
  const input = screen.getByRole('textbox');
  await user.type(input, 'react');
  
  expect(input).toHaveValue('react');
});
// Test doesn't care about implementation (useState vs useReducer)
```

**2. Encourages Accessibility**

```typescript
// RTL queries encourage accessible markup

// ❌ Bad (hard to query)
function Button() {
  return <div className="button" onClick={handleClick}>Click</div>;
}

// Test would need:
// screen.getByClassName('button') // Not available!

// ✅ Good (accessible and testable)
function Button() {
  return <button onClick={handleClick}>Click</button>;
}

// Easy to test:
screen.getByRole('button', { name: /click/i });

// RTL's query priority guides you to write accessible code:
const queryPriority = [
  '1. getByRole',           // Accessible to everyone
  '2. getByLabelText',      // Form elements
  '3. getByPlaceholderText',
  '4. getByText',           // Non-interactive content
  '5. getByDisplayValue',   // Form elements current value
  '6. getByAltText',        // Images
  '7. getByTitle',
  '8. getByTestId',         // Last resort
];
```

**3. Simulates Real User Interactions**

```typescript
import userEvent from '@testing-library/user-event';

// ❌ Enzyme's simulate() - Not realistic
wrapper.find('input').simulate('change', { target: { value: 'test' } });
// Doesn't trigger real browser events

// ✅ userEvent - Realistic browser events
const user = userEvent.setup();
await user.type(screen.getByRole('textbox'), 'test');
// Triggers: keyDown, keyPress, keyUp, input, change
// Just like a real user typing!

await user.click(screen.getByRole('button'));
// Triggers: mouseOver, mouseMove, mouseDown, mouseUp, click

// Complex interactions
await user.dblClick(element);      // Double click
await user.tripleClick(element);   // Triple click
await user.hover(element);         // Hover
await user.unhover(element);       // Unhover
await user.tab();                  // Tab key
await user.keyboard('{Enter}');    // Keyboard input
await user.upload(input, file);    // File upload
```

**4. Faster and More Reliable**

```typescript
const performance = {
  enzyme: {
    shallowRender: 'Fast but unrealistic',
    fullRender: 'Slow, complex setup',
    isolation: 'Tests components in isolation',
  },
  
  rtl: {
    rendering: 'Always full render (faster than Enzyme mount)',
    realism: 'Tests real DOM like production',
    integration: 'Tests components together (more confidence)',
  },
};
```

**5. Better Async Testing**

```typescript
// ✅ RTL has excellent async utilities

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);
  
  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}

// RTL async test
it('loads and displays user', async () => {
  render(<UserProfile userId="123" />);
  
  // Automatically waits for element to appear
  const userName = await screen.findByText('John Doe');
  expect(userName).toBeInTheDocument();
  
  // Or use waitFor for more complex conditions
  await waitFor(() => {
    expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
  });
});

// Built-in async utilities:
// - findBy* queries (wait for element)
// - waitFor() (wait for condition)
// - waitForElementToBeRemoved() (wait for removal)
```

---

#### **5. Migration from Enzyme**

```typescript
// Migration patterns

const migrationGuide = [
  {
    enzyme: 'shallow(<Component />)',
    rtl: 'render(<Component />)',
    note: 'Always use full render',
  },
  {
    enzyme: 'wrapper.find(".class")',
    rtl: 'screen.getByRole("button")',
    note: 'Use accessible queries',
  },
  {
    enzyme: 'wrapper.state()',
    rtl: 'screen.getByText(...)',
    note: 'Test output, not state',
  },
  {
    enzyme: 'wrapper.props()',
    rtl: 'Test behavior, not props',
    note: 'Test what happens, not inputs',
  },
  {
    enzyme: 'wrapper.simulate("click")',
    rtl: 'await user.click(element)',
    note: 'Use userEvent for realism',
  },
];

// Example migration

// ❌ Before (Enzyme)
import { shallow } from 'enzyme';

it('toggles visibility', () => {
  const wrapper = shallow(<Dropdown />);
  expect(wrapper.state('isOpen')).toBe(false);
  
  wrapper.find('button').simulate('click');
  expect(wrapper.state('isOpen')).toBe(true);
  
  expect(wrapper.find('.dropdown-menu')).toHaveLength(1);
});

// ✅ After (RTL)
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('shows dropdown menu when button clicked', async () => {
  const user = userEvent.setup();
  render(<Dropdown />);
  
  // Menu should be hidden initially
  expect(screen.queryByRole('menu')).not.toBeInTheDocument();
  
  // Click button
  await user.click(screen.getByRole('button'));
  
  // Menu should be visible
  expect(screen.getByRole('menu')).toBeInTheDocument();
});
```

---

#### **6. Common RTL Patterns**

```typescript
// Setup and cleanup
import { render, screen, cleanup } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// Automatic cleanup after each test
afterEach(() => {
  cleanup(); // Usually not needed, RTL auto-cleans
});

// Custom render with providers
function renderWithProviders(ui: React.ReactElement) {
  return render(
    <Provider store={store}>
      <Router>
        <ThemeProvider>
          {ui}
        </ThemeProvider>
      </Router>
    </Provider>
  );
}

// Query best practices
const queryExamples = {
  // ✅ Prefer getByRole
  button: screen.getByRole('button', { name: /submit/i }),
  input: screen.getByRole('textbox', { name: /email/i }),
  checkbox: screen.getByRole('checkbox', { name: /agree/i }),
  
  // ✅ Use getByLabelText for forms
  emailInput: screen.getByLabelText(/email/i),
  
  // ✅ Use getByText for content
  heading: screen.getByText(/welcome/i),
  
  // ✅ Use getByTestId only when necessary
  customElement: screen.getByTestId('custom-element'),
  
  // ✅ Use queryBy* for asserting absence
  notExist: expect(screen.queryByText(/error/i)).not.toBeInTheDocument(),
  
  // ✅ Use findBy* for async elements
  asyncElement: await screen.findByText(/loaded/i),
};
```

---

#### **7. Installation & Setup**

```bash
# Install React Testing Library
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event

# For TypeScript
npm install --save-dev @types/jest
```

```typescript
// setupTests.ts
import '@testing-library/jest-dom';

// Custom matchers available:
// - toBeInTheDocument()
// - toHaveTextContent()
// - toHaveValue()
// - toBeDisabled()
// - toBeVisible()
// - and many more...
```

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "setupFilesAfterEnv": ["<rootDir>/setupTests.ts"],
    "testEnvironment": "jsdom"
  }
}
```

---

#### **Summary**

**Choose React Testing Library because:**

1. **User-centric**: Tests what users see and do
2. **Maintainable**: Tests don't break with refactoring
3. **Accessible**: Encourages accessible markup
4. **Realistic**: Simulates real user interactions
5. **Simple**: Easier to learn and use
6. **Active**: Officially recommended, actively maintained
7. **Confidence**: More confident tests

**Enzyme is deprecated** because:
- Maintenance mode (no active development)
- Requires adapters for each React version
- Tests implementation details
- Doesn't encourage accessibility
- Community moved to RTL

**Key Principle:**
> "Test how your software is used, not how it's implemented"

React Testing Library is the **industry standard** for testing React applications in 2024+.

</details>

---

### 132. How do you test React components?

<details>
<summary>View Answer</summary>

**Testing React components** involves **verifying behavior** from the **user's perspective** using **React Testing Library**. Focus on **what the component does**, not **how it does it**.

---

#### **1. Basic Component Test Structure**

```typescript
// Component to test
function Button({ onClick, children, disabled = false }) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}

// Test file: Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  it('renders with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });
  
  it('calls onClick when clicked', async () => {
    const handleClick = jest.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

---

#### **2. Testing Different Component Types**

**A. Presentational Components**

```typescript
// Component
interface UserCardProps {
  name: string;
  email: string;
  avatar: string;
  isOnline: boolean;
}

function UserCard({ name, email, avatar, isOnline }: UserCardProps) {
  return (
    <div data-testid="user-card">
      <img src={avatar} alt={`${name}'s avatar`} />
      <h2>{name}</h2>
      <p>{email}</p>
      {isOnline && <span className="status-online">Online</span>}
    </div>
  );
}

// Test
describe('UserCard', () => {
  const defaultProps = {
    name: 'John Doe',
    email: 'john@example.com',
    avatar: 'https://example.com/avatar.jpg',
    isOnline: false,
  };
  
  it('renders user information', () => {
    render(<UserCard {...defaultProps} />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.getByAltText("John Doe's avatar")).toHaveAttribute(
      'src',
      'https://example.com/avatar.jpg'
    );
  });
  
  it('shows online status when user is online', () => {
    render(<UserCard {...defaultProps} isOnline={true} />);
    expect(screen.getByText('Online')).toBeInTheDocument();
  });
  
  it('does not show online status when user is offline', () => {
    render(<UserCard {...defaultProps} isOnline={false} />);
    expect(screen.queryByText('Online')).not.toBeInTheDocument();
  });
});
```

**B. Components with State**

```typescript
// Component
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}

// Test
describe('Counter', () => {
  it('starts at 0', () => {
    render(<Counter />);
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });
  
  it('increments count', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    await user.click(screen.getByRole('button', { name: /increment/i }));
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
    
    await user.click(screen.getByRole('button', { name: /increment/i }));
    expect(screen.getByText('Count: 2')).toBeInTheDocument();
  });
  
  it('decrements count', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    await user.click(screen.getByRole('button', { name: /decrement/i }));
    expect(screen.getByText('Count: -1')).toBeInTheDocument();
  });
  
  it('resets count to 0', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    // Increment a few times
    await user.click(screen.getByRole('button', { name: /increment/i }));
    await user.click(screen.getByRole('button', { name: /increment/i }));
    expect(screen.getByText('Count: 2')).toBeInTheDocument();
    
    // Reset
    await user.click(screen.getByRole('button', { name: /reset/i }));
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });
});
```

**C. Components with Side Effects (useEffect)**

```typescript
// Component
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// Test
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/users/:userId', (req, res, ctx) => {
    return res(
      ctx.json({
        id: '123',
        name: 'John Doe',
        email: 'john@example.com',
      })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('UserProfile', () => {
  it('displays loading state initially', () => {
    render(<UserProfile userId="123" />);
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
  
  it('displays user data after loading', async () => {
    render(<UserProfile userId="123" />);
    
    // Wait for loading to finish
    expect(await screen.findByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
  });
  
  it('displays error message on failure', async () => {
    server.use(
      rest.get('/api/users/:userId', (req, res, ctx) => {
        return res(ctx.status(500), ctx.json({ message: 'Server error' }));
      })
    );
    
    render(<UserProfile userId="123" />);
    
    expect(await screen.findByText(/error/i)).toBeInTheDocument();
  });
  
  it('refetches when userId changes', async () => {
    const { rerender } = render(<UserProfile userId="123" />);
    
    expect(await screen.findByText('John Doe')).toBeInTheDocument();
    
    // Change userId
    server.use(
      rest.get('/api/users/:userId', (req, res, ctx) => {
        return res(
          ctx.json({
            id: '456',
            name: 'Jane Smith',
            email: 'jane@example.com',
          })
        );
      })
    );
    
    rerender(<UserProfile userId="456" />);
    
    expect(await screen.findByText('Jane Smith')).toBeInTheDocument();
  });
});
```

**D. Form Components**

```typescript
// Component
function LoginForm({ onSubmit }: { onSubmit: (data: any) => void }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({ email: '', password: '' });
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    const newErrors = { email: '', password: '' };
    
    if (!email) newErrors.email = 'Email is required';
    if (!password) newErrors.password = 'Password is required';
    if (password.length < 8) newErrors.password = 'Password must be at least 8 characters';
    
    setErrors(newErrors);
    
    if (!newErrors.email && !newErrors.password) {
      onSubmit({ email, password });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="email">Email</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
        {errors.email && <span role="alert">{errors.email}</span>}
      </div>
      
      <div>
        <label htmlFor="password">Password</label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        {errors.password && <span role="alert">{errors.password}</span>}
      </div>
      
      <button type="submit">Log in</button>
    </form>
  );
}

// Test
describe('LoginForm', () => {
  it('submits form with valid data', async () => {
    const handleSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /log in/i }));
    
    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'john@example.com',
      password: 'password123',
    });
  });
  
  it('shows validation errors for empty fields', async () => {
    const handleSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.click(screen.getByRole('button', { name: /log in/i }));
    
    expect(screen.getByText('Email is required')).toBeInTheDocument();
    expect(screen.getByText('Password is required')).toBeInTheDocument();
    expect(handleSubmit).not.toHaveBeenCalled();
  });
  
  it('shows error for short password', async () => {
    const handleSubmit = jest.fn();
    const user = userEvent.setup();
    
    render(<LoginForm onSubmit={handleSubmit} />);
    
    await user.type(screen.getByLabelText(/email/i), 'john@example.com');
    await user.type(screen.getByLabelText(/password/i), 'short');
    await user.click(screen.getByRole('button', { name: /log in/i }));
    
    expect(
      screen.getByText('Password must be at least 8 characters')
    ).toBeInTheDocument();
    expect(handleSubmit).not.toHaveBeenCalled();
  });
});
```

---

#### **3. Testing with Context**

```typescript
// Context
const ThemeContext = createContext({ theme: 'light', toggleTheme: () => {} });

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const toggleTheme = () => setTheme(theme === 'light' ? 'dark' : 'light');
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Component using context
function ThemeToggle() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <div>
      <p>Current theme: {theme}</p>
      <button onClick={toggleTheme}>Toggle theme</button>
    </div>
  );
}

// Test
function renderWithTheme(ui: React.ReactElement) {
  return render(<ThemeProvider>{ui}</ThemeProvider>);
}

describe('ThemeToggle', () => {
  it('displays current theme', () => {
    renderWithTheme(<ThemeToggle />);
    expect(screen.getByText('Current theme: light')).toBeInTheDocument();
  });
  
  it('toggles theme when button clicked', async () => {
    const user = userEvent.setup();
    renderWithTheme(<ThemeToggle />);
    
    expect(screen.getByText('Current theme: light')).toBeInTheDocument();
    
    await user.click(screen.getByRole('button', { name: /toggle/i }));
    
    expect(screen.getByText('Current theme: dark')).toBeInTheDocument();
  });
});
```

---

#### **4. Testing Async Components**

```typescript
// Component with async data
function SearchResults({ query }: { query: string }) {
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    if (!query) return;
    
    setLoading(true);
    fetch(`/api/search?q=${query}`)
      .then(res => res.json())
      .then(data => {
        setResults(data.results);
        setLoading(false);
      });
  }, [query]);
  
  if (loading) return <div>Loading...</div>;
  
  return (
    <ul>
      {results.map(result => (
        <li key={result.id}>{result.title}</li>
      ))}
    </ul>
  );
}

// Test
import { waitFor, waitForElementToBeRemoved } from '@testing-library/react';

describe('SearchResults', () => {
  it('displays search results', async () => {
    server.use(
      rest.get('/api/search', (req, res, ctx) => {
        return res(
          ctx.json({
            results: [
              { id: 1, title: 'Result 1' },
              { id: 2, title: 'Result 2' },
            ],
          })
        );
      })
    );
    
    render(<SearchResults query="react" />);
    
    // Using findBy (waits for element)
    expect(await screen.findByText('Result 1')).toBeInTheDocument();
    expect(screen.getByText('Result 2')).toBeInTheDocument();
  });
  
  it('shows loading state', async () => {
    render(<SearchResults query="react" />);
    
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    
    // Wait for loading to disappear
    await waitForElementToBeRemoved(() => screen.queryByText('Loading...'));
  });
  
  it('updates results when query changes', async () => {
    const { rerender } = render(<SearchResults query="react" />);
    
    expect(await screen.findByText('Result 1')).toBeInTheDocument();
    
    // Change query
    server.use(
      rest.get('/api/search', (req, res, ctx) => {
        return res(
          ctx.json({
            results: [{ id: 3, title: 'New Result' }],
          })
        );
      })
    );
    
    rerender(<SearchResults query="vue" />);
    
    expect(await screen.findByText('New Result')).toBeInTheDocument();
    expect(screen.queryByText('Result 1')).not.toBeInTheDocument();
  });
});
```

---

#### **5. Best Practices**

```typescript
const testingBestPractices = {
  queries: {
    do: [
      '✅ Use getByRole for accessibility',
      '✅ Use getByLabelText for form fields',
      '✅ Use getByText for content',
      '✅ Use queryBy* to assert absence',
      '✅ Use findBy* for async elements',
    ],
    dont: [
      '❌ Use getByTestId as first choice',
      '❌ Use container.querySelector',
      '❌ Test implementation details',
    ],
  },
  
  assertions: {
    do: [
      '✅ Assert on what user sees',
      '✅ Test behavior, not implementation',
      '✅ Test accessibility',
      '✅ Test error states',
      '✅ Test loading states',
    ],
    dont: [
      '❌ Assert on state',
      '❌ Assert on props',
      '❌ Assert on internal methods',
      '❌ Test styles (use visual regression)',
    ],
  },
  
  structure: {
    do: [
      '✅ One component per test file',
      '✅ Group related tests with describe()',
      '✅ Use descriptive test names',
      '✅ Arrange-Act-Assert pattern',
      '✅ Clean up after tests',
    ],
    dont: [
      '❌ Test multiple behaviors in one test',
      '❌ Share state between tests',
      '❌ Make tests depend on order',
    ],
  },
};

// Example: Good test structure
describe('ProductCard', () => {
  // Arrange: Setup
  const product = {
    id: '1',
    name: 'Product Name',
    price: 99.99,
    image: 'product.jpg',
  };
  
  it('displays product information', () => {
    // Arrange
    render(<ProductCard product={product} />);
    
    // Assert (no Act needed for display test)
    expect(screen.getByText('Product Name')).toBeInTheDocument();
    expect(screen.getByText('$99.99')).toBeInTheDocument();
  });
  
  it('adds product to cart when button clicked', async () => {
    // Arrange
    const handleAddToCart = jest.fn();
    const user = userEvent.setup();
    render(<ProductCard product={product} onAddToCart={handleAddToCart} />);
    
    // Act
    await user.click(screen.getByRole('button', { name: /add to cart/i }));
    
    // Assert
    expect(handleAddToCart).toHaveBeenCalledWith(product);
  });
});
```

---

#### **Summary**

**Testing Workflow:**

1. **Render** the component
2. **Query** for elements (preferably by role/label)
3. **Interact** with elements (userEvent)
4. **Assert** on the output

**Key Principles:**
- Test behavior, not implementation
- Test from user's perspective
- Make tests resilient to refactoring
- Cover happy path and edge cases
- Test loading, error, and empty states
- Ensure accessibility

**Common Patterns:**
- Presentational: Test rendering
- Stateful: Test interactions and state changes
- Forms: Test validation and submission
- Async: Test loading and data display
- Context: Test with providers

Good tests give **confidence** that your components work as users expect.

</details>

---

### 133. What is the difference between render and screen in Testing Library?

<details>
<summary>View Answer</summary>

**`render`** and **`screen`** are two fundamental APIs in React Testing Library with different purposes: `render` **mounts components**, while `screen` **queries the DOM**.

---

#### **1. Core Difference**

```typescript
import { render, screen } from '@testing-library/react';

// render() - Mounts the component and returns utilities
const result = render(<MyComponent />);

// screen - Global object for querying the entire document
const element = screen.getByText('Hello');

const comparison = {
  render: {
    purpose: 'Mount React component to the DOM',
    returns: 'Object with queries and utilities',
    scope: 'Limited to rendered component',
    usage: 'Called once per test',
  },
  
  screen: {
    purpose: 'Query elements in the document',
    returns: 'N/A (contains query methods)',
    scope: 'Entire document.body',
    usage: 'Used for all queries',
  },
};
```

---

#### **2. render() Function**

**What it does:**

```typescript
// render() mounts the component and returns utilities
const renderResult = render(<Counter />);

// Returns an object with:
const returnValue = {
  // Container element
  container: HTMLElement,          // The DOM node where component is mounted
  baseElement: HTMLElement,        // Usually document.body
  
  // Query methods (same as screen)
  getByText: Function,
  getByRole: Function,
  queryByText: Function,
  findByText: Function,
  // ... all query methods
  
  // Utilities
  rerender: Function,              // Re-render with new props
  unmount: Function,               // Unmount the component
  asFragment: Function,            // Get snapshot
  debug: Function,                 // Print DOM tree
};
```

**Common Usage:**

```typescript
// Basic render
render(<MyComponent />);

// Render with props
render(<UserCard name="John" email="john@example.com" />);

// Render with providers
render(
  <Provider store={store}>
    <Router>
      <MyComponent />
    </Router>
  </Provider>
);

// Destructure utilities you need
const { rerender, unmount } = render(<Counter initialCount={0} />);

// Rerender with new props
rerender(<Counter initialCount={5} />);

// Manually unmount
unmount();
```

---

#### **3. screen Object**

**What it is:**

```typescript
// screen is a global object with query methods
import { screen } from '@testing-library/react';

// All queries are available on screen
screen.getByRole('button');
screen.getByText('Hello');
screen.getByLabelText('Email');
screen.queryByText('Optional');
screen.findByText('Async content');

// screen queries the entire document.body
// It's equivalent to:
const { getByText } = render(<Component />);
// vs
render(<Component />);
screen.getByText('text');
```

**Why screen exists:**

```typescript
const reasons = {
  readability: {
    without: `
const { getByText, getByRole, getByLabelText } = render(<Form />);
const title = getByText('Title');
const button = getByRole('button');
const input = getByLabelText('Email');
    `,
    with: `
render(<Form />);
const title = screen.getByText('Title');
const button = screen.getByRole('button');
const input = screen.getByLabelText('Email');
    `,
  },
  
  consistency: 'Always use screen - no need to destructure',
  simplicity: 'One way to query - less cognitive load',
  eslintRule: 'prefer-screen-queries - enforces screen usage',
};
```

---

#### **4. Practical Comparison**

**Old Way (without screen):**

```typescript
// ❌ Old approach - destructuring from render()
it('displays user information', () => {
  const { getByText, getByRole, getByLabelText } = render(<UserForm />);
  
  expect(getByText('User Profile')).toBeInTheDocument();
  expect(getByRole('button', { name: /submit/i })).toBeInTheDocument();
  expect(getByLabelText('Email')).toBeInTheDocument();
});

// Problems:
// - Verbose destructuring
// - Need to update destructuring when adding queries
// - Inconsistent (sometimes you destructure, sometimes you don't)
```

**Modern Way (with screen):**

```typescript
// ✅ Modern approach - using screen
it('displays user information', () => {
  render(<UserForm />);
  
  expect(screen.getByText('User Profile')).toBeInTheDocument();
  expect(screen.getByRole('button', { name: /submit/i })).toBeInTheDocument();
  expect(screen.getByLabelText('Email')).toBeInTheDocument();
});

// Benefits:
// - Clean and consistent
// - No destructuring needed
// - Easy to add more queries
// - Recommended by React Testing Library
```

---

#### **5. When to Use Each**

```typescript
const usageGuide = {
  useScreen: {
    scenarios: [
      'Querying elements (99% of the time)',
      'Multiple queries in the same test',
      'Following React Testing Library recommendations',
    ],
    example: `
render(<Component />);
screen.getByRole('button');
screen.getByText('Hello');
    `,
  },
  
  useRenderReturn: {
    scenarios: [
      'Need to rerender with different props',
      'Need to unmount component',
      'Need container for specific DOM operations',
      'Need to debug the component tree',
    ],
    example: `
const { rerender, unmount, container, debug } = render(<Component />);
rerender(<Component prop="new" />);
container.querySelector('.specific-class'); // Rarely needed
debug(); // Print DOM tree
    `,
  },
};
```

**Examples:**

```typescript
// Example 1: Use screen for queries
it('renders correctly', () => {
  render(<Welcome name="John" />);
  
  // ✅ Use screen for all queries
  expect(screen.getByText('Welcome, John!')).toBeInTheDocument();
  expect(screen.getByRole('button')).toBeInTheDocument();
});

// Example 2: Use render return for rerender
it('updates when props change', () => {
  const { rerender } = render(<Counter count={0} />);
  
  expect(screen.getByText('Count: 0')).toBeInTheDocument();
  
  // Need rerender
  rerender(<Counter count={5} />);
  
  expect(screen.getByText('Count: 5')).toBeInTheDocument();
});

// Example 3: Use render return for unmount
it('cleans up on unmount', () => {
  const cleanup = jest.fn();
  const { unmount } = render(<ComponentWithCleanup onCleanup={cleanup} />);
  
  // Manually unmount to test cleanup
  unmount();
  
  expect(cleanup).toHaveBeenCalled();
});

// Example 4: Use render return for debug
it('debugs component', () => {
  const { debug } = render(<ComplexComponent />);
  
  // Print current DOM tree to console
  debug();
  
  // Or debug specific element
  debug(screen.getByRole('form'));
});
```

---

#### **6. Advanced Patterns**

**Container Access (Rarely Needed):**

```typescript
// ⚠️ Use sparingly - usually screen is better
it('accesses container for specific needs', () => {
  const { container } = render(<Component />);
  
  // Example: Check if container has specific class
  expect(container.firstChild).toHaveClass('wrapper');
  
  // Example: Use querySelector when no better option
  const svgElement = container.querySelector('svg');
  expect(svgElement).toBeInTheDocument();
  
  // Note: Prefer screen queries when possible!
});
```

**Custom Render Function:**

```typescript
// Create custom render that includes providers
import { render as rtlRender } from '@testing-library/react';

function render(ui: React.ReactElement, options = {}) {
  const Wrapper = ({ children }) => (
    <Provider store={store}>
      <Router>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </Router>
    </Provider>
  );
  
  return rtlRender(ui, { wrapper: Wrapper, ...options });
}

// Usage - same as before
it('works with custom render', () => {
  render(<MyComponent />);
  
  // Still use screen for queries
  expect(screen.getByText('Hello')).toBeInTheDocument();
});

// Can still access render return values
it('rerenders with custom render', () => {
  const { rerender } = render(<Counter count={0} />);
  rerender(<Counter count={5} />);
  expect(screen.getByText('Count: 5')).toBeInTheDocument();
});
```

---

#### **7. Common Mistakes**

```typescript
// ❌ Mistake 1: Destructuring everything from render
const { getByText, getByRole, queryByText, findByText } = render(<Component />);
expect(getByText('Hello')).toBeInTheDocument();

// ✅ Fix: Use screen
render(<Component />);
expect(screen.getByText('Hello')).toBeInTheDocument();

// ❌ Mistake 2: Mixing destructured and screen queries
const { getByText } = render(<Component />);
expect(getByText('Title')).toBeInTheDocument();
expect(screen.getByRole('button')).toBeInTheDocument(); // Inconsistent!

// ✅ Fix: Use screen consistently
render(<Component />);
expect(screen.getByText('Title')).toBeInTheDocument();
expect(screen.getByRole('button')).toBeInTheDocument();

// ❌ Mistake 3: Storing render result when not needed
const renderResult = render(<Component />);
expect(renderResult.getByText('Hello')).toBeInTheDocument();

// ✅ Fix: Just use screen
render(<Component />);
expect(screen.getByText('Hello')).toBeInTheDocument();

// ✅ Exception: When you actually need render utilities
const { rerender, unmount } = render(<Component />);
rerender(<Component prop="new" />); // Actually using rerender
```

---

#### **8. screen.debug() Helper**

```typescript
// screen also has debug method
import { screen } from '@testing-library/react';

it('debugs the DOM', () => {
  render(<Component />);
  
  // Print entire document.body
  screen.debug();
  
  // Print specific element
  screen.debug(screen.getByRole('button'));
  
  // Limit output length
  screen.debug(undefined, 10000); // Max 10000 characters
});

// Output:
// <body>
//   <div>
//     <h1>Title</h1>
//     <button>Click me</button>
//   </div>
// </body>
```

---

#### **9. ESLint Rule**

```json
// .eslintrc.json
{
  "extends": [
    "plugin:testing-library/react"
  ],
  "rules": {
    "testing-library/prefer-screen-queries": "error"
  }
}

// This rule enforces using screen over destructured queries
// ❌ Error: const { getByText } = render(<Component />);
// ✅ Pass: render(<Component />); screen.getByText(...);
```

---

#### **Summary**

**render():**
- **Purpose**: Mount component to DOM
- **Returns**: Object with queries + utilities (rerender, unmount, etc.)
- **Use when**: You need rerender, unmount, or container access

**screen:**
- **Purpose**: Query elements in the DOM
- **Contains**: All query methods (getBy*, queryBy*, findBy*)
- **Use when**: Querying elements (almost always)

**Best Practice:**
```typescript
// ✅ Modern recommended pattern
import { render, screen } from '@testing-library/react';

it('test name', async () => {
  // 1. Render component
  render(<Component />);
  
  // 2. Query with screen
  const button = screen.getByRole('button');
  
  // 3. Interact
  await user.click(button);
  
  // 4. Assert with screen
  expect(screen.getByText('Clicked')).toBeInTheDocument();
});

// Only destructure when you need specific utilities
it('test with rerender', () => {
  const { rerender } = render(<Component prop="old" />);
  rerender(<Component prop="new" />);
  expect(screen.getByText('new')).toBeInTheDocument();
});
```

**Key Takeaway:**
> Use **`screen`** for all queries. Only destructure **`render()`** when you need `rerender`, `unmount`, or `container`.

</details>

---

### 134. How do you test hooks with React Testing Library?

<details>
<summary>View Answer</summary>

**Testing hooks** in React requires the **`renderHook`** utility from React Testing Library. You test hooks by **observing their behavior** through their return values and effects.

---

#### **1. The renderHook API**

```typescript
import { renderHook, waitFor } from '@testing-library/react';

// Basic usage
const { result } = renderHook(() => useCustomHook());

const renderHookAPI = {
  purpose: 'Render a hook in a test component',
  
  returns: {
    result: {
      current: 'Current value returned by the hook',
      description: 'Access hook return value via result.current',
    },
    rerender: 'Function to rerender with new props',
    unmount: 'Function to unmount the test component',
  },
  
  usage: 'Test hooks that use React features (state, effects, context)',
};
```

---

#### **2. Testing Simple Hooks**

**Example: Counter Hook**

```typescript
// useCounter.ts
import { useState } from 'react';

export function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = () => setCount(c => c + 1);
  const decrement = () => setCount(c => c - 1);
  const reset = () => setCount(initialValue);
  
  return { count, increment, decrement, reset };
}

// useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    
    expect(result.current.count).toBe(0);
  });
  
  it('initializes with custom value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    expect(result.current.count).toBe(10);
  });
  
  it('increments count', () => {
    const { result } = renderHook(() => useCounter());
    
    // Wrap state updates in act()
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });
  
  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
});
```

---

#### **3. Testing Hooks with Dependencies**

```typescript
// useDebounce.ts
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// useDebounce.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { useDebounce } from './useDebounce';

jest.useFakeTimers();

describe('useDebounce', () => {
  it('returns initial value immediately', () => {
    const { result } = renderHook(() => useDebounce('initial', 500));
    
    expect(result.current).toBe('initial');
  });
  
  it('debounces value changes', () => {
    const { result, rerender } = renderHook(
      ({ value, delay }) => useDebounce(value, delay),
      { initialProps: { value: 'first', delay: 500 } }
    );
    
    expect(result.current).toBe('first');
    
    // Change value
    rerender({ value: 'second', delay: 500 });
    
    // Value hasn't changed yet
    expect(result.current).toBe('first');
    
    // Fast-forward time
    jest.advanceTimersByTime(500);
    
    // Now value should be updated
    expect(result.current).toBe('second');
  });
  
  it('cancels previous timeout on rapid changes', () => {
    const { result, rerender } = renderHook(
      ({ value }) => useDebounce(value, 500),
      { initialProps: { value: 'first' } }
    );
    
    rerender({ value: 'second' });
    jest.advanceTimersByTime(250);
    
    rerender({ value: 'third' });
    jest.advanceTimersByTime(250);
    
    // Should still be 'first' (timeouts were cancelled)
    expect(result.current).toBe('first');
    
    jest.advanceTimersByTime(250);
    
    // Now should be 'third'
    expect(result.current).toBe('third');
  });
});
```

---

#### **4. Testing Async Hooks**

```typescript
// useApi.ts
import { useState, useEffect } from 'react';

export function useApi<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    let cancelled = false;
    
    setLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(data => {
        if (!cancelled) {
          setData(data);
          setLoading(false);
        }
      })
      .catch(err => {
        if (!cancelled) {
          setError(err);
          setLoading(false);
        }
      });
    
    return () => {
      cancelled = true;
    };
  }, [url]);
  
  return { data, loading, error };
}

// useApi.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { useApi } from './useApi';
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(ctx.json({ name: 'John Doe' }));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('useApi', () => {
  it('starts in loading state', () => {
    const { result } = renderHook(() => useApi('/api/users'));
    
    expect(result.current.loading).toBe(true);
    expect(result.current.data).toBe(null);
    expect(result.current.error).toBe(null);
  });
  
  it('fetches data successfully', async () => {
    const { result } = renderHook(() => useApi('/api/users'));
    
    // Wait for loading to finish
    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });
    
    expect(result.current.data).toEqual({ name: 'John Doe' });
    expect(result.current.error).toBe(null);
  });
  
  it('handles errors', async () => {
    server.use(
      rest.get('/api/users', (req, res, ctx) => {
        return res(ctx.status(500), ctx.json({ message: 'Server error' }));
      })
    );
    
    const { result } = renderHook(() => useApi('/api/users'));
    
    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });
    
    expect(result.current.data).toBe(null);
    expect(result.current.error).toBeTruthy();
  });
  
  it('refetches when URL changes', async () => {
    const { result, rerender } = renderHook(
      ({ url }) => useApi(url),
      { initialProps: { url: '/api/users' } }
    );
    
    await waitFor(() => {
      expect(result.current.data).toEqual({ name: 'John Doe' });
    });
    
    // Mock different response for new URL
    server.use(
      rest.get('/api/posts', (req, res, ctx) => {
        return res(ctx.json({ title: 'Post Title' }));
      })
    );
    
    // Change URL
    rerender({ url: '/api/posts' });
    
    await waitFor(() => {
      expect(result.current.data).toEqual({ title: 'Post Title' });
    });
  });
});
```

---

#### **5. Testing Hooks with Context**

```typescript
// AuthContext.tsx
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const login = async (email, password) => {
    const user = await loginAPI(email, password);
    setUser(user);
  };
  
  const logout = () => setUser(null);
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
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

// useAuth.test.tsx
import { renderHook, act, waitFor } from '@testing-library/react';
import { AuthProvider, useAuth } from './AuthContext';

describe('useAuth', () => {
  it('throws error when used outside provider', () => {
    // Suppress console.error for this test
    const spy = jest.spyOn(console, 'error').mockImplementation();
    
    expect(() => {
      renderHook(() => useAuth());
    }).toThrow('useAuth must be used within AuthProvider');
    
    spy.mockRestore();
  });
  
  it('provides user and auth methods', () => {
    const wrapper = ({ children }) => <AuthProvider>{children}</AuthProvider>;
    
    const { result } = renderHook(() => useAuth(), { wrapper });
    
    expect(result.current.user).toBe(null);
    expect(typeof result.current.login).toBe('function');
    expect(typeof result.current.logout).toBe('function');
  });
  
  it('logs in user', async () => {
    const wrapper = ({ children }) => <AuthProvider>{children}</AuthProvider>;
    
    const { result } = renderHook(() => useAuth(), { wrapper });
    
    await act(async () => {
      await result.current.login('test@example.com', 'password');
    });
    
    expect(result.current.user).toEqual({
      email: 'test@example.com',
      name: 'Test User',
    });
  });
  
  it('logs out user', async () => {
    const wrapper = ({ children }) => <AuthProvider>{children}</AuthProvider>;
    
    const { result } = renderHook(() => useAuth(), { wrapper });
    
    // Login first
    await act(async () => {
      await result.current.login('test@example.com', 'password');
    });
    
    expect(result.current.user).toBeTruthy();
    
    // Logout
    act(() => {
      result.current.logout();
    });
    
    expect(result.current.user).toBe(null);
  });
});
```

---

#### **6. Testing Hooks with React Query**

```typescript
// useUser.ts
import { useQuery } from '@tanstack/react-query';

export function useUser(userId: string) {
  return useQuery(['user', userId], () =>
    fetch(`/api/users/${userId}`).then(res => res.json())
  );
}

// useUser.test.ts
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useUser } from './useUser';

function createWrapper() {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false, // Don't retry in tests
      },
    },
  });
  
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}

describe('useUser', () => {
  it('fetches user data', async () => {
    const { result } = renderHook(() => useUser('123'), {
      wrapper: createWrapper(),
    });
    
    expect(result.current.isLoading).toBe(true);
    
    await waitFor(() => {
      expect(result.current.isSuccess).toBe(true);
    });
    
    expect(result.current.data).toEqual({
      id: '123',
      name: 'John Doe',
    });
  });
});
```

---

#### **7. The act() Warning**

```typescript
// act() is needed for state updates

// ❌ Warning: An update to TestComponent inside a test was not wrapped in act(...)
it('increments count', () => {
  const { result } = renderHook(() => useCounter());
  
  result.current.increment(); // ⚠️ Not wrapped in act()!
  
  expect(result.current.count).toBe(1);
});

// ✅ Correct: Wrap state updates in act()
it('increments count', () => {
  const { result } = renderHook(() => useCounter());
  
  act(() => {
    result.current.increment();
  });
  
  expect(result.current.count).toBe(1);
});

// ✅ Async act() for promises
it('logs in user', async () => {
  const { result } = renderHook(() => useAuth());
  
  await act(async () => {
    await result.current.login('email', 'password');
  });
  
  expect(result.current.user).toBeTruthy();
});

// Note: act() is usually not needed with waitFor()
it('loads data', async () => {
  const { result } = renderHook(() => useApi('/api/data'));
  
  // waitFor handles act() internally
  await waitFor(() => {
    expect(result.current.loading).toBe(false);
  });
});
```

---

#### **8. Testing Custom Hook Edge Cases**

```typescript
// useLocalStorage.ts
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });
  
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Failed to save to localStorage:', error);
    }
  }, [key, value]);
  
  return [value, setValue] as const;
}

// useLocalStorage.test.ts
describe('useLocalStorage', () => {
  beforeEach(() => {
    window.localStorage.clear();
  });
  
  it('uses initial value when localStorage is empty', () => {
    const { result } = renderHook(() => useLocalStorage('test', 'initial'));
    
    expect(result.current[0]).toBe('initial');
  });
  
  it('reads from localStorage on mount', () => {
    window.localStorage.setItem('test', JSON.stringify('stored'));
    
    const { result } = renderHook(() => useLocalStorage('test', 'initial'));
    
    expect(result.current[0]).toBe('stored');
  });
  
  it('updates localStorage when value changes', () => {
    const { result } = renderHook(() => useLocalStorage('test', 'initial'));
    
    act(() => {
      result.current[1]('updated');
    });
    
    expect(window.localStorage.getItem('test')).toBe(JSON.stringify('updated'));
  });
  
  it('handles localStorage errors gracefully', () => {
    // Mock localStorage to throw error
    const spy = jest.spyOn(Storage.prototype, 'setItem').mockImplementation(() => {
      throw new Error('QuotaExceededError');
    });
    
    const { result } = renderHook(() => useLocalStorage('test', 'initial'));
    
    // Should not throw
    act(() => {
      result.current[1]('updated');
    });
    
    spy.mockRestore();
  });
});
```

---

#### **9. Best Practices**

```typescript
const hookTestingBestPractices = {
  do: [
    '✅ Use renderHook for hooks that use React features',
    '✅ Wrap state updates in act()',
    '✅ Use waitFor for async operations',
    '✅ Test hook behavior, not implementation',
    '✅ Provide necessary context/providers',
    '✅ Test edge cases and error states',
    '✅ Clean up after tests (timers, localStorage, etc.)',
  ],
  
  dont: [
    '❌ Test pure utility functions with renderHook',
    '❌ Test hooks by rendering full components (usually)',
    '❌ Access hook internals',
    '❌ Forget to wrap state updates in act()',
    '❌ Test implementation details',
  ],
};

// ❌ Don't use renderHook for pure functions
function add(a: number, b: number) {
  return a + b;
}

// Wrong:
it('adds numbers', () => {
  const { result } = renderHook(() => add(2, 3));
  expect(result.current).toBe(5);
});

// Right:
it('adds numbers', () => {
  expect(add(2, 3)).toBe(5);
});

// ✅ Use renderHook for hooks that use React features
function useDouble(value: number) {
  return useMemo(() => value * 2, [value]);
}

// Correct:
it('doubles value', () => {
  const { result } = renderHook(() => useDouble(5));
  expect(result.current).toBe(10);
});
```

---

#### **Summary**

**Testing Hooks Workflow:**

1. **Use renderHook** to render the hook in a test component
2. **Access return value** via `result.current`
3. **Wrap state updates** in `act()`
4. **Use waitFor** for async operations
5. **Provide wrappers** for context/providers if needed

**Key APIs:**
```typescript
// Render hook
const { result, rerender, unmount } = renderHook(() => useHook());

// Access return value
const value = result.current;

// Update state
act(() => {
  result.current.updateFunction();
});

// Async updates
await act(async () => {
  await result.current.asyncFunction();
});

// Wait for async state
await waitFor(() => {
  expect(result.current.loading).toBe(false);
});

// Rerender with new props
rerender({ newProp: 'value' });

// Provide context
const { result } = renderHook(() => useHook(), {
  wrapper: ({ children }) => <Provider>{children}</Provider>,
});
```

**Remember**: Test **behavior**, not **implementation**. Focus on what the hook returns and how it responds to inputs, not how it works internally.

</details>

---

### 135. What is Jest and how does it work with React?

<details>
<summary>View Answer</summary>

**Jest** is a delightful JavaScript testing framework created by Meta (Facebook) that works great with React. It provides a complete testing solution with zero configuration for most React projects.

#### 1. What is Jest?

**Core Features:**
```typescript
// Jest provides:
// 1. Test Runner - Executes test files
// 2. Assertion Library - expect() with matchers
// 3. Mocking System - Mock functions, modules, timers
// 4. Code Coverage - Built-in coverage reporting
// 5. Snapshot Testing - Component output comparison
// 6. Watch Mode - Interactive test development
```

**Philosophy:**
- **Zero Configuration**: Works out of the box for most projects
- **Isolated Tests**: Each test runs in its own environment
- **Fast & Parallel**: Tests run in parallel by default
- **Rich Matchers**: Extensive built-in assertions
- **Great Error Messages**: Clear, helpful failure messages

#### 2. How Jest Works with React

**Test Execution Flow:**
```typescript
// 1. Jest finds test files
// Pattern: *.test.js, *.spec.js, __tests__/*.js

// 2. Sets up test environment
// - jsdom for DOM simulation
// - Babel for JSX transformation
// - Mock implementations

// 3. Runs tests in isolation
// Each test gets fresh module registry

// 4. Reports results
// Pass/fail with coverage data
```

**Basic Test Structure:**
```typescript
// UserProfile.test.tsx
import { render, screen } from '@testing-library/react';
import UserProfile from './UserProfile';

// describe: Group related tests
describe('UserProfile', () => {
  // it/test: Individual test case
  it('renders user name', () => {
    // Arrange: Setup test data
    const user = { name: 'John Doe', email: 'john@example.com' };
    
    // Act: Render component
    render(<UserProfile user={user} />);
    
    // Assert: Verify behavior
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });

  test('displays email when provided', () => {
    const user = { name: 'Jane', email: 'jane@example.com' };
    render(<UserProfile user={user} />);
    
    expect(screen.getByText(/jane@example.com/i)).toBeInTheDocument();
  });
});
```

#### 3. Jest Configuration for React

**Create React App (Zero Config):**
```json
// package.json
{
  "scripts": {
    "test": "react-scripts test",
    "test:coverage": "react-scripts test --coverage --watchAll=false"
  }
}

// CRA includes:
// - Jest pre-configured
// - jsdom environment
// - @testing-library/react
// - @testing-library/jest-dom
```

**Custom Configuration:**
```javascript
// jest.config.js
module.exports = {
  // Test environment
  testEnvironment: 'jsdom',
  
  // Setup files
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.ts'],
  
  // Transform files
  transform: {
    '^.+\\.(ts|tsx|js|jsx)$': ['babel-jest', {
      presets: [
        ['@babel/preset-env', { targets: { node: 'current' } }],
        ['@babel/preset-react', { runtime: 'automatic' }],
        '@babel/preset-typescript'
      ]
    }]
  },
  
  // Module name mapping
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '\\.(jpg|jpeg|png|gif|svg)$': '<rootDir>/__mocks__/fileMock.js',
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  
  // Test match patterns
  testMatch: [
    '**/__tests__/**/*.[jt]s?(x)',
    '**/?(*.)+(spec|test).[jt]s?(x)'
  ],
  
  // Coverage
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
    '!src/reportWebVitals.ts'
  ],
  
  // Coverage thresholds
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
};
```

**Setup File:**
```typescript
// src/setupTests.ts
import '@testing-library/jest-dom';
import { server } from './mocks/server';

// MSW server setup
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  takeRecords() { return []; }
  unobserve() {}
};
```

#### 4. Jest Matchers

**Common Matchers:**
```typescript
// Equality
expect(value).toBe(4);                    // Strict equality (===)
expect(obj).toEqual({ name: 'John' });    // Deep equality
expect(obj).toStrictEqual({ name: 'John' }); // Strict deep equality

// Truthiness
expect(value).toBeTruthy();               // Boolean true
expect(value).toBeFalsy();                // Boolean false
expect(value).toBeNull();                 // null
expect(value).toBeUndefined();            // undefined
expect(value).toBeDefined();              // not undefined

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeGreaterThanOrEqual(3.5);
expect(value).toBeLessThan(5);
expect(value).toBeLessThanOrEqual(4.5);
expect(0.1 + 0.2).toBeCloseTo(0.3);      // Floating point

// Strings
expect('team').toMatch(/ea/);
expect('team').toContain('ea');

// Arrays & Iterables
expect(['apple', 'banana']).toContain('apple');
expect([1, 2, 3]).toHaveLength(3);
expect(new Set([1, 2])).toContainEqual(2);

// Objects
expect({ name: 'John', age: 30 }).toHaveProperty('name');
expect({ name: 'John', age: 30 }).toHaveProperty('name', 'John');
expect(obj).toMatchObject({ name: 'John' }); // Partial match

// Exceptions
expect(() => functionThatThrows()).toThrow();
expect(() => functionThatThrows()).toThrow(Error);
expect(() => functionThatThrows()).toThrow('error message');
expect(() => functionThatThrows()).toThrow(/error/);

// Promises
await expect(promise).resolves.toBe('value');
await expect(promise).rejects.toThrow(Error);
```

**React Testing Library Matchers (jest-dom):**
```typescript
import '@testing-library/jest-dom';

// DOM matchers
expect(element).toBeInTheDocument();
expect(element).toBeVisible();
expect(element).toBeEmpty();
expect(element).toBeDisabled();
expect(element).toBeEnabled();
expect(element).toBeRequired();
expect(element).toHaveTextContent('text');
expect(element).toHaveValue('value');
expect(element).toHaveAttribute('href', '/path');
expect(element).toHaveClass('active');
expect(element).toHaveStyle({ color: 'red' });
expect(element).toHaveFocus();

// Form matchers
expect(input).toBeChecked();
expect(input).toBePartiallyChecked();
expect(select).toHaveDisplayValue('Option 1');
expect(form).toHaveFormValues({ username: 'john' });
```

#### 5. Mocking in Jest

**Mock Functions:**
```typescript
// Create mock function
const mockFn = jest.fn();

// Mock implementation
const mockFn = jest.fn((x) => x * 2);
mockFn(2); // Returns 4

// Mock return value
const mockFn = jest.fn();
mockFn.mockReturnValue(42);
mockFn.mockReturnValueOnce(1).mockReturnValueOnce(2);

// Mock resolved/rejected promises
const mockFn = jest.fn();
mockFn.mockResolvedValue({ data: 'success' });
mockFn.mockRejectedValue(new Error('Failed'));

// Test mock calls
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledTimes(2);
expect(mockFn).toHaveBeenCalledWith('arg1', 'arg2');
expect(mockFn).toHaveBeenLastCalledWith('last');

// Access mock data
expect(mockFn.mock.calls).toEqual([['arg1'], ['arg2']]);
expect(mockFn.mock.results[0].value).toBe(42);
```

**Example with Component:**
```typescript
// Button.test.tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Button from './Button';

test('calls onClick when clicked', async () => {
  const handleClick = jest.fn();
  const user = userEvent.setup();
  
  render(<Button onClick={handleClick}>Click me</Button>);
  
  await user.click(screen.getByRole('button'));
  
  expect(handleClick).toHaveBeenCalledTimes(1);
});

test('calls onClick with correct arguments', async () => {
  const handleClick = jest.fn();
  const user = userEvent.setup();
  
  render(<Button onClick={handleClick} id="submit">Submit</Button>);
  
  const button = screen.getByRole('button');
  await user.click(button);
  
  expect(handleClick).toHaveBeenCalledWith(
    expect.objectContaining({
      type: 'click',
      target: expect.any(HTMLButtonElement)
    })
  );
});
```

**Mocking Modules:**
```typescript
// Mock entire module
jest.mock('./api');
import * as api from './api';

test('fetches user data', async () => {
  // Type-safe mock
  (api.fetchUser as jest.Mock).mockResolvedValue({
    id: 1,
    name: 'John'
  });
  
  const user = await api.fetchUser(1);
  expect(user.name).toBe('John');
});

// Partial module mock
jest.mock('./utils', () => ({
  ...jest.requireActual('./utils'),
  formatDate: jest.fn(() => '2024-01-01')
}));

// Mock with factory
jest.mock('axios', () => ({
  default: {
    get: jest.fn(),
    post: jest.fn()
  }
}));
```

**Spying on Methods:**
```typescript
// Spy on object method
const user = { getName: () => 'John' };
const spy = jest.spyOn(user, 'getName');

user.getName();

expect(spy).toHaveBeenCalled();
spy.mockRestore(); // Restore original implementation

// Spy on console
const consoleSpy = jest.spyOn(console, 'error').mockImplementation();

// Code that logs error

expect(consoleSpy).toHaveBeenCalledWith('Error message');
consoleSpy.mockRestore();
```

**Mock Timers:**
```typescript
test('delays execution', () => {
  jest.useFakeTimers();
  
  const callback = jest.fn();
  setTimeout(callback, 1000);
  
  // Fast-forward time
  jest.advanceTimersByTime(1000);
  
  expect(callback).toHaveBeenCalled();
  
  jest.useRealTimers();
});

test('debounces input', async () => {
  jest.useFakeTimers();
  
  const onChange = jest.fn();
  const user = userEvent.setup({ delay: null });
  
  render(<DebouncedInput onChange={onChange} delay={500} />);
  
  const input = screen.getByRole('textbox');
  await user.type(input, 'test');
  
  // Should not be called yet
  expect(onChange).not.toHaveBeenCalled();
  
  // Fast-forward past debounce
  jest.advanceTimersByTime(500);
  
  expect(onChange).toHaveBeenCalledWith('test');
  
  jest.useRealTimers();
});
```

#### 6. Snapshot Testing

**Basic Snapshot:**
```typescript
// Button.test.tsx
import { render } from '@testing-library/react';
import Button from './Button';

test('matches snapshot', () => {
  const { container } = render(<Button>Click me</Button>);
  expect(container.firstChild).toMatchSnapshot();
});

// Generated snapshot file: Button.test.tsx.snap
// exports[`matches snapshot 1`] = `
// <button
//   class="btn"
//   type="button"
// >
//   Click me
// </button>
// `;
```

**Inline Snapshots:**
```typescript
test('renders correctly', () => {
  const { container } = render(<Badge count={5} />);
  expect(container.firstChild).toMatchInlineSnapshot(`
    <span class="badge">
      5
    </span>
  `);
});
```

**Property Matchers:**
```typescript
test('user object snapshot', () => {
  const user = {
    id: Math.random(),
    createdAt: new Date(),
    name: 'John'
  };
  
  expect(user).toMatchSnapshot({
    id: expect.any(Number),
    createdAt: expect.any(Date)
  });
});
```

**When to Use Snapshots:**
```typescript
// ✅ Good use cases
// - Testing static component output
// - Ensuring UI doesn't change unexpectedly
// - Testing error messages
// - Testing serializable data structures

test('error message', () => {
  const error = new ValidationError('Invalid email');
  expect(error.message).toMatchInlineSnapshot(`"Invalid email"`);
});

// ❌ Avoid for
// - Frequently changing components
// - Large component trees (hard to review)
// - Dynamic data (use property matchers)
// - Replace proper assertions
```

#### 7. Watch Mode

**Interactive Commands:**
```bash
# Run tests in watch mode
npm test

# Available commands:
# › Press f to run only failed tests.
# › Press o to only run tests related to changed files.
# › Press p to filter by a filename regex pattern.
# › Press t to filter by a test name regex pattern.
# › Press q to quit watch mode.
# › Press a to run all tests.
# › Press Enter to trigger a test run.
```

**Watch Plugins:**
```javascript
// jest.config.js
module.exports = {
  watchPlugins: [
    'jest-watch-typeahead/filename',
    'jest-watch-typeahead/testname',
    'jest-watch-select-projects'
  ]
};
```

#### 8. Code Coverage

**Running Coverage:**
```bash
# Generate coverage report
npm test -- --coverage --watchAll=false

# Coverage with specific files
npm test -- --coverage --collectCoverageFrom="src/components/**/*.{ts,tsx}"

# Output formats
npm test -- --coverage --coverageReporters=text
npm test -- --coverage --coverageReporters=html
npm test -- --coverage --coverageReporters=lcov # For CI/CD
```

**Coverage Report:**
```
--------------------|---------|----------|---------|---------|-------------------
File                | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
--------------------|---------|----------|---------|---------|-------------------
All files           |   85.71 |    83.33 |   88.89 |   85.71 |
Button.tsx          |     100 |      100 |     100 |     100 |
UserProfile.tsx     |   71.43 |    66.67 |   77.78 |   71.43 | 23-25,45
--------------------|---------|----------|---------|---------|-------------------
```

**Coverage Configuration:**
```javascript
// jest.config.js
module.exports = {
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
    '!src/**/*.stories.tsx',
    '!src/**/__tests__/**'
  ],
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    },
    './src/components/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90
    }
  },
  coveragePathIgnorePatterns: [
    '/node_modules/',
    '/build/',
    '/coverage/'
  ]
};
```

#### 9. Integration with Build Tools

**Create React App:**
```json
// package.json - Zero configuration needed
{
  "scripts": {
    "test": "react-scripts test",
    "test:coverage": "react-scripts test --coverage --watchAll=false",
    "test:debug": "react-scripts --inspect-brk test --runInBand --no-cache"
  }
}

// Includes:
// - Jest
// - @testing-library/react
// - @testing-library/jest-dom
// - @testing-library/user-event
```

**Vite:**
```javascript
// vite.config.ts - Requires manual setup
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    // For Vitest (recommended with Vite)
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/setupTests.ts'
  }
});

// OR use Jest with Vite
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  transform: {
    '^.+\\.tsx?$': ['@swc/jest', {
      jsc: {
        parser: { syntax: 'typescript', tsx: true },
        transform: { react: { runtime: 'automatic' } }
      }
    }]
  },
  moduleNameMapper: {
    '\\.(css|less|scss)$': 'identity-obj-proxy'
  }
};
```

**Next.js:**
```javascript
// jest.config.js
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  dir: './',
});

const customJestConfig = {
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  testEnvironment: 'jest-environment-jsdom',
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  }
};

module.exports = createJestConfig(customJestConfig);
```

#### 10. Best Practices

**Test Organization:**
```typescript
// ✅ Good: Descriptive test names
describe('UserProfile', () => {
  describe('when user is logged in', () => {
    it('displays user avatar', () => {});
    it('shows logout button', () => {});
  });
  
  describe('when user is logged out', () => {
    it('displays login button', () => {});
    it('hides user menu', () => {});
  });
});

// ❌ Bad: Vague test names
test('test 1', () => {});
test('it works', () => {});
```

**Setup and Teardown:**
```typescript
describe('TodoList', () => {
  let user: ReturnType<typeof userEvent.setup>;
  
  beforeEach(() => {
    user = userEvent.setup();
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  it('adds new todo', async () => {
    render(<TodoList />);
    // Test implementation
  });
});
```

**Key Principles:**

1. **Test Behavior, Not Implementation**
```typescript
// ❌ Bad: Testing implementation details
expect(component.state.count).toBe(1);

// ✅ Good: Testing behavior
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

2. **Use Descriptive Test Names**
```typescript
// ✅ Good
it('displays error message when email is invalid', () => {});

// ❌ Bad
it('test email', () => {});
```

3. **One Assertion Per Test (Generally)**
```typescript
// ✅ Good: Focused test
it('displays user name', () => {
  render(<UserCard user={{ name: 'John' }} />);
  expect(screen.getByText('John')).toBeInTheDocument();
});

// Sometimes multiple assertions are OK if testing single behavior
it('validates form fields', () => {
  render(<Form />);
  const input = screen.getByLabelText('Email');
  expect(input).toBeRequired();
  expect(input).toHaveAttribute('type', 'email');
});
```

4. **Avoid Test Interdependence**
```typescript
// ❌ Bad: Tests depend on each other
let userId;
test('creates user', () => {
  userId = createUser();
});
test('updates user', () => {
  updateUser(userId); // Depends on previous test
});

// ✅ Good: Independent tests
test('updates user', () => {
  const userId = createUser();
  updateUser(userId);
  // Test assertions
});
```

5. **Clean Up After Tests**
```typescript
afterEach(() => {
  jest.clearAllMocks();
  jest.restoreAllMocks();
  cleanup(); // RTL automatic cleanup
});
```

#### Summary

**Jest with React provides:**

1. **Complete Testing Solution**: Test runner, assertions, mocking, coverage in one framework
2. **Zero Configuration**: Works out of the box with Create React App
3. **Rich Matchers**: Extensive built-in assertions plus jest-dom for React
4. **Powerful Mocking**: Mock functions, modules, timers, and more
5. **Snapshot Testing**: Track component output changes
6. **Watch Mode**: Interactive test development workflow
7. **Code Coverage**: Built-in coverage reporting with Istanbul
8. **Great DX**: Fast, parallel execution with helpful error messages

**Key Jest APIs for React:**
- `describe()` / `it()` / `test()` - Test structure
- `expect()` with matchers - Assertions
- `jest.fn()` - Mock functions
- `jest.mock()` - Module mocking
- `jest.spyOn()` - Method spying
- `beforeEach()` / `afterEach()` - Setup/teardown
- `toMatchSnapshot()` - Snapshot testing
- Coverage reporting - Quality metrics

</details>

---

### 136. How do you mock API calls in tests?

<details>
<summary>View Answer</summary>

Mocking API calls is essential for reliable, fast, and predictable tests. There are multiple approaches, with **Mock Service Worker (MSW)** being the modern, recommended solution.

#### 1. Mock Service Worker (MSW) - Recommended

**Why MSW?**
```typescript
// MSW intercepts requests at the network level
// Benefits:
// 1. Works with any request library (fetch, axios, etc.)
// 2. Same mock definitions for tests and development
// 3. No need to mock fetch/axios modules
// 4. Realistic network behavior
// 5. Type-safe with TypeScript
```

**Installation:**
```bash
npm install msw --save-dev
```

**Setup:**
```typescript
// src/mocks/handlers.ts
import { rest } from 'msw';

export const handlers = [
  // GET request
  rest.get('/api/users/:id', (req, res, ctx) => {
    const { id } = req.params;
    
    return res(
      ctx.status(200),
      ctx.json({
        id,
        name: 'John Doe',
        email: 'john@example.com'
      })
    );
  }),
  
  // POST request
  rest.post('/api/users', async (req, res, ctx) => {
    const newUser = await req.json();
    
    return res(
      ctx.status(201),
      ctx.json({
        id: '123',
        ...newUser
      })
    );
  }),
  
  // Error response
  rest.get('/api/users/error', (req, res, ctx) => {
    return res(
      ctx.status(500),
      ctx.json({ message: 'Internal server error' })
    );
  })
];
```

**Server Setup:**
```typescript
// src/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

**Test Setup:**
```typescript
// src/setupTests.ts
import '@testing-library/jest-dom';
import { server } from './mocks/server';

// Start server before all tests
beforeAll(() => server.listen());

// Reset handlers after each test
afterEach(() => server.resetHandlers());

// Clean up after all tests
afterAll(() => server.close());
```

**Basic Example:**
```typescript
// UserProfile.tsx
import { useEffect, useState } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return null;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

export default UserProfile;
```

**Test with MSW:**
```typescript
// UserProfile.test.tsx
import { render, screen, waitFor } from '@testing-library/react';
import { rest } from 'msw';
import { server } from '../mocks/server';
import UserProfile from './UserProfile';

describe('UserProfile', () => {
  it('displays user data after loading', async () => {
    render(<UserProfile userId="1" />);
    
    // Initially shows loading
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    
    // Wait for data to load
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
    
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });
  
  it('displays error message on failure', async () => {
    // Override handler for this test
    server.use(
      rest.get('/api/users/:id', (req, res, ctx) => {
        return res(
          ctx.status(500),
          ctx.json({ message: 'Server error' })
        );
      })
    );
    
    render(<UserProfile userId="1" />);
    
    await waitFor(() => {
      expect(screen.getByText(/Error:/)).toBeInTheDocument();
    });
  });
});
```

**Advanced MSW Patterns:**
```typescript
// Dynamic responses
rest.get('/api/users/:id', (req, res, ctx) => {
  const { id } = req.params;
  
  if (id === 'invalid') {
    return res(
      ctx.status(404),
      ctx.json({ message: 'User not found' })
    );
  }
  
  return res(ctx.json({ id, name: `User ${id}` }));
});

// Delayed responses
rest.get('/api/slow', (req, res, ctx) => {
  return res(
    ctx.delay(2000), // 2 second delay
    ctx.json({ data: 'slow response' })
  );
});

// Request headers
rest.get('/api/protected', (req, res, ctx) => {
  const auth = req.headers.get('Authorization');
  
  if (!auth || !auth.startsWith('Bearer ')) {
    return res(
      ctx.status(401),
      ctx.json({ message: 'Unauthorized' })
    );
  }
  
  return res(ctx.json({ data: 'protected data' }));
});

// Query parameters
rest.get('/api/users', (req, res, ctx) => {
  const page = req.url.searchParams.get('page') || '1';
  const limit = req.url.searchParams.get('limit') || '10';
  
  return res(
    ctx.json({
      users: [],
      pagination: { page, limit }
    })
  );
});

// Request body validation
rest.post('/api/users', async (req, res, ctx) => {
  const body = await req.json();
  
  if (!body.email || !body.name) {
    return res(
      ctx.status(400),
      ctx.json({ message: 'Missing required fields' })
    );
  }
  
  return res(
    ctx.status(201),
    ctx.json({ id: '123', ...body })
  );
});
```

**GraphQL with MSW:**
```typescript
import { graphql } from 'msw';

export const handlers = [
  graphql.query('GetUser', (req, res, ctx) => {
    const { id } = req.variables;
    
    return res(
      ctx.data({
        user: {
          id,
          name: 'John Doe',
          email: 'john@example.com'
        }
      })
    );
  }),
  
  graphql.mutation('CreateUser', (req, res, ctx) => {
    const { input } = req.variables;
    
    return res(
      ctx.data({
        createUser: {
          id: '123',
          ...input
        }
      })
    );
  })
];
```

#### 2. jest.mock() - Module Mocking

**Mocking fetch Globally:**
```typescript
// Option 1: Mock in test file
global.fetch = jest.fn();

test('fetches user data', async () => {
  const mockUser = { id: '1', name: 'John' };
  
  (global.fetch as jest.Mock).mockResolvedValueOnce({
    ok: true,
    json: async () => mockUser
  });
  
  render(<UserProfile userId="1" />);
  
  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument();
  });
  
  expect(global.fetch).toHaveBeenCalledWith('/api/users/1');
});

// Option 2: Mock in setup file
// src/setupTests.ts
global.fetch = jest.fn();

beforeEach(() => {
  (global.fetch as jest.Mock).mockClear();
});
```

**Mocking axios:**
```typescript
// Mock axios module
jest.mock('axios');
import axios from 'axios';

const mockedAxios = axios as jest.Mocked<typeof axios>;

test('fetches user with axios', async () => {
  const mockUser = { id: '1', name: 'John' };
  
  mockedAxios.get.mockResolvedValueOnce({
    data: mockUser,
    status: 200,
    statusText: 'OK',
    headers: {},
    config: {} as any
  });
  
  render(<UserProfile userId="1" />);
  
  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument();
  });
  
  expect(mockedAxios.get).toHaveBeenCalledWith('/api/users/1');
});

// Mock axios instance
jest.mock('../api/axios-instance');
import { api } from '../api/axios-instance';

const mockedApi = api as jest.Mocked<typeof api>;

test('uses custom axios instance', async () => {
  mockedApi.get.mockResolvedValueOnce({ data: { name: 'John' } });
  // Test implementation
});
```

**Mocking Custom API Module:**
```typescript
// api/users.ts
export async function fetchUser(id: string) {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error('Failed to fetch');
  return response.json();
}

export async function createUser(data: CreateUserData) {
  const response = await fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  return response.json();
}

// UserProfile.test.tsx
jest.mock('../api/users');
import * as userApi from '../api/users';

const mockedUserApi = userApi as jest.Mocked<typeof userApi>;

test('fetches user using api module', async () => {
  mockedUserApi.fetchUser.mockResolvedValueOnce({
    id: '1',
    name: 'John'
  });
  
  render(<UserProfile userId="1" />);
  
  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument();
  });
  
  expect(mockedUserApi.fetchUser).toHaveBeenCalledWith('1');
});
```

#### 3. jest.spyOn() - Spying on Methods

```typescript
test('spies on fetch', async () => {
  const fetchSpy = jest.spyOn(global, 'fetch').mockResolvedValueOnce({
    ok: true,
    json: async () => ({ id: '1', name: 'John' })
  } as Response);
  
  render(<UserProfile userId="1" />);
  
  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument();
  });
  
  expect(fetchSpy).toHaveBeenCalledTimes(1);
  
  fetchSpy.mockRestore();
});
```

#### 4. Testing Different Scenarios

**Loading State:**
```typescript
test('shows loading state initially', () => {
  // MSW will handle the request, but render immediately
  render(<UserProfile userId="1" />);
  
  expect(screen.getByText('Loading...')).toBeInTheDocument();
});

test('shows loading with delayed response', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      return res(
        ctx.delay(100),
        ctx.json({ id: '1', name: 'John' })
      );
    })
  );
  
  render(<UserProfile userId="1" />);
  
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  
  // Wait for loading to finish
  await waitFor(() => {
    expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
  });
});
```

**Success Response:**
```typescript
test('displays user data on success', async () => {
  render(<UserProfile userId="1" />);
  
  // Wait for API call and rendering
  const userName = await screen.findByText('John Doe');
  expect(userName).toBeInTheDocument();
  
  // Verify all expected data is displayed
  expect(screen.getByText('john@example.com')).toBeInTheDocument();
});

test('handles different user data', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      return res(
        ctx.json({
          id: '2',
          name: 'Jane Smith',
          email: 'jane@example.com'
        })
      );
    })
  );
  
  render(<UserProfile userId="2" />);
  
  await waitFor(() => {
    expect(screen.getByText('Jane Smith')).toBeInTheDocument();
  });
});
```

**Error Responses:**
```typescript
test('displays error on 404', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      return res(
        ctx.status(404),
        ctx.json({ message: 'User not found' })
      );
    })
  );
  
  render(<UserProfile userId="999" />);
  
  await waitFor(() => {
    expect(screen.getByText(/Error:/)).toBeInTheDocument();
  });
});

test('handles 500 server error', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      return res(
        ctx.status(500),
        ctx.json({ message: 'Internal server error' })
      );
    })
  );
  
  render(<UserProfile userId="1" />);
  
  await waitFor(() => {
    expect(screen.getByText(/Error:/)).toBeInTheDocument();
  });
});
```

**Network Errors:**
```typescript
test('handles network failure', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res) => {
      return res.networkError('Failed to connect');
    })
  );
  
  render(<UserProfile userId="1" />);
  
  await waitFor(() => {
    expect(screen.getByText(/Error:/)).toBeInTheDocument();
  });
});
```

**Retry Logic:**
```typescript
// Component with retry
function UserProfileWithRetry({ userId }: { userId: string }) {
  const [retries, setRetries] = useState(0);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error('Failed');
        const data = await response.json();
        setUser(data);
      } catch (error) {
        if (retries < 3) {
          setRetries(r => r + 1);
        } else {
          setError('Failed after retries');
        }
      }
    };
    fetchData();
  }, [userId, retries]);
  
  // Render logic
}

// Test retry behavior
test('retries failed requests', async () => {
  let callCount = 0;
  
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      callCount++;
      
      // Fail first 2 attempts
      if (callCount <= 2) {
        return res(ctx.status(500));
      }
      
      // Succeed on 3rd attempt
      return res(
        ctx.json({ id: '1', name: 'John' })
      );
    })
  );
  
  render(<UserProfileWithRetry userId="1" />);
  
  await waitFor(() => {
    expect(screen.getByText('John')).toBeInTheDocument();
  });
  
  // Verify retry happened
  expect(callCount).toBe(3);
});
```

#### 5. Testing with React Query

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { renderHook, waitFor } from '@testing-library/react';

function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: async () => {
      const response = await fetch(`/api/users/${id}`);
      if (!response.ok) throw new Error('Failed to fetch');
      return response.json();
    }
  });
}

test('fetches user with React Query', async () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false } // Disable retry for tests
    }
  });
  
  const wrapper = ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
  
  const { result } = renderHook(() => useUser('1'), { wrapper });
  
  // Initially loading
  expect(result.current.isLoading).toBe(true);
  
  // Wait for data
  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });
  
  expect(result.current.data).toEqual({
    id: '1',
    name: 'John Doe',
    email: 'john@example.com'
  });
});

test('handles error with React Query', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      return res(ctx.status(500));
    })
  );
  
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false }
    }
  });
  
  const wrapper = ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
  
  const { result } = renderHook(() => useUser('1'), { wrapper });
  
  await waitFor(() => {
    expect(result.current.isError).toBe(true);
  });
  
  expect(result.current.error).toBeInstanceOf(Error);
});
```

#### 6. Best Practices

**1. Prefer MSW Over jest.mock()**
```typescript
// ✅ Good: MSW (realistic network mocking)
server.use(
  rest.get('/api/users/:id', (req, res, ctx) => {
    return res(ctx.json({ id: '1', name: 'John' }));
  })
);

// ❌ Less ideal: Mocking fetch module
global.fetch = jest.fn().mockResolvedValue({
  json: () => Promise.resolve({ id: '1', name: 'John' })
});
```

**2. Keep Mock Data Realistic**
```typescript
// ✅ Good: Realistic data structure
const mockUser = {
  id: '123',
  name: 'John Doe',
  email: 'john@example.com',
  createdAt: '2024-01-01T00:00:00Z',
  role: 'user'
};

// ❌ Bad: Minimal/unrealistic data
const mockUser = { name: 'test' };
```

**3. Test Both Happy and Error Paths**
```typescript
describe('UserProfile', () => {
  it('displays user on success', async () => {
    // Test happy path
  });
  
  it('handles 404 error', async () => {
    // Test error path
  });
  
  it('handles network failure', async () => {
    // Test network error
  });
});
```

**4. Reset Handlers Between Tests**
```typescript
afterEach(() => {
  server.resetHandlers(); // Reset to original handlers
  jest.clearAllMocks();    // Clear mock call history
});
```

**5. Use Proper HTTP Status Codes**
```typescript
// ✅ Good: Correct status codes
rest.get('/api/users/:id', (req, res, ctx) => {
  return res(
    ctx.status(404), // Correct status
    ctx.json({ message: 'Not found' })
  );
});

// ❌ Bad: Always 200
rest.get('/api/users/:id', (req, res, ctx) => {
  return res(
    ctx.json({ error: 'Not found' }) // Status is 200!
  );
});
```

**6. Test Loading States**
```typescript
test('shows loading indicator', async () => {
  server.use(
    rest.get('/api/users/:id', (req, res, ctx) => {
      return res(
        ctx.delay(100), // Add delay
        ctx.json({ id: '1', name: 'John' })
      );
    })
  );
  
  render(<UserProfile userId="1" />);
  
  expect(screen.getByText('Loading...')).toBeInTheDocument();
  
  await waitFor(() => {
    expect(screen.queryByText('Loading...')).not.toBeInTheDocument();
  });
});
```

**7. Verify Request Parameters**
```typescript
test('sends correct request', async () => {
  let capturedRequest: any;
  
  server.use(
    rest.post('/api/users', async (req, res, ctx) => {
      capturedRequest = {
        body: await req.json(),
        headers: Object.fromEntries(req.headers.entries())
      };
      return res(ctx.json({ id: '123' }));
    })
  );
  
  // Trigger API call
  render(<CreateUserForm />);
  // Fill form and submit
  
  await waitFor(() => {
    expect(capturedRequest.body).toEqual({
      name: 'John',
      email: 'john@example.com'
    });
    expect(capturedRequest.headers['content-type']).toBe('application/json');
  });
});
```

#### Summary

**Mocking API Calls:**

1. **MSW (Recommended)**: Network-level interception, realistic mocking, works with any library
2. **jest.mock()**: Module-level mocking, good for unit tests
3. **jest.spyOn()**: Method-level spying, temporary mocks

**Testing Scenarios:**
- ✅ Loading states
- ✅ Success responses
- ✅ Error responses (4xx, 5xx)
- ✅ Network failures
- ✅ Retry logic
- ✅ Request parameters

**Best Practices:**
- Use MSW for realistic network mocking
- Keep mock data close to real API shapes
- Test both happy and error paths
- Reset handlers between tests
- Use proper HTTP status codes
- Test loading and error states
- Verify request parameters when needed

</details>

---

### 137. What is the difference between unit and integration tests?

<details>
<summary>View Answer</summary>

**Unit tests** verify individual components or functions in isolation, while **integration tests** verify how multiple components work together. Both are essential for comprehensive test coverage.

#### 1. Unit Tests

**Definition:**
```typescript
// Unit tests:
// - Test a single component/function in isolation
// - Mock all dependencies
// - Fast execution
// - Focus on component's internal logic
// - High number of tests (base of test pyramid)
```

**Example - Testing Pure Function:**
```typescript
// utils/formatters.ts
export function formatCurrency(amount: number, currency: string = 'USD'): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency
  }).format(amount);
}

export function truncateText(text: string, maxLength: number): string {
  if (text.length <= maxLength) return text;
  return text.slice(0, maxLength) + '...';
}

// utils/formatters.test.ts
import { formatCurrency, truncateText } from './formatters';

describe('formatCurrency', () => {
  it('formats USD correctly', () => {
    expect(formatCurrency(1234.56)).toBe('$1,234.56');
  });
  
  it('formats EUR correctly', () => {
    expect(formatCurrency(1234.56, 'EUR')).toBe('€1,234.56');
  });
  
  it('handles zero', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });
  
  it('handles negative numbers', () => {
    expect(formatCurrency(-100)).toBe('-$100.00');
  });
});

describe('truncateText', () => {
  it('returns original text when shorter than max length', () => {
    expect(truncateText('Hello', 10)).toBe('Hello');
  });
  
  it('truncates text longer than max length', () => {
    expect(truncateText('Hello World', 5)).toBe('Hello...');
  });
  
  it('handles exact length', () => {
    expect(truncateText('Hello', 5)).toBe('Hello');
  });
});
```

**Example - Testing Component in Isolation:**
```typescript
// Button.tsx
import { ButtonHTMLAttributes } from 'react';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  loading?: boolean;
}

export default function Button({
  variant = 'primary',
  size = 'medium',
  loading = false,
  disabled,
  children,
  ...props
}: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled || loading}
      {...props}
    >
      {loading ? 'Loading...' : children}
    </button>
  );
}

// Button.test.tsx - UNIT TEST
import { render, screen } from '@testing-library/react';
import Button from './Button';

describe('Button - Unit Tests', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  it('applies primary variant by default', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveClass('btn-primary');
  });
  
  it('applies secondary variant', () => {
    render(<Button variant="secondary">Click me</Button>);
    expect(screen.getByRole('button')).toHaveClass('btn-secondary');
  });
  
  it('applies danger variant', () => {
    render(<Button variant="danger">Delete</Button>);
    expect(screen.getByRole('button')).toHaveClass('btn-danger');
  });
  
  it('applies size classes', () => {
    render(<Button size="large">Click me</Button>);
    expect(screen.getByRole('button')).toHaveClass('btn-large');
  });
  
  it('shows loading text when loading', () => {
    render(<Button loading>Click me</Button>);
    expect(screen.getByText('Loading...')).toBeInTheDocument();
    expect(screen.queryByText('Click me')).not.toBeInTheDocument();
  });
  
  it('is disabled when loading', () => {
    render(<Button loading>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
  
  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
  
  it('passes through additional props', () => {
    render(<Button data-testid="custom-button">Click me</Button>);
    expect(screen.getByTestId('custom-button')).toBeInTheDocument();
  });
});
```

**Example - Testing Custom Hook in Isolation:**
```typescript
// useCounter.ts
import { useState, useCallback } from 'react';

export function useCounter(initialValue: number = 0) {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCount(c => c + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCount(c => c - 1);
  }, []);
  
  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);
  
  return { count, increment, decrement, reset };
}

// useCounter.test.ts - UNIT TEST
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter - Unit Tests', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });
  
  it('initializes with provided value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });
  
  it('increments count', () => {
    const { result } = renderHook(() => useCounter());
    
    act(() => {
      result.current.increment();
    });
    
    expect(result.current.count).toBe(1);
  });
  
  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));
    
    act(() => {
      result.current.decrement();
    });
    
    expect(result.current.count).toBe(4);
  });
  
  it('resets to initial value', () => {
    const { result } = renderHook(() => useCounter(10));
    
    act(() => {
      result.current.increment();
      result.current.increment();
    });
    
    expect(result.current.count).toBe(12);
    
    act(() => {
      result.current.reset();
    });
    
    expect(result.current.count).toBe(10);
  });
});
```

#### 2. Integration Tests

**Definition:**
```typescript
// Integration tests:
// - Test multiple components working together
// - Test interactions between components
// - May mock external APIs, but not internal components
// - Slower than unit tests
// - Middle layer of test pyramid
// - Catch issues in component communication
```

**Example - Testing Component Integration:**
```typescript
// TodoList.tsx
import { useState } from 'react';
import TodoItem from './TodoItem';
import TodoInput from './TodoInput';

interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

export default function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  
  const addTodo = (text: string) => {
    setTodos(prev => [...prev, {
      id: Date.now().toString(),
      text,
      completed: false
    }]);
  };
  
  const toggleTodo = (id: string) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const deleteTodo = (id: string) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };
  
  return (
    <div>
      <h1>Todo List</h1>
      <TodoInput onAdd={addTodo} />
      <ul>
        {todos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={toggleTodo}
            onDelete={deleteTodo}
          />
        ))}
      </ul>
      {todos.length === 0 && <p>No todos yet</p>}
    </div>
  );
}

// TodoInput.tsx
import { useState, FormEvent } from 'react';

interface TodoInputProps {
  onAdd: (text: string) => void;
}

export default function TodoInput({ onAdd }: TodoInputProps) {
  const [text, setText] = useState('');
  
  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    if (text.trim()) {
      onAdd(text);
      setText('');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={text}
        onChange={e => setText(e.target.value)}
        placeholder="Add a todo..."
      />
      <button type="submit">Add</button>
    </form>
  );
}

// TodoItem.tsx
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

interface TodoItemProps {
  todo: Todo;
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
}

export default function TodoItem({ todo, onToggle, onDelete }: TodoItemProps) {
  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
}

// TodoList.test.tsx - INTEGRATION TEST
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import TodoList from './TodoList';

describe('TodoList - Integration Tests', () => {
  it('adds a new todo', async () => {
    const user = userEvent.setup();
    render(<TodoList />);
    
    const input = screen.getByPlaceholderText('Add a todo...');
    const addButton = screen.getByRole('button', { name: 'Add' });
    
    // Type and submit
    await user.type(input, 'Buy groceries');
    await user.click(addButton);
    
    // Verify todo appears in list
    expect(screen.getByText('Buy groceries')).toBeInTheDocument();
    
    // Verify input is cleared
    expect(input).toHaveValue('');
  });
  
  it('toggles todo completion', async () => {
    const user = userEvent.setup();
    render(<TodoList />);
    
    // Add a todo
    await user.type(screen.getByPlaceholderText('Add a todo...'), 'Test todo');
    await user.click(screen.getByRole('button', { name: 'Add' }));
    
    const checkbox = screen.getByRole('checkbox');
    const todoText = screen.getByText('Test todo');
    
    // Initially not completed
    expect(checkbox).not.toBeChecked();
    expect(todoText).toHaveStyle({ textDecoration: 'none' });
    
    // Toggle completion
    await user.click(checkbox);
    
    expect(checkbox).toBeChecked();
    expect(todoText).toHaveStyle({ textDecoration: 'line-through' });
    
    // Toggle back
    await user.click(checkbox);
    
    expect(checkbox).not.toBeChecked();
    expect(todoText).toHaveStyle({ textDecoration: 'none' });
  });
  
  it('deletes a todo', async () => {
    const user = userEvent.setup();
    render(<TodoList />);
    
    // Add a todo
    await user.type(screen.getByPlaceholderText('Add a todo...'), 'Delete me');
    await user.click(screen.getByRole('button', { name: 'Add' }));
    
    expect(screen.getByText('Delete me')).toBeInTheDocument();
    
    // Delete todo
    await user.click(screen.getByRole('button', { name: 'Delete' }));
    
    expect(screen.queryByText('Delete me')).not.toBeInTheDocument();
    expect(screen.getByText('No todos yet')).toBeInTheDocument();
  });
  
  it('manages multiple todos', async () => {
    const user = userEvent.setup();
    render(<TodoList />);
    
    // Add multiple todos
    const input = screen.getByPlaceholderText('Add a todo...');
    const addButton = screen.getByRole('button', { name: 'Add' });
    
    await user.type(input, 'First todo');
    await user.click(addButton);
    
    await user.type(input, 'Second todo');
    await user.click(addButton);
    
    await user.type(input, 'Third todo');
    await user.click(addButton);
    
    // Verify all todos are present
    expect(screen.getByText('First todo')).toBeInTheDocument();
    expect(screen.getByText('Second todo')).toBeInTheDocument();
    expect(screen.getByText('Third todo')).toBeInTheDocument();
    
    // Toggle middle todo
    const checkboxes = screen.getAllByRole('checkbox');
    await user.click(checkboxes[1]);
    
    expect(checkboxes[0]).not.toBeChecked();
    expect(checkboxes[1]).toBeChecked();
    expect(checkboxes[2]).not.toBeChecked();
    
    // Delete first todo
    const deleteButtons = screen.getAllByRole('button', { name: 'Delete' });
    await user.click(deleteButtons[0]);
    
    expect(screen.queryByText('First todo')).not.toBeInTheDocument();
    expect(screen.getByText('Second todo')).toBeInTheDocument();
    expect(screen.getByText('Third todo')).toBeInTheDocument();
  });
  
  it('prevents adding empty todos', async () => {
    const user = userEvent.setup();
    render(<TodoList />);
    
    const addButton = screen.getByRole('button', { name: 'Add' });
    
    // Try to add empty todo
    await user.click(addButton);
    
    expect(screen.getByText('No todos yet')).toBeInTheDocument();
    
    // Try with spaces only
    await user.type(screen.getByPlaceholderText('Add a todo...'), '   ');
    await user.click(addButton);
    
    expect(screen.getByText('No todos yet')).toBeInTheDocument();
  });
});
```

**Example - Testing with API Integration:**
```typescript
// UserDashboard.tsx
import { useEffect, useState } from 'react';
import UserProfile from './UserProfile';
import UserPosts from './UserPosts';

interface User {
  id: string;
  name: string;
  email: string;
}

interface Post {
  id: string;
  title: string;
  content: string;
}

export default function UserDashboard({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    Promise.all([
      fetch(`/api/users/${userId}`).then(r => r.json()),
      fetch(`/api/users/${userId}/posts`).then(r => r.json())
    ])
      .then(([userData, postsData]) => {
        setUser(userData);
        setPosts(postsData);
        setLoading(false);
      })
      .catch(() => {
        setError('Failed to load dashboard');
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Loading dashboard...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return null;
  
  return (
    <div>
      <UserProfile user={user} />
      <UserPosts posts={posts} />
    </div>
  );
}

// UserDashboard.test.tsx - INTEGRATION TEST
import { render, screen, waitFor } from '@testing-library/react';
import { rest } from 'msw';
import { server } from '../mocks/server';
import UserDashboard from './UserDashboard';

describe('UserDashboard - Integration Tests', () => {
  it('loads and displays user profile and posts', async () => {
    server.use(
      rest.get('/api/users/:id', (req, res, ctx) => {
        return res(
          ctx.json({
            id: '1',
            name: 'John Doe',
            email: 'john@example.com'
          })
        );
      }),
      rest.get('/api/users/:id/posts', (req, res, ctx) => {
        return res(
          ctx.json([
            { id: '1', title: 'First Post', content: 'Content 1' },
            { id: '2', title: 'Second Post', content: 'Content 2' }
          ])
        );
      })
    );
    
    render(<UserDashboard userId="1" />);
    
    // Shows loading initially
    expect(screen.getByText('Loading dashboard...')).toBeInTheDocument();
    
    // Wait for data to load
    await waitFor(() => {
      expect(screen.getByText('John Doe')).toBeInTheDocument();
    });
    
    // Verify both profile and posts are displayed
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.getByText('First Post')).toBeInTheDocument();
    expect(screen.getByText('Second Post')).toBeInTheDocument();
  });
  
  it('handles API failures gracefully', async () => {
    server.use(
      rest.get('/api/users/:id', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );
    
    render(<UserDashboard userId="1" />);
    
    await waitFor(() => {
      expect(screen.getByText(/Error:/)).toBeInTheDocument();
    });
  });
});
```

#### 3. Key Differences

**Comparison Table:**

| Aspect | Unit Tests | Integration Tests |
|--------|-----------|-------------------|
| **Scope** | Single component/function | Multiple components together |
| **Dependencies** | All mocked | Some real, some mocked |
| **Speed** | Very fast (milliseconds) | Slower (seconds) |
| **Quantity** | Many (70-80% of tests) | Moderate (15-20% of tests) |
| **Purpose** | Verify logic correctness | Verify component interaction |
| **Isolation** | Complete isolation | Partial isolation |
| **Debugging** | Easy to pinpoint issues | Harder to identify root cause |
| **Maintenance** | Low maintenance | Higher maintenance |
| **Example** | Test Button renders correctly | Test Form submits data to API |

**Visual Comparison:**
```typescript
// UNIT TEST: Test Button in isolation
test('Button shows loading state', () => {
  render(<Button loading>Submit</Button>);
  expect(screen.getByText('Loading...')).toBeInTheDocument();
});

// INTEGRATION TEST: Test Button within Form
test('Form shows loading during submission', async () => {
  const user = userEvent.setup();
  render(<LoginForm />); // Contains multiple components
  
  await user.type(screen.getByLabelText('Email'), 'test@example.com');
  await user.type(screen.getByLabelText('Password'), 'password');
  await user.click(screen.getByRole('button', { name: 'Login' }));
  
  // Verify button shows loading (integration with form state)
  expect(screen.getByRole('button')).toHaveTextContent('Loading...');
});
```

#### 4. Testing Pyramid

**Pyramid Structure:**
```typescript
/*
        /\
       /  \     E2E Tests (5-10%)
      /    \    - Full user flows
     /------\   - Slow, expensive
    /        \
   /  INTEG  \  Integration Tests (15-20%)
  /            \ - Component interactions
 /    UNIT      \ Unit Tests (70-80%)
/________________\ - Individual functions/components
                   - Fast, cheap, many tests
*/

// Example distribution for a feature:
// Feature: User Registration

// UNIT TESTS (70-80%)
// - validateEmail function
// - validatePassword function
// - formatPhoneNumber function
// - Input component renders correctly
// - Button component handles clicks
// - PasswordStrength component shows correct level

// INTEGRATION TESTS (15-20%)
// - RegistrationForm validates all fields together
// - RegistrationForm submits data correctly
// - Error messages appear when validation fails

// E2E TESTS (5-10%)
// - Complete registration flow from landing page to success
```

#### 5. When to Use Each

**Use Unit Tests When:**
```typescript
// ✅ Testing pure functions
test('adds two numbers', () => {
  expect(add(2, 3)).toBe(5);
});

// ✅ Testing utility functions
test('formats date correctly', () => {
  expect(formatDate('2024-01-01')).toBe('January 1, 2024');
});

// ✅ Testing component props and rendering
test('Badge displays count', () => {
  render(<Badge count={5} />);
  expect(screen.getByText('5')).toBeInTheDocument();
});

// ✅ Testing hooks in isolation
test('useToggle switches state', () => {
  const { result } = renderHook(() => useToggle());
  act(() => result.current.toggle());
  expect(result.current.value).toBe(true);
});

// ✅ Testing individual component behavior
test('Modal closes when X is clicked', async () => {
  const onClose = jest.fn();
  const user = userEvent.setup();
  
  render(<Modal isOpen onClose={onClose}>Content</Modal>);
  await user.click(screen.getByLabelText('Close'));
  
  expect(onClose).toHaveBeenCalled();
});
```

**Use Integration Tests When:**
```typescript
// ✅ Testing component interactions
test('SearchBar filters ProductList', async () => {
  const user = userEvent.setup();
  render(<ProductSearch />);
  
  await user.type(screen.getByPlaceholderText('Search...'), 'laptop');
  
  expect(screen.getByText('MacBook Pro')).toBeInTheDocument();
  expect(screen.queryByText('iPhone')).not.toBeInTheDocument();
});

// ✅ Testing form submission flow
test('ContactForm sends data and shows success', async () => {
  const user = userEvent.setup();
  render(<ContactForm />);
  
  await user.type(screen.getByLabelText('Name'), 'John');
  await user.type(screen.getByLabelText('Email'), 'john@example.com');
  await user.type(screen.getByLabelText('Message'), 'Hello');
  await user.click(screen.getByRole('button', { name: 'Send' }));
  
  await waitFor(() => {
    expect(screen.getByText('Message sent!')).toBeInTheDocument();
  });
});

// ✅ Testing state management across components
test('Cart updates when items are added', async () => {
  const user = userEvent.setup();
  render(<ShoppingApp />);
  
  await user.click(screen.getByRole('button', { name: 'Add to Cart' }));
  
  expect(screen.getByText('Cart (1)')).toBeInTheDocument();
});

// ✅ Testing data flow through multiple components
test('Parent component passes data to children', async () => {
  render(
    <UserProvider>
      <Header />
      <Sidebar />
      <MainContent />
    </UserProvider>
  );
  
  await waitFor(() => {
    expect(screen.getByText('Welcome, John')).toBeInTheDocument();
  });
});
```

#### 6. Best Practices

**For Unit Tests:**
```typescript
// ✅ Mock all dependencies
jest.mock('../api/userService');

// ✅ Test one thing at a time
test('increments counter', () => {
  // Test only increment logic
});

// ✅ Keep tests fast
test('renders immediately', () => {
  render(<Component />);
  // No async operations
});

// ✅ Use descriptive names
test('displays error message when email is invalid', () => {});

// ❌ Avoid testing implementation details
// Bad
expect(component.state.count).toBe(1);
// Good
expect(screen.getByText('Count: 1')).toBeInTheDocument();
```

**For Integration Tests:**
```typescript
// ✅ Test realistic user scenarios
test('user can complete checkout process', async () => {
  // Test full workflow
});

// ✅ Mock only external dependencies
// Mock API, but not internal components
server.use(
  rest.post('/api/checkout', (req, res, ctx) => {
    return res(ctx.json({ success: true }));
  })
);

// ✅ Test component communication
test('child component notifies parent', async () => {
  // Test data flow between components
});

// ✅ Use findBy for async operations
const element = await screen.findByText('Success');

// ❌ Don't test every edge case (use unit tests)
// Focus on main workflows
```

**Balanced Approach:**
```typescript
// Feature: User Profile Edit

// UNIT TESTS (more tests, focused)
describe('ProfileForm - Unit', () => {
  it('validates name field');
  it('validates email format');
  it('shows error for empty fields');
  it('enables save button when form is valid');
  it('disables save button when form is invalid');
});

// INTEGRATION TESTS (fewer tests, broader)
describe('ProfileForm - Integration', () => {
  it('saves profile and shows success message');
  it('handles API errors gracefully');
  it('updates user context after save');
});
```

#### Summary

**Unit Tests:**
- Test individual components/functions in isolation
- Mock all dependencies
- Fast, cheap, easy to debug
- 70-80% of your test suite
- Focus on logic correctness

**Integration Tests:**
- Test multiple components working together
- Mock only external dependencies (APIs)
- Slower, more expensive, harder to debug
- 15-20% of your test suite
- Focus on component interaction

**Key Principle:**
> Write tests that give you confidence in your code without slowing down development. Use unit tests to catch bugs early and integration tests to ensure components work together correctly.

**Testing Strategy:**
1. Start with unit tests for core logic
2. Add integration tests for critical user flows
3. Use E2E tests sparingly for complete workflows
4. Follow the testing pyramid
5. Test behavior, not implementation

</details>

---

### 138. How do you test user interactions?

<details>
<summary>View Answer</summary>

Testing user interactions ensures your React components respond correctly to user input. **@testing-library/user-event** is the recommended way to simulate realistic user interactions.

#### 1. Setup: userEvent vs fireEvent

**Why userEvent?**
```typescript
// userEvent: Simulates complete user interactions
// - More realistic (triggers multiple events)
// - Asynchronous (like real user behavior)
// - Better for accessibility testing
// - Recommended by Testing Library

// fireEvent: Dispatches single DOM events
// - Synchronous
// - Less realistic
// - Use only for specific edge cases
```

**Basic Setup:**
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('example test', async () => {
  // Setup userEvent
  const user = userEvent.setup();
  
  render(<MyComponent />);
  
  // Interact with component
  await user.click(screen.getByRole('button'));
});
```

#### 2. Click Events

**Basic Clicks:**
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Counter from './Counter';

// Counter.tsx
export default function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <button onClick={() => setCount(c => c - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}

// Counter.test.tsx
test('increments counter on click', async () => {
  const user = userEvent.setup();
  render(<Counter />);
  
  const incrementButton = screen.getByRole('button', { name: 'Increment' });
  
  await user.click(incrementButton);
  
  expect(screen.getByText('Count: 1')).toBeInTheDocument();
});

test('decrements counter on click', async () => {
  const user = userEvent.setup();
  render(<Counter />);
  
  const decrementButton = screen.getByRole('button', { name: 'Decrement' });
  
  await user.click(decrementButton);
  
  expect(screen.getByText('Count: -1')).toBeInTheDocument();
});

test('resets counter on click', async () => {
  const user = userEvent.setup();
  render(<Counter />);
  
  // Increment a few times
  const incrementButton = screen.getByRole('button', { name: 'Increment' });
  await user.click(incrementButton);
  await user.click(incrementButton);
  await user.click(incrementButton);
  
  expect(screen.getByText('Count: 3')).toBeInTheDocument();
  
  // Reset
  await user.click(screen.getByRole('button', { name: 'Reset' }));
  
  expect(screen.getByText('Count: 0')).toBeInTheDocument();
});
```

**Double Click:**
```typescript
test('handles double click', async () => {
  const user = userEvent.setup();
  const handleDoubleClick = jest.fn();
  
  render(<button onDoubleClick={handleDoubleClick}>Double Click Me</button>);
  
  await user.dblClick(screen.getByRole('button'));
  
  expect(handleDoubleClick).toHaveBeenCalledTimes(1);
});
```

**Right Click (Context Menu):**
```typescript
test('shows context menu on right click', async () => {
  const user = userEvent.setup();
  const handleContextMenu = jest.fn((e) => e.preventDefault());
  
  render(
    <div onContextMenu={handleContextMenu}>
      Right click me
    </div>
  );
  
  await user.pointer({ keys: '[MouseRight]', target: screen.getByText('Right click me') });
  
  expect(handleContextMenu).toHaveBeenCalled();
});
```

#### 3. Typing and Input

**Text Input:**
```typescript
// LoginForm.tsx
export default function LoginForm({ onSubmit }: { onSubmit: (data: any) => void }) {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    onSubmit({ email, password });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="email">Email</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={e => setEmail(e.target.value)}
      />
      
      <label htmlFor="password">Password</label>
      <input
        id="password"
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
      />
      
      <button type="submit">Login</button>
    </form>
  );
}

// LoginForm.test.tsx
test('allows user to type email and password', async () => {
  const user = userEvent.setup();
  const handleSubmit = jest.fn();
  
  render(<LoginForm onSubmit={handleSubmit} />);
  
  const emailInput = screen.getByLabelText('Email');
  const passwordInput = screen.getByLabelText('Password');
  
  // Type into inputs
  await user.type(emailInput, 'test@example.com');
  await user.type(passwordInput, 'password123');
  
  // Verify values
  expect(emailInput).toHaveValue('test@example.com');
  expect(passwordInput).toHaveValue('password123');
  
  // Submit form
  await user.click(screen.getByRole('button', { name: 'Login' }));
  
  expect(handleSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123'
  });
});
```

**Clear and Replace Text:**
```typescript
test('clears input field', async () => {
  const user = userEvent.setup();
  render(<input defaultValue="Initial text" />);
  
  const input = screen.getByRole('textbox');
  
  // Clear the input
  await user.clear(input);
  
  expect(input).toHaveValue('');
});

test('replaces text in input', async () => {
  const user = userEvent.setup();
  render(<input defaultValue="Old text" />);
  
  const input = screen.getByRole('textbox');
  
  // Clear and type new text
  await user.clear(input);
  await user.type(input, 'New text');
  
  expect(input).toHaveValue('New text');
});
```

**Typing Special Characters:**
```typescript
test('types special characters', async () => {
  const user = userEvent.setup();
  render(<input />);
  
  const input = screen.getByRole('textbox');
  
  // Type with special characters
  await user.type(input, 'Hello{Space}World{Enter}');
  await user.type(input, '{Backspace}{Backspace}!');
  
  expect(input).toHaveValue('Hello World!');
});

test('types with modifiers', async () => {
  const user = userEvent.setup();
  const handleKeyDown = jest.fn();
  
  render(<input onKeyDown={handleKeyDown} />);
  
  const input = screen.getByRole('textbox');
  
  // Type with Ctrl/Cmd modifier
  await user.type(input, '{Control>}a{/Control}');
  
  expect(handleKeyDown).toHaveBeenCalledWith(
    expect.objectContaining({
      key: 'a',
      ctrlKey: true
    })
  );
});
```

**Paste:**
```typescript
test('pastes text into input', async () => {
  const user = userEvent.setup();
  render(<input />);
  
  const input = screen.getByRole('textbox');
  
  await user.click(input);
  await user.paste('Pasted text');
  
  expect(input).toHaveValue('Pasted text');
});
```

#### 4. Select and Dropdown

**Select Element:**
```typescript
test('selects option from dropdown', async () => {
  const user = userEvent.setup();
  const handleChange = jest.fn();
  
  render(
    <select onChange={handleChange}>
      <option value="">Select...</option>
      <option value="red">Red</option>
      <option value="blue">Blue</option>
      <option value="green">Green</option>
    </select>
  );
  
  const select = screen.getByRole('combobox');
  
  // Select by value
  await user.selectOptions(select, 'blue');
  
  expect(select).toHaveValue('blue');
  expect(handleChange).toHaveBeenCalled();
});

test('selects option by label', async () => {
  const user = userEvent.setup();
  
  render(
    <select>
      <option value="r">Red</option>
      <option value="b">Blue</option>
    </select>
  );
  
  const select = screen.getByRole('combobox');
  
  // Select by visible text
  await user.selectOptions(select, 'Blue');
  
  expect(select).toHaveValue('b');
});

test('selects multiple options', async () => {
  const user = userEvent.setup();
  
  render(
    <select multiple>
      <option value="1">Option 1</option>
      <option value="2">Option 2</option>
      <option value="3">Option 3</option>
    </select>
  );
  
  const select = screen.getByRole('listbox');
  
  // Select multiple options
  await user.selectOptions(select, ['1', '3']);
  
  const selectedOptions = Array.from(
    (select as HTMLSelectElement).selectedOptions
  ).map(option => option.value);
  
  expect(selectedOptions).toEqual(['1', '3']);
});
```

#### 5. Checkboxes and Radio Buttons

**Checkbox:**
```typescript
test('toggles checkbox', async () => {
  const user = userEvent.setup();
  const handleChange = jest.fn();
  
  render(
    <label>
      <input type="checkbox" onChange={handleChange} />
      Accept terms
    </label>
  );
  
  const checkbox = screen.getByRole('checkbox');
  
  // Initially unchecked
  expect(checkbox).not.toBeChecked();
  
  // Check
  await user.click(checkbox);
  expect(checkbox).toBeChecked();
  expect(handleChange).toHaveBeenCalledTimes(1);
  
  // Uncheck
  await user.click(checkbox);
  expect(checkbox).not.toBeChecked();
  expect(handleChange).toHaveBeenCalledTimes(2);
});
```

**Radio Buttons:**
```typescript
test('selects radio button', async () => {
  const user = userEvent.setup();
  const handleChange = jest.fn();
  
  render(
    <div onChange={handleChange}>
      <label>
        <input type="radio" name="color" value="red" />
        Red
      </label>
      <label>
        <input type="radio" name="color" value="blue" />
        Blue
      </label>
      <label>
        <input type="radio" name="color" value="green" />
        Green
      </label>
    </div>
  );
  
  const redRadio = screen.getByLabelText('Red');
  const blueRadio = screen.getByLabelText('Blue');
  
  // Select red
  await user.click(redRadio);
  expect(redRadio).toBeChecked();
  expect(blueRadio).not.toBeChecked();
  
  // Select blue (red becomes unchecked)
  await user.click(blueRadio);
  expect(redRadio).not.toBeChecked();
  expect(blueRadio).toBeChecked();
});
```

#### 6. Keyboard Navigation

**Tab Navigation:**
```typescript
test('navigates form with tab key', async () => {
  const user = userEvent.setup();
  
  render(
    <form>
      <input placeholder="First" />
      <input placeholder="Second" />
      <input placeholder="Third" />
      <button>Submit</button>
    </form>
  );
  
  const firstInput = screen.getByPlaceholderText('First');
  const secondInput = screen.getByPlaceholderText('Second');
  const thirdInput = screen.getByPlaceholderText('Third');
  const submitButton = screen.getByRole('button');
  
  // Start at first input
  firstInput.focus();
  expect(firstInput).toHaveFocus();
  
  // Tab to second
  await user.tab();
  expect(secondInput).toHaveFocus();
  
  // Tab to third
  await user.tab();
  expect(thirdInput).toHaveFocus();
  
  // Tab to button
  await user.tab();
  expect(submitButton).toHaveFocus();
  
  // Shift+Tab to go back
  await user.tab({ shift: true });
  expect(thirdInput).toHaveFocus();
});
```

**Enter Key:**
```typescript
test('submits form with Enter key', async () => {
  const user = userEvent.setup();
  const handleSubmit = jest.fn(e => e.preventDefault());
  
  render(
    <form onSubmit={handleSubmit}>
      <input />
      <button type="submit">Submit</button>
    </form>
  );
  
  const input = screen.getByRole('textbox');
  
  await user.type(input, 'test{Enter}');
  
  expect(handleSubmit).toHaveBeenCalled();
});
```

**Escape Key:**
```typescript
test('closes modal with Escape key', async () => {
  const user = userEvent.setup();
  const handleClose = jest.fn();
  
  render(
    <div role="dialog">
      <button onClick={handleClose}>Close</button>
      <input onKeyDown={(e) => {
        if (e.key === 'Escape') handleClose();
      }} />
    </div>
  );
  
  const input = screen.getByRole('textbox');
  
  await user.type(input, '{Escape}');
  
  expect(handleClose).toHaveBeenCalled();
});
```

**Arrow Keys:**
```typescript
test('navigates list with arrow keys', async () => {
  const user = userEvent.setup();
  const handleKeyDown = jest.fn();
  
  render(
    <ul onKeyDown={handleKeyDown}>
      <li tabIndex={0}>Item 1</li>
      <li tabIndex={0}>Item 2</li>
      <li tabIndex={0}>Item 3</li>
    </ul>
  );
  
  const firstItem = screen.getByText('Item 1');
  firstItem.focus();
  
  await user.keyboard('{ArrowDown}');
  await user.keyboard('{ArrowUp}');
  
  expect(handleKeyDown).toHaveBeenCalledTimes(2);
});
```

#### 7. Hover and Focus

**Hover:**
```typescript
test('shows tooltip on hover', async () => {
  const user = userEvent.setup();
  
  render(
    <div>
      <button>Hover me</button>
      <div role="tooltip" style={{ display: 'none' }}>Tooltip</div>
    </div>
  );
  
  const button = screen.getByRole('button');
  
  // Hover over button
  await user.hover(button);
  
  // Tooltip should appear (assuming component implements this)
  // await waitFor(() => {
  //   expect(screen.getByRole('tooltip')).toBeVisible();
  // });
  
  // Unhover
  await user.unhover(button);
});
```

**Focus and Blur:**
```typescript
test('validates input on blur', async () => {
  const user = userEvent.setup();
  
  render(
    <input
      onBlur={(e) => {
        if (!e.target.value) {
          e.target.setCustomValidity('Required');
        }
      }}
    />
  );
  
  const input = screen.getByRole('textbox');
  
  // Focus input
  await user.click(input);
  expect(input).toHaveFocus();
  
  // Tab away (triggers blur)
  await user.tab();
  expect(input).not.toHaveFocus();
});
```

#### 8. File Upload

```typescript
test('uploads file', async () => {
  const user = userEvent.setup();
  const handleChange = jest.fn();
  
  render(
    <input
      type="file"
      onChange={handleChange}
    />
  );
  
  const fileInput = screen.getByRole('textbox', { hidden: true });
  
  const file = new File(['hello'], 'hello.txt', { type: 'text/plain' });
  
  await user.upload(fileInput, file);
  
  expect(handleChange).toHaveBeenCalled();
  expect(fileInput.files).toHaveLength(1);
  expect(fileInput.files?.[0]).toBe(file);
});

test('uploads multiple files', async () => {
  const user = userEvent.setup();
  
  render(<input type="file" multiple />);
  
  const fileInput = screen.getByRole('textbox', { hidden: true });
  
  const files = [
    new File(['content1'], 'file1.txt', { type: 'text/plain' }),
    new File(['content2'], 'file2.txt', { type: 'text/plain' })
  ];
  
  await user.upload(fileInput, files);
  
  expect(fileInput.files).toHaveLength(2);
});
```

#### 9. Complex Interactions

**Form with Multiple Interactions:**
```typescript
test('completes registration form', async () => {
  const user = userEvent.setup();
  const handleSubmit = jest.fn(e => e.preventDefault());
  
  render(
    <form onSubmit={handleSubmit}>
      <input placeholder="Username" />
      <input type="email" placeholder="Email" />
      <input type="password" placeholder="Password" />
      <select>
        <option value="">Select country...</option>
        <option value="us">United States</option>
        <option value="uk">United Kingdom</option>
      </select>
      <label>
        <input type="checkbox" />
        I agree to terms
      </label>
      <button type="submit">Register</button>
    </form>
  );
  
  // Fill out form
  await user.type(screen.getByPlaceholderText('Username'), 'johndoe');
  await user.type(screen.getByPlaceholderText('Email'), 'john@example.com');
  await user.type(screen.getByPlaceholderText('Password'), 'password123');
  await user.selectOptions(screen.getByRole('combobox'), 'us');
  await user.click(screen.getByRole('checkbox'));
  
  // Submit
  await user.click(screen.getByRole('button', { name: 'Register' }));
  
  expect(handleSubmit).toHaveBeenCalled();
});
```

**Async Interactions:**
```typescript
test('handles loading state during submission', async () => {
  const user = userEvent.setup();
  
  render(<SubmitForm />);
  
  await user.type(screen.getByPlaceholderText('Name'), 'John');
  await user.click(screen.getByRole('button', { name: 'Submit' }));
  
  // Shows loading state
  expect(screen.getByText('Submitting...')).toBeInTheDocument();
  
  // Wait for success message
  await waitFor(() => {
    expect(screen.getByText('Success!')).toBeInTheDocument();
  });
});
```

#### 10. Best Practices

**1. Always Use userEvent.setup()**
```typescript
// ✅ Good: Setup at test level
test('handles click', async () => {
  const user = userEvent.setup();
  render(<Button />);
  await user.click(screen.getByRole('button'));
});

// ❌ Bad: Import default (deprecated)
import userEvent from '@testing-library/user-event';
await userEvent.click(button); // Old API
```

**2. Await All Interactions**
```typescript
// ✅ Good: Await userEvent calls
await user.click(button);
await user.type(input, 'text');

// ❌ Bad: Missing await
user.click(button); // Will cause issues
```

**3. Use Semantic Queries**
```typescript
// ✅ Good: Query by role/label
const button = screen.getByRole('button', { name: 'Submit' });
const input = screen.getByLabelText('Email');

// ❌ Bad: Query by test ID
const button = screen.getByTestId('submit-button');
```

**4. Test User Perspective**
```typescript
// ✅ Good: Test as user would interact
test('user can login', async () => {
  const user = userEvent.setup();
  render(<LoginForm />);
  
  await user.type(screen.getByLabelText('Email'), 'test@example.com');
  await user.type(screen.getByLabelText('Password'), 'password');
  await user.click(screen.getByRole('button', { name: 'Login' }));
  
  await waitFor(() => {
    expect(screen.getByText('Welcome!')).toBeInTheDocument();
  });
});

// ❌ Bad: Test implementation details
test('updates state', async () => {
  const { result } = renderHook(() => useAuth());
  act(() => result.current.login('email', 'password'));
  expect(result.current.user).toBeDefined();
});
```

**5. Handle Async Operations**
```typescript
// ✅ Good: Wait for async results
await user.click(button);
await waitFor(() => {
  expect(screen.getByText('Success')).toBeInTheDocument();
});

// Or use findBy (combines getBy + waitFor)
expect(await screen.findByText('Success')).toBeInTheDocument();

// ❌ Bad: Not waiting
await user.click(button);
expect(screen.getByText('Success')).toBeInTheDocument(); // May fail
```

**6. Clean Interactions**
```typescript
// ✅ Good: Clear, sequential interactions
test('fills form', async () => {
  const user = userEvent.setup();
  render(<Form />);
  
  await user.type(screen.getByLabelText('Name'), 'John');
  await user.type(screen.getByLabelText('Email'), 'john@example.com');
  await user.click(screen.getByRole('button', { name: 'Submit' }));
  
  await waitFor(() => {
    expect(screen.getByText('Submitted')).toBeInTheDocument();
  });
});

// ❌ Bad: Complex, hard to follow
test('form test', async () => {
  const user = userEvent.setup();
  render(<Form />);
  await user.type(screen.getByLabelText('Name'), 'John');
  expect(screen.getByLabelText('Name')).toHaveValue('John');
  await user.type(screen.getByLabelText('Email'), 'john@example.com');
  expect(screen.getByLabelText('Email')).toHaveValue('john@example.com');
  // Too many intermediate assertions
});
```

#### Summary

**Testing User Interactions:**

1. **Use userEvent**: `const user = userEvent.setup()`
2. **Await interactions**: All userEvent methods are async
3. **Common interactions**:
   - `user.click()` - Click elements
   - `user.type()` - Type text
   - `user.clear()` - Clear inputs
   - `user.selectOptions()` - Select from dropdown
   - `user.upload()` - Upload files
   - `user.tab()` - Tab navigation
   - `user.hover()` - Hover elements

4. **Best practices**:
   - Test from user's perspective
   - Use semantic queries (role, label)
   - Handle async operations with waitFor/findBy
   - Keep tests readable and focused
   - Await all user interactions

**Key Pattern:**
```typescript
test('user interaction test', async () => {
  // 1. Setup
  const user = userEvent.setup();
  
  // 2. Render
  render(<Component />);
  
  // 3. Interact
  await user.type(screen.getByLabelText('Input'), 'text');
  await user.click(screen.getByRole('button'));
  
  // 4. Assert
  await waitFor(() => {
    expect(screen.getByText('Result')).toBeInTheDocument();
  });
});
```

</details>

---

### 139. What is Vitest and how does it compare to Jest?

<details>
<summary>View Answer</summary>

**Vitest** is a modern, fast testing framework designed for Vite projects. It's Jest-compatible and offers native ESM support, making it an excellent choice for modern React applications.

#### 1. What is Vitest?

**Overview:**
```typescript
// Vitest is:
// - A Vite-native unit test framework
// - Jest-compatible API
// - Lightning fast with HMR for tests
// - Native ESM, TypeScript, and JSX support
// - Smart & instant watch mode
// - Built on Vite's transformation pipeline
```

**Key Features:**
```typescript
// 1. Speed: Uses Vite's dev server for transformations
// 2. ESM First: Native ES module support
// 3. HMR: Hot Module Replacement for tests
// 4. Jest Compatible: Easy migration from Jest
// 5. TypeScript: First-class TypeScript support
// 6. Multi-threading: Worker threads for parallel execution
// 7. UI Mode: Visual test interface
// 8. In-source testing: Test alongside your code
```

#### 2. Installation & Setup

**Install Vitest:**
```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom
npm install -D jsdom # or happy-dom for DOM environment
```

**Basic Configuration:**
```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    // Test environment
    environment: 'jsdom', // or 'happy-dom'
    
    // Global test APIs (describe, it, expect)
    globals: true,
    
    // Setup files
    setupFiles: ['./src/setupTests.ts'],
    
    // Coverage configuration
    coverage: {
      provider: 'v8', // or 'istanbul'
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/setupTests.ts',
      ]
    },
    
    // CSS handling
    css: true,
  }
});
```

**Setup File:**
```typescript
// src/setupTests.ts
import { expect, afterEach } from 'vitest';
import { cleanup } from '@testing-library/react';
import * as matchers from '@testing-library/jest-dom/matchers';

// Extend Vitest's expect with jest-dom matchers
expect.extend(matchers);

// Cleanup after each test
afterEach(() => {
  cleanup();
});
```

**Package Scripts:**
```json
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

#### 3. Vitest vs Jest Comparison

**Feature Comparison:**

| Feature | Vitest | Jest |
|---------|--------|------|
| **Speed** | ⚡ Very fast (uses Vite) | Fast |
| **ESM Support** | ✅ Native | 🔶 Experimental |
| **TypeScript** | ✅ No config needed | Needs babel/ts-jest |
| **Watch Mode** | ✅ HMR, instant | Fast, but slower |
| **Configuration** | Via vite.config.ts | Separate jest.config.js |
| **API** | Jest-compatible | Jest API |
| **Ecosystem** | Growing | Mature, extensive |
| **UI Mode** | ✅ Built-in | ❌ Third-party only |
| **In-source testing** | ✅ Supported | ❌ Not supported |
| **Snapshot testing** | ✅ Supported | ✅ Supported |
| **Mocking** | Jest-compatible | Full mocking system |
| **Coverage** | v8 or istanbul | istanbul |
| **React support** | ✅ Excellent | ✅ Excellent |
| **Best for** | Vite projects | Any project |

**Performance Comparison:**
```typescript
// Typical test execution times:

// Jest:
// - Initial run: ~3-5 seconds
// - Watch mode change: ~1-2 seconds
// - Uses Babel/ts-jest for transforms

// Vitest:
// - Initial run: ~1-2 seconds
// - Watch mode change: ~100-500ms
// - Uses Vite's esbuild for transforms
// - HMR makes subsequent runs nearly instant
```

#### 4. API Compatibility

**Jest-Compatible API:**
```typescript
// Most Jest APIs work in Vitest without changes

// Test structure
import { describe, it, expect, beforeEach, afterEach } from 'vitest';

describe('Counter', () => {
  it('increments count', () => {
    expect(1 + 1).toBe(2);
  });
});

// Mocking
import { vi } from 'vitest';

const mockFn = vi.fn();
vi.mock('./api');
vi.spyOn(console, 'log');

// Timers
vi.useFakeTimers();
vi.advanceTimersByTime(1000);
vi.useRealTimers();

// Matchers
expect(value).toBe(1);
expect(value).toEqual({ name: 'John' });
expect(fn).toHaveBeenCalled();
```

**Vitest Example:**
```typescript
// Button.test.tsx (works with both Vitest and Jest)
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import Button from './Button';

describe('Button', () => {
  it('renders children', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  it('calls onClick when clicked', async () => {
    const handleClick = vi.fn();
    const user = userEvent.setup();
    
    render(<Button onClick={handleClick}>Click me</Button>);
    
    await user.click(screen.getByRole('button'));
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  it('is disabled when disabled prop is true', () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole('button')).toBeDisabled();
  });
});
```

#### 5. Unique Vitest Features

**UI Mode:**
```bash
# Run tests with visual UI
npm run test:ui

# Opens browser with:
# - Test file tree
# - Test results
# - Coverage visualization
# - Console output
# - Time tracking
```

**In-Source Testing:**
```typescript
// src/utils/math.ts
export function add(a: number, b: number): number {
  return a + b;
}

// Tests in the same file!
if (import.meta.vitest) {
  const { it, expect } = import.meta.vitest;
  
  it('adds numbers', () => {
    expect(add(2, 3)).toBe(5);
  });
  
  it('handles negative numbers', () => {
    expect(add(-1, -2)).toBe(-3);
  });
}

// Enable in vite.config.ts
export default defineConfig({
  define: {
    'import.meta.vitest': 'undefined',
  },
  test: {
    includeSource: ['src/**/*.{js,ts,jsx,tsx}']
  }
});
```

**Type Testing:**
```typescript
import { expectTypeOf, assertType } from 'vitest';

// Type assertions
expectTypeOf<string>().toBeString();
expectTypeOf<{ name: string }>().toHaveProperty('name');

function greet(name: string) {
  return `Hello ${name}`;
}

expectTypeOf(greet).parameter(0).toBeString();
expectTypeOf(greet).returns.toBeString();
```

**Concurrent Tests:**
```typescript
import { describe, it } from 'vitest';

// Run tests in parallel
describe.concurrent('API tests', () => {
  it('fetches user 1', async () => {
    const user = await fetchUser(1);
    expect(user.name).toBe('John');
  });
  
  it('fetches user 2', async () => {
    const user = await fetchUser(2);
    expect(user.name).toBe('Jane');
  });
  
  it('fetches user 3', async () => {
    const user = await fetchUser(3);
    expect(user.name).toBe('Bob');
  });
});
```

**Benchmark Testing:**
```typescript
import { bench, describe } from 'vitest';

describe('sorting algorithms', () => {
  bench('bubble sort', () => {
    bubbleSort([5, 3, 8, 1, 2]);
  });
  
  bench('quick sort', () => {
    quickSort([5, 3, 8, 1, 2]);
  });
});
```

#### 6. Migrating from Jest to Vitest

**Step 1: Install Dependencies**
```bash
npm uninstall jest @types/jest ts-jest
npm install -D vitest @vitest/ui jsdom
```

**Step 2: Update Configuration**
```typescript
// Remove jest.config.js

// Update vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/setupTests.ts',
  }
});
```

**Step 3: Update Setup File**
```typescript
// src/setupTests.ts
// Before (Jest):
import '@testing-library/jest-dom';

// After (Vitest):
import { expect } from 'vitest';
import * as matchers from '@testing-library/jest-dom/matchers';

expect.extend(matchers);
```

**Step 4: Update Test Files**
```typescript
// Before (Jest):
import { jest } from '@jest/globals';
const mockFn = jest.fn();
jest.mock('./module');

// After (Vitest):
import { vi } from 'vitest';
const mockFn = vi.fn();
vi.mock('./module');

// Or use globals:
const mockFn = vi.fn(); // if globals: true
```

**Step 5: Update Package Scripts**
```json
// package.json
{
  "scripts": {
    // Before:
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    
    // After:
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage"
  }
}
```

**Step 6: Update TypeScript Config**
```json
// tsconfig.json
{
  "compilerOptions": {
    "types": [
      // Remove: "jest"
      "vitest/globals", // Add this if using globals: true
      "@testing-library/jest-dom"
    ]
  }
}
```

#### 7. Common Patterns

**Mocking with Vitest:**
```typescript
import { vi } from 'vitest';

// Mock function
const mockFn = vi.fn();
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ data: 'success' });

// Mock module
vi.mock('./api', () => ({
  fetchUser: vi.fn(() => Promise.resolve({ id: 1, name: 'John' }))
}));

// Spy on method
const spy = vi.spyOn(console, 'error').mockImplementation(() => {});

// Mock timers
vi.useFakeTimers();
vi.advanceTimersByTime(1000);
vi.runAllTimers();
vi.useRealTimers();

// Clear mocks
vi.clearAllMocks();
vi.resetAllMocks();
vi.restoreAllMocks();
```

**Snapshot Testing:**
```typescript
import { expect } from 'vitest';
import { render } from '@testing-library/react';

test('matches snapshot', () => {
  const { container } = render(<Button>Click me</Button>);
  expect(container.firstChild).toMatchSnapshot();
});

// Inline snapshots
test('renders correctly', () => {
  const { container } = render(<Badge count={5} />);
  expect(container.firstChild).toMatchInlineSnapshot(`
    <span class="badge">
      5
    </span>
  `);
});
```

**Testing Async Code:**
```typescript
import { waitFor } from '@testing-library/react';

test('loads data', async () => {
  render(<UserProfile userId="1" />);
  
  // Wait for element to appear
  const name = await screen.findByText('John Doe');
  expect(name).toBeInTheDocument();
  
  // Or use waitFor
  await waitFor(() => {
    expect(screen.getByText('John Doe')).toBeInTheDocument();
  });
});
```

#### 8. When to Use Vitest vs Jest

**Use Vitest When:**
```typescript
// ✅ Using Vite as build tool
// ✅ Want fastest possible test execution
// ✅ Need native ESM support
// ✅ Want modern developer experience
// ✅ Starting a new project
// ✅ Value built-in UI mode
// ✅ Want in-source testing

// Example project structure:
// my-vite-app/
//   vite.config.ts
//   src/
//     components/
//       Button.tsx
//       Button.test.tsx  ← Vitest
```

**Use Jest When:**
```typescript
// ✅ Using Webpack, Create React App
// ✅ Need mature, battle-tested ecosystem
// ✅ Require specific Jest plugins
// ✅ Large existing Jest test suite
// ✅ Team expertise with Jest
// ✅ Need maximum compatibility

// Example project structure:
// my-cra-app/
//   package.json (react-scripts)
//   src/
//     components/
//       Button.tsx
//       Button.test.tsx  ← Jest (via react-scripts)
```

**Migration Decision Matrix:**
```typescript
/*
Should I migrate from Jest to Vitest?

YES if:
- Already using Vite
- Tests are slow
- Want better DX
- Small to medium test suite
- Team is open to change

NO if:
- Using Create React App (stick with Jest)
- Large test suite (high migration cost)
- Custom Jest plugins/setup
- Team unfamiliar with Vite
- Tests are already fast enough

COMPROMISE:
- Use Vitest for new features
- Keep Jest for existing tests
- Gradual migration over time
*/
```

#### 9. Advanced Configuration

**Multi-Environment Testing:**
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    // Browser-like environment for components
    environment: 'jsdom',
    
    // Multiple test environments
    environmentOptions: {
      jsdom: {
        resources: 'usable',
      },
    },
    
    // Test file patterns
    include: ['**/*.{test,spec}.{js,ts,jsx,tsx}'],
    exclude: ['node_modules', 'dist', '.next'],
    
    // Global setup/teardown
    globalSetup: './tests/globalSetup.ts',
    
    // Test timeout
    testTimeout: 10000,
    
    // Threads
    threads: true,
    maxThreads: 4,
    minThreads: 1,
  }
});
```

**Custom Matchers:**
```typescript
// tests/matchers.ts
import { expect } from 'vitest';

expect.extend({
  toBeWithinRange(received: number, floor: number, ceiling: number) {
    const pass = received >= floor && received <= ceiling;
    return {
      pass,
      message: () =>
        pass
          ? `expected ${received} not to be within range ${floor} - ${ceiling}`
          : `expected ${received} to be within range ${floor} - ${ceiling}`,
    };
  },
});

// Use in tests
expect(100).toBeWithinRange(90, 110);
```

#### 10. Best Practices

**1. Use Vitest's Native Features:**
```typescript
// ✅ Good: Use vi namespace
import { vi } from 'vitest';
const mockFn = vi.fn();

// ❌ Avoid: Don't try to import jest
import { jest } from '@jest/globals'; // Not needed
```

**2. Leverage Watch Mode:**
```bash
# Start watch mode (default)
npm test

# Tests re-run instantly on changes due to HMR
# Much faster than Jest watch mode
```

**3. Use UI Mode for Debugging:**
```bash
npm run test:ui

# Visual interface helps:
# - See test structure
# - Debug failures
# - View coverage
# - Track performance
```

**4. Configure Coverage Properly:**
```typescript
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8', // Faster than istanbul
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/setupTests.ts',
        '**/*.d.ts',
        '**/*.config.*',
        '**/mockData/*',
        'src/main.tsx',
      ],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80,
      },
    },
  },
});
```

**5. Keep Tests Fast:**
```typescript
// ✅ Good: Fast unit tests
test('adds numbers', () => {
  expect(add(2, 3)).toBe(5);
});

// ⚠️ Slower: Integration tests (use sparingly)
test('full app flow', async () => {
  render(<App />);
  // Multiple interactions and API calls
});
```

#### Summary

**Vitest:**
- Modern, fast testing framework for Vite projects
- Jest-compatible API makes migration easy
- Native ESM and TypeScript support
- HMR for instant test feedback
- Built-in UI mode for visual testing
- Best choice for new Vite-based React projects

**Key Advantages:**
1. ⚡ **Speed**: 2-10x faster than Jest
2. 🔄 **HMR**: Instant test re-runs
3. 📦 **Zero Config**: Works with Vite out of the box
4. 🎨 **UI Mode**: Visual test interface
5. 🔧 **Modern**: ESM, TypeScript native

**When to Choose:**
- **Vitest**: New projects with Vite, modern stack, speed priority
- **Jest**: CRA projects, existing test suites, mature ecosystem needs

**Migration Effort:**
- Most Jest tests work in Vitest with minimal changes
- Main change: `jest` → `vi` for mocking
- Configuration moves from jest.config.js to vite.config.ts
- Can be done incrementally for large projects

</details>

---

### 140. How do you achieve high test coverage?

<details>
<summary>View Answer</summary>

Achieving high test coverage ensures your code is thoroughly tested. However, **quality matters more than quantity** - aim for meaningful tests that catch real bugs.

#### 1. Understanding Test Coverage

**Coverage Metrics:**
```typescript
// Four main coverage metrics:

// 1. Line Coverage
// Percentage of code lines executed during tests
function add(a: number, b: number) {
  return a + b; // ✅ Line executed
}

// 2. Branch Coverage
// Percentage of conditional branches executed
function isPositive(n: number) {
  if (n > 0) {     // ✅ True branch tested
    return true;
  }
  return false;    // ✅ False branch tested
}

// 3. Function Coverage
// Percentage of functions called
function multiply(a: number, b: number) {
  return a * b; // ✅ Function called
}

// 4. Statement Coverage
// Percentage of statements executed
const result = calculate(); // ✅ Statement executed
console.log(result);        // ✅ Statement executed
```

**Coverage Report Example:**
```
--------------------|---------|----------|---------|---------|-------------------
File                | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
--------------------|---------|----------|---------|---------|-------------------
All files           |   87.50 |    83.33 |   90.00 |   87.50 |
Button.tsx          |     100 |      100 |     100 |     100 |
UserProfile.tsx     |   75.00 |    66.67 |   80.00 |   75.00 | 23-25,45
utils/helpers.ts    |   90.00 |    85.00 |     100 |   90.00 | 67,89
--------------------|---------|----------|---------|---------|-------------------
```

#### 2. Setting Up Coverage

**With Jest:**
```bash
# Run tests with coverage
npm test -- --coverage --watchAll=false

# With specific reporters
npm test -- --coverage --coverageReporters=text
npm test -- --coverage --coverageReporters=html
npm test -- --coverage --coverageReporters=lcov
```

**Jest Configuration:**
```javascript
// jest.config.js
module.exports = {
  // Files to collect coverage from
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
    '!src/reportWebVitals.ts',
    '!src/**/*.stories.tsx',
    '!src/**/__tests__/**',
  ],
  
  // Coverage thresholds (fail if below)
  coverageThresholds: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    // Stricter for critical paths
    './src/components/': {
      branches: 90,
      functions: 90,
      lines: 90,
      statements: 90,
    },
  },
  
  // Ignore patterns
  coveragePathIgnorePatterns: [
    '/node_modules/',
    '/build/',
    '/coverage/',
  ],
  
  // Report formats
  coverageReporters: [
    'text',        // Console output
    'text-summary', // Brief summary
    'html',        // HTML report
    'lcov',        // For CI/CD tools
  ],
};
```

**With Vitest:**
```typescript
// vite.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    coverage: {
      provider: 'v8', // or 'istanbul'
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/setupTests.ts',
        '**/*.d.ts',
        '**/*.config.*',
        'dist/',
      ],
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 80,
        statements: 80,
      },
    },
  },
});
```

**Package Scripts:**
```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:coverage": "jest --coverage --watchAll=false",
    "test:coverage:watch": "jest --coverage --watch",
    "test:coverage:ci": "jest --coverage --ci --maxWorkers=2"
  }
}
```

#### 3. Strategies for High Coverage

**Strategy 1: Test All Code Paths**
```typescript
// UserStatus.tsx
function UserStatus({ user }: { user: User | null }) {
  if (!user) {
    return <div>Not logged in</div>;
  }
  
  if (user.role === 'admin') {
    return <div>Admin: {user.name}</div>;
  }
  
  if (user.role === 'premium') {
    return <div>Premium: {user.name}</div>;
  }
  
  return <div>User: {user.name}</div>;
}

// UserStatus.test.tsx - Test ALL branches
describe('UserStatus', () => {
  it('shows not logged in when user is null', () => {
    render(<UserStatus user={null} />);
    expect(screen.getByText('Not logged in')).toBeInTheDocument();
  });
  
  it('shows admin status for admin users', () => {
    const user = { name: 'Admin', role: 'admin' };
    render(<UserStatus user={user} />);
    expect(screen.getByText('Admin: Admin')).toBeInTheDocument();
  });
  
  it('shows premium status for premium users', () => {
    const user = { name: 'Premium', role: 'premium' };
    render(<UserStatus user={user} />);
    expect(screen.getByText('Premium: Premium')).toBeInTheDocument();
  });
  
  it('shows regular user status for other roles', () => {
    const user = { name: 'User', role: 'user' };
    render(<UserStatus user={user} />);
    expect(screen.getByText('User: User')).toBeInTheDocument();
  });
});
```

**Strategy 2: Test Edge Cases**
```typescript
// utils/validators.ts
export function validateEmail(email: string): boolean {
  if (!email) return false;
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

export function truncate(text: string, maxLength: number): string {
  if (text.length <= maxLength) return text;
  return text.slice(0, maxLength) + '...';
}

// validators.test.ts - Test edge cases
describe('validateEmail', () => {
  // Happy path
  it('returns true for valid email', () => {
    expect(validateEmail('test@example.com')).toBe(true);
  });
  
  // Edge cases
  it('returns false for empty string', () => {
    expect(validateEmail('')).toBe(false);
  });
  
  it('returns false for email without @', () => {
    expect(validateEmail('testexample.com')).toBe(false);
  });
  
  it('returns false for email without domain', () => {
    expect(validateEmail('test@')).toBe(false);
  });
  
  it('returns false for email without TLD', () => {
    expect(validateEmail('test@example')).toBe(false);
  });
  
  it('returns false for email with spaces', () => {
    expect(validateEmail('test @example.com')).toBe(false);
  });
});

describe('truncate', () => {
  it('returns original text when shorter than max', () => {
    expect(truncate('Hello', 10)).toBe('Hello');
  });
  
  it('returns original text when equal to max', () => {
    expect(truncate('Hello', 5)).toBe('Hello');
  });
  
  it('truncates text longer than max', () => {
    expect(truncate('Hello World', 5)).toBe('Hello...');
  });
  
  it('handles empty string', () => {
    expect(truncate('', 5)).toBe('');
  });
  
  it('handles maxLength of 0', () => {
    expect(truncate('Hello', 0)).toBe('...');
  });
});
```

**Strategy 3: Test Error Handling**
```typescript
// UserProfile.tsx
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    fetchUser(userId)
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;
  
  return <div>{user.name}</div>;
}

// UserProfile.test.tsx - Test all states
describe('UserProfile', () => {
  it('shows loading state initially', () => {
    render(<UserProfile userId="1" />);
    expect(screen.getByText('Loading...')).toBeInTheDocument();
  });
  
  it('shows user data on success', async () => {
    server.use(
      rest.get('/api/users/:id', (req, res, ctx) => {
        return res(ctx.json({ id: '1', name: 'John' }));
      })
    );
    
    render(<UserProfile userId="1" />);
    expect(await screen.findByText('John')).toBeInTheDocument();
  });
  
  it('shows error message on failure', async () => {
    server.use(
      rest.get('/api/users/:id', (req, res, ctx) => {
        return res(ctx.status(500));
      })
    );
    
    render(<UserProfile userId="1" />);
    expect(await screen.findByText(/Error:/)).toBeInTheDocument();
  });
  
  it('shows no user message when null', async () => {
    server.use(
      rest.get('/api/users/:id', (req, res, ctx) => {
        return res(ctx.json(null));
      })
    );
    
    render(<UserProfile userId="1" />);
    expect(await screen.findByText('No user found')).toBeInTheDocument();
  });
});
```

**Strategy 4: Test User Interactions**
```typescript
// Counter.tsx
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <button onClick={() => setCount(c => c - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}

// Counter.test.tsx - Test all interactions
describe('Counter', () => {
  it('renders initial count', () => {
    render(<Counter />);
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });
  
  it('increments count', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    await user.click(screen.getByRole('button', { name: 'Increment' }));
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });
  
  it('decrements count', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    await user.click(screen.getByRole('button', { name: 'Decrement' }));
    expect(screen.getByText('Count: -1')).toBeInTheDocument();
  });
  
  it('resets count', async () => {
    const user = userEvent.setup();
    render(<Counter />);
    
    await user.click(screen.getByRole('button', { name: 'Increment' }));
    await user.click(screen.getByRole('button', { name: 'Increment' }));
    await user.click(screen.getByRole('button', { name: 'Reset' }));
    
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });
});
```

**Strategy 5: Test Hooks Thoroughly**
```typescript
// useLocalStorage.ts
export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue] as const;
}

// useLocalStorage.test.ts - Comprehensive hook tests
describe('useLocalStorage', () => {
  beforeEach(() => {
    localStorage.clear();
  });
  
  it('returns initial value when no stored value', () => {
    const { result } = renderHook(() => useLocalStorage('test', 'default'));
    expect(result.current[0]).toBe('default');
  });
  
  it('returns stored value when available', () => {
    localStorage.setItem('test', JSON.stringify('stored'));
    const { result } = renderHook(() => useLocalStorage('test', 'default'));
    expect(result.current[0]).toBe('stored');
  });
  
  it('updates stored value', () => {
    const { result } = renderHook(() => useLocalStorage('test', 'initial'));
    
    act(() => {
      result.current[1]('updated');
    });
    
    expect(result.current[0]).toBe('updated');
    expect(localStorage.getItem('test')).toBe(JSON.stringify('updated'));
  });
  
  it('updates with function', () => {
    const { result } = renderHook(() => useLocalStorage('test', 0));
    
    act(() => {
      result.current[1](prev => prev + 1);
    });
    
    expect(result.current[0]).toBe(1);
  });
  
  it('handles JSON parse error', () => {
    localStorage.setItem('test', 'invalid json');
    const { result } = renderHook(() => useLocalStorage('test', 'default'));
    expect(result.current[0]).toBe('default');
  });
  
  it('handles storage error gracefully', () => {
    const { result } = renderHook(() => useLocalStorage('test', 'initial'));
    
    // Mock localStorage.setItem to throw
    const spy = jest.spyOn(Storage.prototype, 'setItem')
      .mockImplementation(() => {
        throw new Error('Storage full');
      });
    
    act(() => {
      result.current[1]('new value');
    });
    
    // Should not crash
    expect(result.current[0]).toBe('new value');
    
    spy.mockRestore();
  });
});
```

#### 4. Identifying Coverage Gaps

**View Coverage Report:**
```bash
# Generate HTML report
npm run test:coverage

# Open in browser
open coverage/lcov-report/index.html

# The report shows:
# - Green: Covered lines
# - Red: Uncovered lines
# - Yellow: Partially covered branches
```

**Reading Coverage Report:**
```typescript
// Example: Uncovered lines highlighted

function calculateDiscount(price: number, userType: string) {
  let discount = 0;
  
  if (price > 100) {               // ✅ Covered
    discount = 0.1;
  }
  
  if (userType === 'premium') {    // ❌ Not covered (never tested)
    discount += 0.05;
  }
  
  if (userType === 'employee') {   // ❌ Not covered (never tested)
    discount = 0.5;
  }
  
  return price * (1 - discount);   // ✅ Covered
}

// Add missing tests:
test('applies premium discount', () => {
  expect(calculateDiscount(200, 'premium')).toBe(170); // 15% off
});

test('applies employee discount', () => {
  expect(calculateDiscount(200, 'employee')).toBe(100); // 50% off
});
```

**Using Coverage to Find Gaps:**
```typescript
// Coverage report shows:
// ProductCard.tsx: 75% branch coverage
// Uncovered: lines 23-25

// ProductCard.tsx
function ProductCard({ product }: { product: Product }) {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      {product.onSale && (          // ⚠️ True branch covered
        <span>On Sale!</span>       // ❌ Lines 23-25 never executed
      )}
    </div>
  );
}

// Add missing test:
test('shows sale badge when product is on sale', () => {
  const product = { name: 'Item', price: 99, onSale: true };
  render(<ProductCard product={product} />);
  expect(screen.getByText('On Sale!')).toBeInTheDocument();
});
```

#### 5. Coverage Best Practices

**1. Don't Chase 100% Coverage**
```typescript
// ❌ Bad: Testing trivial code just for coverage
test('exports constant', () => {
  expect(API_URL).toBeDefined();
});

// ✅ Good: Focus on meaningful tests
test('fetches data from correct API endpoint', async () => {
  await fetchUsers();
  expect(global.fetch).toHaveBeenCalledWith(API_URL + '/users');
});
```

**2. Set Realistic Thresholds**
```javascript
// jest.config.js
coverageThresholds: {
  global: {
    branches: 80,    // 80-90% is realistic
    functions: 80,
    lines: 80,
    statements: 80,
  },
  // Higher for critical paths
  './src/payment/': {
    branches: 95,
    functions: 95,
    lines: 95,
    statements: 95,
  },
  // Lower for UI components
  './src/components/': {
    branches: 70,
    functions: 75,
    lines: 75,
    statements: 75,
  },
}
```

**3. Exclude Non-Critical Files**
```javascript
collectCoverageFrom: [
  'src/**/*.{js,jsx,ts,tsx}',
  // Exclude:
  '!src/index.tsx',              // Entry point
  '!src/reportWebVitals.ts',     // CRA boilerplate
  '!src/**/*.d.ts',              // Type definitions
  '!src/**/*.stories.tsx',       // Storybook stories
  '!src/setupTests.ts',          // Test setup
  '!src/mocks/**',               // Mock data
  '!src/**/__tests__/**',        // Test files themselves
]
```

**4. Write Meaningful Tests**
```typescript
// ❌ Bad: High coverage, low value
test('component renders', () => {
  render(<LoginForm />);
  expect(screen.getByRole('form')).toBeInTheDocument();
});

// ✅ Good: Tests actual functionality
test('submits login form with credentials', async () => {
  const user = userEvent.setup();
  const onSubmit = jest.fn();
  
  render(<LoginForm onSubmit={onSubmit} />);
  
  await user.type(screen.getByLabelText('Email'), 'test@example.com');
  await user.type(screen.getByLabelText('Password'), 'password123');
  await user.click(screen.getByRole('button', { name: 'Login' }));
  
  expect(onSubmit).toHaveBeenCalledWith({
    email: 'test@example.com',
    password: 'password123',
  });
});
```

**5. Monitor Coverage Over Time**
```javascript
// CI/CD integration
// .github/workflows/test.yml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install dependencies
        run: npm ci
      - name: Run tests with coverage
        run: npm run test:coverage
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v2
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true
```

**6. Review Coverage in PRs**
```typescript
// package.json - Fail CI if coverage drops
{
  "scripts": {
    "test:ci": "jest --coverage --ci --maxWorkers=2"
  },
  "jest": {
    "coverageThresholds": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

#### 6. Common Pitfalls

**Pitfall 1: Testing Implementation Details**
```typescript
// ❌ Bad: High coverage, brittle tests
test('counter state updates', () => {
  const { container } = render(<Counter />);
  const component = container.querySelector('.counter');
  expect(component.state.count).toBe(0); // Testing internals
});

// ✅ Good: Testing behavior
test('counter displays count', () => {
  render(<Counter />);
  expect(screen.getByText('Count: 0')).toBeInTheDocument();
});
```

**Pitfall 2: Ignoring Branch Coverage**
```typescript
// 100% line coverage, but 50% branch coverage!
function getGreeting(hour: number) {
  return hour < 12 ? 'Good morning' : 'Good afternoon';
}

// ❌ Incomplete test (only one branch)
test('returns greeting', () => {
  expect(getGreeting(10)).toBe('Good morning');
});

// ✅ Complete test (both branches)
test('returns morning greeting before noon', () => {
  expect(getGreeting(10)).toBe('Good morning');
});

test('returns afternoon greeting after noon', () => {
  expect(getGreeting(14)).toBe('Good afternoon');
});
```

**Pitfall 3: Not Testing Error Cases**
```typescript
// getUserAge has error handling, but it's not tested!
function getUserAge(user: User | null) {
  try {
    return user!.age;
  } catch (error) {
    return null; // ❌ This line is never covered
  }
}

// Add error case test:
test('returns null when user is null', () => {
  expect(getUserAge(null)).toBe(null);
});
```

#### Summary

**Achieving High Test Coverage:**

1. **Set up coverage tools**:
   - Configure Jest/Vitest with coverage options
   - Set realistic thresholds (80-90%)
   - Exclude non-critical files

2. **Test systematically**:
   - Test all code paths (if/else, switch, ternary)
   - Test edge cases (null, empty, extreme values)
   - Test error handling (try/catch, promises)
   - Test user interactions (clicks, typing, forms)

3. **Monitor coverage**:
   - Review HTML coverage reports
   - Identify uncovered lines/branches
   - Add tests for gaps
   - Track coverage in CI/CD

4. **Quality over quantity**:
   - Don't chase 100% coverage
   - Write meaningful tests
   - Test behavior, not implementation
   - Focus on critical paths

5. **Best practices**:
   - Set per-directory thresholds
   - Review coverage in pull requests
   - Monitor coverage trends over time
   - Balance unit and integration tests

**Remember**: High coverage doesn't guarantee bug-free code. Focus on writing tests that catch real bugs and give you confidence in your code.

</details>
