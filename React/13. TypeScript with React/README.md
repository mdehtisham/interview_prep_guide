# TypeScript with React

> Advanced / Senior Level (3-5 years)

---

## Questions

121. How do you type React components with TypeScript?
122. What is FC (FunctionComponent) type and should you use it?
123. How do you type props with TypeScript?
124. How do you type useState with TypeScript?
125. How do you type useRef with TypeScript?
126. How do you type useContext with TypeScript?
127. What are generic components in TypeScript?
128. How do you type children prop in TypeScript?
129. How do you type event handlers in React with TypeScript?
130. What is PropsWithChildren type?

---

## Detailed Answers

### 121. How do you type React components with TypeScript?

<details>
<summary>View Answer</summary>

**Typing React components** with TypeScript ensures type safety for props, state, and events, preventing runtime errors and improving developer experience with autocomplete and IntelliSense.

#### Methods to Type React Components

1. **Function components with explicit props type**
2. **React.FC / React.FunctionComponent** (not recommended anymore)
3. **Class components with typed props and state**
4. **Generic components**
5. **Components with children**
6. **Components with ref forwarding**

---

#### 1. Function Components (Recommended)

**Basic Function Component:**
```tsx
import React from 'react';

interface Props {
  name: string;
  age: number;
  onUpdate?: (name: string) => void;
}

// Method 1: Explicit return type (recommended)
function UserCard({ name, age, onUpdate }: Props): JSX.Element {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {onUpdate && <button onClick={() => onUpdate(name)}>Update</button>}
    </div>
  );
}

// Method 2: Implicit return type (also fine)
function UserCard({ name, age, onUpdate }: Props) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
    </div>
  );
}

// Method 3: Arrow function
const UserCard = ({ name, age, onUpdate }: Props): JSX.Element => {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
    </div>
  );
};

export default UserCard;
```

**With Optional Props:**
```tsx
interface ButtonProps {
  label: string;
  onClick: () => void;
  disabled?: boolean; // Optional
  variant?: 'primary' | 'secondary'; // Optional with union type
  icon?: React.ReactNode; // Optional React element
}

function Button({ 
  label, 
  onClick, 
  disabled = false, // Default value
  variant = 'primary', 
  icon 
}: ButtonProps) {
  return (
    <button 
      onClick={onClick} 
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {icon && <span className="icon">{icon}</span>}
      {label}
    </button>
  );
}
```

---

#### 2. Components with Children

**Using React.ReactNode (Recommended):**
```tsx
import React from 'react';

interface CardProps {
  title: string;
  children: React.ReactNode; // Accepts any valid React child
}

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Usage
<Card title="My Card">
  <p>This is content</p>
  <button>Click me</button>
</Card>
```

**PropsWithChildren Helper (React 18+):**
```tsx
import React, { PropsWithChildren } from 'react';

interface CardProps {
  title: string;
  variant?: 'default' | 'highlighted';
}

// PropsWithChildren automatically adds children: React.ReactNode
function Card({ title, variant = 'default', children }: PropsWithChildren<CardProps>) {
  return (
    <div className={`card card-${variant}`}>
      <h2>{title}</h2>
      <div className="card-body">{children}</div>
    </div>
  );
}
```

**Specific Children Types:**
```tsx
import React from 'react';

// Only accept string children
interface TextBoxProps {
  children: string;
}

function TextBox({ children }: TextBoxProps) {
  return <p className="text-box">{children}</p>;
}

// Only accept single child element
interface WrapperProps {
  children: React.ReactElement;
}

function Wrapper({ children }: WrapperProps) {
  return <div className="wrapper">{children}</div>;
}

// Only accept array of elements
interface ListProps {
  children: React.ReactElement[];
}

function List({ children }: ListProps) {
  return <ul>{children}</ul>;
}

// Accept render function
interface RenderProps {
  children: (data: string) => React.ReactNode;
}

function DataProvider({ children }: RenderProps) {
  const data = "Some data";
  return <div>{children(data)}</div>;
}

// Usage
<DataProvider>
  {(data) => <p>{data}</p>}
</DataProvider>
```

---

#### 3. Class Components

**Basic Class Component:**
```tsx
import React, { Component } from 'react';

interface Props {
  name: string;
  age: number;
}

interface State {
  count: number;
  isActive: boolean;
}

// Component<PropsType, StateType>
class Counter extends Component<Props, State> {
  // Type the state
  state: State = {
    count: 0,
    isActive: false,
  };

  increment = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    const { name, age } = this.props;
    const { count, isActive } = this.state;

    return (
      <div>
        <h2>{name} ({age})</h2>
        <p>Count: {count}</p>
        <button onClick={this.increment}>Increment</button>
      </div>
    );
  }
}

export default Counter;
```

**Class Component with Lifecycle Methods:**
```tsx
import React, { Component } from 'react';

interface Props {
  userId: string;
}

interface State {
  user: User | null;
  loading: boolean;
  error: string | null;
}

interface User {
  id: string;
  name: string;
  email: string;
}

class UserProfile extends Component<Props, State> {
  state: State = {
    user: null,
    loading: true,
    error: null,
  };

  componentDidMount() {
    this.fetchUser();
  }

  componentDidUpdate(prevProps: Props) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
    }
  }

  async fetchUser() {
    try {
      const response = await fetch(`/api/users/${this.props.userId}`);
      const user: User = await response.json();
      this.setState({ user, loading: false });
    } catch (error) {
      this.setState({ error: (error as Error).message, loading: false });
    }
  }

  render() {
    const { user, loading, error } = this.state;

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
}
```

---

#### 4. Event Handlers

**Typing Event Handlers:**
```tsx
import React from 'react';

interface FormProps {
  onSubmit: (data: FormData) => void;
}

interface FormData {
  email: string;
  password: string;
}

function LoginForm({ onSubmit }: FormProps) {
  // onClick event
  const handleClick = (event: React.MouseEvent<HTMLButtonElement>) => {
    console.log('Clicked:', event.currentTarget);
  };

  // onChange event for input
  const handleInputChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    console.log('Input value:', event.target.value);
  };

  // onChange for select
  const handleSelectChange = (event: React.ChangeEvent<HTMLSelectElement>) => {
    console.log('Selected:', event.target.value);
  };

  // onSubmit event
  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    const formData = new FormData(event.currentTarget);
    onSubmit({
      email: formData.get('email') as string,
      password: formData.get('password') as string,
    });
  };

  // onKeyDown event
  const handleKeyDown = (event: React.KeyboardEvent<HTMLInputElement>) => {
    if (event.key === 'Enter') {
      console.log('Enter pressed');
    }
  };

  // onFocus event
  const handleFocus = (event: React.FocusEvent<HTMLInputElement>) => {
    console.log('Input focused');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        name="email" 
        onChange={handleInputChange}
        onKeyDown={handleKeyDown}
        onFocus={handleFocus}
      />
      <input name="password" type="password" />
      <button type="submit" onClick={handleClick}>
        Login
      </button>
    </form>
  );
}
```

**Common Event Types:**
```tsx
// Mouse events
React.MouseEvent<HTMLDivElement>
React.MouseEvent<HTMLButtonElement>

// Form events
React.FormEvent<HTMLFormElement>
React.ChangeEvent<HTMLInputElement>
React.ChangeEvent<HTMLTextAreaElement>
React.ChangeEvent<HTMLSelectElement>

// Keyboard events
React.KeyboardEvent<HTMLInputElement>

// Focus events
React.FocusEvent<HTMLInputElement>

// Generic event
React.SyntheticEvent
```

---

#### 5. Generic Components

**Generic List Component:**
```tsx
import React from 'react';

interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// Usage with different types
interface User {
  id: string;
  name: string;
}

interface Product {
  id: number;
  title: string;
}

function App() {
  const users: User[] = [{ id: '1', name: 'John' }];
  const products: Product[] = [{ id: 1, title: 'Phone' }];

  return (
    <>
      <List
        items={users}
        renderItem={(user) => <span>{user.name}</span>}
        keyExtractor={(user) => user.id}
      />
      
      <List
        items={products}
        renderItem={(product) => <span>{product.title}</span>}
        keyExtractor={(product) => product.id}
      />
    </>
  );
}
```

**Generic Table Component:**
```tsx
interface Column<T> {
  key: keyof T;
  header: string;
  render?: (value: T[keyof T], item: T) => React.ReactNode;
}

interface TableProps<T> {
  data: T[];
  columns: Column<T>[];
  onRowClick?: (item: T) => void;
}

function Table<T extends { id: string | number }>({
  data,
  columns,
  onRowClick,
}: TableProps<T>) {
  return (
    <table>
      <thead>
        <tr>
          {columns.map((column) => (
            <th key={String(column.key)}>{column.header}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((item) => (
          <tr key={item.id} onClick={() => onRowClick?.(item)}>
            {columns.map((column) => (
              <td key={String(column.key)}>
                {column.render
                  ? column.render(item[column.key], item)
                  : String(item[column.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// Usage
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

const users: User[] = [
  { id: '1', name: 'John', email: 'john@example.com', age: 30 },
];

const columns: Column<User>[] = [
  { key: 'name', header: 'Name' },
  { key: 'email', header: 'Email' },
  { 
    key: 'age', 
    header: 'Age',
    render: (age) => <strong>{age} years old</strong>
  },
];

<Table data={users} columns={columns} onRowClick={(user) => console.log(user)} />;
```

---

#### 6. Ref Forwarding

**forwardRef with TypeScript:**
```tsx
import React, { forwardRef, useRef } from 'react';

interface InputProps {
  label: string;
  placeholder?: string;
}

// forwardRef with generic types
const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, placeholder }, ref) => {
    return (
      <div>
        <label>{label}</label>
        <input ref={ref} placeholder={placeholder} />
      </div>
    );
  }
);

Input.displayName = 'Input';

// Usage
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleFocus = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <Input ref={inputRef} label="Email" placeholder="Enter email" />
      <button onClick={handleFocus}>Focus Input</button>
    </div>
  );
}
```

**useImperativeHandle with TypeScript:**
```tsx
import React, { forwardRef, useImperativeHandle, useRef } from 'react';

interface FancyInputProps {
  label: string;
}

// Define the methods exposed via ref
interface FancyInputRef {
  focus: () => void;
  clear: () => void;
  getValue: () => string;
}

const FancyInput = forwardRef<FancyInputRef, FancyInputProps>(
  ({ label }, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);

    useImperativeHandle(ref, () => ({
      focus: () => {
        inputRef.current?.focus();
      },
      clear: () => {
        if (inputRef.current) {
          inputRef.current.value = '';
        }
      },
      getValue: () => {
        return inputRef.current?.value || '';
      },
    }));

    return (
      <div>
        <label>{label}</label>
        <input ref={inputRef} />
      </div>
    );
  }
);

FancyInput.displayName = 'FancyInput';

// Usage
function Form() {
  const inputRef = useRef<FancyInputRef>(null);

  const handleSubmit = () => {
    const value = inputRef.current?.getValue();
    console.log('Value:', value);
    inputRef.current?.clear();
  };

  return (
    <div>
      <FancyInput ref={inputRef} label="Name" />
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
}
```

---

#### 7. Higher-Order Components (HOCs)

**Typing HOCs:**
```tsx
import React, { ComponentType } from 'react';

interface WithLoadingProps {
  loading: boolean;
}

// HOC that adds loading prop
function withLoading<P extends object>(
  Component: ComponentType<P>
): ComponentType<P & WithLoadingProps> {
  return ({ loading, ...props }: WithLoadingProps & P) => {
    if (loading) {
      return <div>Loading...</div>;
    }
    return <Component {...(props as P)} />;
  };
}

// Original component
interface UserListProps {
  users: User[];
}

function UserList({ users }: UserListProps) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// Enhanced component
const UserListWithLoading = withLoading(UserList);

// Usage
<UserListWithLoading users={users} loading={false} />;
```

---

#### 8. Context with TypeScript

**Typed Context:**
```tsx
import React, { createContext, useContext, useState, ReactNode } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
}

interface AuthContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

// Create context with type
const AuthContext = createContext<AuthContextType | undefined>(undefined);

interface AuthProviderProps {
  children: ReactNode;
}

export function AuthProvider({ children }: AuthProviderProps) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    // Login logic
    const userData: User = { id: '1', name: 'John', email };
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  const value: AuthContextType = {
    user,
    login,
    logout,
    isAuthenticated: !!user,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// Custom hook with type checking
export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// Usage
function Profile() {
  const { user, logout } = useAuth();

  return (
    <div>
      <h2>{user?.name}</h2>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

---

#### Best Practices

**1. Use Interface for Props:**
```tsx
// GOOD - Use interface
interface ButtonProps {
  label: string;
  onClick: () => void;
}

// Also OK - Use type
type ButtonProps = {
  label: string;
  onClick: () => void;
};
```

**2. Avoid React.FC (Modern Approach):**
```tsx
// GOOD - Modern approach
function Button({ label, onClick }: ButtonProps) {
  return <button onClick={onClick}>{label}</button>;
}

// AVOID - React.FC (has issues)
const Button: React.FC<ButtonProps> = ({ label, onClick }) => {
  return <button onClick={onClick}>{label}</button>;
};
```

**3. Explicit Return Types for Complex Components:**
```tsx
// GOOD - Explicit return type
function ComplexComponent(): JSX.Element | null {
  if (condition) return null;
  return <div>Content</div>;
}
```

**4. Use React.ReactNode for Children:**
```tsx
// GOOD
interface Props {
  children: React.ReactNode;
}

// AVOID - Too restrictive
interface Props {
  children: JSX.Element;
}
```

**5. Type Event Handlers Inline or Extract:**
```tsx
// GOOD - Inline
<button onClick={(e: React.MouseEvent) => console.log(e)} />

// GOOD - Extracted
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  console.log(e);
};
```

---

#### Common Patterns Summary

```tsx
// 1. Basic component
interface Props { name: string; }
function Component({ name }: Props) { return <div>{name}</div>; }

// 2. With children
interface Props { children: React.ReactNode; }

// 3. With optional props
interface Props { name?: string; onClick?: () => void; }

// 4. Event handler
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {};

// 5. Generic component
function List<T>({ items }: { items: T[] }) {}

// 6. Ref forwarding
const Input = forwardRef<HTMLInputElement, Props>((props, ref) => {});

// 7. Context
const Context = createContext<ContextType | undefined>(undefined);

// 8. Class component
class Component extends React.Component<Props, State> {}
```

---

#### Interview Tips

✅ **Key Points:**
- Type props with interface or type
- Avoid React.FC in modern code
- Use React.ReactNode for children
- Type event handlers explicitly
- Use generics for reusable components

✅ **When to Mention:**
- TypeScript + React setup
- Type safety in components
- Props validation
- Event handling
- Generic/reusable components

✅ **Common Follow-ups:**
- "Interface vs Type for props?"
- "Why avoid React.FC?"
- "How to type children?"
- "How to type event handlers?"
- "How to create generic components?"

✅ **Perfect Answer Structure:**
1. Define: TypeScript adds type safety to React components
2. Basic: Show function component with typed props
3. Children: Use React.ReactNode for children prop
4. Events: Show typed event handlers
5. Advanced: Generics, refs, HOCs
6. Best practices: Avoid FC, use explicit types

</details>

---

### 122. What is FC (FunctionComponent) type and should you use it?

<details>
<summary>View Answer</summary>

**React.FC** (or **React.FunctionComponent**) is a TypeScript type provided by React to type function components. However, **it's no longer recommended** in modern React development due to several limitations and issues.

#### What is React.FC?

**Definition:**
```tsx
import React from 'react';

interface Props {
  name: string;
  age: number;
}

// Using React.FC
const User: React.FC<Props> = ({ name, age }) => {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
    </div>
  );
};

// Same as:
const User: React.FunctionComponent<Props> = ({ name, age }) => {
  // ...
};
```

**What React.FC Provides:**
- Automatically types the `children` prop (before React 18)
- Types the return value as `React.ReactElement`
- Provides static properties like `displayName`, `propTypes`, `defaultProps`

---

#### Why NOT to Use React.FC (Modern Approach)

**❌ Problem 1: Implicit Children (Before React 18)**

```tsx
interface Props {
  name: string;
}

// Using React.FC - children is implicit (confusing!)
const Card: React.FC<Props> = ({ name }) => {
  return <div>{name}</div>;
};

// This compiles but might not be intended:
<Card name="John">
  <p>Unexpected children!</p>  {/* TypeScript doesn't complain */}
</Card>
```

**✅ Better: Explicit children when needed**

```tsx
interface Props {
  name: string;
  // Explicitly declare children if needed
  children?: React.ReactNode;
}

function Card({ name, children }: Props) {
  return (
    <div>
      <h2>{name}</h2>
      {children}
    </div>
  );
}

// Now it's clear this component accepts children
```

---

**❌ Problem 2: Generic Components Don't Work Well**

```tsx
// Doesn't work properly with React.FC
const List: React.FC<{ items: T[] }> = ({ items }) => {  // ❌ Error!
  return <ul>{items.map(item => <li key={item.id}>{item.name}</li>)}</ul>;
};
```

**✅ Better: Regular function with generics**

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// Usage with type inference
<List 
  items={users} 
  renderItem={(user) => <span>{user.name}</span>}  // user is typed!
/>
```

---

**❌ Problem 3: defaultProps Don't Work Properly**

```tsx
interface Props {
  name: string;
  age?: number;
}

const User: React.FC<Props> = ({ name, age = 18 }) => {
  return <div>{name} - {age}</div>;
};

// defaultProps don't work well with React.FC
User.defaultProps = {
  age: 18,  // TypeScript may not infer this correctly
};
```

**✅ Better: Use default parameters**

```tsx
interface Props {
  name: string;
  age?: number;
}

function User({ name, age = 18 }: Props) {
  return <div>{name} - {age}</div>;
}
```

---

**❌ Problem 4: No Contextual Typing**

```tsx
// With React.FC - no contextual typing
const Button: React.FC<{ onClick: () => void }> = ({ onClick }) => {
  return <button onClick={onClick}>Click</button>;
};
```

**✅ Better: Contextual typing works**

```tsx
interface ButtonProps {
  onClick: () => void;
}

function Button({ onClick }: ButtonProps) {
  return <button onClick={onClick}>Click</button>;
}

// Usage - TypeScript infers the function type
<Button onClick={() => console.log('clicked')} />
```

---

#### React 18 Changes

In **React 18**, `React.FC` was updated to:
- **Remove implicit children** (fixes Problem 1)
- Now you must explicitly add `children` to props

```tsx
// React 18 - children NO LONGER implicit
const Card: React.FC<{ title: string }> = ({ title }) => {
  return <div>{title}</div>;
};

// This now errors (good!):
<Card title="Title">
  <p>Children</p>  {/* ❌ Error: children not in props */}
</Card>

// Must explicitly add children:
interface CardProps {
  title: string;
  children: React.ReactNode;
}

const Card: React.FC<CardProps> = ({ title, children }) => {
  return <div>{title}{children}</div>;
};
```

But even in React 18, **regular functions are still preferred**.

---

#### Modern Approach: Regular Functions

**✅ Recommended Pattern:**

```tsx
import React from 'react';

interface Props {
  name: string;
  age: number;
  onUpdate?: (name: string) => void;
}

// Option 1: Regular function (best)
function User({ name, age, onUpdate }: Props) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      {onUpdate && <button onClick={() => onUpdate(name)}>Update</button>}
    </div>
  );
}

// Option 2: Arrow function (also good)
const User = ({ name, age, onUpdate }: Props) => {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
    </div>
  );
};

// Option 3: Explicit return type (when needed)
function User({ name, age }: Props): JSX.Element {
  return <div>{name} - {age}</div>;
}

export default User;
```

---

#### When React.FC Might Still Be Used

**Legacy Codebases:**
```tsx
// Older projects might still use React.FC
const Button: React.FC<ButtonProps> = (props) => {
  return <button {...props} />;
};
```

**Team Conventions:**
- Some teams still prefer React.FC for consistency
- It's not "wrong", just not recommended for new code

**With displayName:**
```tsx
const Button: React.FC<Props> = (props) => {
  return <button {...props} />;
};

Button.displayName = 'Button';  // Easier with React.FC

// vs.

function Button(props: Props) {
  return <button {...props} />;
}

Button.displayName = 'Button';  // Still works fine
```

---

#### Complete Comparison

**React.FC Approach:**
```tsx
import React from 'react';

interface Props {
  title: string;
  description?: string;
  children?: React.ReactNode;  // Must be explicit in React 18
}

const Card: React.FC<Props> = ({ title, description, children }) => {
  return (
    <div className="card">
      <h2>{title}</h2>
      {description && <p>{description}</p>}
      {children}
    </div>
  );
};

Card.displayName = 'Card';

export default Card;
```

**Modern Approach:**
```tsx
import React from 'react';

interface CardProps {
  title: string;
  description?: string;
  children?: React.ReactNode;
}

function Card({ title, description, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {description && <p>{description}</p>}
      {children}
    </div>
  );
}

Card.displayName = 'Card';  // Optional

export default Card;
```

---

#### Real-World Examples

**Example 1: Button Component**

**❌ With React.FC:**
```tsx
import React from 'react';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick: () => void;
  children: React.ReactNode;
}

const Button: React.FC<ButtonProps> = ({ 
  variant = 'primary', 
  disabled = false, 
  onClick, 
  children 
}) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
};
```

**✅ Modern Approach:**
```tsx
import React from 'react';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick: () => void;
  children: React.ReactNode;
}

function Button({ 
  variant = 'primary', 
  disabled = false, 
  onClick, 
  children 
}: ButtonProps) {
  return (
    <button 
      className={`btn btn-${variant}`}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

export default Button;
```

**Example 2: Generic List Component**

**❌ Doesn't work with React.FC:**
```tsx
// This doesn't work!
const List: React.FC<ListProps<T>> = ({ items, renderItem }) => {  // ❌
  return <ul>{items.map(renderItem)}</ul>;
};
```

**✅ Works with regular function:**
```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// Perfect type inference!
<List
  items={users}
  renderItem={(user) => <span>{user.name}</span>}  // user is User type
  keyExtractor={(user) => user.id}
/>
```

---

#### Migration Guide

**If you have existing code with React.FC:**

**Before:**
```tsx
import React from 'react';

interface Props {
  name: string;
}

const User: React.FC<Props> = ({ name }) => {
  return <div>{name}</div>;
};

export default User;
```

**After:**
```tsx
import React from 'react';

interface Props {
  name: string;
}

function User({ name }: Props) {
  return <div>{name}</div>;
}

// Or arrow function
const User = ({ name }: Props) => {
  return <div>{name}</div>;
};

export default User;
```

---

#### Official React Team Recommendation

From the React TypeScript Cheatsheet and React team:

> "We recommend against using React.FC for typing function components. Regular function declarations provide better type inference and don't have the downsides of React.FC."

---

#### Summary Table

| Feature | React.FC | Regular Function |
|---------|----------|------------------|
| **Implicit children** | ✅ (React <18) | ❌ Explicit |
| | ❌ (React 18+) | |
| **Generic components** | ❌ Doesn't work | ✅ Works perfectly |
| **defaultProps** | ❌ Issues | ✅ Use defaults |
| **Type inference** | ❌ Limited | ✅ Better |
| **Verbosity** | More verbose | Less verbose |
| **Modern best practice** | ❌ Not recommended | ✅ Recommended |
| **displayName** | ✅ Slightly easier | ✅ Still works |

---

#### Best Practices

**✅ DO:**
```tsx
// Use regular function
function Component({ prop }: Props) {
  return <div>{prop}</div>;
}

// Or arrow function
const Component = ({ prop }: Props) => {
  return <div>{prop}</div>;
};

// Explicit children when needed
interface Props {
  children?: React.ReactNode;
}

// Use generics
function List<T>(props: ListProps<T>) {
  // ...
}
```

**❌ DON'T:**
```tsx
// Avoid React.FC
const Component: React.FC<Props> = ({ prop }) => {
  return <div>{prop}</div>;
};

// Don't rely on implicit children (React <18)
const Card: React.FC<{ title: string }> = ({ title }) => {
  // children is implicit but unclear
};
```

---

#### Interview Tips

✅ **Key Points:**
- React.FC is a TypeScript type for function components
- **Not recommended** in modern React development
- Issues: implicit children (old), no generics, defaultProps problems
- Modern approach: regular functions with explicit prop types
- React 18 removed implicit children from React.FC

✅ **When to Mention:**
- TypeScript + React discussions
- Component typing strategies
- Legacy code vs modern patterns
- Generic components
- Migration from older React code

✅ **Common Follow-ups:**
- "Why is React.FC not recommended?"
- "What changed in React 18?"
- "How to type children properly?"
- "What about generic components?"
- "Should we migrate away from React.FC?"

✅ **Perfect Answer Structure:**
1. Define: React.FC is a TypeScript type for function components
2. Why not: Implicit children confusing, no generics, issues with defaults
3. React 18: Removed implicit children but still not preferred
4. Modern: Use regular functions with explicit types
5. Example: Show side-by-side comparison
6. Conclusion: Regular functions are best practice

</details>

---

### 123. How do you type props with TypeScript?

<details>
<summary>View Answer</summary>

**Typing props** in TypeScript ensures type safety, provides autocomplete, and catches errors at compile time. There are multiple ways to define and use props types depending on your component needs.

#### Basic Props Typing

**Using Interface (Recommended):**
```tsx
import React from 'react';

interface UserCardProps {
  name: string;
  age: number;
  email: string;
}

function UserCard({ name, age, email }: UserCardProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
    </div>
  );
}

export default UserCard;
```

**Using Type:**
```tsx
type UserCardProps = {
  name: string;
  age: number;
  email: string;
};

function UserCard({ name, age, email }: UserCardProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
    </div>
  );
}
```

**Interface vs Type:**
- **Interface**: Can be extended, better for objects, slightly better error messages
- **Type**: Can use unions/intersections, more flexible
- **Recommendation**: Use `interface` for props (React convention)

---

#### Optional Props

```tsx
interface ButtonProps {
  label: string;           // Required
  onClick: () => void;     // Required
  disabled?: boolean;      // Optional
  variant?: 'primary' | 'secondary';  // Optional with union type
  icon?: React.ReactNode;  // Optional React element
}

function Button({ 
  label, 
  onClick, 
  disabled = false,        // Default value
  variant = 'primary',     // Default value
  icon 
}: ButtonProps) {
  return (
    <button 
      onClick={onClick} 
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {icon && <span className="icon">{icon}</span>}
      {label}
    </button>
  );
}
```

---

#### Props with Children

**Using React.ReactNode:**
```tsx
import React from 'react';

interface CardProps {
  title: string;
  subtitle?: string;
  children: React.ReactNode;  // Accepts any valid React child
}

function Card({ title, subtitle, children }: CardProps) {
  return (
    <div className="card">
      <div className="card-header">
        <h2>{title}</h2>
        {subtitle && <p>{subtitle}</p>}
      </div>
      <div className="card-body">{children}</div>
    </div>
  );
}

// Usage
<Card title="My Card">
  <p>Content goes here</p>
  <button>Click me</button>
</Card>
```

**Using PropsWithChildren (React 18+):**
```tsx
import React, { PropsWithChildren } from 'react';

interface CardProps {
  title: string;
  variant?: 'default' | 'highlighted';
}

// PropsWithChildren automatically adds children: React.ReactNode
function Card({ title, variant = 'default', children }: PropsWithChildren<CardProps>) {
  return (
    <div className={`card card-${variant}`}>
      <h2>{title}</h2>
      <div>{children}</div>
    </div>
  );
}
```

**Specific Children Types:**
```tsx
// Only string children
interface TextProps {
  children: string;
}

function Text({ children }: TextProps) {
  return <p>{children}</p>;
}

// Only single React element
interface WrapperProps {
  children: React.ReactElement;
}

function Wrapper({ children }: WrapperProps) {
  return <div className="wrapper">{children}</div>;
}

// Render function as children
interface DataProviderProps {
  children: (data: string) => React.ReactNode;
}

function DataProvider({ children }: DataProviderProps) {
  const data = "Hello";
  return <div>{children(data)}</div>;
}

// Usage
<DataProvider>
  {(data) => <p>{data}</p>}
</DataProvider>
```

---

#### Function Props (Event Handlers)

```tsx
interface FormProps {
  onSubmit: (data: FormData) => void;
  onChange?: (field: string, value: string) => void;
  onValidate?: (data: FormData) => boolean;
}

interface FormData {
  email: string;
  password: string;
}

function LoginForm({ onSubmit, onChange, onValidate }: FormProps) {
  const [formData, setFormData] = useState<FormData>({
    email: '',
    password: '',
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    if (onValidate && !onValidate(formData)) {
      return;
    }
    
    onSubmit(formData);
  };

  const handleChange = (field: keyof FormData, value: string) => {
    setFormData(prev => ({ ...prev, [field]: value }));
    onChange?.(field, value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        value={formData.email}
        onChange={(e) => handleChange('email', e.target.value)}
      />
      <input 
        type="password"
        value={formData.password}
        onChange={(e) => handleChange('password', e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

#### Union Types for Props

```tsx
type ButtonVariant = 'primary' | 'secondary' | 'danger';
type ButtonSize = 'small' | 'medium' | 'large';

interface ButtonProps {
  label: string;
  variant: ButtonVariant;
  size: ButtonSize;
  onClick: () => void;
}

function Button({ label, variant, size, onClick }: ButtonProps) {
  return (
    <button 
      className={`btn btn-${variant} btn-${size}`}
      onClick={onClick}
    >
      {label}
    </button>
  );
}

// TypeScript enforces valid values
<Button label="Click" variant="primary" size="medium" onClick={() => {}} />
<Button label="Click" variant="invalid" size="huge" onClick={() => {}} />  // ❌ Error!
```

---

#### Extending Props

**Extending Interfaces:**
```tsx
interface BaseProps {
  id: string;
  className?: string;
}

interface ButtonProps extends BaseProps {
  label: string;
  onClick: () => void;
}

interface InputProps extends BaseProps {
  value: string;
  onChange: (value: string) => void;
}

function Button({ id, className, label, onClick }: ButtonProps) {
  return (
    <button id={id} className={className} onClick={onClick}>
      {label}
    </button>
  );
}
```

**Intersection Types:**
```tsx
type BaseProps = {
  id: string;
  className?: string;
};

type ButtonProps = BaseProps & {
  label: string;
  onClick: () => void;
};

function Button(props: ButtonProps) {
  return <button {...props}>{props.label}</button>;
}
```

---

#### Extending HTML Element Props

**Button Extending HTML Attributes:**
```tsx
import React from 'react';

interface CustomButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
  loading?: boolean;
}

function CustomButton({ 
  variant = 'primary', 
  loading = false,
  children,
  disabled,
  ...rest  // All other button attributes
}: CustomButtonProps) {
  return (
    <button 
      className={`btn btn-${variant}`}
      disabled={disabled || loading}
      {...rest}
    >
      {loading ? 'Loading...' : children}
    </button>
  );
}

// Can use any standard button props!
<CustomButton 
  variant="primary"
  onClick={() => {}}
  type="submit"
  aria-label="Submit"
  data-testid="submit-btn"
>
  Submit
</CustomButton>
```

**Input Extending HTML Attributes:**
```tsx
interface CustomInputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

function CustomInput({ label, error, ...rest }: CustomInputProps) {
  return (
    <div>
      <label>{label}</label>
      <input {...rest} />
      {error && <span className="error">{error}</span>}
    </div>
  );
}

// Accepts all standard input props
<CustomInput 
  label="Email"
  type="email"
  placeholder="Enter email"
  required
  maxLength={50}
/>
```

**Div Extending HTML Attributes:**
```tsx
interface CardProps extends React.HTMLAttributes<HTMLDivElement> {
  title: string;
  variant?: 'default' | 'highlighted';
}

function Card({ title, variant = 'default', children, ...rest }: CardProps) {
  return (
    <div className={`card card-${variant}`} {...rest}>
      <h2>{title}</h2>
      {children}
    </div>
  );
}

// Can use any div attributes
<Card 
  title="My Card"
  onClick={() => console.log('clicked')}
  onMouseEnter={() => {}}
  data-id="card-1"
>
  Content
</Card>
```

---

#### Omit and Pick Utility Types

**Using Omit:**
```tsx
interface UserProps {
  id: string;
  name: string;
  email: string;
  password: string;
}

// Create props without password
type PublicUserProps = Omit<UserProps, 'password'>;

function UserCard({ id, name, email }: PublicUserProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
}
```

**Using Pick:**
```tsx
interface UserProps {
  id: string;
  name: string;
  email: string;
  age: number;
  address: string;
}

// Only pick specific props
type UserSummaryProps = Pick<UserProps, 'name' | 'email'>;

function UserSummary({ name, email }: UserSummaryProps) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}
```

**Combining with Custom Props:**
```tsx
interface UserProps {
  id: string;
  name: string;
  email: string;
}

type UserCardProps = Pick<UserProps, 'name' | 'email'> & {
  onEdit: () => void;
  isActive: boolean;
};

function UserCard({ name, email, onEdit, isActive }: UserCardProps) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      <span>{isActive ? 'Active' : 'Inactive'}</span>
      <button onClick={onEdit}>Edit</button>
    </div>
  );
}
```

---

#### Props with Generic Types

**Generic List Component:**
```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
  emptyMessage?: string;
}

function List<T>({
  items,
  renderItem,
  keyExtractor,
  emptyMessage = 'No items',
}: ListProps<T>) {
  if (items.length === 0) {
    return <p>{emptyMessage}</p>;
  }

  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// Usage with different types
interface User {
  id: string;
  name: string;
}

const users: User[] = [{ id: '1', name: 'John' }];

<List
  items={users}
  renderItem={(user) => <span>{user.name}</span>}  // user is typed as User!
  keyExtractor={(user) => user.id}
/>
```

**Generic Form Field:**
```tsx
interface FormFieldProps<T> {
  value: T;
  onChange: (value: T) => void;
  label: string;
  validator?: (value: T) => boolean;
}

function FormField<T>({
  value,
  onChange,
  label,
  validator,
}: FormFieldProps<T>) {
  const isValid = validator ? validator(value) : true;

  return (
    <div>
      <label>{label}</label>
      <input
        value={String(value)}
        onChange={(e) => onChange(e.target.value as T)}
      />
      {!isValid && <span className="error">Invalid</span>}
    </div>
  );
}
```

---

#### Discriminated Unions (Conditional Props)

**Button with Loading or Success State:**
```tsx
type ButtonState =
  | { state: 'idle'; onClick: () => void }
  | { state: 'loading' }
  | { state: 'success'; message: string };

type ButtonProps = ButtonState & {
  label: string;
};

function Button(props: ButtonProps) {
  const { label, state } = props;

  if (state === 'loading') {
    return <button disabled>Loading...</button>;
  }

  if (state === 'success') {
    return <button disabled>{props.message}</button>;
  }

  // state is 'idle', onClick is available
  return <button onClick={props.onClick}>{label}</button>;
}

// Usage
<Button label="Click" state="idle" onClick={() => {}} />
<Button label="Click" state="loading" />
<Button label="Click" state="success" message="Done!" />
```

**Form Field with Different Types:**
```tsx
type FieldProps =
  | {
      type: 'text';
      value: string;
      onChange: (value: string) => void;
    }
  | {
      type: 'number';
      value: number;
      onChange: (value: number) => void;
    }
  | {
      type: 'boolean';
      value: boolean;
      onChange: (value: boolean) => void;
    };

function Field(props: FieldProps) {
  if (props.type === 'text') {
    return (
      <input
        type="text"
        value={props.value}
        onChange={(e) => props.onChange(e.target.value)}
      />
    );
  }

  if (props.type === 'number') {
    return (
      <input
        type="number"
        value={props.value}
        onChange={(e) => props.onChange(Number(e.target.value))}
      />
    );
  }

  return (
    <input
      type="checkbox"
      checked={props.value}
      onChange={(e) => props.onChange(e.target.checked)}
    />
  );
}
```

---

#### Props with Refs

**Component Accepting Ref:**
```tsx
import React, { forwardRef } from 'react';

interface InputProps {
  label: string;
  placeholder?: string;
}

// forwardRef with generics: forwardRef<RefType, PropsType>
const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, placeholder }, ref) => {
    return (
      <div>
        <label>{label}</label>
        <input ref={ref} placeholder={placeholder} />
      </div>
    );
  }
);

Input.displayName = 'Input';

// Usage
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleFocus = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <Input ref={inputRef} label="Email" />
      <button onClick={handleFocus}>Focus</button>
    </div>
  );
}
```

---

#### Props Validation with Readonly

```tsx
interface UserProps {
  readonly id: string;        // Cannot be modified
  readonly name: string;
  age: number;
}

function UserCard(props: Readonly<UserProps>) {
  // props.id = '123';  // ❌ Error: Cannot assign to 'id' because it is a read-only property
  
  return (
    <div>
      <h2>{props.name}</h2>
      <p>Age: {props.age}</p>
    </div>
  );
}
```

---

#### Default Props Pattern

**Using Default Parameters (Recommended):**
```tsx
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  size?: 'small' | 'medium' | 'large';
}

function Button({
  label,
  variant = 'primary',
  disabled = false,
  size = 'medium',
}: ButtonProps) {
  return (
    <button 
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
    >
      {label}
    </button>
  );
}
```

**Using Object Destructuring with Defaults:**
```tsx
function Button(props: ButtonProps) {
  const {
    label,
    variant = 'primary',
    disabled = false,
    size = 'medium',
  } = props;

  return (
    <button className={`btn btn-${variant} btn-${size}`} disabled={disabled}>
      {label}
    </button>
  );
}
```

---

#### Complex Props Example

```tsx
import React, { ReactNode } from 'react';

type Status = 'idle' | 'loading' | 'success' | 'error';

interface DataTableProps<T> {
  // Data
  data: T[];
  columns: ColumnConfig<T>[];
  
  // State
  status: Status;
  error?: string;
  
  // Callbacks
  onRowClick?: (row: T) => void;
  onSort?: (key: keyof T) => void;
  onFilter?: (key: keyof T, value: string) => void;
  
  // Customization
  emptyMessage?: string;
  loadingComponent?: ReactNode;
  errorComponent?: (error: string) => ReactNode;
  
  // Pagination
  pagination?: {
    page: number;
    pageSize: number;
    total: number;
    onPageChange: (page: number) => void;
  };
}

interface ColumnConfig<T> {
  key: keyof T;
  header: string;
  sortable?: boolean;
  filterable?: boolean;
  render?: (value: T[keyof T], row: T) => ReactNode;
}

function DataTable<T extends { id: string | number }>(props: DataTableProps<T>) {
  // Implementation
  return <div>Table</div>;
}
```

---

#### Best Practices

**1. Use Interface for Props:**
```tsx
// GOOD
interface Props {
  name: string;
}

// Also OK
type Props = {
  name: string;
};
```

**2. Name Props Interface After Component:**
```tsx
// GOOD
interface ButtonProps { /* ... */ }
function Button(props: ButtonProps) { /* ... */ }

// Or use Props suffix
interface UserCardProps { /* ... */ }
```

**3. Use Optional Props with ?:**
```tsx
interface Props {
  required: string;
  optional?: string;  // Use ? for optional
}
```

**4. Provide Default Values:**
```tsx
function Button({ variant = 'primary', disabled = false }: ButtonProps) {
  // Defaults in destructuring
}
```

**5. Use Specific Types:**
```tsx
// GOOD - Specific types
interface Props {
  status: 'idle' | 'loading' | 'success';
  count: number;
}

// AVOID - Too generic
interface Props {
  status: string;
  count: any;
}
```

---

#### Interview Tips

✅ **Key Points:**
- Use `interface` or `type` to define props
- `interface` preferred for props by convention
- Use `?` for optional props, provide defaults
- Use `React.ReactNode` for children
- Extend HTML attributes with `React.ButtonHTMLAttributes` etc.
- Use generics for reusable components

✅ **When to Mention:**
- TypeScript + React setup
- Component prop validation
- Type safety discussions
- Reusable component design
- API design for components

✅ **Common Follow-ups:**
- "Interface vs Type for props?"
- "How to make props optional?"
- "How to extend HTML element props?"
- "How to type children?"
- "What are generic component props?"

✅ **Perfect Answer Structure:**
1. Basic: Show interface with required props
2. Optional: Use `?` and default values
3. Children: Use `React.ReactNode`
4. Extend: Extend HTML attributes for native elements
5. Advanced: Generic props, discriminated unions
6. Best practices: Use interface, specific types, defaults

</details>

---

### 124. How do you type useState with TypeScript?

<details>
<summary>View Answer</summary>

**Typing useState** with TypeScript ensures type safety for state values and setter functions. TypeScript can often infer types automatically, but explicit typing is recommended for complex types.

#### Basic useState Typing

**Type Inference (Simple Types):**
```tsx
import { useState } from 'react';

// TypeScript infers type automatically
const [count, setCount] = useState(0);  // inferred as number
const [name, setName] = useState('John');  // inferred as string
const [isActive, setIsActive] = useState(true);  // inferred as boolean

// Can only set values of the inferred type
setCount(10);  // ✅ OK
setCount('10');  // ❌ Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

**Explicit Typing:**
```tsx
// Explicit type annotation
const [count, setCount] = useState<number>(0);
const [name, setName] = useState<string>('John');
const [isActive, setIsActive] = useState<boolean>(true);
```

---

#### Typing with Objects

**Object State:**
```tsx
import { useState } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

function UserProfile() {
  // Explicit type for object state
  const [user, setUser] = useState<User>({
    id: '1',
    name: 'John',
    email: 'john@example.com',
    age: 30,
  });

  // Update user
  const updateUser = () => {
    setUser({
      ...user,
      name: 'Jane',
    });
  };

  // TypeScript ensures type safety
  // setUser({ id: '1' });  // ❌ Error: Missing properties

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

**Partial Updates:**
```tsx
interface FormData {
  email: string;
  password: string;
  rememberMe: boolean;
}

function LoginForm() {
  const [formData, setFormData] = useState<FormData>({
    email: '',
    password: '',
    rememberMe: false,
  });

  // Update specific field
  const updateField = (field: keyof FormData, value: FormData[typeof field]) => {
    setFormData(prev => ({
      ...prev,
      [field]: value,
    }));
  };

  return (
    <form>
      <input
        value={formData.email}
        onChange={(e) => updateField('email', e.target.value)}
      />
      <input
        type="password"
        value={formData.password}
        onChange={(e) => updateField('password', e.target.value)}
      />
      <input
        type="checkbox"
        checked={formData.rememberMe}
        onChange={(e) => updateField('rememberMe', e.target.checked)}
      />
    </form>
  );
}
```

---

#### Typing with Null or Undefined

**Nullable State:**
```tsx
interface User {
  id: string;
  name: string;
}

function UserProfile() {
  // State can be User or null
  const [user, setUser] = useState<User | null>(null);

  // Fetch user
  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => setUser(data));
  }, []);

  // Type guard needed
  if (!user) {
    return <div>Loading...</div>;
  }

  // TypeScript knows user is User here
  return <div>{user.name}</div>;
}
```

**Optional vs Undefined:**
```tsx
interface Product {
  id: string;
  name: string;
}

// Option 1: null as initial value
const [product, setProduct] = useState<Product | null>(null);

// Option 2: undefined as initial value
const [product, setProduct] = useState<Product | undefined>(undefined);

// Option 3: undefined without initial value
const [product, setProduct] = useState<Product>();  // undefined by default

// Checking for null/undefined
if (product) {
  // product is Product here
  console.log(product.name);
}
```

---

#### Typing with Arrays

**Array State:**
```tsx
interface Todo {
  id: string;
  text: string;
  completed: boolean;
}

function TodoList() {
  // Array of objects
  const [todos, setTodos] = useState<Todo[]>([]);

  // Add todo
  const addTodo = (text: string) => {
    const newTodo: Todo = {
      id: Date.now().toString(),
      text,
      completed: false,
    };
    setTodos([...todos, newTodo]);
  };

  // Remove todo
  const removeTodo = (id: string) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  // Toggle completed
  const toggleTodo = (id: string) => {
    setTodos(
      todos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          {todo.text}
          <button onClick={() => removeTodo(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

**Array of Primitives:**
```tsx
function TagInput() {
  const [tags, setTags] = useState<string[]>([]);

  const addTag = (tag: string) => {
    setTags([...tags, tag]);
  };

  const removeTag = (index: number) => {
    setTags(tags.filter((_, i) => i !== index));
  };

  return (
    <div>
      {tags.map((tag, index) => (
        <span key={index}>
          {tag}
          <button onClick={() => removeTag(index)}>×</button>
        </span>
      ))}
    </div>
  );
}
```

---

#### Union Types

**Multiple Possible Types:**
```tsx
type Status = 'idle' | 'loading' | 'success' | 'error';

function DataFetcher() {
  const [status, setStatus] = useState<Status>('idle');

  // TypeScript ensures only valid values
  setStatus('loading');  // ✅ OK
  setStatus('pending');  // ❌ Error: "pending" is not assignable to type Status

  return <div>Status: {status}</div>;
}
```

**Complex Union Types:**
```tsx
type LoadingState = { status: 'loading' };
type SuccessState = { status: 'success'; data: string };
type ErrorState = { status: 'error'; error: string };

type State = LoadingState | SuccessState | ErrorState;

function DataComponent() {
  const [state, setState] = useState<State>({ status: 'loading' });

  // Type-safe state updates
  setState({ status: 'success', data: 'Hello' });
  setState({ status: 'error', error: 'Failed' });
  // setState({ status: 'success' });  // ❌ Error: missing 'data'

  // Type narrowing
  if (state.status === 'loading') {
    return <div>Loading...</div>;
  }

  if (state.status === 'error') {
    return <div>Error: {state.error}</div>;  // TypeScript knows 'error' exists
  }

  // TypeScript knows this is SuccessState
  return <div>Data: {state.data}</div>;
}
```

---

#### Generic State

**Using Generic Types:**
```tsx
interface ApiResponse<T> {
  data: T;
  loading: boolean;
  error: string | null;
}

interface User {
  id: string;
  name: string;
}

function UserProfile() {
  const [response, setResponse] = useState<ApiResponse<User>>({
    data: null!,  // or use User | null
    loading: true,
    error: null,
  });

  useEffect(() => {
    fetch('/api/user')
      .then(res => res.json())
      .then(data => {
        setResponse({
          data,
          loading: false,
          error: null,
        });
      })
      .catch(error => {
        setResponse({
          data: null!,
          loading: false,
          error: error.message,
        });
      });
  }, []);

  return <div>{response.data?.name}</div>;
}
```

---

#### Function State

**State as Function:**
```tsx
type Operation = (a: number, b: number) => number;

function Calculator() {
  // Store function in state (must wrap in function)
  const [operation, setOperation] = useState<Operation>(() => (
    (a, b) => a + b
  ));

  // Update function
  const setAddition = () => {
    setOperation(() => (a: number, b: number) => a + b);
  };

  const setMultiplication = () => {
    setOperation(() => (a: number, b: number) => a * b);
  };

  const result = operation(5, 3);

  return (
    <div>
      <p>Result: {result}</p>
      <button onClick={setAddition}>Add</button>
      <button onClick={setMultiplication}>Multiply</button>
    </div>
  );
}
```

---

#### Lazy Initial State

**Complex Initial State:**
```tsx
interface ExpensiveData {
  items: string[];
  total: number;
}

function DataComponent() {
  // Lazy initialization - function only runs once
  const [data, setData] = useState<ExpensiveData>(() => {
    // Expensive computation
    const items = Array.from({ length: 1000 }, (_, i) => `Item ${i}`);
    return {
      items,
      total: items.length,
    };
  });

  return <div>Total: {data.total}</div>;
}
```

**Reading from localStorage:**
```tsx
interface Settings {
  theme: 'light' | 'dark';
  language: string;
}

function App() {
  const [settings, setSettings] = useState<Settings>(() => {
    const stored = localStorage.getItem('settings');
    return stored ? JSON.parse(stored) : {
      theme: 'light',
      language: 'en',
    };
  });

  // Save to localStorage on change
  useEffect(() => {
    localStorage.setItem('settings', JSON.stringify(settings));
  }, [settings]);

  return <div>Theme: {settings.theme}</div>;
}
```

---

#### Common Patterns

**Form State:**
```tsx
interface FormValues {
  username: string;
  email: string;
  password: string;
}

function RegisterForm() {
  const [values, setValues] = useState<FormValues>({
    username: '',
    email: '',
    password: '',
  });

  const handleChange = (field: keyof FormValues) => (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    setValues({
      ...values,
      [field]: e.target.value,
    });
  };

  return (
    <form>
      <input value={values.username} onChange={handleChange('username')} />
      <input value={values.email} onChange={handleChange('email')} />
      <input value={values.password} onChange={handleChange('password')} type="password" />
    </form>
  );
}
```

**Modal State:**
```tsx
interface ModalState {
  isOpen: boolean;
  content: React.ReactNode | null;
}

function App() {
  const [modal, setModal] = useState<ModalState>({
    isOpen: false,
    content: null,
  });

  const openModal = (content: React.ReactNode) => {
    setModal({ isOpen: true, content });
  };

  const closeModal = () => {
    setModal({ isOpen: false, content: null });
  };

  return (
    <div>
      <button onClick={() => openModal(<p>Modal content</p>)}>Open</button>
      {modal.isOpen && (
        <div className="modal">
          {modal.content}
          <button onClick={closeModal}>Close</button>
        </div>
      )}
    </div>
  );
}
```

**Async Data Loading:**
```tsx
interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function useAsync<T>() {
  const [state, setState] = useState<AsyncState<T>>({
    data: null,
    loading: false,
    error: null,
  });

  const execute = async (promise: Promise<T>) => {
    setState({ data: null, loading: true, error: null });
    try {
      const data = await promise;
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ data: null, loading: false, error: error as Error });
    }
  };

  return { ...state, execute };
}

// Usage
interface User {
  id: string;
  name: string;
}

function UserProfile({ userId }: { userId: string }) {
  const { data: user, loading, error, execute } = useAsync<User>();

  useEffect(() => {
    execute(fetch(`/api/users/${userId}`).then(r => r.json()));
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!user) return null;

  return <div>{user.name}</div>;
}
```

---

#### Best Practices

**1. Let TypeScript Infer Simple Types:**
```tsx
// GOOD - Inference
const [count, setCount] = useState(0);

// Unnecessary - Explicit
const [count, setCount] = useState<number>(0);
```

**2. Explicitly Type Complex Types:**
```tsx
// GOOD - Explicit for objects
interface User { id: string; name: string; }
const [user, setUser] = useState<User | null>(null);

// AVOID - Inference not clear
const [user, setUser] = useState(null);  // Type is null
```

**3. Use Union Types for States:**
```tsx
// GOOD
type Status = 'idle' | 'loading' | 'success' | 'error';
const [status, setStatus] = useState<Status>('idle');

// AVOID
const [status, setStatus] = useState('idle');  // Type is string
```

**4. Initialize with Correct Type:**
```tsx
// GOOD
const [items, setItems] = useState<string[]>([]);

// AVOID - Wrong type inference
const [items, setItems] = useState([]);  // Type is never[]
```

**5. Use Type Guards:**
```tsx
const [user, setUser] = useState<User | null>(null);

// GOOD - Type guard
if (user) {
  console.log(user.name);  // TypeScript knows user is User
}

// AVOID - Non-null assertion
console.log(user!.name);  // Unsafe
```

---

#### Common Mistakes

**❌ Mistake 1: Not Typing Null States**
```tsx
// BAD - TypeScript thinks user is always defined
const [user, setUser] = useState({ id: '', name: '' });

// GOOD - Allow null
const [user, setUser] = useState<User | null>(null);
```

**❌ Mistake 2: Wrong Array Type**
```tsx
// BAD - Type is never[]
const [items, setItems] = useState([]);
items.push('item');  // ❌ Error

// GOOD - Explicit type
const [items, setItems] = useState<string[]>([]);
```

**❌ Mistake 3: Not Using Union Types**
```tsx
// BAD - Type is string (too generic)
const [status, setStatus] = useState('idle');
setStatus('invalid');  // ✅ No error, but should be!

// GOOD - Union type
type Status = 'idle' | 'loading' | 'success';
const [status, setStatus] = useState<Status>('idle');
setStatus('invalid');  // ❌ Error
```

---

#### Interview Tips

✅ **Key Points:**
- TypeScript infers simple types automatically
- Explicitly type objects, nullable states, arrays
- Use union types for fixed set of values
- Use `Type | null` for nullable state
- Initialize arrays with `useState<T[]>([])`

✅ **When to Mention:**
- TypeScript + React setup
- State management discussions
- Type safety importance
- Preventing runtime errors
- Complex state structures

✅ **Common Follow-ups:**
- "When do you need explicit types?"
- "How to type nullable state?"
- "How to type arrays in useState?"
- "What about union types?"
- "How to handle complex state objects?"

✅ **Perfect Answer Structure:**
1. Basic: TypeScript infers simple types automatically
2. Explicit: Use explicit types for objects and nullable states
3. Null: `useState<User | null>(null)` for nullable
4. Arrays: `useState<Type[]>([])` for arrays
5. Union: Use union types for fixed values
6. Best practices: Explicit for complex, inference for simple

</details>

---

### 125. How do you type useRef with TypeScript?

<details>
<summary>View Answer</summary>

**Typing useRef** with TypeScript requires understanding the difference between refs for DOM elements vs refs for mutable values. The type parameter determines what the ref can hold and how it behaves.

#### Three Main Use Cases

1. **DOM element refs** - `useRef<HTMLElement>(null)`
2. **Mutable value refs** - `useRef<Type>(initialValue)`
3. **Read-only refs** - For forwarding refs

---

#### 1. DOM Element Refs

**Basic Input Ref:**
```tsx
import { useRef, useEffect } from 'react';

function InputComponent() {
  // Type: React.RefObject<HTMLInputElement>
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    // Must check for null before accessing
    if (inputRef.current) {
      inputRef.current.focus();
    }
    
    // Or use optional chaining
    inputRef.current?.focus();
  }, []);

  return <input ref={inputRef} type="text" />;
}
```

**Different HTML Elements:**
```tsx
function RefExamples() {
  // Input element
  const inputRef = useRef<HTMLInputElement>(null);
  
  // Button element
  const buttonRef = useRef<HTMLButtonElement>(null);
  
  // Div element
  const divRef = useRef<HTMLDivElement>(null);
  
  // Textarea element
  const textareaRef = useRef<HTMLTextAreaElement>(null);
  
  // Select element
  const selectRef = useRef<HTMLSelectElement>(null);
  
  // Canvas element
  const canvasRef = useRef<HTMLCanvasElement>(null);
  
  // Video element
  const videoRef = useRef<HTMLVideoElement>(null);
  
  // Any HTML element
  const elementRef = useRef<HTMLElement>(null);

  const handleFocus = () => {
    inputRef.current?.focus();
    buttonRef.current?.click();
    
    // Access element properties
    console.log(divRef.current?.clientWidth);
    console.log(textareaRef.current?.value);
    console.log(selectRef.current?.selectedIndex);
    
    // Canvas operations
    const ctx = canvasRef.current?.getContext('2d');
    
    // Video controls
    videoRef.current?.play();
  };

  return (
    <>
      <input ref={inputRef} />
      <button ref={buttonRef}>Click</button>
      <div ref={divRef}>Content</div>
      <textarea ref={textareaRef} />
      <select ref={selectRef}>
        <option>Option 1</option>
      </select>
      <canvas ref={canvasRef} />
      <video ref={videoRef} />
    </>
  );
}
```

**Practical Example - Form Focus:**
```tsx
function LoginForm() {
  const emailRef = useRef<HTMLInputElement>(null);
  const passwordRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    
    // Validate and focus on error
    if (!emailRef.current?.value) {
      emailRef.current?.focus();
      return;
    }
    
    if (!passwordRef.current?.value) {
      passwordRef.current?.focus();
      return;
    }
    
    // Submit form
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={emailRef} type="email" placeholder="Email" />
      <input ref={passwordRef} type="password" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

#### 2. Mutable Value Refs

**Storing Mutable Values:**
```tsx
import { useRef, useEffect } from 'react';

function Timer() {
  // Type: React.MutableRefObject<number | null>
  const intervalRef = useRef<number | null>(null);
  const countRef = useRef<number>(0);

  useEffect(() => {
    // Store interval ID
    intervalRef.current = window.setInterval(() => {
      countRef.current += 1;
      console.log(countRef.current);
    }, 1000);

    return () => {
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);

  return <div>Check console for count</div>;
}
```

**Previous Value Tracking:**
```tsx
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Storing Timeout/Interval IDs:**
```tsx
function Debouncer() {
  const [value, setValue] = useState('');
  const [debouncedValue, setDebouncedValue] = useState('');
  const timeoutRef = useRef<NodeJS.Timeout | null>(null);

  useEffect(() => {
    // Clear previous timeout
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }

    // Set new timeout
    timeoutRef.current = setTimeout(() => {
      setDebouncedValue(value);
    }, 500);

    return () => {
      if (timeoutRef.current) {
        clearTimeout(timeoutRef.current);
      }
    };
  }, [value]);

  return (
    <div>
      <input value={value} onChange={(e) => setValue(e.target.value)} />
      <p>Debounced: {debouncedValue}</p>
    </div>
  );
}
```

**Storing Callback References:**
```tsx
interface WebSocketMessage {
  type: string;
  data: any;
}

function WebSocketComponent() {
  const wsRef = useRef<WebSocket | null>(null);
  const onMessageRef = useRef<((msg: WebSocketMessage) => void) | null>(null);

  useEffect(() => {
    wsRef.current = new WebSocket('ws://localhost:8080');

    wsRef.current.onmessage = (event) => {
      const message: WebSocketMessage = JSON.parse(event.data);
      onMessageRef.current?.(message);
    };

    return () => {
      wsRef.current?.close();
    };
  }, []);

  const setMessageHandler = (handler: (msg: WebSocketMessage) => void) => {
    onMessageRef.current = handler;
  };

  return <div>WebSocket Component</div>;
}
```

---

#### 3. Ref Forwarding

**forwardRef with TypeScript:**
```tsx
import { forwardRef, useRef } from 'react';

interface InputProps {
  label: string;
  placeholder?: string;
}

// forwardRef<RefType, PropsType>
const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, placeholder }, ref) => {
    return (
      <div>
        <label>{label}</label>
        <input ref={ref} placeholder={placeholder} />
      </div>
    );
  }
);

Input.displayName = 'Input';

// Usage
function Form() {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleFocus = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <Input ref={inputRef} label="Email" placeholder="Enter email" />
      <button onClick={handleFocus}>Focus Input</button>
    </div>
  );
}
```

**useImperativeHandle with TypeScript:**
```tsx
import { forwardRef, useImperativeHandle, useRef } from 'react';

interface FancyInputProps {
  label: string;
}

// Define the methods exposed via ref
interface FancyInputHandle {
  focus: () => void;
  clear: () => void;
  getValue: () => string;
  setValue: (value: string) => void;
}

const FancyInput = forwardRef<FancyInputHandle, FancyInputProps>(
  ({ label }, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);

    useImperativeHandle(ref, () => ({
      focus: () => {
        inputRef.current?.focus();
      },
      clear: () => {
        if (inputRef.current) {
          inputRef.current.value = '';
        }
      },
      getValue: () => {
        return inputRef.current?.value || '';
      },
      setValue: (value: string) => {
        if (inputRef.current) {
          inputRef.current.value = value;
        }
      },
    }));

    return (
      <div>
        <label>{label}</label>
        <input ref={inputRef} />
      </div>
    );
  }
);

FancyInput.displayName = 'FancyInput';

// Usage
function Form() {
  const inputRef = useRef<FancyInputHandle>(null);

  const handleSubmit = () => {
    const value = inputRef.current?.getValue();
    console.log('Value:', value);
    inputRef.current?.clear();
  };

  const handlePrefill = () => {
    inputRef.current?.setValue('test@example.com');
    inputRef.current?.focus();
  };

  return (
    <div>
      <FancyInput ref={inputRef} label="Email" />
      <button onClick={handleSubmit}>Submit</button>
      <button onClick={handlePrefill}>Prefill</button>
    </div>
  );
}
```

---

#### 4. RefObject vs MutableRefObject

**Understanding the Difference:**
```tsx
import { useRef, MutableRefObject, RefObject } from 'react';

function RefTypes() {
  // RefObject<T> - .current is readonly (for DOM refs)
  const inputRef: RefObject<HTMLInputElement> = useRef<HTMLInputElement>(null);
  // inputRef.current = document.createElement('input');  // ❌ Error: readonly

  // MutableRefObject<T> - .current is mutable (for values)
  const countRef: MutableRefObject<number> = useRef<number>(0);
  countRef.current = 10;  // ✅ OK

  // When initialized with null, it's RefObject
  const ref1 = useRef<HTMLDivElement>(null);
  // Type: RefObject<HTMLDivElement>
  // ref1.current = div;  // ❌ Error

  // When initialized with non-null, it's MutableRefObject
  const ref2 = useRef<number>(0);
  // Type: MutableRefObject<number>
  ref2.current = 5;  // ✅ OK

  return <div>Ref Types Example</div>;
}
```

---

#### 5. Complex Ref Examples

**Canvas Ref:**
```tsx
function Canvas() {
  const canvasRef = useRef<HTMLCanvasElement>(null);

  useEffect(() => {
    const canvas = canvasRef.current;
    if (!canvas) return;

    const ctx = canvas.getContext('2d');
    if (!ctx) return;

    // Draw on canvas
    ctx.fillStyle = 'blue';
    ctx.fillRect(0, 0, 200, 200);
  }, []);

  return <canvas ref={canvasRef} width={400} height={400} />;
}
```

**Video Ref with Controls:**
```tsx
function VideoPlayer({ src }: { src: string }) {
  const videoRef = useRef<HTMLVideoElement>(null);

  const play = () => {
    videoRef.current?.play();
  };

  const pause = () => {
    videoRef.current?.pause();
  };

  const seek = (time: number) => {
    if (videoRef.current) {
      videoRef.current.currentTime = time;
    }
  };

  const setVolume = (volume: number) => {
    if (videoRef.current) {
      videoRef.current.volume = Math.max(0, Math.min(1, volume));
    }
  };

  return (
    <div>
      <video ref={videoRef} src={src} />
      <button onClick={play}>Play</button>
      <button onClick={pause}>Pause</button>
      <button onClick={() => seek(10)}>Skip to 10s</button>
      <button onClick={() => setVolume(0.5)}>50% Volume</button>
    </div>
  );
}
```

**Scroll Position Tracking:**
```tsx
function ScrollTracker() {
  const containerRef = useRef<HTMLDivElement>(null);
  const scrollPositionRef = useRef<{ x: number; y: number }>({ x: 0, y: 0 });

  const handleScroll = () => {
    if (containerRef.current) {
      scrollPositionRef.current = {
        x: containerRef.current.scrollLeft,
        y: containerRef.current.scrollTop,
      };
    }
  };

  const scrollToTop = () => {
    containerRef.current?.scrollTo({ top: 0, behavior: 'smooth' });
  };

  const restoreScrollPosition = () => {
    if (containerRef.current) {
      containerRef.current.scrollLeft = scrollPositionRef.current.x;
      containerRef.current.scrollTop = scrollPositionRef.current.y;
    }
  };

  return (
    <div>
      <div
        ref={containerRef}
        onScroll={handleScroll}
        style={{ height: 400, overflow: 'auto' }}
      >
        {/* Long content */}
      </div>
      <button onClick={scrollToTop}>Scroll to Top</button>
      <button onClick={restoreScrollPosition}>Restore Position</button>
    </div>
  );
}
```

**Multiple Refs in List:**
```tsx
function ItemList({ items }: { items: string[] }) {
  const itemRefs = useRef<Array<HTMLDivElement | null>>([]);

  const scrollToItem = (index: number) => {
    itemRefs.current[index]?.scrollIntoView({ behavior: 'smooth' });
  };

  return (
    <div>
      {items.map((item, index) => (
        <div
          key={index}
          ref={(el) => (itemRefs.current[index] = el)}
        >
          {item}
        </div>
      ))}
      <button onClick={() => scrollToItem(5)}>Scroll to Item 5</button>
    </div>
  );
}
```

---

#### 6. Custom Hooks with Refs

**useClickOutside Hook:**
```tsx
import { useEffect, useRef } from 'react';

function useClickOutside<T extends HTMLElement = HTMLElement>(
  handler: () => void
) {
  const ref = useRef<T>(null);

  useEffect(() => {
    const handleClick = (event: MouseEvent) => {
      if (ref.current && !ref.current.contains(event.target as Node)) {
        handler();
      }
    };

    document.addEventListener('mousedown', handleClick);
    return () => {
      document.removeEventListener('mousedown', handleClick);
    };
  }, [handler]);

  return ref;
}

// Usage
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useClickOutside<HTMLDivElement>(() => {
    setIsOpen(false);
  });

  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && (
        <div className="dropdown-menu">
          <div>Option 1</div>
          <div>Option 2</div>
        </div>
      )}
    </div>
  );
}
```

**useHover Hook:**
```tsx
function useHover<T extends HTMLElement = HTMLElement>() {
  const [isHovered, setIsHovered] = useState(false);
  const ref = useRef<T>(null);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const handleMouseEnter = () => setIsHovered(true);
    const handleMouseLeave = () => setIsHovered(false);

    element.addEventListener('mouseenter', handleMouseEnter);
    element.addEventListener('mouseleave', handleMouseLeave);

    return () => {
      element.removeEventListener('mouseenter', handleMouseEnter);
      element.removeEventListener('mouseleave', handleMouseLeave);
    };
  }, []);

  return [ref, isHovered] as const;
}

// Usage
function HoverCard() {
  const [ref, isHovered] = useHover<HTMLDivElement>();

  return (
    <div ref={ref} style={{ background: isHovered ? 'blue' : 'gray' }}>
      Hover me!
    </div>
  );
}
```

**useIntersectionObserver Hook:**
```tsx
function useIntersectionObserver<T extends HTMLElement = HTMLElement>(
  options?: IntersectionObserverInit
) {
  const [isIntersecting, setIsIntersecting] = useState(false);
  const ref = useRef<T>(null);

  useEffect(() => {
    const element = ref.current;
    if (!element) return;

    const observer = new IntersectionObserver(([entry]) => {
      setIsIntersecting(entry.isIntersecting);
    }, options);

    observer.observe(element);

    return () => {
      observer.disconnect();
    };
  }, [options]);

  return [ref, isIntersecting] as const;
}

// Usage
function LazyImage({ src }: { src: string }) {
  const [ref, isVisible] = useIntersectionObserver<HTMLDivElement>();

  return (
    <div ref={ref}>
      {isVisible ? (
        <img src={src} alt="Lazy loaded" />
      ) : (
        <div>Loading...</div>
      )}
    </div>
  );
}
```

---

#### Best Practices

**1. Always Initialize DOM Refs with null:**
```tsx
// GOOD
const inputRef = useRef<HTMLInputElement>(null);

// AVOID - TypeScript error
const inputRef = useRef<HTMLInputElement>();
```

**2. Check for null Before Accessing:**
```tsx
// GOOD - Optional chaining
inputRef.current?.focus();

// GOOD - Explicit check
if (inputRef.current) {
  inputRef.current.focus();
}

// AVOID - Non-null assertion (unsafe)
inputRef.current!.focus();
```

**3. Use Specific Element Types:**
```tsx
// GOOD - Specific
const inputRef = useRef<HTMLInputElement>(null);

// AVOID - Too generic
const inputRef = useRef<HTMLElement>(null);
```

**4. Mutable Refs Don't Need null:**
```tsx
// GOOD - For values
const countRef = useRef<number>(0);

// AVOID - Unnecessary null
const countRef = useRef<number | null>(null);
```

**5. Type Custom Ref Interfaces:**
```tsx
// GOOD - Type the handle
interface CustomHandle {
  focus: () => void;
  clear: () => void;
}

const ref = useRef<CustomHandle>(null);
```

---

#### Common Patterns Summary

```tsx
// DOM element ref
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();

// Mutable value ref
const countRef = useRef<number>(0);
countRef.current = 10;

// Timeout/Interval ref
const timeoutRef = useRef<NodeJS.Timeout | null>(null);

// forwardRef
const Input = forwardRef<HTMLInputElement, Props>((props, ref) => {
  return <input ref={ref} />;
});

// useImperativeHandle
interface Handle {
  focus: () => void;
}
const Component = forwardRef<Handle, Props>((props, ref) => {
  useImperativeHandle(ref, () => ({ focus: () => {} }));
});

// Custom hook with ref
function useHook<T extends HTMLElement>() {
  const ref = useRef<T>(null);
  return ref;
}
```

---

#### Interview Tips

✅ **Key Points:**
- DOM refs: `useRef<HTMLElement>(null)` - returns RefObject
- Mutable refs: `useRef<Type>(initialValue)` - returns MutableRefObject
- Always check for null before accessing DOM refs
- Use forwardRef to pass refs to child components
- useImperativeHandle to expose custom methods via ref

✅ **When to Mention:**
- DOM manipulation in React
- Focus management
- Storing mutable values without re-renders
- Integrating with third-party libraries
- Custom hooks that need element references

✅ **Common Follow-ups:**
- "What's the difference between RefObject and MutableRefObject?"
- "When to use useRef vs useState?"
- "How to forward refs to child components?"
- "What is useImperativeHandle?"
- "How to store previous values with useRef?"

✅ **Perfect Answer Structure:**
1. Define: useRef creates mutable refs that persist across renders
2. DOM refs: `useRef<HTMLInputElement>(null)` for elements
3. Mutable refs: `useRef<number>(0)` for values
4. Check null: Always use optional chaining or check
5. forwardRef: Pass refs to child components
6. Use cases: Focus, intervals, previous values, DOM measurements

</details>

---

### 126. How do you type useContext with TypeScript?

<details>
<summary>View Answer</summary>

**Typing useContext** with TypeScript ensures type safety for context values and prevents common errors like using context outside of providers. The key is properly typing the Context creation and providing good defaults or undefined handling.

#### Basic Context Typing

**Simple Context:**
```tsx
import { createContext, useContext, ReactNode } from 'react';

interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

// Create context with type
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

interface ThemeProviderProps {
  children: ReactNode;
}

export function ThemeProvider({ children }: ThemeProviderProps) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  const value: ThemeContextType = {
    theme,
    toggleTheme,
  };

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom hook with type safety
export function useTheme(): ThemeContextType {
  const context = useContext(ThemeContext);
  
  if (context === undefined) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  
  return context;
}

// Usage
function Button() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme}>
      Current theme: {theme}
    </button>
  );
}
```

---

#### Two Approaches to Context Typing

**Approach 1: Context with undefined (Recommended):**
```tsx
interface UserContextType {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

// Context starts as undefined
const UserContext = createContext<UserContextType | undefined>(undefined);

export function UserProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    const userData = await loginApi(email, password);
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
}

// Hook with runtime check
export function useUser(): UserContextType {
  const context = useContext(UserContext);
  
  if (context === undefined) {
    throw new Error('useUser must be used within UserProvider');
  }
  
  return context;
}
```

**Approach 2: Context with Default Value (Less Common):**
```tsx
interface CounterContextType {
  count: number;
  increment: () => void;
  decrement: () => void;
}

// Default value (usually not practical)
const defaultValue: CounterContextType = {
  count: 0,
  increment: () => console.warn('No provider'),
  decrement: () => console.warn('No provider'),
};

const CounterContext = createContext<CounterContextType>(defaultValue);

export function CounterProvider({ children }: { children: ReactNode }) {
  const [count, setCount] = useState(0);

  const value: CounterContextType = {
    count,
    increment: () => setCount(c => c + 1),
    decrement: () => setCount(c => c - 1),
  };

  return (
    <CounterContext.Provider value={value}>
      {children}
    </CounterContext.Provider>
  );
}

// Hook doesn't need undefined check
export function useCounter(): CounterContextType {
  return useContext(CounterContext);
}
```

---

#### Real-World Examples

**Authentication Context:**
```tsx
import { createContext, useContext, useState, useEffect, ReactNode } from 'react';

interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

interface AuthContextType {
  user: User | null;
  loading: boolean;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  updateUser: (data: Partial<User>) => void;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  // Check for existing session on mount
  useEffect(() => {
    checkSession();
  }, []);

  const checkSession = async () => {
    try {
      const userData = await fetch('/api/me').then(r => r.json());
      setUser(userData);
    } catch {
      setUser(null);
    } finally {
      setLoading(false);
    }
  };

  const login = async (email: string, password: string) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email, password }),
    });
    const userData = await response.json();
    setUser(userData);
  };

  const logout = async () => {
    await fetch('/api/logout', { method: 'POST' });
    setUser(null);
  };

  const updateUser = (data: Partial<User>) => {
    setUser(prev => prev ? { ...prev, ...data } : null);
  };

  const value: AuthContextType = {
    user,
    loading,
    isAuthenticated: !!user,
    login,
    logout,
    updateUser,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  
  if (context === undefined) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  
  return context;
}

// Usage
function Profile() {
  const { user, logout, isAuthenticated } = useAuth();

  if (!isAuthenticated || !user) {
    return <div>Please log in</div>;
  }

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <p>Role: {user.role}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

**Settings Context with Persistence:**
```tsx
interface Settings {
  theme: 'light' | 'dark' | 'auto';
  language: string;
  notifications: boolean;
  fontSize: 'small' | 'medium' | 'large';
}

interface SettingsContextType {
  settings: Settings;
  updateSettings: (updates: Partial<Settings>) => void;
  resetSettings: () => void;
}

const defaultSettings: Settings = {
  theme: 'light',
  language: 'en',
  notifications: true,
  fontSize: 'medium',
};

const SettingsContext = createContext<SettingsContextType | undefined>(undefined);

export function SettingsProvider({ children }: { children: ReactNode }) {
  const [settings, setSettings] = useState<Settings>(() => {
    const stored = localStorage.getItem('settings');
    return stored ? JSON.parse(stored) : defaultSettings;
  });

  // Persist to localStorage
  useEffect(() => {
    localStorage.setItem('settings', JSON.stringify(settings));
  }, [settings]);

  const updateSettings = (updates: Partial<Settings>) => {
    setSettings(prev => ({ ...prev, ...updates }));
  };

  const resetSettings = () => {
    setSettings(defaultSettings);
  };

  const value: SettingsContextType = {
    settings,
    updateSettings,
    resetSettings,
  };

  return (
    <SettingsContext.Provider value={value}>
      {children}
    </SettingsContext.Provider>
  );
}

export function useSettings(): SettingsContextType {
  const context = useContext(SettingsContext);
  
  if (context === undefined) {
    throw new Error('useSettings must be used within SettingsProvider');
  }
  
  return context;
}

// Usage
function SettingsPanel() {
  const { settings, updateSettings } = useSettings();

  return (
    <div>
      <select
        value={settings.theme}
        onChange={(e) => updateSettings({ theme: e.target.value as Settings['theme'] })}
      >
        <option value="light">Light</option>
        <option value="dark">Dark</option>
        <option value="auto">Auto</option>
      </select>
      
      <input
        type="checkbox"
        checked={settings.notifications}
        onChange={(e) => updateSettings({ notifications: e.target.checked })}
      />
    </div>
  );
}
```

**Shopping Cart Context:**
```tsx
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
  image: string;
}

interface CartContextType {
  items: CartItem[];
  total: number;
  itemCount: number;
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
}

const CartContext = createContext<CartContextType | undefined>(undefined);

export function CartProvider({ children }: { children: ReactNode }) {
  const [items, setItems] = useState<CartItem[]>([]);

  const addItem = (item: Omit<CartItem, 'quantity'>) => {
    setItems(prev => {
      const existing = prev.find(i => i.id === item.id);
      if (existing) {
        return prev.map(i =>
          i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
        );
      }
      return [...prev, { ...item, quantity: 1 }];
    });
  };

  const removeItem = (id: string) => {
    setItems(prev => prev.filter(item => item.id !== id));
  };

  const updateQuantity = (id: string, quantity: number) => {
    if (quantity <= 0) {
      removeItem(id);
      return;
    }
    setItems(prev =>
      prev.map(item => (item.id === id ? { ...item, quantity } : item))
    );
  };

  const clearCart = () => {
    setItems([]);
  };

  const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const itemCount = items.reduce((sum, item) => sum + item.quantity, 0);

  const value: CartContextType = {
    items,
    total,
    itemCount,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
  };

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

export function useCart(): CartContextType {
  const context = useContext(CartContext);
  
  if (context === undefined) {
    throw new Error('useCart must be used within CartProvider');
  }
  
  return context;
}

// Usage
function CartSummary() {
  const { items, total, itemCount, removeItem } = useCart();

  return (
    <div>
      <h2>Cart ({itemCount} items)</h2>
      {items.map(item => (
        <div key={item.id}>
          <span>{item.name} x {item.quantity}</span>
          <span>${item.price * item.quantity}</span>
          <button onClick={() => removeItem(item.id)}>Remove</button>
        </div>
      ))}
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

---

#### Multiple Contexts

**Combining Multiple Contexts:**
```tsx
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <SettingsProvider>
          <CartProvider>
            <Router />
          </CartProvider>
        </SettingsProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

// Using multiple contexts
function Header() {
  const { user, logout } = useAuth();
  const { theme, toggleTheme } = useTheme();
  const { itemCount } = useCart();

  return (
    <header>
      <span>Welcome, {user?.name}</span>
      <span>Cart: {itemCount}</span>
      <button onClick={toggleTheme}>Theme: {theme}</button>
      <button onClick={logout}>Logout</button>
    </header>
  );
}
```

**Compose Provider Component:**
```tsx
interface ProviderProps {
  children: ReactNode;
}

const providers = [
  AuthProvider,
  ThemeProvider,
  SettingsProvider,
  CartProvider,
];

export function AppProviders({ children }: ProviderProps) {
  return providers.reduceRight(
    (acc, Provider) => <Provider>{acc}</Provider>,
    children
  );
}

// Usage
function App() {
  return (
    <AppProviders>
      <Router />
    </AppProviders>
  );
}
```

---

#### Generic Context

**Generic Context Factory:**
```tsx
import { createContext, useContext, ReactNode } from 'react';

function createGenericContext<T>() {
  const Context = createContext<T | undefined>(undefined);

  function useGenericContext(): T {
    const context = useContext(Context);
    if (context === undefined) {
      throw new Error('useContext must be used within Provider');
    }
    return context;
  }

  return [Context.Provider, useGenericContext] as const;
}

// Usage
interface TodoContextType {
  todos: Todo[];
  addTodo: (text: string) => void;
}

const [TodoProvider, useTodos] = createGenericContext<TodoContextType>();

export function TodoContextProvider({ children }: { children: ReactNode }) {
  const [todos, setTodos] = useState<Todo[]>([]);

  const addTodo = (text: string) => {
    setTodos([...todos, { id: Date.now().toString(), text, completed: false }]);
  };

  return (
    <TodoProvider value={{ todos, addTodo }}>
      {children}
    </TodoProvider>
  );
}

// Use the custom hook
function TodoList() {
  const { todos } = useTodos();
  return <ul>{todos.map(todo => <li key={todo.id}>{todo.text}</li>)}</ul>;
}
```

---

#### Context with Reducer

**Context + useReducer Pattern:**
```tsx
import { createContext, useContext, useReducer, ReactNode } from 'react';

interface State {
  count: number;
  history: number[];
}

type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' }
  | { type: 'set'; payload: number };

interface CounterContextType {
  state: State;
  dispatch: React.Dispatch<Action>;
}

const CounterContext = createContext<CounterContextType | undefined>(undefined);

function counterReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return {
        count: state.count + 1,
        history: [...state.history, state.count + 1],
      };
    case 'decrement':
      return {
        count: state.count - 1,
        history: [...state.history, state.count - 1],
      };
    case 'reset':
      return { count: 0, history: [0] };
    case 'set':
      return {
        count: action.payload,
        history: [...state.history, action.payload],
      };
    default:
      return state;
  }
}

export function CounterProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    history: [0],
  });

  return (
    <CounterContext.Provider value={{ state, dispatch }}>
      {children}
    </CounterContext.Provider>
  );
}

export function useCounter(): CounterContextType {
  const context = useContext(CounterContext);
  
  if (context === undefined) {
    throw new Error('useCounter must be used within CounterProvider');
  }
  
  return context;
}

// Usage
function Counter() {
  const { state, dispatch } = useCounter();

  return (
    <div>
      <p>Count: {state.count}</p>
      <p>History: {state.history.join(', ')}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

---

#### Best Practices

**1. Always Type Context:**
```tsx
// GOOD
const Context = createContext<ContextType | undefined>(undefined);

// AVOID
const Context = createContext(null);  // Type is null
```

**2. Create Custom Hook:**
```tsx
// GOOD - Custom hook with error handling
export function useMyContext(): ContextType {
  const context = useContext(MyContext);
  if (context === undefined) {
    throw new Error('useMyContext must be used within Provider');
  }
  return context;
}

// AVOID - Direct useContext usage
export const MyContext = createContext<ContextType | undefined>(undefined);
// Consumers have to handle undefined everywhere
```

**3. Separate Context and Provider:**
```tsx
// GOOD - Export only hook and provider
export function ThemeProvider({ children }) { /* ... */ }
export function useTheme() { /* ... */ }

// Don't export context directly
const ThemeContext = createContext<ThemeContextType | undefined>(undefined);
```

**4. Type Provider Props:**
```tsx
// GOOD
interface ProviderProps {
  children: ReactNode;
  initialValue?: string;
}

export function MyProvider({ children, initialValue }: ProviderProps) {
  // ...
}
```

**5. Use Meaningful Names:**
```tsx
// GOOD - Clear naming
const AuthContext = createContext<AuthContextType | undefined>(undefined);
export function useAuth() { /* ... */ }

// AVOID - Generic names
const Context = createContext<Type | undefined>(undefined);
export function useContext() { /* ... */ }  // Name conflict!
```

---

#### Common Patterns Summary

```tsx
// 1. Create context with type
interface ContextType {
  value: string;
  setValue: (value: string) => void;
}
const Context = createContext<ContextType | undefined>(undefined);

// 2. Create provider
export function Provider({ children }: { children: ReactNode }) {
  const [value, setValue] = useState('');
  return <Context.Provider value={{ value, setValue }}>{children}</Context.Provider>;
}

// 3. Create custom hook
export function useCustomContext(): ContextType {
  const context = useContext(Context);
  if (!context) throw new Error('Must be used within Provider');
  return context;
}

// 4. Usage
function Component() {
  const { value, setValue } = useCustomContext();
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
}
```

---

#### Interview Tips

✅ **Key Points:**
- Type context with `createContext<Type | undefined>(undefined)`
- Create custom hook with error handling
- Don't export context directly, only hook and provider
- Use meaningful names for context and hooks
- Combine with useReducer for complex state

✅ **When to Mention:**
- Global state management
- Prop drilling solutions
- Theme/auth/settings management
- TypeScript best practices
- Component composition patterns

✅ **Common Follow-ups:**
- "Why use undefined instead of default value?"
- "How to combine multiple contexts?"
- "Context vs Redux?"
- "How to optimize context performance?"
- "When to use Context vs prop drilling?"

✅ **Perfect Answer Structure:**
1. Define: createContext with type or undefined
2. Provider: Component that provides the value
3. Custom hook: Wrapper with error handling
4. Example: Show auth or theme context
5. Best practices: Custom hook, type safety, error handling
6. Advanced: Multiple contexts, reducer pattern

</details>

---

### 127. What are generic components in TypeScript?

<details>
<summary>View Answer</summary>

**Generic components** in TypeScript allow you to create reusable components that work with multiple types while maintaining type safety. They use type parameters to make the component flexible and type-safe at the same time.

#### Basic Generic Component

**Simple List Component:**
```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor?: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor ? keyExtractor(item) : index}>
          {renderItem(item)}
        </li>
      ))}
    </ul>
  );
}

// Usage with different types
interface User {
  id: number;
  name: string;
}

interface Product {
  id: string;
  title: string;
  price: number;
}

function App() {
  const users: User[] = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' },
  ];

  const products: Product[] = [
    { id: 'p1', title: 'Laptop', price: 999 },
    { id: 'p2', title: 'Phone', price: 599 },
  ];

  return (
    <div>
      {/* Type inferred as List<User> */}
      <List
        items={users}
        renderItem={(user) => <span>{user.name}</span>}
        keyExtractor={(user) => user.id}
      />

      {/* Type inferred as List<Product> */}
      <List
        items={products}
        renderItem={(product) => (
          <div>
            {product.title} - ${product.price}
          </div>
        )}
        keyExtractor={(product) => product.id}
      />
    </div>
  );
}
```

---

#### Generic Components Patterns

**1. Select/Dropdown Component:**
```tsx
interface Option<T> {
  value: T;
  label: string;
  disabled?: boolean;
}

interface SelectProps<T> {
  options: Option<T>[];
  value: T;
  onChange: (value: T) => void;
  placeholder?: string;
}

function Select<T extends string | number>({
  options,
  value,
  onChange,
  placeholder,
}: SelectProps<T>) {
  return (
    <select
      value={value as string | number}
      onChange={(e) => {
        const selectedValue = e.target.value;
        const option = options.find(
          (opt) => String(opt.value) === selectedValue
        );
        if (option) {
          onChange(option.value);
        }
      }}
    >
      {placeholder && <option value="">{placeholder}</option>}
      {options.map((option) => (
        <option
          key={String(option.value)}
          value={option.value as string | number}
          disabled={option.disabled}
        >
          {option.label}
        </option>
      ))}
    </select>
  );
}

// Usage
type Status = 'active' | 'inactive' | 'pending';

function StatusSelector() {
  const [status, setStatus] = useState<Status>('active');

  const options: Option<Status>[] = [
    { value: 'active', label: 'Active' },
    { value: 'inactive', label: 'Inactive' },
    { value: 'pending', label: 'Pending' },
  ];

  return (
    <Select
      options={options}
      value={status}
      onChange={setStatus}
      placeholder="Select status"
    />
  );
}

// With numbers
function AgeSelector() {
  const [age, setAge] = useState<number>(18);

  const options: Option<number>[] = [
    { value: 18, label: '18+' },
    { value: 21, label: '21+' },
    { value: 65, label: '65+' },
  ];

  return <Select options={options} value={age} onChange={setAge} />;
}
```

**2. Table Component:**
```tsx
interface Column<T> {
  key: keyof T;
  header: string;
  render?: (value: T[keyof T], item: T) => React.ReactNode;
  width?: string | number;
}

interface TableProps<T> {
  data: T[];
  columns: Column<T>[];
  onRowClick?: (item: T) => void;
  keyExtractor: (item: T) => string | number;
}

function Table<T>({
  data,
  columns,
  onRowClick,
  keyExtractor,
}: TableProps<T>) {
  return (
    <table>
      <thead>
        <tr>
          {columns.map((column) => (
            <th key={String(column.key)} style={{ width: column.width }}>
              {column.header}
            </th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map((item) => (
          <tr
            key={keyExtractor(item)}
            onClick={() => onRowClick?.(item)}
            style={{ cursor: onRowClick ? 'pointer' : 'default' }}
          >
            {columns.map((column) => {
              const value = item[column.key];
              return (
                <td key={String(column.key)}>
                  {column.render ? column.render(value, item) : String(value)}
                </td>
              );
            })}
          </tr>
        ))}
      </tbody>
    </table>
  );
}

// Usage
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
  createdAt: Date;
}

function UserTable() {
  const users: User[] = [
    {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
      role: 'admin',
      createdAt: new Date('2024-01-15'),
    },
    {
      id: 2,
      name: 'Jane Smith',
      email: 'jane@example.com',
      role: 'user',
      createdAt: new Date('2024-02-20'),
    },
  ];

  const columns: Column<User>[] = [
    {
      key: 'id',
      header: 'ID',
      width: 60,
    },
    {
      key: 'name',
      header: 'Name',
      render: (value) => <strong>{value}</strong>,
    },
    {
      key: 'email',
      header: 'Email',
    },
    {
      key: 'role',
      header: 'Role',
      render: (value) => (
        <span className={`badge-${value}`}>{value.toUpperCase()}</span>
      ),
    },
    {
      key: 'createdAt',
      header: 'Created',
      render: (value) => (value as Date).toLocaleDateString(),
    },
  ];

  const handleRowClick = (user: User) => {
    console.log('Clicked user:', user);
  };

  return (
    <Table
      data={users}
      columns={columns}
      keyExtractor={(user) => user.id}
      onRowClick={handleRowClick}
    />
  );
}
```

**3. Form Field Component:**
```tsx
interface FormFieldProps<T> {
  label: string;
  value: T;
  onChange: (value: T) => void;
  error?: string;
  disabled?: boolean;
  render: (props: {
    value: T;
    onChange: (value: T) => void;
    disabled?: boolean;
  }) => React.ReactNode;
}

function FormField<T>({
  label,
  value,
  onChange,
  error,
  disabled,
  render,
}: FormFieldProps<T>) {
  return (
    <div className="form-field">
      <label>{label}</label>
      {render({ value, onChange, disabled })}
      {error && <span className="error">{error}</span>}
    </div>
  );
}

// Usage
function UserForm() {
  const [name, setName] = useState<string>('');
  const [age, setAge] = useState<number>(0);
  const [active, setActive] = useState<boolean>(false);

  return (
    <form>
      <FormField
        label="Name"
        value={name}
        onChange={setName}
        render={({ value, onChange }) => (
          <input
            type="text"
            value={value}
            onChange={(e) => onChange(e.target.value)}
          />
        )}
      />

      <FormField
        label="Age"
        value={age}
        onChange={setAge}
        render={({ value, onChange }) => (
          <input
            type="number"
            value={value}
            onChange={(e) => onChange(Number(e.target.value))}
          />
        )}
      />

      <FormField
        label="Active"
        value={active}
        onChange={setActive}
        render={({ value, onChange }) => (
          <input
            type="checkbox"
            checked={value}
            onChange={(e) => onChange(e.target.checked)}
          />
        )}
      />
    </form>
  );
}
```

**4. Data Fetcher Component:**
```tsx
import { useState, useEffect } from 'react';

interface DataFetcherProps<T> {
  url: string;
  children: (state: {
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

  const fetchData = async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch(url);
      if (!response.ok) throw new Error('Failed to fetch');
      const json = await response.json();
      setData(json);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [url]);

  return <>{children({ data, loading, error, refetch: fetchData })}</>;
}

// Usage
interface User {
  id: number;
  name: string;
  email: string;
}

function UserList() {
  return (
    <DataFetcher<User[]> url="/api/users">
      {({ data, loading, error, refetch }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error.message}</div>;
        if (!data) return <div>No data</div>;

        return (
          <div>
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
      }}
    </DataFetcher>
  );
}
```

---

#### Advanced Generic Patterns

**5. Constrained Generics:**
```tsx
// Generic constrained to objects with an 'id' property
interface WithId {
  id: string | number;
}

interface ListProps<T extends WithId> {
  items: T[];
  onItemClick: (item: T) => void;
  selectedId?: T['id'];
}

function SelectableList<T extends WithId>({
  items,
  onItemClick,
  selectedId,
}: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li
          key={item.id}
          onClick={() => onItemClick(item)}
          className={item.id === selectedId ? 'selected' : ''}
        >
          {/* TypeScript knows item has 'id' */}
          ID: {item.id}
        </li>
      ))}
    </ul>
  );
}

// Usage - Any type with 'id' works
interface Product extends WithId {
  name: string;
  price: number;
}

function ProductSelector() {
  const products: Product[] = [
    { id: 1, name: 'Laptop', price: 999 },
    { id: 2, name: 'Phone', price: 599 },
  ];

  return (
    <SelectableList
      items={products}
      onItemClick={(product) => console.log(product.name)}
    />
  );
}
```

**6. Multiple Type Parameters:**
```tsx
interface MapperProps<TInput, TOutput> {
  items: TInput[];
  mapper: (item: TInput) => TOutput;
  children: (mappedItems: TOutput[]) => React.ReactNode;
}

function Mapper<TInput, TOutput>({
  items,
  mapper,
  children,
}: MapperProps<TInput, TOutput>) {
  const mappedItems = items.map(mapper);
  return <>{children(mappedItems)}</>;
}

// Usage
interface User {
  id: number;
  firstName: string;
  lastName: string;
}

interface UserDisplay {
  id: number;
  fullName: string;
}

function UsersList() {
  const users: User[] = [
    { id: 1, firstName: 'John', lastName: 'Doe' },
    { id: 2, firstName: 'Jane', lastName: 'Smith' },
  ];

  return (
    <Mapper<User, UserDisplay>
      items={users}
      mapper={(user) => ({
        id: user.id,
        fullName: `${user.firstName} ${user.lastName}`,
      })}
    >
      {(displayUsers) => (
        <ul>
          {displayUsers.map((user) => (
            <li key={user.id}>{user.fullName}</li>
          ))}
        </ul>
      )}
    </Mapper>
  );
}
```

**7. Generic with Default Types:**
```tsx
interface PaginatedListProps<T, TKey extends keyof T = keyof T> {
  items: T[];
  itemsPerPage: number;
  renderItem: (item: T) => React.ReactNode;
  keyField?: TKey;
}

function PaginatedList<T extends Record<string, any>>(
  { items, itemsPerPage, renderItem, keyField }: PaginatedListProps<T>
) {
  const [page, setPage] = useState(0);

  const startIndex = page * itemsPerPage;
  const endIndex = startIndex + itemsPerPage;
  const pageItems = items.slice(startIndex, endIndex);
  const totalPages = Math.ceil(items.length / itemsPerPage);

  return (
    <div>
      <div className="items">
        {pageItems.map((item, index) => (
          <div key={keyField ? String(item[keyField]) : index}>
            {renderItem(item)}
          </div>
        ))}
      </div>
      <div className="pagination">
        <button onClick={() => setPage(p => Math.max(0, p - 1))} disabled={page === 0}>
          Previous
        </button>
        <span>
          Page {page + 1} of {totalPages}
        </span>
        <button
          onClick={() => setPage(p => Math.min(totalPages - 1, p + 1))}
          disabled={page >= totalPages - 1}
        >
          Next
        </button>
      </div>
    </div>
  );
}

// Usage
interface Article {
  id: string;
  title: string;
  content: string;
}

function ArticleList() {
  const articles: Article[] = [/* ... */];

  return (
    <PaginatedList
      items={articles}
      itemsPerPage={10}
      keyField="id"
      renderItem={(article) => (
        <div>
          <h3>{article.title}</h3>
          <p>{article.content}</p>
        </div>
      )}
    />
  );
}
```

**8. Generic Hook:**
```tsx
import { useState } from 'react';

function useLocalStorage<T>(key: string, initialValue: T) {
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

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage<'light' | 'dark'>('theme', 'light');
  const [fontSize, setFontSize] = useLocalStorage<number>('fontSize', 16);
  const [user, setUser] = useLocalStorage<User | null>('user', null);

  return (
    <div>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <input
        type="number"
        value={fontSize}
        onChange={(e) => setFontSize(Number(e.target.value))}
      />
    </div>
  );
}
```

---

#### Real-World Example: Autocomplete

```tsx
import { useState, useEffect, useRef } from 'react';

interface AutocompleteProps<T> {
  options: T[];
  value: T | null;
  onChange: (value: T | null) => void;
  getOptionLabel: (option: T) => string;
  getOptionValue: (option: T) => string | number;
  filterOptions?: (options: T[], inputValue: string) => T[];
  placeholder?: string;
  disabled?: boolean;
}

function Autocomplete<T>({
  options,
  value,
  onChange,
  getOptionLabel,
  getOptionValue,
  filterOptions,
  placeholder = 'Search...',
  disabled = false,
}: AutocompleteProps<T>) {
  const [inputValue, setInputValue] = useState('');
  const [isOpen, setIsOpen] = useState(false);
  const [filteredOptions, setFilteredOptions] = useState<T[]>(options);
  const wrapperRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const filtered = filterOptions
      ? filterOptions(options, inputValue)
      : options.filter((option) =>
          getOptionLabel(option)
            .toLowerCase()
            .includes(inputValue.toLowerCase())
        );
    setFilteredOptions(filtered);
  }, [inputValue, options, filterOptions, getOptionLabel]);

  useEffect(() => {
    const handleClickOutside = (event: MouseEvent) => {
      if (wrapperRef.current && !wrapperRef.current.contains(event.target as Node)) {
        setIsOpen(false);
      }
    };

    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, []);

  const handleSelect = (option: T) => {
    onChange(option);
    setInputValue(getOptionLabel(option));
    setIsOpen(false);
  };

  const handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setInputValue(e.target.value);
    setIsOpen(true);
    if (!e.target.value) {
      onChange(null);
    }
  };

  return (
    <div ref={wrapperRef} className="autocomplete">
      <input
        type="text"
        value={inputValue}
        onChange={handleInputChange}
        onFocus={() => setIsOpen(true)}
        placeholder={placeholder}
        disabled={disabled}
      />
      {isOpen && filteredOptions.length > 0 && (
        <ul className="autocomplete-options">
          {filteredOptions.map((option) => {
            const optionValue = getOptionValue(option);
            return (
              <li
                key={optionValue}
                onClick={() => handleSelect(option)}
                className={
                  value && getOptionValue(value) === optionValue ? 'selected' : ''
                }
              >
                {getOptionLabel(option)}
              </li>
            );
          })}
        </ul>
      )}
    </div>
  );
}

// Usage
interface Country {
  code: string;
  name: string;
  population: number;
}

function CountrySelector() {
  const [selectedCountry, setSelectedCountry] = useState<Country | null>(null);

  const countries: Country[] = [
    { code: 'US', name: 'United States', population: 331000000 },
    { code: 'UK', name: 'United Kingdom', population: 67000000 },
    { code: 'CA', name: 'Canada', population: 38000000 },
  ];

  return (
    <Autocomplete<Country>
      options={countries}
      value={selectedCountry}
      onChange={setSelectedCountry}
      getOptionLabel={(country) => country.name}
      getOptionValue={(country) => country.code}
      placeholder="Select a country"
    />
  );
}
```

---

#### Best Practices

**1. Type Inference:**
```tsx
// GOOD - Type inferred from props
const users: User[] = [/* ... */];
<List items={users} renderItem={(user) => <div>{user.name}</div>} />

// AVOID - Explicit type parameter (unnecessary)
<List<User> items={users} renderItem={(user) => <div>{user.name}</div>} />
```

**2. Constraints:**
```tsx
// GOOD - Constrain generic types
function Component<T extends { id: string }>(props: { item: T }) {
  return <div>{props.item.id}</div>;
}

// AVOID - Unconstrained (less type safety)
function Component<T>(props: { item: T }) {
  return <div>{(props.item as any).id}</div>;  // Type assertion needed
}
```

**3. Default Types:**
```tsx
// GOOD - Provide sensible defaults
interface Props<T = string> {
  value: T;
}

// Can be used without type parameter
const comp1: Props = { value: 'hello' };  // T defaults to string
const comp2: Props<number> = { value: 42 };  // T explicitly number
```

---

#### Interview Tips

✅ **Key Points:**
- Generic components use type parameters: `<T>`
- Infer types from props when possible
- Constrain generics with `extends` for better type safety
- Use multiple type parameters when needed: `<TInput, TOutput>`
- Common use cases: Lists, tables, form fields, data fetchers

✅ **When to Mention:**
- Reusable component libraries
- Type-safe collections (lists, tables, dropdowns)
- Higher-order components
- Custom hooks with flexible types
- Component composition patterns

✅ **Common Follow-ups:**
- "How do you constrain generic types?"
- "When to use generics vs union types?"
- "How to provide default generic types?"
- "Can you use multiple type parameters?"
- "How do generics work with React.memo?"

✅ **Perfect Answer Structure:**
1. Define: Components with type parameters for flexibility
2. Basic example: Generic List component
3. Type inference: TypeScript infers T from props
4. Constraints: Use `extends` to require certain properties
5. Use cases: Lists, tables, form fields, data fetchers
6. Benefits: Type safety + reusability

</details>

---

### 128. How do you type children prop in TypeScript?

<details>
<summary>View Answer</summary>

**Typing the children prop** in TypeScript has evolved over time. The modern approach uses `ReactNode` from React types, which covers all valid React children including elements, strings, numbers, fragments, and portals.

#### Modern Approach (Recommended)

**Using ReactNode:**
```tsx
import { ReactNode } from 'react';

interface CardProps {
  title: string;
  children: ReactNode;
}

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">{children}</div>
    </div>
  );
}

// Usage - accepts any valid React content
function App() {
  return (
    <>
      {/* String children */}
      <Card title="Text">Hello World</Card>

      {/* Element children */}
      <Card title="Element">
        <p>This is a paragraph</p>
      </Card>

      {/* Multiple children */}
      <Card title="Multiple">
        <p>First paragraph</p>
        <p>Second paragraph</p>
        <button>Click me</button>
      </Card>

      {/* Fragment */}
      <Card title="Fragment">
        <>
          <span>One</span>
          <span>Two</span>
        </>
      </Card>

      {/* Conditional children */}
      <Card title="Conditional">
        {true && <p>Shown</p>}
        {false && <p>Hidden</p>}
      </Card>

      {/* Null/undefined */}
      <Card title="Empty">{null}</Card>
    </>
  );
}
```

---

#### All Children Types

**ReactNode (Most Common):**
```tsx
import { ReactNode } from 'react';

interface Props {
  children: ReactNode;  // Accepts everything
}

// ReactNode = ReactElement | string | number | ReactFragment |
//             ReactPortal | boolean | null | undefined

function Container({ children }: Props) {
  return <div>{children}</div>;
}
```

**ReactElement (Specific Elements):**
```tsx
import { ReactElement } from 'react';

interface Props {
  children: ReactElement;  // Only React elements (JSX)
}

function Wrapper({ children }: Props) {
  return <div className="wrapper">{children}</div>;
}

// Usage
<Wrapper>
  <p>This works</p>  {/* ✅ */}
</Wrapper>

<Wrapper>
  Hello  {/* ❌ Error: string not assignable to ReactElement */}
</Wrapper>
```

**ReactElement with Type Constraints:**
```tsx
import { ReactElement } from 'react';

interface ButtonProps {
  label: string;
}

function Button({ label }: ButtonProps) {
  return <button>{label}</button>;
}

interface Props {
  // Only accepts Button components
  children: ReactElement<ButtonProps>;
}

function ButtonGroup({ children }: Props) {
  return <div className="button-group">{children}</div>;
}

// Usage
<ButtonGroup>
  <Button label="Click" />  {/* ✅ */}
</ButtonGroup>

<ButtonGroup>
  <div>Not a button</div>  {/* ❌ Error */}
</ButtonGroup>
```

**JSX.Element:**
```tsx
interface Props {
  children: JSX.Element;  // Single JSX element
}

function SingleChild({ children }: Props) {
  return <div>{children}</div>;
}

// Usage
<SingleChild>
  <p>Single element</p>  {/* ✅ */}
</SingleChild>

<SingleChild>
  <p>One</p>
  <p>Two</p>  {/* ❌ Error: multiple children */}
</SingleChild>
```

**JSX.Element Array:**
```tsx
interface Props {
  children: JSX.Element[];  // Array of elements
}

function List({ children }: Props) {
  return (
    <ul>
      {children.map((child, index) => (
        <li key={index}>{child}</li>
      ))}
    </ul>
  );
}

// Usage
<List>
  {[<span key="1">Item 1</span>, <span key="2">Item 2</span>]}
</List>
```

---

#### PropsWithChildren Utility

**Using PropsWithChildren:**
```tsx
import { PropsWithChildren } from 'react';

// Automatically adds children: ReactNode
interface CardProps {
  title: string;
  variant?: 'primary' | 'secondary';
}

function Card({ title, variant = 'primary', children }: PropsWithChildren<CardProps>) {
  return (
    <div className={`card card-${variant}`}>
      <h2>{title}</h2>
      {children}
    </div>
  );
}

// Equivalent to:
interface CardPropsLong {
  title: string;
  variant?: 'primary' | 'secondary';
  children: ReactNode;
}
```

**Generic PropsWithChildren:**
```tsx
import { PropsWithChildren } from 'react';

function Wrapper<T>(props: PropsWithChildren<{ data: T }>) {
  return (
    <div>
      {props.children}
    </div>
  );
}

// Usage
<Wrapper data={{ id: 1, name: 'Test' }}>
  <p>Content</p>
</Wrapper>
```

---

#### Render Props Pattern

**Children as Function:**
```tsx
interface RenderProps<T> {
  data: T;
  loading: boolean;
  error: Error | null;
}

interface DataProviderProps<T> {
  url: string;
  children: (props: RenderProps<T>) => ReactNode;
}

function DataProvider<T>({ url, children }: DataProviderProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetch(url)
      .then((r) => r.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);

  return <>{children({ data: data!, loading, error })}</>;
}

// Usage
interface User {
  id: number;
  name: string;
}

function UserList() {
  return (
    <DataProvider<User[]> url="/api/users">
      {({ data, loading, error }) => {
        if (loading) return <div>Loading...</div>;
        if (error) return <div>Error: {error.message}</div>;
        return (
          <ul>
            {data?.map((user) => <li key={user.id}>{user.name}</li>)}
          </ul>
        );
      }}
    </DataProvider>
  );
}
```

---

#### Specific Children Patterns

**Optional Children:**
```tsx
interface Props {
  title: string;
  children?: ReactNode;  // Optional
}

function Card({ title, children }: Props) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children && <div className="content">{children}</div>}
    </div>
  );
}

// Usage
<Card title="With Content">
  <p>Content here</p>
</Card>

<Card title="Without Content" />  {/* No children */}
```

**Multiple Named Children:**
```tsx
import { ReactNode } from 'react';

interface LayoutProps {
  header: ReactNode;
  sidebar: ReactNode;
  content: ReactNode;
  footer?: ReactNode;
}

function Layout({ header, sidebar, content, footer }: LayoutProps) {
  return (
    <div className="layout">
      <header>{header}</header>
      <div className="main">
        <aside>{sidebar}</aside>
        <main>{content}</main>
      </div>
      {footer && <footer>{footer}</footer>}
    </div>
  );
}

// Usage
function App() {
  return (
    <Layout
      header={<h1>My App</h1>}
      sidebar={
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
      }
      content={
        <div>
          <h2>Welcome</h2>
          <p>Content goes here</p>
        </div>
      }
      footer={<p>&copy; 2024</p>}
    />
  );
}
```

**Children with Slots:**
```tsx
import { ReactNode, ReactElement } from 'react';

interface ModalProps {
  title: ReactNode;
  actions: ReactNode;
  children: ReactNode;
}

function Modal({ title, actions, children }: ModalProps) {
  return (
    <div className="modal">
      <div className="modal-header">
        <h2>{title}</h2>
      </div>
      <div className="modal-body">{children}</div>
      <div className="modal-footer">{actions}</div>
    </div>
  );
}

// Usage
function ConfirmDialog() {
  return (
    <Modal
      title="Confirm Action"
      actions={
        <>
          <button>Cancel</button>
          <button>Confirm</button>
        </>
      }
    >
      <p>Are you sure you want to proceed?</p>
    </Modal>
  );
}
```

---

#### Children Manipulation

**React.Children API:**
```tsx
import { Children, ReactNode, cloneElement, ReactElement } from 'react';

interface ListProps {
  children: ReactNode;
  spacing?: number;
}

function List({ children, spacing = 10 }: ListProps) {
  // Count children
  const count = Children.count(children);

  // Map over children
  const items = Children.map(children, (child, index) => {
    if (!child) return null;

    // Clone and add props
    return cloneElement(child as ReactElement, {
      style: {
        marginBottom: index < count - 1 ? spacing : 0,
      },
    });
  });

  return <div className="list">{items}</div>;
}

// Usage
function App() {
  return (
    <List spacing={20}>
      <div>Item 1</div>
      <div>Item 2</div>
      <div>Item 3</div>
    </List>
  );
}
```

**Filter Children:**
```tsx
import { Children, ReactNode, ReactElement, isValidElement } from 'react';

interface Tab {
  label: string;
  children: ReactNode;
}

function TabPanel({ label, children }: Tab) {
  return <div data-label={label}>{children}</div>;
}

interface TabsProps {
  children: ReactNode;
}

function Tabs({ children }: TabsProps) {
  const [activeTab, setActiveTab] = useState(0);

  // Filter valid TabPanel children
  const tabs = Children.toArray(children).filter(
    (child): child is ReactElement<Tab> =>
      isValidElement(child) && child.type === TabPanel
  );

  return (
    <div>
      <div className="tab-headers">
        {tabs.map((tab, index) => (
          <button
            key={index}
            onClick={() => setActiveTab(index)}
            className={activeTab === index ? 'active' : ''}
          >
            {tab.props.label}
          </button>
        ))}
      </div>
      <div className="tab-content">{tabs[activeTab]}</div>
    </div>
  );
}

// Usage
function App() {
  return (
    <Tabs>
      <TabPanel label="Tab 1">
        <p>Content 1</p>
      </TabPanel>
      <TabPanel label="Tab 2">
        <p>Content 2</p>
      </TabPanel>
      <TabPanel label="Tab 3">
        <p>Content 3</p>
      </TabPanel>
    </Tabs>
  );
}
```

---

#### Legacy Approaches (Avoid)

**FC Type (Deprecated):**
```tsx
import { FC } from 'react';

// OLD - React 17 and earlier
const Card: FC<{ title: string }> = ({ title, children }) => {
  // children implicit from FC
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
};

// MODERN - Explicitly type children
interface CardProps {
  title: string;
  children: ReactNode;
}

function Card({ title, children }: CardProps) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  );
}
```

---

#### Comparison Table

| Type | Accepts | Use Case |
|------|---------|----------|
| `ReactNode` | Everything | General purpose (recommended) |
| `ReactElement` | JSX elements only | When you need specific elements |
| `JSX.Element` | Single JSX element | Stricter than ReactElement |
| `JSX.Element[]` | Array of elements | Multiple specific children |
| `ReactNode \| undefined` | Optional children | When children may be omitted |
| `(props: T) => ReactNode` | Render prop function | Render props pattern |
| `PropsWithChildren<T>` | Adds ReactNode to type | Utility for adding children |

---

#### Best Practices

**1. Use ReactNode for General Cases:**
```tsx
// GOOD - Flexible
interface Props {
  children: ReactNode;
}

// AVOID - Too restrictive
interface Props {
  children: JSX.Element;
}
```

**2. Make Children Optional When Appropriate:**
```tsx
// GOOD - Optional when not always needed
interface Props {
  title: string;
  children?: ReactNode;
}

// AVOID - Required when it could be optional
interface Props {
  title: string;
  children: ReactNode;  // Forces always providing children
}
```

**3. Use Named Props for Multiple Content Areas:**
```tsx
// GOOD - Clear intent
interface Props {
  header: ReactNode;
  content: ReactNode;
  footer: ReactNode;
}

// AVOID - Single children with complex structure
interface Props {
  children: ReactNode;  // Unclear what structure is expected
}
```

**4. Type Function Children Explicitly:**
```tsx
// GOOD - Clear render prop type
interface Props<T> {
  data: T;
  children: (data: T) => ReactNode;
}

// AVOID - Any or unknown
interface Props {
  children: Function;  // No type safety
}
```

---

#### Common Patterns Summary

```tsx
import { ReactNode, PropsWithChildren } from 'react';

// 1. Basic children
interface Props1 {
  children: ReactNode;
}

// 2. Optional children
interface Props2 {
  children?: ReactNode;
}

// 3. With PropsWithChildren
type Props3 = PropsWithChildren<{ title: string }>;

// 4. Render prop
interface Props4<T> {
  data: T;
  children: (data: T) => ReactNode;
}

// 5. Multiple content slots
interface Props5 {
  header: ReactNode;
  content: ReactNode;
  footer?: ReactNode;
}

// 6. Specific element type
interface Props6 {
  children: ReactElement<ButtonProps>;
}
```

---

#### Interview Tips

✅ **Key Points:**
- `ReactNode` is the most flexible and recommended type
- Covers all valid React children (elements, strings, numbers, fragments, etc.)
- `PropsWithChildren<T>` utility adds children to existing props
- Use `ReactElement` when you need specific element types
- Make children optional with `?` when appropriate

✅ **When to Mention:**
- Component composition patterns
- Layout components
- Container components
- Render props pattern
- Children manipulation with React.Children API

✅ **Common Follow-ups:**
- "What's the difference between ReactNode and ReactElement?"
- "Should I use FC type?"
- "How to type render props?"
- "How to manipulate children with TypeScript?"
- "What is PropsWithChildren?"

✅ **Perfect Answer Structure:**
1. Define: ReactNode is the standard type for children
2. ReactNode includes: elements, strings, numbers, fragments, null, undefined
3. PropsWithChildren: Utility that adds children prop
4. Alternative: ReactElement for specific JSX elements
5. Optional: Use `children?:` when not always required
6. Render props: Type as `(props: T) => ReactNode`

</details>

---

### 129. How do you type event handlers in React with TypeScript?

<details>
<summary>View Answer</summary>

**Typing event handlers** in React with TypeScript requires using the correct event types from React. The key is understanding React's synthetic event system and using specific event types like `React.MouseEvent`, `React.ChangeEvent`, `React.FormEvent`, etc.

#### Basic Event Handler Types

**Mouse Events:**
```tsx
import { MouseEvent } from 'react';

function Button() {
  // Method 1: Inline type
  const handleClick = (event: MouseEvent<HTMLButtonElement>) => {
    console.log('Clicked!', event.currentTarget.value);
  };

  // Method 2: Type in JSX
  const handleHover = (e: MouseEvent<HTMLDivElement>) => {
    console.log('Hovered', e.clientX, e.clientY);
  };

  return (
    <div>
      <button onClick={handleClick}>Click Me</button>
      <div onMouseEnter={handleHover}>Hover Me</div>
    </div>
  );
}
```

**Change Events (Input):**
```tsx
import { ChangeEvent } from 'react';

function Form() {
  const handleInputChange = (event: ChangeEvent<HTMLInputElement>) => {
    console.log(event.target.value);
    console.log(event.target.checked); // For checkboxes
  };

  const handleTextareaChange = (event: ChangeEvent<HTMLTextAreaElement>) => {
    console.log(event.target.value);
  };

  const handleSelectChange = (event: ChangeEvent<HTMLSelectElement>) => {
    console.log(event.target.value);
    console.log(event.target.selectedIndex);
  };

  return (
    <form>
      <input type="text" onChange={handleInputChange} />
      <input type="checkbox" onChange={handleInputChange} />
      <textarea onChange={handleTextareaChange} />
      <select onChange={handleSelectChange}>
        <option value="1">Option 1</option>
        <option value="2">Option 2</option>
      </select>
    </form>
  );
}
```

**Form Events:**
```tsx
import { FormEvent } from 'react';

function LoginForm() {
  const handleSubmit = (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    
    const formData = new FormData(event.currentTarget);
    const email = formData.get('email');
    const password = formData.get('password');
    
    console.log({ email, password });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" required />
      <input name="password" type="password" required />
      <button type="submit">Login</button>
    </form>
  );
}
```

**Keyboard Events:**
```tsx
import { KeyboardEvent } from 'react';

function SearchInput() {
  const handleKeyDown = (event: KeyboardEvent<HTMLInputElement>) => {
    if (event.key === 'Enter') {
      console.log('Enter pressed');
      event.preventDefault();
    }
    
    if (event.key === 'Escape') {
      console.log('Escape pressed');
    }
    
    // Modifiers
    if (event.ctrlKey && event.key === 's') {
      console.log('Ctrl+S pressed');
      event.preventDefault();
    }
  };

  const handleKeyUp = (event: KeyboardEvent<HTMLInputElement>) => {
    console.log('Key released:', event.key);
  };

  return (
    <input
      type="text"
      onKeyDown={handleKeyDown}
      onKeyUp={handleKeyUp}
      placeholder="Press Enter or Escape"
    />
  );
}
```

---

#### All Common Event Types

**Complete Reference:**
```tsx
import {
  MouseEvent,
  ChangeEvent,
  FormEvent,
  KeyboardEvent,
  FocusEvent,
  ClipboardEvent,
  DragEvent,
  TouchEvent,
  WheelEvent,
  AnimationEvent,
  TransitionEvent,
} from 'react';

function AllEvents() {
  // Mouse Events
  const onClick = (e: MouseEvent<HTMLButtonElement>) => {};
  const onDoubleClick = (e: MouseEvent<HTMLButtonElement>) => {};
  const onMouseDown = (e: MouseEvent<HTMLDivElement>) => {};
  const onMouseUp = (e: MouseEvent<HTMLDivElement>) => {};
  const onMouseEnter = (e: MouseEvent<HTMLDivElement>) => {};
  const onMouseLeave = (e: MouseEvent<HTMLDivElement>) => {};
  const onMouseMove = (e: MouseEvent<HTMLDivElement>) => {};
  const onContextMenu = (e: MouseEvent<HTMLDivElement>) => {};

  // Change Events
  const onInputChange = (e: ChangeEvent<HTMLInputElement>) => {};
  const onTextareaChange = (e: ChangeEvent<HTMLTextAreaElement>) => {};
  const onSelectChange = (e: ChangeEvent<HTMLSelectElement>) => {};

  // Form Events
  const onSubmit = (e: FormEvent<HTMLFormElement>) => {};
  const onReset = (e: FormEvent<HTMLFormElement>) => {};

  // Keyboard Events
  const onKeyDown = (e: KeyboardEvent<HTMLInputElement>) => {};
  const onKeyUp = (e: KeyboardEvent<HTMLInputElement>) => {};
  const onKeyPress = (e: KeyboardEvent<HTMLInputElement>) => {}; // Deprecated

  // Focus Events
  const onFocus = (e: FocusEvent<HTMLInputElement>) => {};
  const onBlur = (e: FocusEvent<HTMLInputElement>) => {};

  // Clipboard Events
  const onCopy = (e: ClipboardEvent<HTMLDivElement>) => {};
  const onCut = (e: ClipboardEvent<HTMLDivElement>) => {};
  const onPaste = (e: ClipboardEvent<HTMLDivElement>) => {};

  // Drag Events
  const onDrag = (e: DragEvent<HTMLDivElement>) => {};
  const onDragStart = (e: DragEvent<HTMLDivElement>) => {};
  const onDragEnd = (e: DragEvent<HTMLDivElement>) => {};
  const onDragEnter = (e: DragEvent<HTMLDivElement>) => {};
  const onDragLeave = (e: DragEvent<HTMLDivElement>) => {};
  const onDragOver = (e: DragEvent<HTMLDivElement>) => {};
  const onDrop = (e: DragEvent<HTMLDivElement>) => {};

  // Touch Events (Mobile)
  const onTouchStart = (e: TouchEvent<HTMLDivElement>) => {};
  const onTouchMove = (e: TouchEvent<HTMLDivElement>) => {};
  const onTouchEnd = (e: TouchEvent<HTMLDivElement>) => {};

  // Wheel Events
  const onWheel = (e: WheelEvent<HTMLDivElement>) => {};

  // Animation Events
  const onAnimationStart = (e: AnimationEvent<HTMLDivElement>) => {};
  const onAnimationEnd = (e: AnimationEvent<HTMLDivElement>) => {};

  // Transition Events
  const onTransitionEnd = (e: TransitionEvent<HTMLDivElement>) => {};

  return <div>Event Examples</div>;
}
```

---

#### Real-World Examples

**1. Controlled Input with State:**
```tsx
import { ChangeEvent, useState } from 'react';

function ControlledInput() {
  const [value, setValue] = useState('');
  const [error, setError] = useState('');

  const handleChange = (event: ChangeEvent<HTMLInputElement>) => {
    const newValue = event.target.value;
    setValue(newValue);

    // Validation
    if (newValue.length < 3) {
      setError('Must be at least 3 characters');
    } else {
      setError('');
    }
  };

  return (
    <div>
      <input
        type="text"
        value={value}
        onChange={handleChange}
        placeholder="Enter text"
      />
      {error && <span className="error">{error}</span>}
    </div>
  );
}
```

**2. Form with Multiple Inputs:**
```tsx
import { ChangeEvent, FormEvent, useState } from 'react';

interface FormData {
  username: string;
  email: string;
  password: string;
  age: number;
  agree: boolean;
}

function RegistrationForm() {
  const [formData, setFormData] = useState<FormData>({
    username: '',
    email: '',
    password: '',
    age: 0,
    agree: false,
  });

  const handleInputChange = (event: ChangeEvent<HTMLInputElement>) => {
    const { name, value, type, checked } = event.target;
    
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : type === 'number' ? Number(value) : value,
    }));
  };

  const handleSubmit = (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    console.log('Form data:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="username"
        value={formData.username}
        onChange={handleInputChange}
        placeholder="Username"
      />
      
      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleInputChange}
        placeholder="Email"
      />
      
      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={handleInputChange}
        placeholder="Password"
      />
      
      <input
        type="number"
        name="age"
        value={formData.age}
        onChange={handleInputChange}
        placeholder="Age"
      />
      
      <label>
        <input
          type="checkbox"
          name="agree"
          checked={formData.agree}
          onChange={handleInputChange}
        />
        I agree to terms
      </label>
      
      <button type="submit">Register</button>
    </form>
  );
}
```

**3. Search with Debounce:**
```tsx
import { ChangeEvent, useState, useEffect } from 'react';

function SearchBar() {
  const [query, setQuery] = useState('');
  const [debouncedQuery, setDebouncedQuery] = useState('');

  const handleChange = (event: ChangeEvent<HTMLInputElement>) => {
    setQuery(event.target.value);
  };

  // Debounce
  useEffect(() => {
    const timeout = setTimeout(() => {
      setDebouncedQuery(query);
    }, 500);

    return () => clearTimeout(timeout);
  }, [query]);

  // Search when debounced query changes
  useEffect(() => {
    if (debouncedQuery) {
      console.log('Searching for:', debouncedQuery);
      // Perform search API call
    }
  }, [debouncedQuery]);

  return (
    <input
      type="text"
      value={query}
      onChange={handleChange}
      placeholder="Search..."
    />
  );
}
```

**4. Drag and Drop:**
```tsx
import { DragEvent, useState } from 'react';

interface Item {
  id: string;
  text: string;
}

function DragDropList() {
  const [items, setItems] = useState<Item[]>([
    { id: '1', text: 'Item 1' },
    { id: '2', text: 'Item 2' },
    { id: '3', text: 'Item 3' },
  ]);
  const [draggingId, setDraggingId] = useState<string | null>(null);

  const handleDragStart = (event: DragEvent<HTMLDivElement>, id: string) => {
    setDraggingId(id);
    event.dataTransfer.effectAllowed = 'move';
  };

  const handleDragOver = (event: DragEvent<HTMLDivElement>) => {
    event.preventDefault();
    event.dataTransfer.dropEffect = 'move';
  };

  const handleDrop = (event: DragEvent<HTMLDivElement>, targetId: string) => {
    event.preventDefault();
    
    if (!draggingId || draggingId === targetId) return;

    const dragIndex = items.findIndex(item => item.id === draggingId);
    const targetIndex = items.findIndex(item => item.id === targetId);

    const newItems = [...items];
    const [draggedItem] = newItems.splice(dragIndex, 1);
    newItems.splice(targetIndex, 0, draggedItem);

    setItems(newItems);
    setDraggingId(null);
  };

  const handleDragEnd = () => {
    setDraggingId(null);
  };

  return (
    <div>
      {items.map(item => (
        <div
          key={item.id}
          draggable
          onDragStart={(e) => handleDragStart(e, item.id)}
          onDragOver={handleDragOver}
          onDrop={(e) => handleDrop(e, item.id)}
          onDragEnd={handleDragEnd}
          style={{
            padding: '10px',
            margin: '5px',
            background: draggingId === item.id ? '#ddd' : '#fff',
            border: '1px solid #ccc',
            cursor: 'move',
          }}
        >
          {item.text}
        </div>
      ))}
    </div>
  );
}
```

**5. Keyboard Shortcuts:**
```tsx
import { KeyboardEvent, useEffect } from 'react';

function Editor() {
  const handleKeyDown = (event: KeyboardEvent<HTMLTextAreaElement>) => {
    // Save: Ctrl+S or Cmd+S
    if ((event.ctrlKey || event.metaKey) && event.key === 's') {
      event.preventDefault();
      console.log('Save triggered');
    }

    // Bold: Ctrl+B or Cmd+B
    if ((event.ctrlKey || event.metaKey) && event.key === 'b') {
      event.preventDefault();
      console.log('Bold triggered');
    }

    // Tab handling
    if (event.key === 'Tab') {
      event.preventDefault();
      // Insert tab or indent
    }
  };

  // Global keyboard listener
  useEffect(() => {
    const handleGlobalKeyDown = (event: globalThis.KeyboardEvent) => {
      if (event.key === 'Escape') {
        console.log('Global escape pressed');
      }
    };

    window.addEventListener('keydown', handleGlobalKeyDown);
    return () => window.removeEventListener('keydown', handleGlobalKeyDown);
  }, []);

  return (
    <textarea
      onKeyDown={handleKeyDown}
      placeholder="Try Ctrl+S or Ctrl+B"
      rows={10}
      cols={50}
    />
  );
}
```

**6. Click Outside Detection:**
```tsx
import { MouseEvent, useEffect, useRef } from 'react';

function Dropdown() {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useRef<HTMLDivElement>(null);

  const handleButtonClick = (event: MouseEvent<HTMLButtonElement>) => {
    event.stopPropagation();
    setIsOpen(!isOpen);
  };

  useEffect(() => {
    const handleClickOutside = (event: globalThis.MouseEvent) => {
      if (dropdownRef.current && !dropdownRef.current.contains(event.target as Node)) {
        setIsOpen(false);
      }
    };

    if (isOpen) {
      document.addEventListener('mousedown', handleClickOutside);
    }

    return () => {
      document.removeEventListener('mousedown', handleClickOutside);
    };
  }, [isOpen]);

  return (
    <div ref={dropdownRef}>
      <button onClick={handleButtonClick}>Toggle Menu</button>
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

#### Handler Function Types

**Explicit Function Types:**
```tsx
import { MouseEventHandler, ChangeEventHandler, FormEventHandler } from 'react';

interface ButtonProps {
  onClick: MouseEventHandler<HTMLButtonElement>;
  label: string;
}

function Button({ onClick, label }: ButtonProps) {
  return <button onClick={onClick}>{label}</button>;
}

interface InputProps {
  value: string;
  onChange: ChangeEventHandler<HTMLInputElement>;
}

function Input({ value, onChange }: InputProps) {
  return <input type="text" value={value} onChange={onChange} />;
}

interface FormProps {
  onSubmit: FormEventHandler<HTMLFormElement>;
  children: ReactNode;
}

function Form({ onSubmit, children }: FormProps) {
  return <form onSubmit={onSubmit}>{children}</form>;
}
```

**Generic Event Handlers:**
```tsx
import { MouseEvent } from 'react';

interface ItemProps<T> {
  item: T;
  onClick: (item: T, event: MouseEvent<HTMLDivElement>) => void;
}

function Item<T>({ item, onClick }: ItemProps<T>) {
  return (
    <div onClick={(e) => onClick(item, e)}>
      {JSON.stringify(item)}
    </div>
  );
}

// Usage
interface User {
  id: number;
  name: string;
}

function UserList() {
  const handleUserClick = (user: User, event: MouseEvent<HTMLDivElement>) => {
    console.log('Clicked user:', user.name);
    console.log('Mouse position:', event.clientX, event.clientY);
  };

  const users: User[] = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' },
  ];

  return (
    <div>
      {users.map(user => (
        <Item key={user.id} item={user} onClick={handleUserClick} />
      ))}
    </div>
  );
}
```

---

#### Event Properties Reference

**Mouse Event Properties:**
```tsx
import { MouseEvent } from 'react';

function handleClick(e: MouseEvent<HTMLDivElement>) {
  // Mouse position
  console.log(e.clientX, e.clientY); // Relative to viewport
  console.log(e.pageX, e.pageY);     // Relative to document
  console.log(e.screenX, e.screenY); // Relative to screen
  console.log(e.movementX, e.movementY); // Movement since last event

  // Mouse buttons
  console.log(e.button);  // 0: left, 1: middle, 2: right
  console.log(e.buttons); // Bitmask of pressed buttons

  // Modifiers
  console.log(e.ctrlKey);  // Ctrl key pressed
  console.log(e.shiftKey); // Shift key pressed
  console.log(e.altKey);   // Alt key pressed
  console.log(e.metaKey);  // Meta key (Cmd on Mac, Win on Windows)

  // Target
  console.log(e.target);        // Element that triggered the event
  console.log(e.currentTarget); // Element the handler is attached to

  // Prevent default & stop propagation
  e.preventDefault();
  e.stopPropagation();
}
```

**Keyboard Event Properties:**
```tsx
import { KeyboardEvent } from 'react';

function handleKeyDown(e: KeyboardEvent<HTMLInputElement>) {
  // Key identification
  console.log(e.key);      // 'a', 'Enter', 'Escape', etc.
  console.log(e.code);     // 'KeyA', 'Enter', 'Escape', etc.
  console.log(e.keyCode);  // Deprecated numeric code

  // Modifiers
  console.log(e.ctrlKey);
  console.log(e.shiftKey);
  console.log(e.altKey);
  console.log(e.metaKey);

  // Repeat
  console.log(e.repeat); // True if key is held down

  // Common patterns
  if (e.key === 'Enter') { /* ... */ }
  if (e.key === 'Escape') { /* ... */ }
  if (e.ctrlKey && e.key === 's') { /* ... */ }
}
```

**Form/Change Event Properties:**
```tsx
import { ChangeEvent, FormEvent } from 'react';

function handleChange(e: ChangeEvent<HTMLInputElement>) {
  const { value, name, type, checked } = e.target;
  
  console.log(value);   // Current value
  console.log(name);    // Input name attribute
  console.log(type);    // Input type (text, checkbox, etc.)
  console.log(checked); // For checkboxes/radios
}

function handleSubmit(e: FormEvent<HTMLFormElement>) {
  e.preventDefault();
  
  const form = e.currentTarget;
  const formData = new FormData(form);
  
  // Get form values
  const email = formData.get('email');
  const password = formData.get('password');
}
```

---

#### Best Practices

**1. Use Specific Element Types:**
```tsx
// GOOD - Specific element type
const handleClick = (e: MouseEvent<HTMLButtonElement>) => {};

// AVOID - Generic element
const handleClick = (e: MouseEvent<HTMLElement>) => {};
```

**2. Extract Event Handlers:**
```tsx
// GOOD - Extracted and typed
function Form() {
  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    // Handle submit
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}

// AVOID - Inline without clear types
function Form() {
  return <form onSubmit={(e) => { /* ... */ }}>...</form>;
}
```

**3. Type Handler Props:**
```tsx
import { MouseEventHandler } from 'react';

// GOOD - Clear handler type
interface Props {
  onClick: MouseEventHandler<HTMLButtonElement>;
}

// ALSO GOOD - Explicit type
interface Props {
  onClick: (event: MouseEvent<HTMLButtonElement>) => void;
}
```

**4. Prevent Default When Needed:**
```tsx
// GOOD - Prevent default for form submit
const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
  e.preventDefault();
  // Process form
};

// GOOD - Prevent default for custom links
const handleLinkClick = (e: MouseEvent<HTMLAnchorElement>) => {
  e.preventDefault();
  // Custom navigation
};
```

---

#### Common Patterns Summary

```tsx
import {
  MouseEvent,
  ChangeEvent,
  FormEvent,
  KeyboardEvent,
  FocusEvent,
} from 'react';

// Mouse click
const onClick = (e: MouseEvent<HTMLButtonElement>) => {};

// Input change
const onChange = (e: ChangeEvent<HTMLInputElement>) => {};

// Form submit
const onSubmit = (e: FormEvent<HTMLFormElement>) => {
  e.preventDefault();
};

// Keyboard
const onKeyDown = (e: KeyboardEvent<HTMLInputElement>) => {
  if (e.key === 'Enter') {}
};

// Focus
const onFocus = (e: FocusEvent<HTMLInputElement>) => {};

// Handler prop type
interface Props {
  onClick: (e: MouseEvent<HTMLButtonElement>) => void;
}
```

---

#### Interview Tips

✅ **Key Points:**
- React uses synthetic events (cross-browser compatibility)
- Generic types: `MouseEvent<HTMLButtonElement>`
- Element type matters: HTMLInputElement, HTMLButtonElement, etc.
- Import from 'react': `import { MouseEvent } from 'react'`
- Use `preventDefault()` to stop default behavior
- Use `stopPropagation()` to prevent bubbling

✅ **When to Mention:**
- Form handling in TypeScript
- User interactions (clicks, typing, drag/drop)
- Event delegation
- Synthetic events vs native events
- Type safety for event handlers

✅ **Common Follow-ups:**
- "What's the difference between event.target and event.currentTarget?"
- "What are React synthetic events?"
- "How to type custom event handlers?"
- "When to use preventDefault vs stopPropagation?"
- "How to handle events in child components?"

✅ **Perfect Answer Structure:**
1. Define: React synthetic events with TypeScript types
2. Basic types: MouseEvent, ChangeEvent, FormEvent, KeyboardEvent
3. Generic parameter: Specify element type `<HTMLInputElement>`
4. Import: From 'react' package
5. Common patterns: Forms, inputs, clicks, keyboard shortcuts
6. Properties: target, currentTarget, preventDefault, stopPropagation

</details>

---

### 130. What is PropsWithChildren type?

<details>
<summary>View Answer</summary>

**PropsWithChildren** is a TypeScript utility type provided by React that automatically adds the `children` prop to your component's props interface. It's a convenient way to type components that accept children without manually adding `children: ReactNode` to every interface.

#### Basic Usage

**Without PropsWithChildren:**
```tsx
import { ReactNode } from 'react';

interface CardProps {
  title: string;
  children: ReactNode;  // Must add manually
}

function Card({ title, children }: CardProps) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  );
}
```

**With PropsWithChildren:**
```tsx
import { PropsWithChildren } from 'react';

interface CardProps {
  title: string;
  // No need to add children manually
}

function Card({ title, children }: PropsWithChildren<CardProps>) {
  return (
    <div className="card">
      <h2>{title}</h2>
      {children}
    </div>
  );
}

// Usage - same as before
<Card title="My Card">
  <p>Card content here</p>
</Card>
```

---

#### How It Works

**Type Definition:**
```tsx
// React's internal definition (simplified)
type PropsWithChildren<P = unknown> = P & { children?: ReactNode };

// It's essentially doing:
interface CardProps {
  title: string;
}

// PropsWithChildren<CardProps> becomes:
interface CardPropsWithChildren {
  title: string;
  children?: ReactNode;
}
```

**Equivalent Code:**
```tsx
import { ReactNode } from 'react';

// Using PropsWithChildren
import { PropsWithChildren } from 'react';
type Props1 = PropsWithChildren<{ title: string }>;

// Is equivalent to:
type Props2 = {
  title: string;
  children?: ReactNode;
};

// Both are the same:
const props1: Props1 = { title: 'Test', children: <div>Content</div> };
const props2: Props2 = { title: 'Test', children: <div>Content</div> };
```

---

#### Real-World Examples

**1. Layout Components:**
```tsx
import { PropsWithChildren } from 'react';

interface ContainerProps {
  maxWidth?: 'sm' | 'md' | 'lg' | 'xl';
  padding?: number;
}

function Container({
  maxWidth = 'lg',
  padding = 20,
  children,
}: PropsWithChildren<ContainerProps>) {
  return (
    <div
      style={{
        maxWidth: maxWidth === 'sm' ? 640 : maxWidth === 'md' ? 768 : 1024,
        padding,
        margin: '0 auto',
      }}
    >
      {children}
    </div>
  );
}

// Usage
function App() {
  return (
    <Container maxWidth="md" padding={30}>
      <h1>Welcome</h1>
      <p>Content goes here</p>
    </Container>
  );
}
```

**2. Card Components:**
```tsx
import { PropsWithChildren } from 'react';

interface CardProps {
  title: string;
  subtitle?: string;
  variant?: 'default' | 'outlined' | 'elevated';
  onClick?: () => void;
}

function Card({
  title,
  subtitle,
  variant = 'default',
  onClick,
  children,
}: PropsWithChildren<CardProps>) {
  return (
    <div className={`card card-${variant}`} onClick={onClick}>
      <div className="card-header">
        <h3>{title}</h3>
        {subtitle && <p className="subtitle">{subtitle}</p>}
      </div>
      <div className="card-content">{children}</div>
    </div>
  );
}

// Usage
function Dashboard() {
  return (
    <div>
      <Card title="Statistics" subtitle="Monthly overview" variant="elevated">
        <p>Total Users: 1,234</p>
        <p>Revenue: $45,678</p>
      </Card>

      <Card title="Recent Activity">
        <ul>
          <li>User A logged in</li>
          <li>User B made a purchase</li>
        </ul>
      </Card>
    </div>
  );
}
```

**3. Modal/Dialog Components:**
```tsx
import { PropsWithChildren } from 'react';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  size?: 'sm' | 'md' | 'lg';
}

function Modal({
  isOpen,
  onClose,
  title,
  size = 'md',
  children,
}: PropsWithChildren<ModalProps>) {
  if (!isOpen) return null;

  return (
    <div className="modal-overlay" onClick={onClose}>
      <div
        className={`modal modal-${size}`}
        onClick={(e) => e.stopPropagation()}
      >
        <div className="modal-header">
          <h2>{title}</h2>
          <button onClick={onClose}>×</button>
        </div>
        <div className="modal-body">{children}</div>
      </div>
    </div>
  );
}

// Usage
function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>Open Modal</button>
      
      <Modal
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
        title="Confirm Action"
        size="sm"
      >
        <p>Are you sure you want to proceed?</p>
        <div className="modal-actions">
          <button onClick={() => setIsOpen(false)}>Cancel</button>
          <button onClick={() => setIsOpen(false)}>Confirm</button>
        </div>
      </Modal>
    </>
  );
}
```

**4. Provider Components:**
```tsx
import { PropsWithChildren, createContext, useState } from 'react';

interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);

interface ThemeProviderProps {
  defaultTheme?: 'light' | 'dark';
}

function ThemeProvider({
  defaultTheme = 'light',
  children,
}: PropsWithChildren<ThemeProviderProps>) {
  const [theme, setTheme] = useState<'light' | 'dark'>(defaultTheme);

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Usage
function App() {
  return (
    <ThemeProvider defaultTheme="dark">
      <MainContent />
    </ThemeProvider>
  );
}
```

**5. Button Group:**
```tsx
import { PropsWithChildren } from 'react';

interface ButtonGroupProps {
  orientation?: 'horizontal' | 'vertical';
  spacing?: number;
  variant?: 'contained' | 'outlined' | 'text';
}

function ButtonGroup({
  orientation = 'horizontal',
  spacing = 8,
  variant = 'contained',
  children,
}: PropsWithChildren<ButtonGroupProps>) {
  return (
    <div
      className={`button-group button-group-${orientation}`}
      style={{
        display: 'flex',
        flexDirection: orientation === 'horizontal' ? 'row' : 'column',
        gap: spacing,
      }}
    >
      {children}
    </div>
  );
}

// Usage
function Toolbar() {
  return (
    <ButtonGroup orientation="horizontal" spacing={12}>
      <button>Save</button>
      <button>Cancel</button>
      <button>Delete</button>
    </ButtonGroup>
  );
}
```

---

#### When to Use PropsWithChildren

**Use PropsWithChildren when:**
```tsx
// ✅ Component accepts children
function Wrapper({ children }: PropsWithChildren<{}>) {
  return <div className="wrapper">{children}</div>;
}

// ✅ Component with props + children
interface Props {
  title: string;
}
function Card({ title, children }: PropsWithChildren<Props>) {
  return <div><h2>{title}</h2>{children}</div>;
}

// ✅ Generic component with children
function Container<T>({ data, children }: PropsWithChildren<{ data: T }>) {
  return <div>{children}</div>;
}
```

**Don't use when children is required:**
```tsx
// ❌ PropsWithChildren makes children optional
function RequiresChildren({ children }: PropsWithChildren<{}>) {
  // children might be undefined!
  return <div>{children}</div>;
}

// ✅ Make children explicitly required
interface Props {
  children: ReactNode;  // Required
}
function RequiresChildren({ children }: Props) {
  return <div>{children}</div>;
}
```

---

#### PropsWithChildren vs Manual Children

**Comparison:**
```tsx
import { ReactNode, PropsWithChildren } from 'react';

// Option 1: PropsWithChildren (children optional)
interface Props1 {
  title: string;
}
function Component1({ title, children }: PropsWithChildren<Props1>) {
  return <div>{title} {children}</div>;
}

// Option 2: Manual children (required)
interface Props2 {
  title: string;
  children: ReactNode;
}
function Component2({ title, children }: Props2) {
  return <div>{title} {children}</div>;
}

// Option 3: Manual children (optional)
interface Props3 {
  title: string;
  children?: ReactNode;
}
function Component3({ title, children }: Props3) {
  return <div>{title} {children}</div>;
}

// Usage differences:
<Component1 title="Test" />  // ✅ OK - children optional
<Component2 title="Test" />  // ❌ Error - children required
<Component3 title="Test" />  // ✅ OK - children optional

// Props1 and Props3 are equivalent when using PropsWithChildren
```

---

#### Advanced Patterns

**1. Generic PropsWithChildren:**
```tsx
import { PropsWithChildren } from 'react';

interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => ReactNode;
}

function List<T>({
  items,
  renderItem,
  children,
}: PropsWithChildren<ListProps<T>>) {
  return (
    <div>
      {children}  {/* Optional header */}
      <ul>
        {items.map((item, index) => (
          <li key={index}>{renderItem(item)}</li>
        ))}
      </ul>
    </div>
  );
}

// Usage
interface User {
  id: number;
  name: string;
}

function UserList() {
  const users: User[] = [{ id: 1, name: 'John' }];

  return (
    <List items={users} renderItem={(user) => user.name}>
      <h2>User List</h2>  {/* Optional children */}
    </List>
  );
}
```

**2. Conditional Rendering with Children:**
```tsx
import { PropsWithChildren } from 'react';

interface ConditionalProps {
  condition: boolean;
  fallback?: ReactNode;
}

function Conditional({
  condition,
  fallback = null,
  children,
}: PropsWithChildren<ConditionalProps>) {
  return <>{condition ? children : fallback}</>;
}

// Usage
function App() {
  const [isLoggedIn, setIsLoggedIn] = useState(false);

  return (
    <Conditional
      condition={isLoggedIn}
      fallback={<div>Please log in</div>}
    >
      <Dashboard />
    </Conditional>
  );
}
```

**3. Empty Props with Children:**
```tsx
import { PropsWithChildren } from 'react';

// When component only needs children (no other props)
function Wrapper({ children }: PropsWithChildren) {
  return <div className="wrapper">{children}</div>;
}

// Equivalent to:
function Wrapper2({ children }: PropsWithChildren<{}>) {
  return <div className="wrapper">{children}</div>;
}

// Usage
<Wrapper>
  <p>Content</p>
</Wrapper>
```

**4. Combining with Other Utilities:**
```tsx
import { PropsWithChildren, ComponentPropsWithoutRef } from 'react';

// Extend HTML div props + custom props + children
type CustomDivProps = PropsWithChildren<
  ComponentPropsWithoutRef<'div'> & {
    variant?: 'primary' | 'secondary';
  }
>;

function CustomDiv({ variant = 'primary', children, ...props }: CustomDivProps) {
  return (
    <div {...props} className={`custom-div-${variant} ${props.className || ''}`}>
      {children}
    </div>
  );
}

// Usage
<CustomDiv variant="secondary" onClick={() => {}} style={{ padding: 20 }}>
  Content here
</CustomDiv>
```

---

#### Best Practices

**1. Use for Optional Children:**
```tsx
// GOOD - PropsWithChildren for optional children
function Card({ title, children }: PropsWithChildren<{ title: string }>) {
  return <div>{title} {children}</div>;
}

// AVOID - Manual typing when PropsWithChildren works
interface Props {
  title: string;
  children?: ReactNode;
}
```

**2. Required Children - Skip PropsWithChildren:**
```tsx
// GOOD - Explicit required children
interface Props {
  children: ReactNode;  // Required
}

// AVOID - PropsWithChildren makes it optional
function Component({ children }: PropsWithChildren<{}>) {
  return <div>{children}</div>;  // children might be undefined
}
```

**3. Type Alias for Reusability:**
```tsx
// GOOD - Reusable type
type CardProps = PropsWithChildren<{
  title: string;
  subtitle?: string;
}>;

function Card({ title, subtitle, children }: CardProps) {
  return <div>...</div>;
}
```

**4. Empty Props Object:**
```tsx
// GOOD - Explicit empty object
function Wrapper({ children }: PropsWithChildren<{}>) {
  return <div>{children}</div>;
}

// ALSO GOOD - Omit generic (defaults to {})
function Wrapper({ children }: PropsWithChildren) {
  return <div>{children}</div>;
}
```

---

#### Common Patterns Summary

```tsx
import { PropsWithChildren, ReactNode } from 'react';

// 1. Simple wrapper (no props)
function Wrapper({ children }: PropsWithChildren) {
  return <div>{children}</div>;
}

// 2. Component with props
interface Props {
  title: string;
}
function Card({ title, children }: PropsWithChildren<Props>) {
  return <div><h2>{title}</h2>{children}</div>;
}

// 3. Generic component
function Container<T>({ data, children }: PropsWithChildren<{ data: T }>) {
  return <div>{children}</div>;
}

// 4. Type alias
type ModalProps = PropsWithChildren<{
  isOpen: boolean;
  onClose: () => void;
}>;

// 5. Required children (don't use PropsWithChildren)
interface RequiredChildrenProps {
  children: ReactNode;  // Required
}
```

---

#### Interview Tips

✅ **Key Points:**
- PropsWithChildren is a utility type from React
- Automatically adds `children?: ReactNode` to props
- Generic type: `PropsWithChildren<YourProps>`
- Makes children optional (not required)
- Equivalent to manually adding `children?: ReactNode`

✅ **When to Mention:**
- TypeScript component typing
- Layout/container components
- Props with optional children
- Provider components
- Type utilities in React

✅ **Common Follow-ups:**
- "What does PropsWithChildren add?"
- "When to use PropsWithChildren vs manual children prop?"
- "Is children required or optional with PropsWithChildren?"
- "Can you use PropsWithChildren with generic components?"
- "What's the difference between PropsWithChildren and FC?"

✅ **Perfect Answer Structure:**
1. Define: Utility type that adds children prop
2. Type signature: `PropsWithChildren<YourProps>`
3. Adds: `children?: ReactNode` (optional)
4. Use case: Container/layout components with optional children
5. Alternative: Manually add `children: ReactNode` when required
6. Example: Card or Modal component

</details>
