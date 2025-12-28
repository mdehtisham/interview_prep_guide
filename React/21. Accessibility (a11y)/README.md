# Accessibility (a11y)

> Expert / Architect Level (5+ years)

---

## Questions

201. How do you make React applications accessible?
202. What is ARIA and how do you use it in React?
203. How do you implement keyboard navigation?
204. What is focus management in React?
205. How do you test accessibility in React apps?
206. What is the React ARIA library?
207. How do you handle screen reader compatibility?
208. What are semantic HTML elements and their importance?
209. How do you implement skip links in React?
210. What is color contrast and why does it matter?

---

## Detailed Answers

### 201. How do you make React applications accessible?

<details>
<summary>View Answer</summary>

**Making React applications accessible** ensures that people with disabilities can use your app effectively. Accessibility (a11y) is not optional—it's a legal requirement in many countries and affects 15-20% of users worldwide.

#### 1. Follow WCAG Guidelines

**WCAG 2.1 Principles (POUR):**
```typescript
/*
Perceivable:
- Text alternatives for images
- Captions for videos
- Sufficient color contrast
- Resizable text

Operable:
- Keyboard accessible
- Enough time to interact
- No seizure triggers
- Clear navigation

Understandable:
- Readable text
- Predictable behavior
- Input assistance
- Error prevention

Robust:
- Compatible with assistive technologies
- Valid HTML
- Proper ARIA usage
*/
```

#### 2. Use Semantic HTML

**Basic Structure:**
```typescript
// ❌ BAD: Divs everywhere
function App() {
  return (
    <div className="app">
      <div className="header">
        <div className="nav">
          <div onClick={goHome}>Home</div>
          <div onClick={goAbout}>About</div>
        </div>
      </div>
      <div className="content">
        <div className="article">Article content</div>
      </div>
      <div className="footer">Footer</div>
    </div>
  );
}

// ✅ GOOD: Semantic elements
function App() {
  return (
    <div className="app">
      <header>
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
      </header>
      <main>
        <article>Article content</article>
      </main>
      <footer>Footer</footer>
    </div>
  );
}

// Screen readers announce:
// "navigation landmark"
// "main landmark"
// "contentinfo landmark"
```

**Accessible Form:**
```typescript
// ❌ BAD: No labels, poor structure
function BadForm() {
  return (
    <div>
      <input type="text" placeholder="Name" />
      <input type="email" placeholder="Email" />
      <div onClick={submit}>Submit</div>
    </div>
  );
}

// ✅ GOOD: Proper labels and semantics
function GoodForm() {
  return (
    <form onSubmit={handleSubmit}>
      <label htmlFor="name">
        Name:
        <input
          id="name"
          type="text"
          required
          aria-required="true"
          aria-describedby="name-hint"
        />
        <span id="name-hint" className="hint">
          Enter your full name
        </span>
      </label>
      
      <label htmlFor="email">
        Email:
        <input
          id="email"
          type="email"
          required
          aria-required="true"
          aria-invalid={errors.email ? 'true' : 'false'}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <span id="email-error" role="alert">
            {errors.email}
          </span>
        )}
      </label>
      
      <button type="submit">Submit</button>
    </form>
  );
}
```

#### 3. Keyboard Navigation

**Interactive Elements:**
```typescript
// ❌ BAD: Div with click handler (not keyboard accessible)
function BadButton() {
  return (
    <div onClick={() => alert('Clicked')}>
      Click me
    </div>
  );
}

// ✅ GOOD: Proper button
function GoodButton() {
  return (
    <button onClick={() => alert('Clicked')}>
      Click me
    </button>
  );
}

// ✅ If you MUST use div (rare cases):
function DivButton() {
  return (
    <div
      role="button"
      tabIndex={0}
      onClick={handleClick}
      onKeyDown={(e) => {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault();
          handleClick();
        }
      }}
    >
      Click me
    </div>
  );
}
```

**Focus Management:**
```typescript
import { useRef, useEffect } from 'react';

function Modal({ isOpen, onClose, children }) {
  const closeButtonRef = useRef<HTMLButtonElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);
  
  useEffect(() => {
    if (isOpen) {
      // Save previous focus
      previousFocusRef.current = document.activeElement as HTMLElement;
      
      // Focus close button when modal opens
      closeButtonRef.current?.focus();
      
      // Trap focus in modal
      const handleTab = (e: KeyboardEvent) => {
        if (e.key === 'Tab') {
          const focusableElements = modal.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
          );
          const firstElement = focusableElements[0] as HTMLElement;
          const lastElement = focusableElements[focusableElements.length - 1] as HTMLElement;
          
          if (e.shiftKey && document.activeElement === firstElement) {
            e.preventDefault();
            lastElement.focus();
          } else if (!e.shiftKey && document.activeElement === lastElement) {
            e.preventDefault();
            firstElement.focus();
          }
        }
      };
      
      document.addEventListener('keydown', handleTab);
      return () => document.removeEventListener('keydown', handleTab);
    } else {
      // Restore focus when modal closes
      previousFocusRef.current?.focus();
    }
  }, [isOpen]);
  
  if (!isOpen) return null;
  
  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
    >
      <h2 id="modal-title">Modal Title</h2>
      {children}
      <button ref={closeButtonRef} onClick={onClose}>
        Close
      </button>
    </div>
  );
}
```

#### 4. Alt Text for Images

**Descriptive Alt Text:**
```typescript
// ❌ BAD
<img src="photo.jpg" alt="image" />
<img src="logo.png" />  {/* Missing alt */}
<img src="chart.png" alt="chart" />  {/* Too vague */}

// ✅ GOOD
<img src="photo.jpg" alt="Woman giving presentation at tech conference" />
<img src="logo.png" alt="Company Name" />
<img src="chart.png" alt="Bar chart showing 30% increase in sales from Q1 to Q2" />

// ✅ Decorative images
<img src="decorative-border.png" alt="" role="presentation" />
```

**Dynamic Alt Text:**
```typescript
function ProductImage({ product }: { product: Product }) {
  const altText = `${product.name} - ${product.color} ${product.size}`;
  
  return (
    <img
      src={product.image}
      alt={altText}
      loading="lazy"
    />
  );
}

// Example output:
// "Leather Jacket - Black Medium"
```

#### 5. ARIA Attributes

**Common ARIA Patterns:**
```typescript
// Accordion
function Accordion({ title, children, isOpen, toggle }) {
  const id = useId();
  
  return (
    <div>
      <button
        aria-expanded={isOpen}
        aria-controls={`panel-${id}`}
        onClick={toggle}
      >
        {title}
      </button>
      <div
        id={`panel-${id}`}
        role="region"
        aria-labelledby={`button-${id}`}
        hidden={!isOpen}
      >
        {children}
      </div>
    </div>
  );
}

// Tab panel
function Tabs({ tabs, activeTab, setActiveTab }) {
  return (
    <div>
      <div role="tablist" aria-label="Content tabs">
        {tabs.map((tab, index) => (
          <button
            key={tab.id}
            role="tab"
            aria-selected={activeTab === index}
            aria-controls={`panel-${tab.id}`}
            id={`tab-${tab.id}`}
            onClick={() => setActiveTab(index)}
          >
            {tab.label}
          </button>
        ))}
      </div>
      
      {tabs.map((tab, index) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={activeTab !== index}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

#### 6. Color Contrast

**WCAG Standards:**
```typescript
/*
WCAG AA (Minimum):
- Normal text: 4.5:1
- Large text (18pt+): 3:1

WCAG AAA (Enhanced):
- Normal text: 7:1
- Large text: 4.5:1

Examples:

❌ Fail: #777 text on #fff background (2.8:1)
✅ Pass: #595959 text on #fff background (4.6:1)
✅ Pass: #000 text on #fff background (21:1)

Check with:
- Chrome DevTools (Lighthouse)
- WebAIM Contrast Checker
- axe DevTools
*/

// Good contrast examples
const colors = {
  // Text on white background
  primary: '#0066CC',    // 4.5:1 ✅
  secondary: '#555555',  // 8.6:1 ✅
  error: '#D32F2F',      // 4.5:1 ✅
  
  // Text on dark background
  lightText: '#FFFFFF',  // 21:1 ✅
};
```

#### 7. Screen Reader Support

**Live Regions:**
```typescript
function Toast({ message, type }) {
  return (
    <div
      role="alert"  // Announces immediately
      aria-live="assertive"  // Interrupts current announcement
      className={`toast toast-${type}`}
    >
      {message}
    </div>
  );
}

// For less urgent updates
function StatusMessage({ message }) {
  return (
    <div
      role="status"
      aria-live="polite"  // Waits for pause
    >
      {message}
    </div>
  );
}
```

**Accessible Loading States:**
```typescript
function LoadingButton({ isLoading, children, ...props }) {
  return (
    <button
      {...props}
      aria-busy={isLoading}
      aria-live="polite"
      disabled={isLoading}
    >
      {isLoading ? (
        <>
          <span className="spinner" aria-hidden="true" />
          <span className="sr-only">Loading...</span>
        </>
      ) : (
        children
      )}
    </button>
  );
}

// CSS for screen-reader-only text
/*
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
*/
```

#### 8. Testing Accessibility

**Automated Testing:**
```typescript
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('Button is accessible', async () => {
  const { container } = render(<Button>Click me</Button>);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

**Manual Testing Checklist:**
```typescript
/*
✅ Keyboard Navigation:
- Tab through all interactive elements
- Enter/Space activates buttons
- Escape closes modals
- Arrow keys navigate menus

✅ Screen Reader:
- Turn on VoiceOver (Mac) or NVDA (Windows)
- Navigate with screen reader on
- Check announcements are clear
- Verify landmarks are announced

✅ Zoom:
- Zoom to 200%
- Check layout doesn't break
- Text remains readable

✅ Color:
- Check contrast ratios
- Disable images, check alt text
- View in grayscale mode
*/
```

#### 9. Accessibility Checklist

**Implementation Checklist:**
```typescript
/*
✅ Semantic HTML:
- Use <header>, <nav>, <main>, <footer>
- Use <button> for buttons, <a> for links
- Use proper heading hierarchy (h1 → h2 → h3)

✅ Forms:
- Label all inputs with <label htmlFor>
- Show errors with role="alert"
- Mark required fields with aria-required
- Provide helpful error messages

✅ Images:
- Alt text for all images
- Empty alt="" for decorative images
- Descriptive alt text (not "image" or "photo")

✅ Keyboard:
- All interactive elements accessible via keyboard
- Visible focus indicators
- Logical tab order
- Skip links for main content

✅ ARIA:
- Use semantic HTML first
- ARIA when semantic HTML insufficient
- Test with screen readers
- aria-label for icon buttons

✅ Color & Contrast:
- 4.5:1 for normal text
- 3:1 for large text
- Don't rely on color alone

✅ Focus Management:
- Manage focus in modals
- Restore focus after actions
- Trap focus in modals

✅ Testing:
- Automated tests (axe, Lighthouse)
- Manual keyboard testing
- Screen reader testing
- Zoom testing (200%)
*/
```

#### 10. Common Patterns

**Accessible Button:**
```typescript
interface ButtonProps {
  children: React.ReactNode;
  onClick: () => void;
  isLoading?: boolean;
  disabled?: boolean;
  ariaLabel?: string;
}

function AccessibleButton({
  children,
  onClick,
  isLoading = false,
  disabled = false,
  ariaLabel,
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled || isLoading}
      aria-label={ariaLabel}
      aria-busy={isLoading}
    >
      {children}
      {isLoading && (
        <span className="sr-only">Loading...</span>
      )}
    </button>
  );
}
```

**Accessible Link:**
```typescript
// ❌ BAD: "Click here" links
<a href="/about">Click here</a> to learn more about us.

// ✅ GOOD: Descriptive links
<a href="/about">Learn more about our company</a>

// ✅ GOOD: External link with indicator
function ExternalLink({ href, children }) {
  return (
    <a
      href={href}
      target="_blank"
      rel="noopener noreferrer"
    >
      {children}
      <span className="sr-only">(opens in new window)</span>
      <span aria-hidden="true">↗</span>
    </a>
  );
}
```

#### Summary

**Core Principles:**

1. **Semantic HTML** - Use proper elements
2. **Keyboard Navigation** - Everything accessible without mouse
3. **ARIA** - Enhance semantics when needed
4. **Alt Text** - Describe images meaningfully
5. **Color Contrast** - Meet WCAG standards
6. **Focus Management** - Control and indicate focus
7. **Screen Readers** - Test with actual assistive tech
8. **Testing** - Automate and manually test

**Impact:**
> "15-20% of users have disabilities. Accessibility isn't optional—it's essential!"

**Remember:**
> "If it's not accessible by keyboard, it's not accessible."

**Golden Rule:**
> "Semantic HTML first, ARIA when necessary, test with real users."

</details>

---

### 202. What is ARIA and how do you use it in React?

<details>
<summary>View Answer</summary>

**ARIA (Accessible Rich Internet Applications)** is a set of attributes that make web content more accessible to people with disabilities. ARIA provides semantic information to assistive technologies when HTML alone is insufficient.

#### 1. ARIA Basics

**What is ARIA?**
```typescript
/*
ARIA consists of:

1. Roles: What an element is
   - role="button"
   - role="dialog"
   - role="navigation"

2. Properties: Characteristics of elements
   - aria-label="Close menu"
   - aria-labelledby="title-id"
   - aria-describedby="desc-id"

3. States: Current condition of elements
   - aria-expanded="true"
   - aria-hidden="false"
   - aria-disabled="true"

When to use ARIA:
✅ When semantic HTML is insufficient
✅ For complex interactive widgets
✅ For dynamic content updates

When NOT to use ARIA:
❌ When semantic HTML exists
❌ As a substitute for proper HTML
❌ Without testing with screen readers

First Rule of ARIA:
"Don't use ARIA if you can use native HTML instead!"
*/
```

#### 2. ARIA Roles

**Common Roles:**
```typescript
// ❌ BAD: Using divs without roles
function BadNavigation() {
  return (
    <div className="nav">
      <div onClick={goHome}>Home</div>
      <div onClick={goAbout}>About</div>
    </div>
  );
}

// ✅ BETTER: Using roles
function BetterNavigation() {
  return (
    <div role="navigation">
      <div role="button" tabIndex={0} onClick={goHome}>Home</div>
      <div role="button" tabIndex={0} onClick={goAbout}>About</div>
    </div>
  );
}

// ✅ BEST: Using semantic HTML
function BestNavigation() {
  return (
    <nav>
      <button onClick={goHome}>Home</button>
      <button onClick={goAbout}>About</button>
    </nav>
  );
}

// Semantic HTML provides role automatically:
// <nav> = role="navigation"
// <button> = role="button"
// <main> = role="main"
```

**Landmark Roles:**
```typescript
function App() {
  return (
    <div className="app">
      {/* ✅ Header landmark */}
      <header role="banner">
        <h1>My App</h1>
      </header>
      
      {/* ✅ Navigation landmark */}
      <nav role="navigation" aria-label="Main navigation">
        <a href="/">Home</a>
        <a href="/about">About</a>
      </nav>
      
      {/* ✅ Main landmark */}
      <main role="main">
        <h2>Content</h2>
        {/* Multiple navigation, so label them */}
        <nav aria-label="Article navigation">
          <a href="#section1">Section 1</a>
        </nav>
      </main>
      
      {/* ✅ Complementary landmark */}
      <aside role="complementary">
        <h2>Sidebar</h2>
      </aside>
      
      {/* ✅ Footer landmark */}
      <footer role="contentinfo">
        <p>&copy; 2025</p>
      </footer>
    </div>
  );
}

// Screen reader announces:
// "banner landmark"
// "navigation landmark"
// "main landmark"
// "complementary landmark"
// "contentinfo landmark"
```

**Widget Roles:**
```typescript
// Dialog/Modal
function Modal({ isOpen, title, children, onClose }) {
  if (!isOpen) return null;
  
  return (
    <div
      role="dialog"  // Identifies as dialog
      aria-modal="true"  // Modal behavior
      aria-labelledby="modal-title"  // Label reference
    >
      <h2 id="modal-title">{title}</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  );
}

// Alert
function Alert({ message, type }) {
  return (
    <div
      role="alert"  // Announces immediately
      className={`alert alert-${type}`}
    >
      {message}
    </div>
  );
}

// Tab panel
function TabPanel({ id, isActive, children }) {
  return (
    <div
      role="tabpanel"  // Tab content
      id={`panel-${id}`}
      aria-labelledby={`tab-${id}`}
      hidden={!isActive}
    >
      {children}
    </div>
  );
}
```

#### 3. ARIA Properties

**aria-label:**
```typescript
// For elements without visible text
function IconButton() {
  return (
    <button aria-label="Close menu">
      <XIcon />  {/* Visual icon, no text */}
    </button>
  );
}

// Screen reader announces: "Close menu, button"

function SearchInput() {
  return (
    <input
      type="search"
      aria-label="Search products"
      placeholder="Search..."
    />
  );
}
```

**aria-labelledby:**
```typescript
// Reference existing element for label
function ProductCard({ product }) {
  return (
    <article aria-labelledby={`product-${product.id}`}>
      <h3 id={`product-${product.id}`}>{product.name}</h3>
      <img
        src={product.image}
        alt=""
        aria-labelledby={`product-${product.id}`}
      />
      <p>{product.description}</p>
    </article>
  );
}

// Multiple labels
function PriceTag() {
  return (
    <div aria-labelledby="product-name price-value">
      <h3 id="product-name">Laptop</h3>
      <span id="price-value">$999</span>
    </div>
  );
}
// Screen reader: "Laptop $999"
```

**aria-describedby:**
```typescript
// Additional description
function PasswordInput() {
  const [error, setError] = useState('');
  
  return (
    <div>
      <label htmlFor="password">Password:</label>
      <input
        id="password"
        type="password"
        aria-describedby="password-hint password-error"
        aria-invalid={error ? 'true' : 'false'}
      />
      <p id="password-hint">
        Must be at least 8 characters with numbers and symbols
      </p>
      {error && (
        <p id="password-error" role="alert">
          {error}
        </p>
      )}
    </div>
  );
}

// Screen reader announces:
// "Password, edit text"
// "Must be at least 8 characters..."
// If error: "Password is too short" (announced immediately)
```

#### 4. ARIA States

**aria-expanded:**
```typescript
function Accordion({ title, children }) {
  const [isOpen, setIsOpen] = useState(false);
  const id = useId();
  
  return (
    <div>
      <button
        aria-expanded={isOpen}  // true or false
        aria-controls={`content-${id}`}
        onClick={() => setIsOpen(!isOpen)}
      >
        {title}
        <span aria-hidden="true">{isOpen ? '▼' : '▶'}</span>
      </button>
      
      <div
        id={`content-${id}`}
        role="region"
        hidden={!isOpen}
      >
        {children}
      </div>
    </div>
  );
}

// Screen reader announces:
// "Button, collapsed" or "Button, expanded"
```

**aria-hidden:**
```typescript
function Modal({ isOpen, children }) {
  return (
    <>
      {/* Hide main content from screen readers */}
      <div aria-hidden={isOpen}>
        <main>Main content</main>
      </div>
      
      {/* Show modal */}
      {isOpen && (
        <div role="dialog" aria-modal="true">
          {children}
        </div>
      )}
    </>
  );
}

// Decorative icons
function Button({ children }) {
  return (
    <button>
      {children}
      <span className="icon" aria-hidden="true">→</span>
    </button>
  );
}
// Screen reader ignores the arrow icon
```

**aria-disabled:**
```typescript
function DisabledButton({ isDisabled, children, onClick }) {
  return (
    <button
      disabled={isDisabled}
      aria-disabled={isDisabled}
      onClick={isDisabled ? undefined : onClick}
    >
      {children}
    </button>
  );
}

// Use aria-disabled when you want to keep element focusable
function FocusableDisabledButton({ isDisabled, children, tooltip }) {
  return (
    <button
      aria-disabled={isDisabled}
      aria-describedby="tooltip"
      onClick={(e) => {
        if (isDisabled) {
          e.preventDefault();
          return;
        }
        handleClick();
      }}
    >
      {children}
      {isDisabled && (
        <span id="tooltip" role="tooltip">
          {tooltip}
        </span>
      )}
    </button>
  );
}
```

**aria-selected:**
```typescript
function Tabs({ tabs, activeTab, setActiveTab }) {
  return (
    <div>
      <div role="tablist">
        {tabs.map((tab, index) => (
          <button
            key={tab.id}
            role="tab"
            aria-selected={activeTab === index}  // true/false
            aria-controls={`panel-${tab.id}`}
            onClick={() => setActiveTab(index)}
          >
            {tab.label}
          </button>
        ))}
      </div>
      
      {tabs.map((tab, index) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          hidden={activeTab !== index}
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

#### 5. Live Regions

**aria-live:**
```typescript
// Polite: Wait for pause
function StatusMessage({ message }) {
  return (
    <div
      role="status"
      aria-live="polite"
      aria-atomic="true"
    >
      {message}
    </div>
  );
}

// Assertive: Interrupt immediately
function ErrorAlert({ error }) {
  return (
    <div
      role="alert"
      aria-live="assertive"
      aria-atomic="true"
    >
      {error}
    </div>
  );
}

// Off: No announcements
function SilentUpdate({ count }) {
  return (
    <div aria-live="off">
      {count} items
    </div>
  );
}
```

**Real-World Example:**
```typescript
function ShoppingCart() {
  const [items, setItems] = useState([]);
  const [announcement, setAnnouncement] = useState('');
  
  const addItem = (item: Item) => {
    setItems([...items, item]);
    setAnnouncement(`${item.name} added to cart. ${items.length + 1} items total.`);
  };
  
  const removeItem = (itemId: string) => {
    const item = items.find(i => i.id === itemId);
    setItems(items.filter(i => i.id !== itemId));
    setAnnouncement(`${item.name} removed from cart. ${items.length - 1} items remaining.`);
  };
  
  return (
    <div>
      {/* Visual cart */}
      <div>
        {items.map(item => (
          <div key={item.id}>
            {item.name}
            <button
              onClick={() => removeItem(item.id)}
              aria-label={`Remove ${item.name} from cart`}
            >
              Remove
            </button>
          </div>
        ))}
      </div>
      
      {/* Screen reader announcements */}
      <div
        role="status"
        aria-live="polite"
        aria-atomic="true"
        className="sr-only"
      >
        {announcement}
      </div>
    </div>
  );
}
```

#### 6. Form Validation

**Accessible Form Errors:**
```typescript
function LoginForm() {
  const [email, setEmail] = useState('');
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const newErrors: Record<string, string> = {};
    
    if (!email) {
      newErrors.email = 'Email is required';
    } else if (!email.includes('@')) {
      newErrors.email = 'Email must be valid';
    }
    
    setErrors(newErrors);
  };
  
  return (
    <form onSubmit={handleSubmit} noValidate>
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          aria-required="true"
          aria-invalid={errors.email ? 'true' : 'false'}
          aria-describedby={errors.email ? 'email-error' : undefined}
        />
        {errors.email && (
          <p id="email-error" role="alert">
            {errors.email}
          </p>
        )}
      </div>
      
      <button type="submit">Login</button>
    </form>
  );
}

// Screen reader announces:
// "Email, edit text, required, invalid"
// Then: "Email is required" (as alert)
```

#### 7. React-Specific ARIA

**camelCase in React:**
```typescript
// HTML uses kebab-case
<div aria-label="Close" aria-hidden="true"></div>

// React uses camelCase for multi-word attributes
// But ARIA attributes stay kebab-case!
function Component() {
  return (
    <div
      aria-label="Close"           // ✅ Correct
      aria-labelledby="title"      // ✅ Correct
      aria-describedby="desc"      // ✅ Correct
      ariaLabel="Close"            // ❌ Wrong!
      ariaLabelledBy="title"       // ❌ Wrong!
    />
  );
}
```

**Dynamic ARIA:**
```typescript
function DynamicARIA({ isExpanded, isDisabled }) {
  return (
    <button
      aria-expanded={isExpanded}  // boolean converted to "true"/"false"
      aria-disabled={isDisabled}
      aria-label={isDisabled ? 'Disabled button' : 'Active button'}
    >
      Click me
    </button>
  );
}
```

#### 8. Common Mistakes

**Mistakes to Avoid:**
```typescript
// ❌ MISTAKE 1: Redundant ARIA
<button role="button">Click</button>
// Button already has implicit role="button"

// ✅ Correct
<button>Click</button>

// ❌ MISTAKE 2: Conflicting ARIA
<input type="checkbox" role="button" />
// Don't override native semantics!

// ✅ Correct
<input type="checkbox" />

// ❌ MISTAKE 3: Empty aria-label
<button aria-label="">Click</button>

// ✅ Correct
<button aria-label="Close menu">×</button>

// ❌ MISTAKE 4: aria-hidden on focusable elements
<button aria-hidden="true">Click</button>
// Element is hidden but focusable - confusing!

// ✅ Correct
<button aria-hidden="true" tabIndex={-1}>Click</button>
// Or just don't render it

// ❌ MISTAKE 5: Using ARIA instead of semantic HTML
<div role="button" tabIndex={0} onClick={click}>Click</div>

// ✅ Correct
<button onClick={click}>Click</button>
```

#### 9. Testing ARIA

**Testing with Screen Readers:**
```typescript
/*
Test with:
- VoiceOver (Mac): Cmd + F5
- NVDA (Windows): Free, open source
- JAWS (Windows): Commercial

Test checklist:
✅ Elements announced correctly
✅ Roles announced ("button", "navigation")
✅ States announced ("expanded", "selected")
✅ Landmarks navigable
✅ Form errors announced
✅ Live regions announce updates
*/
```

**Automated Testing:**
```typescript
import { render } from '@testing-library/react';
import { axe } from 'jest-axe';

test('Accordion is accessible', async () => {
  const { container } = render(
    <Accordion title="Section 1">
      <p>Content</p>
    </Accordion>
  );
  
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

test('Button has correct ARIA', () => {
  const { getByRole } = render(
    <button aria-expanded="true" aria-controls="menu">
      Menu
    </button>
  );
  
  const button = getByRole('button', { name: 'Menu' });
  expect(button).toHaveAttribute('aria-expanded', 'true');
  expect(button).toHaveAttribute('aria-controls', 'menu');
});
```

#### 10. Best Practices

**ARIA Checklist:**
```typescript
/*
✅ DO:
- Use semantic HTML first
- Add ARIA when HTML is insufficient
- Test with screen readers
- Use aria-label for icon-only buttons
- Use aria-describedby for additional context
- Use aria-live for dynamic updates
- Keep ARIA attributes in sync with state

❌ DON'T:
- Use ARIA when semantic HTML exists
- Override native semantics
- Use empty aria-label
- Forget to test with screen readers
- Use aria-hidden on focusable elements
- Use ARIA as a substitute for fixing HTML

Remember:
"No ARIA is better than bad ARIA!"
*/
```

#### Summary

**ARIA Components:**

1. **Roles** - What an element is
   - `role="button"`, `role="dialog"`, `role="navigation"`

2. **Properties** - Element characteristics
   - `aria-label`, `aria-labelledby`, `aria-describedby`

3. **States** - Current condition
   - `aria-expanded`, `aria-hidden`, `aria-disabled`

4. **Live Regions** - Dynamic updates
   - `aria-live="polite"`, `role="alert"`

**When to Use ARIA:**
> "Semantic HTML first. ARIA when HTML is insufficient. Test with screen readers!"

**First Rule of ARIA:**
> "Don't use ARIA if you can use native HTML instead!"

**Golden Rule:**
> "No ARIA is better than bad ARIA. When in doubt, use semantic HTML."

</details>

---

### 203. How do you implement keyboard navigation?

<details>
<summary>View Answer</summary>

**Keyboard navigation** is essential for accessibility—many users rely entirely on keyboards or keyboard-like devices to navigate the web. Every interactive element must be accessible without a mouse.

#### 1. Basic Keyboard Navigation

**Standard Keys:**
```typescript
/*
Essential Keyboard Controls:

Tab: Move focus forward
Shift + Tab: Move focus backward
Enter: Activate buttons, links, submit forms
Space: Activate buttons, toggle checkboxes
Escape: Close modals, cancel actions
Arrow keys: Navigate within components (menus, tabs, lists)
Home/End: Jump to first/last item
Page Up/Down: Scroll content

Accessibility Requirements:
✅ All interactive elements focusable with Tab
✅ Logical tab order (left-to-right, top-to-bottom)
✅ Visible focus indicators
✅ Enter/Space activate buttons
✅ Escape closes modals/dialogs
✅ No keyboard traps
*/
```

#### 2. Tab Navigation

**tabIndex:**
```typescript
// tabIndex values:
// -1: Not in tab order, but programmatically focusable
//  0: In natural tab order
// >0: Custom tab order (AVOID - confusing!)

// ❌ BAD: Positive tabIndex (confusing order)
function BadTabOrder() {
  return (
    <div>
      <button tabIndex={3}>Third</button>
      <button tabIndex={1}>First</button>
      <button tabIndex={2}>Second</button>
    </div>
  );
}

// ✅ GOOD: Natural tab order
function GoodTabOrder() {
  return (
    <div>
      <button>First</button>
      <button>Second</button>
      <button>Third</button>
    </div>
  );
}

// ✅ GOOD: Use tabIndex={-1} for programmatic focus only
function ModalContent() {
  return (
    <div
      tabIndex={-1}  // Not in tab order
      ref={focusOnMount}  // But can be focused programmatically
    >
      Modal content
    </div>
  );
}
```

**Making Divs Interactive:**
```typescript
// ❌ BAD: Div with onClick (not keyboard accessible)
function BadButton() {
  return (
    <div onClick={handleClick}>
      Click me
    </div>
  );
}

// ✅ GOOD: Proper button
function GoodButton() {
  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}

// ✅ OK: If you MUST use div (rare cases)
function CustomButton() {
  const handleKeyDown = (e: React.KeyboardEvent) => {
    // Activate on Enter or Space
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      handleClick();
    }
  };
  
  return (
    <div
      role="button"
      tabIndex={0}  // Make focusable
      onClick={handleClick}
      onKeyDown={handleKeyDown}
    >
      Click me
    </div>
  );
}
```

#### 3. Arrow Key Navigation

**Menu Navigation:**
```typescript
import { useState, useRef, useEffect } from 'react';

function Dropdown({ items }: { items: string[] }) {
  const [isOpen, setIsOpen] = useState(false);
  const [focusedIndex, setFocusedIndex] = useState(0);
  const itemRefs = useRef<(HTMLButtonElement | null)[]>([]);
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (!isOpen) {
      // Open on Enter, Space, or Arrow Down
      if (e.key === 'Enter' || e.key === ' ' || e.key === 'ArrowDown') {
        e.preventDefault();
        setIsOpen(true);
        setFocusedIndex(0);
      }
      return;
    }
    
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex((prev) => 
          prev < items.length - 1 ? prev + 1 : 0  // Wrap to first
        );
        break;
        
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex((prev) => 
          prev > 0 ? prev - 1 : items.length - 1  // Wrap to last
        );
        break;
        
      case 'Home':
        e.preventDefault();
        setFocusedIndex(0);
        break;
        
      case 'End':
        e.preventDefault();
        setFocusedIndex(items.length - 1);
        break;
        
      case 'Escape':
        e.preventDefault();
        setIsOpen(false);
        break;
        
      case 'Enter':
      case ' ':
        e.preventDefault();
        handleSelect(items[focusedIndex]);
        setIsOpen(false);
        break;
    }
  };
  
  // Focus item when focusedIndex changes
  useEffect(() => {
    if (isOpen) {
      itemRefs.current[focusedIndex]?.focus();
    }
  }, [focusedIndex, isOpen]);
  
  return (
    <div onKeyDown={handleKeyDown}>
      <button
        aria-expanded={isOpen}
        aria-haspopup="true"
        onClick={() => setIsOpen(!isOpen)}
      >
        Menu
      </button>
      
      {isOpen && (
        <div role="menu">
          {items.map((item, index) => (
            <button
              key={item}
              ref={(el) => (itemRefs.current[index] = el)}
              role="menuitem"
              tabIndex={-1}  // Controlled by arrow keys
              onClick={() => handleSelect(item)}
            >
              {item}
            </button>
          ))}
        </div>
      )}
    </div>
  );
}
```

**Tab Navigation:**
```typescript
function Tabs({ tabs }: { tabs: Tab[] }) {
  const [activeTab, setActiveTab] = useState(0);
  const tabRefs = useRef<(HTMLButtonElement | null)[]>([]);
  
  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    let newIndex = index;
    
    switch (e.key) {
      case 'ArrowRight':
        e.preventDefault();
        newIndex = index < tabs.length - 1 ? index + 1 : 0;
        break;
        
      case 'ArrowLeft':
        e.preventDefault();
        newIndex = index > 0 ? index - 1 : tabs.length - 1;
        break;
        
      case 'Home':
        e.preventDefault();
        newIndex = 0;
        break;
        
      case 'End':
        e.preventDefault();
        newIndex = tabs.length - 1;
        break;
        
      default:
        return;
    }
    
    setActiveTab(newIndex);
    tabRefs.current[newIndex]?.focus();
  };
  
  return (
    <div>
      <div role="tablist" aria-label="Content tabs">
        {tabs.map((tab, index) => (
          <button
            key={tab.id}
            ref={(el) => (tabRefs.current[index] = el)}
            role="tab"
            aria-selected={activeTab === index}
            aria-controls={`panel-${tab.id}`}
            tabIndex={activeTab === index ? 0 : -1}  // Only active tab in tab order
            onClick={() => setActiveTab(index)}
            onKeyDown={(e) => handleKeyDown(e, index)}
          >
            {tab.label}
          </button>
        ))}
      </div>
      
      {tabs.map((tab, index) => (
        <div
          key={tab.id}
          role="tabpanel"
          id={`panel-${tab.id}`}
          aria-labelledby={`tab-${tab.id}`}
          hidden={activeTab !== index}
          tabIndex={0}  // Make panel scrollable with keyboard
        >
          {tab.content}
        </div>
      ))}
    </div>
  );
}
```

#### 4. Focus Trap (Modal)

**Trap Focus in Modal:**
```typescript
import { useEffect, useRef } from 'react';

function Modal({ isOpen, onClose, children }) {
  const modalRef = useRef<HTMLDivElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);
  
  useEffect(() => {
    if (!isOpen) return;
    
    // Save current focus
    previousFocusRef.current = document.activeElement as HTMLElement;
    
    // Get all focusable elements
    const getFocusableElements = () => {
      return modalRef.current?.querySelectorAll<HTMLElement>(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
    };
    
    // Focus first element
    const focusableElements = getFocusableElements();
    if (focusableElements && focusableElements.length > 0) {
      focusableElements[0].focus();
    }
    
    // Trap focus
    const handleTab = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return;
      
      const focusableElements = getFocusableElements();
      if (!focusableElements || focusableElements.length === 0) return;
      
      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];
      
      if (e.shiftKey) {
        // Shift + Tab: Move backward
        if (document.activeElement === firstElement) {
          e.preventDefault();
          lastElement.focus();
        }
      } else {
        // Tab: Move forward
        if (document.activeElement === lastElement) {
          e.preventDefault();
          firstElement.focus();
        }
      }
    };
    
    // Close on Escape
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        onClose();
      }
    };
    
    document.addEventListener('keydown', handleTab);
    document.addEventListener('keydown', handleEscape);
    
    return () => {
      document.removeEventListener('keydown', handleTab);
      document.removeEventListener('keydown', handleEscape);
      
      // Restore focus
      previousFocusRef.current?.focus();
    };
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay">
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
      >
        <h2 id="modal-title">Modal Title</h2>
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}
```

#### 5. Keyboard Shortcuts

**Global Shortcuts:**
```typescript
import { useEffect } from 'react';

function useKeyboardShortcuts() {
  useEffect(() => {
    const handleKeyDown = (e: KeyboardEvent) => {
      // Ignore if user is typing in input
      if (e.target instanceof HTMLInputElement || 
          e.target instanceof HTMLTextAreaElement) {
        return;
      }
      
      // Cmd/Ctrl + K: Open search
      if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
        e.preventDefault();
        openSearch();
      }
      
      // / : Focus search
      if (e.key === '/') {
        e.preventDefault();
        focusSearch();
      }
      
      // ? : Show help
      if (e.key === '?') {
        e.preventDefault();
        showHelp();
      }
    };
    
    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, []);
}

function App() {
  useKeyboardShortcuts();
  
  return (
    <div>
      {/* Shortcut hint */}
      <button aria-label="Search (Ctrl+K)">
        Search
      </button>
    </div>
  );
}
```

**Accessible Shortcuts:**
```typescript
function KeyboardShortcuts() {
  return (
    <div role="region" aria-label="Keyboard shortcuts">
      <h2>Keyboard Shortcuts</h2>
      <dl>
        <dt><kbd>Ctrl</kbd> + <kbd>K</kbd></dt>
        <dd>Open search</dd>
        
        <dt><kbd>/</kbd></dt>
        <dd>Focus search</dd>
        
        <dt><kbd>?</kbd></dt>
        <dd>Show this help</dd>
        
        <dt><kbd>Esc</kbd></dt>
        <dd>Close dialog</dd>
      </dl>
    </div>
  );
}

// CSS for kbd element
/*
kbd {
  padding: 2px 6px;
  border: 1px solid #ccc;
  border-radius: 3px;
  background: #f5f5f5;
  font-family: monospace;
  font-size: 0.9em;
}
*/
```

#### 6. Skip Links

**Skip to Main Content:**
```typescript
function Layout({ children }) {
  return (
    <div>
      {/* Skip link - first focusable element */}
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      
      <header>
        <nav>
          {/* 20+ navigation links */}
        </nav>
      </header>
      
      <main id="main-content" tabIndex={-1}>
        {children}
      </main>
    </div>
  );
}

// CSS: Show skip link on focus
/*
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
*/
```

#### 7. Focus Indicators

**Visible Focus:**
```css
/* ❌ BAD: Removing focus outline */
* {
  outline: none;
}

button:focus {
  outline: none;
}

/* ✅ GOOD: Custom focus indicator */
button:focus-visible {
  outline: 2px solid #007bff;
  outline-offset: 2px;
}

/* Focus-visible: Only show outline on keyboard focus, not mouse click */
button:focus:not(:focus-visible) {
  outline: none;
}

/* High contrast focus */
@media (prefers-contrast: high) {
  button:focus-visible {
    outline: 3px solid currentColor;
    outline-offset: 3px;
  }
}
```

**React Component:**
```typescript
function FocusButton({ children, ...props }) {
  return (
    <button
      {...props}
      className="focus-button"
    >
      {children}
    </button>
  );
}

// CSS
/*
.focus-button:focus-visible {
  outline: 2px solid #007bff;
  outline-offset: 2px;
  box-shadow: 0 0 0 4px rgba(0, 123, 255, 0.25);
}
*/
```

#### 8. Roving Tab Index

**Radio Group Pattern:**
```typescript
function RadioGroup({ options, value, onChange }) {
  const [focusedIndex, setFocusedIndex] = useState(0);
  const radioRefs = useRef<(HTMLInputElement | null)[]>([]);
  
  const handleKeyDown = (e: React.KeyboardEvent, index: number) => {
    let newIndex = index;
    
    switch (e.key) {
      case 'ArrowDown':
      case 'ArrowRight':
        e.preventDefault();
        newIndex = index < options.length - 1 ? index + 1 : 0;
        break;
        
      case 'ArrowUp':
      case 'ArrowLeft':
        e.preventDefault();
        newIndex = index > 0 ? index - 1 : options.length - 1;
        break;
        
      default:
        return;
    }
    
    setFocusedIndex(newIndex);
    onChange(options[newIndex].value);
    radioRefs.current[newIndex]?.focus();
  };
  
  return (
    <div role="radiogroup" aria-label="Options">
      {options.map((option, index) => (
        <label key={option.value}>
          <input
            ref={(el) => (radioRefs.current[index] = el)}
            type="radio"
            name="options"
            value={option.value}
            checked={value === option.value}
            onChange={() => onChange(option.value)}
            onKeyDown={(e) => handleKeyDown(e, index)}
            tabIndex={value === option.value ? 0 : -1}  // Only checked in tab order
          />
          {option.label}
        </label>
      ))}
    </div>
  );
}
```

#### 9. Testing Keyboard Navigation

**Manual Testing:**
```typescript
/*
Keyboard Testing Checklist:

✅ Unplug mouse and navigate entire app
✅ Tab through all interactive elements
✅ Focus indicators visible
✅ Tab order logical (left-to-right, top-to-bottom)
✅ Enter activates buttons/links
✅ Space toggles checkboxes/buttons
✅ Escape closes modals
✅ Arrow keys work in custom components
✅ No keyboard traps
✅ Skip links work
✅ All content reachable
*/
```

**Automated Testing:**
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('Button is keyboard accessible', async () => {
  const handleClick = jest.fn();
  render(<button onClick={handleClick}>Click me</button>);
  
  const button = screen.getByRole('button', { name: 'Click me' });
  
  // Tab to button
  await userEvent.tab();
  expect(button).toHaveFocus();
  
  // Activate with Enter
  await userEvent.keyboard('{Enter}');
  expect(handleClick).toHaveBeenCalled();
  
  // Activate with Space
  await userEvent.keyboard(' ');
  expect(handleClick).toHaveBeenCalledTimes(2);
});

test('Modal traps focus', async () => {
  render(
    <Modal isOpen>
      <button>First</button>
      <button>Last</button>
    </Modal>
  );
  
  const first = screen.getByRole('button', { name: 'First' });
  const last = screen.getByRole('button', { name: 'Last' });
  
  // Should focus first button
  expect(first).toHaveFocus();
  
  // Tab to last button
  await userEvent.tab();
  expect(last).toHaveFocus();
  
  // Tab wraps to first
  await userEvent.tab();
  expect(first).toHaveFocus();
  
  // Shift+Tab goes to last
  await userEvent.tab({ shift: true });
  expect(last).toHaveFocus();
});
```

#### 10. Best Practices

**Keyboard Navigation Checklist:**
```typescript
/*
✅ All interactive elements focusable with Tab
✅ Logical tab order (use DOM order, not tabIndex)
✅ Visible focus indicators
✅ Enter/Space activate buttons
✅ Escape closes modals/menus
✅ Arrow keys for widget navigation (tabs, menus)
✅ Skip links for navigation
✅ No keyboard traps
✅ Focus management (modals, routes)
✅ Test with keyboard only

Avoid:
❌ Positive tabIndex values
❌ Removing focus outlines without replacement
❌ Keyboard traps
❌ Non-interactive elements with click handlers
❌ Forgetting to test with keyboard
*/
```

#### Summary

**Essential Keys:**
- **Tab**: Navigate forward
- **Shift+Tab**: Navigate backward
- **Enter/Space**: Activate
- **Escape**: Close/cancel
- **Arrows**: Navigate within components

**Key Principles:**
1. **All interactive elements keyboard accessible**
2. **Logical tab order** (DOM order)
3. **Visible focus indicators**
4. **Focus management** in modals
5. **No keyboard traps**

**Golden Rule:**
> "If you can't navigate with keyboard only, it's not accessible!"

**Testing:**
> "Unplug your mouse and try to use your app. Everything should work!"

</details>

---

### 204. What is focus management in React?

<details>
<summary>View Answer</summary>

**Focus management** is the practice of controlling which element receives keyboard focus at any given time. Proper focus management is critical for keyboard and screen reader users to navigate your application efficiently.

#### 1. Why Focus Management Matters

**The Problem:**
```typescript
/*
Common Focus Issues:

1. Lost Focus:
   - User opens modal → focus stays on button behind modal
   - User closes modal → focus goes to <body>, user must Tab from start

2. Keyboard Traps:
   - User can Tab into element but not out
   - Focus stuck in modal without Escape

3. Unexpected Focus:
   - Page loads, focus on random element
   - Route changes, focus jumps unpredictably

4. No Focus Indicators:
   - User can't see where they are
   - Tab key seems broken

Impact:
- Keyboard users: Frustrating, slow navigation
- Screen reader users: Lost context, confusion
- Result: Inaccessible application
*/
```

#### 2. Basic Focus Control

**useRef and focus():**
```typescript
import { useRef, useEffect } from 'react';

function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useEffect(() => {
    // Focus input when component mounts
    inputRef.current?.focus();
  }, []);
  
  return (
    <input
      ref={inputRef}
      type="text"
      placeholder="Auto-focused"
    />
  );
}

// Or use autoFocus prop (careful with SSR!)
function SimpleAutoFocus() {
  return (
    <input
      autoFocus  // Only on client-side
      type="text"
    />
  );
}
```

**Focus on Condition:**
```typescript
function ConditionalFocus({ shouldFocus }: { shouldFocus: boolean }) {
  const buttonRef = useRef<HTMLButtonElement>(null);
  
  useEffect(() => {
    if (shouldFocus) {
      buttonRef.current?.focus();
    }
  }, [shouldFocus]);
  
  return <button ref={buttonRef}>Click me</button>;
}
```

#### 3. Modal Focus Management

**Complete Modal Implementation:**
```typescript
import { useEffect, useRef } from 'react';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

function Modal({ isOpen, onClose, title, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);
  const closeButtonRef = useRef<HTMLButtonElement>(null);
  const previousFocusRef = useRef<HTMLElement | null>(null);
  
  useEffect(() => {
    if (!isOpen) return;
    
    // 1. Save current focus
    previousFocusRef.current = document.activeElement as HTMLElement;
    
    // 2. Focus first focusable element (close button)
    setTimeout(() => {
      closeButtonRef.current?.focus();
    }, 0);
    
    // 3. Trap focus in modal
    const handleTab = (e: KeyboardEvent) => {
      if (e.key !== 'Tab' || !modalRef.current) return;
      
      const focusableElements = modalRef.current.querySelectorAll<HTMLElement>(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      
      if (focusableElements.length === 0) return;
      
      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];
      
      if (e.shiftKey) {
        // Shift+Tab: Wrap from first to last
        if (document.activeElement === firstElement) {
          e.preventDefault();
          lastElement.focus();
        }
      } else {
        // Tab: Wrap from last to first
        if (document.activeElement === lastElement) {
          e.preventDefault();
          firstElement.focus();
        }
      }
    };
    
    // 4. Close on Escape
    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        onClose();
      }
    };
    
    document.addEventListener('keydown', handleTab);
    document.addEventListener('keydown', handleEscape);
    
    return () => {
      document.removeEventListener('keydown', handleTab);
      document.removeEventListener('keydown', handleEscape);
      
      // 5. Restore focus when modal closes
      previousFocusRef.current?.focus();
    };
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  return (
    <div className="modal-overlay" onClick={onClose}>
      <div
        ref={modalRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        onClick={(e) => e.stopPropagation()}
      >
        <h2 id="modal-title">{title}</h2>
        {children}
        <button ref={closeButtonRef} onClick={onClose}>
          Close
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
      <button onClick={() => setIsOpen(true)}>
        Open Modal
      </button>
      
      <Modal
        isOpen={isOpen}
        onClose={() => setIsOpen(false)}
        title="My Modal"
      >
        <p>Modal content</p>
      </Modal>
    </div>
  );
}
```

**Reusable useFocusTrap Hook:**
```typescript
import { useEffect, useRef } from 'react';

function useFocusTrap(isActive: boolean) {
  const containerRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    if (!isActive) return;
    
    const container = containerRef.current;
    if (!container) return;
    
    // Save and restore focus
    const previousFocus = document.activeElement as HTMLElement;
    
    // Focus first element
    const focusableElements = container.querySelectorAll<HTMLElement>(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    
    if (focusableElements.length > 0) {
      focusableElements[0].focus();
    }
    
    // Trap focus
    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return;
      
      const focusableElements = container.querySelectorAll<HTMLElement>(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      
      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];
      
      if (e.shiftKey && document.activeElement === firstElement) {
        e.preventDefault();
        lastElement.focus();
      } else if (!e.shiftKey && document.activeElement === lastElement) {
        e.preventDefault();
        firstElement.focus();
      }
    };
    
    document.addEventListener('keydown', handleKeyDown);
    
    return () => {
      document.removeEventListener('keydown', handleKeyDown);
      previousFocus?.focus();
    };
  }, [isActive]);
  
  return containerRef;
}

// Usage
function SimpleModal({ isOpen, onClose }) {
  const modalRef = useFocusTrap(isOpen);
  
  if (!isOpen) return null;
  
  return (
    <div ref={modalRef} role="dialog" aria-modal="true">
      <h2>Modal</h2>
      <button onClick={onClose}>Close</button>
    </div>
  );
}
```

#### 4. Route Change Focus Management

**Focus on Route Change:**
```typescript
import { useEffect, useRef } from 'react';
import { useLocation } from 'react-router-dom';

function Layout({ children }) {
  const location = useLocation();
  const mainRef = useRef<HTMLElement>(null);
  
  // Focus main content on route change
  useEffect(() => {
    mainRef.current?.focus();
    
    // Announce page change to screen readers
    const pageTitle = document.title;
    const announcement = document.getElementById('route-announcer');
    if (announcement) {
      announcement.textContent = `Navigated to ${pageTitle}`;
    }
  }, [location.pathname]);
  
  return (
    <div>
      {/* Screen reader announcement */}
      <div
        id="route-announcer"
        role="status"
        aria-live="polite"
        className="sr-only"
      />
      
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      
      <header>
        <nav>{/* Navigation */}</nav>
      </header>
      
      <main
        id="main-content"
        ref={mainRef}
        tabIndex={-1}  // Make focusable programmatically
      >
        {children}
      </main>
    </div>
  );
}
```

**Next.js Route Focus:**
```typescript
import { useEffect, useRef } from 'react';
import { useRouter } from 'next/router';

export default function Layout({ children }) {
  const router = useRouter();
  const mainRef = useRef<HTMLElement>(null);
  
  useEffect(() => {
    const handleRouteChange = () => {
      mainRef.current?.focus();
    };
    
    router.events.on('routeChangeComplete', handleRouteChange);
    
    return () => {
      router.events.off('routeChangeComplete', handleRouteChange);
    };
  }, [router.events]);
  
  return (
    <main ref={mainRef} tabIndex={-1}>
      {children}
    </main>
  );
}
```

#### 5. Focus After Actions

**Delete Item Focus:**
```typescript
function TodoList() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const itemRefs = useRef<Map<string, HTMLButtonElement>>(new Map());
  
  const deleteTodo = (id: string) => {
    const index = todos.findIndex(t => t.id === id);
    setTodos(todos.filter(t => t.id !== id));
    
    // Focus next item, or previous if last item deleted
    setTimeout(() => {
      const nextTodo = todos[index + 1] || todos[index - 1];
      if (nextTodo) {
        itemRefs.current.get(nextTodo.id)?.focus();
      }
    }, 0);
  };
  
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>
          {todo.text}
          <button
            ref={(el) => {
              if (el) itemRefs.current.set(todo.id, el);
              else itemRefs.current.delete(todo.id);
            }}
            onClick={() => deleteTodo(todo.id)}
            aria-label={`Delete ${todo.text}`}
          >
            Delete
          </button>
        </li>
      ))}
    </ul>
  );
}
```

**Form Submission Focus:**
```typescript
function ContactForm() {
  const [submitted, setSubmitted] = useState(false);
  const successMessageRef = useRef<HTMLDivElement>(null);
  const firstInputRef = useRef<HTMLInputElement>(null);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    await submitForm();
    setSubmitted(true);
    
    // Focus success message
    setTimeout(() => {
      successMessageRef.current?.focus();
    }, 0);
  };
  
  if (submitted) {
    return (
      <div
        ref={successMessageRef}
        tabIndex={-1}
        role="status"
        aria-live="polite"
      >
        <h2>Thank you!</h2>
        <p>Your message has been sent.</p>
        <button onClick={() => setSubmitted(false)}>
          Send another message
        </button>
      </div>
    );
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={firstInputRef}
        type="text"
        placeholder="Name"
      />
      <button type="submit">Send</button>
    </form>
  );
}
```

#### 6. Dropdown Focus Management

**Accessible Dropdown:**
```typescript
import { useState, useRef, useEffect } from 'react';

function Dropdown({ label, items, onSelect }) {
  const [isOpen, setIsOpen] = useState(false);
  const [focusedIndex, setFocusedIndex] = useState(0);
  const buttonRef = useRef<HTMLButtonElement>(null);
  const itemRefs = useRef<(HTMLButtonElement | null)[]>([]);
  
  // Focus item when index changes
  useEffect(() => {
    if (isOpen) {
      itemRefs.current[focusedIndex]?.focus();
    }
  }, [focusedIndex, isOpen]);
  
  const openDropdown = () => {
    setIsOpen(true);
    setFocusedIndex(0);
  };
  
  const closeDropdown = () => {
    setIsOpen(false);
    buttonRef.current?.focus();  // Return focus to button
  };
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (!isOpen) {
      if (e.key === 'Enter' || e.key === ' ' || e.key === 'ArrowDown') {
        e.preventDefault();
        openDropdown();
      }
      return;
    }
    
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault();
        setFocusedIndex((prev) => 
          prev < items.length - 1 ? prev + 1 : prev
        );
        break;
        
      case 'ArrowUp':
        e.preventDefault();
        setFocusedIndex((prev) => prev > 0 ? prev - 1 : prev);
        break;
        
      case 'Home':
        e.preventDefault();
        setFocusedIndex(0);
        break;
        
      case 'End':
        e.preventDefault();
        setFocusedIndex(items.length - 1);
        break;
        
      case 'Escape':
        e.preventDefault();
        closeDropdown();
        break;
        
      case 'Enter':
      case ' ':
        e.preventDefault();
        onSelect(items[focusedIndex]);
        closeDropdown();
        break;
    }
  };
  
  return (
    <div onKeyDown={handleKeyDown}>
      <button
        ref={buttonRef}
        aria-haspopup="true"
        aria-expanded={isOpen}
        onClick={() => (isOpen ? closeDropdown() : openDropdown())}
      >
        {label}
      </button>
      
      {isOpen && (
        <div role="menu">
          {items.map((item, index) => (
            <button
              key={item.id}
              ref={(el) => (itemRefs.current[index] = el)}
              role="menuitem"
              tabIndex={-1}  // Managed by arrow keys
              onClick={() => {
                onSelect(item);
                closeDropdown();
              }}
            >
              {item.label}
            </button>
          ))}
        </div>
      )}
    </div>
  );
}
```

#### 7. Skip Links

**Skip to Main Content:**
```typescript
function SkipLink() {
  return (
    <a
      href="#main-content"
      className="skip-link"
      onClick={(e) => {
        e.preventDefault();
        const main = document.getElementById('main-content');
        if (main) {
          main.focus();
          main.scrollIntoView();
        }
      }}
    >
      Skip to main content
    </a>
  );
}

function Layout({ children }) {
  return (
    <div>
      <SkipLink />
      
      <header>
        <nav>{/* 50+ links */}</nav>
      </header>
      
      <main id="main-content" tabIndex={-1}>
        {children}
      </main>
    </div>
  );
}

// CSS
/*
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  z-index: 1000;
  text-decoration: none;
}

.skip-link:focus {
  top: 0;
}
*/
```

#### 8. Focus Indicators

**Custom Focus Styles:**
```css
/* Modern focus indicator */
*:focus-visible {
  outline: 2px solid #007bff;
  outline-offset: 2px;
}

/* Remove outline on mouse click, keep on keyboard */
*:focus:not(:focus-visible) {
  outline: none;
}

/* Button specific */
button:focus-visible {
  outline: 2px solid #007bff;
  outline-offset: 2px;
  box-shadow: 0 0 0 4px rgba(0, 123, 255, 0.25);
}

/* Input specific */
input:focus-visible,
textarea:focus-visible {
  outline: 2px solid #007bff;
  border-color: #007bff;
}

/* High contrast mode */
@media (prefers-contrast: high) {
  *:focus-visible {
    outline: 3px solid currentColor;
    outline-offset: 3px;
  }
}
```

#### 9. Testing Focus Management

**Manual Testing:**
```typescript
/*
Focus Management Testing Checklist:

✅ Modal:
  - Opens: Focus moves to modal
  - Tab: Focus stays in modal (trapped)
  - Escape: Modal closes, focus returns to trigger

✅ Route Change:
  - Navigate to new page
  - Focus moves to main content
  - Screen reader announces page

✅ Delete Item:
  - Delete item from list
  - Focus moves to next/previous item
  - Not lost to <body>

✅ Form Submission:
  - Submit form
  - Focus moves to success message
  - Or to first error

✅ Dropdown:
  - Open: Focus first item
  - Close: Focus returns to button
  - Escape works

✅ Skip Links:
  - Tab: Skip link appears
  - Activate: Focus moves to main
*/
```

**Automated Testing:**
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('Modal manages focus correctly', async () => {
  const handleClose = jest.fn();
  const user = userEvent.setup();
  
  const { rerender } = render(
    <>
      <button>Trigger</button>
      <Modal isOpen={false} onClose={handleClose} />
    </>
  );
  
  const trigger = screen.getByRole('button', { name: 'Trigger' });
  trigger.focus();
  expect(trigger).toHaveFocus();
  
  // Open modal
  rerender(
    <>
      <button>Trigger</button>
      <Modal isOpen={true} onClose={handleClose} />
    </>
  );
  
  // Focus should move to modal
  const closeButton = screen.getByRole('button', { name: 'Close' });
  expect(closeButton).toHaveFocus();
  
  // Close modal
  await user.click(closeButton);
  
  // Focus should return to trigger
  expect(trigger).toHaveFocus();
});
```

#### 10. Best Practices

**Focus Management Checklist:**
```typescript
/*
✅ Modals:
  - Save previous focus
  - Focus first element in modal
  - Trap focus in modal
  - Restore focus on close

✅ Route Changes:
  - Focus main content on navigation
  - Announce page change to screen readers
  - Skip links available

✅ Dynamic Content:
  - Focus new content when it appears
  - Focus next item after deletion
  - Focus error messages

✅ Forms:
  - Focus first error on validation
  - Focus success message after submit
  - Don't auto-focus inputs unnecessarily

✅ Dropdowns:
  - Focus first item on open
  - Return focus to trigger on close
  - Arrow key navigation

✅ Focus Indicators:
  - Always visible
  - High contrast
  - Never remove without replacement
*/
```

#### Summary

**Core Focus Management:**

1. **Save and Restore** - Remember where focus was
2. **Focus on Open** - Move focus to new content
3. **Trap Focus** - Keep focus in modals
4. **Focus on Close** - Return focus to trigger
5. **Focus on Delete** - Move to next/previous item
6. **Route Changes** - Focus main content
7. **Visible Indicators** - Always show focus

**Key Principle:**
> "Never leave users wondering where they are. Always manage focus explicitly!"

**Golden Rule:**
> "Save focus, move focus, restore focus. In that order!"

</details>

---

### 205. How do you test accessibility in React apps?

<details>
<summary>View Answer</summary>

**Testing accessibility** ensures your React app works for everyone, including users with disabilities. A combination of automated tools, manual testing, and user testing provides comprehensive coverage.

#### 1. Automated Testing Tools

**axe-core (Most Popular):**
```bash
npm install --save-dev @axe-core/react
```

```typescript
// Development mode accessibility checking
import React from 'react';
import ReactDOM from 'react-dom';

if (process.env.NODE_ENV !== 'production') {
  import('@axe-core/react').then((axe) => {
    axe.default(React, ReactDOM, 1000);  // Check every 1 second
  });
}

import App from './App';

ReactDOM.render(<App />, document.getElementById('root'));

// Logs accessibility violations to console in development
```

**jest-axe (Unit Testing):**
```bash
npm install --save-dev jest-axe
```

```typescript
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('Button has no accessibility violations', async () => {
  const { container } = render(
    <button onClick={() => {}}>Click me</button>
  );
  
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

test('Form is accessible', async () => {
  const { container } = render(
    <form>
      <label htmlFor="name">Name:</label>
      <input id="name" type="text" />
      <button type="submit">Submit</button>
    </form>
  );
  
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

// Test with specific rules
test('Check only color contrast', async () => {
  const { container } = render(<App />);
  
  const results = await axe(container, {
    rules: {
      'color-contrast': { enabled: true },
    },
  });
  
  expect(results).toHaveNoViolations();
});
```

**React Testing Library:**
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('Button is accessible by role', () => {
  render(<button>Click me</button>);
  
  // Find by accessible role
  const button = screen.getByRole('button', { name: 'Click me' });
  expect(button).toBeInTheDocument();
});

test('Image has alt text', () => {
  render(<img src="photo.jpg" alt="Person smiling" />);
  
  const image = screen.getByAltText('Person smiling');
  expect(image).toBeInTheDocument();
});

test('Form is accessible', () => {
  render(
    <form>
      <label htmlFor="email">Email:</label>
      <input id="email" type="email" aria-required="true" />
    </form>
  );
  
  // Find by label text
  const input = screen.getByLabelText('Email:');
  expect(input).toBeRequired();
  expect(input).toHaveAttribute('aria-required', 'true');
});

test('Modal traps focus', async () => {
  const user = userEvent.setup();
  
  render(
    <Modal isOpen>
      <button>First</button>
      <button>Last</button>
    </Modal>
  );
  
  const first = screen.getByRole('button', { name: 'First' });
  const last = screen.getByRole('button', { name: 'Last' });
  
  // Focus should start on first button
  expect(first).toHaveFocus();
  
  // Tab to last button
  await user.tab();
  expect(last).toHaveFocus();
  
  // Tab should wrap to first
  await user.tab();
  expect(first).toHaveFocus();
});

test('Error message is announced', () => {
  render(
    <div role="alert">Error: Password is required</div>
  );
  
  const alert = screen.getByRole('alert');
  expect(alert).toHaveTextContent('Error: Password is required');
});
```

#### 2. Browser DevTools

**Chrome Lighthouse:**
```typescript
/*
How to use:

1. Open Chrome DevTools (F12)
2. Go to "Lighthouse" tab
3. Select "Accessibility" category
4. Click "Generate report"

Lighthouse checks:
✅ [aria-*] attributes are valid
✅ [role] attributes are valid
✅ Buttons have accessible names
✅ Images have alt text
✅ Form elements have labels
✅ Color contrast meets WCAG
✅ [id] attributes are unique
✅ Heading order is logical
✅ HTML lang attribute exists
✅ Lists contain only <li> elements
✅ No keyboard traps

Score:
- 90-100: Good
- 50-89: Needs improvement
- 0-49: Poor

Goal: 100/100
*/
```

**Chrome Accessibility Tree:**
```typescript
/*
How to view:

1. Open Chrome DevTools
2. Elements tab
3. Click "Accessibility" pane (bottom)
4. Inspect element

Shows:
- Accessible name
- Role
- States (expanded, selected, etc.)
- Description
- Keyboard focusable

Example:
<button aria-label="Close menu" aria-expanded="true">
  ×
</button>

Accessibility Tree:
{
  role: "button",
  name: "Close menu",
  focusable: true,
  expanded: true
}
*/
```

**axe DevTools Extension:**
```bash
# Install from Chrome Web Store:
# https://chrome.google.com/webstore/detail/axe-devtools

# Usage:
# 1. Open DevTools
# 2. Go to "axe DevTools" tab
# 3. Click "Scan ALL of my page"
# 4. Review violations

# Shows:
# - Issue description
# - WCAG level (A, AA, AAA)
# - Affected elements
# - How to fix
# - "Inspect node" button
```

#### 3. Manual Keyboard Testing

**Keyboard Testing Checklist:**
```typescript
/*
Complete Manual Test (15 minutes):

1. UNPLUG YOUR MOUSE!

2. Tab Navigation:
   ✅ Press Tab repeatedly
   ✅ All interactive elements focusable
   ✅ Focus indicator always visible
   ✅ Logical tab order (top to bottom, left to right)
   ✅ No elements skipped
   ✅ No keyboard traps

3. Shift+Tab:
   ✅ Reverse navigation works
   ✅ Same order in reverse

4. Enter Key:
   ✅ Activates buttons
   ✅ Submits forms
   ✅ Follows links

5. Space Key:
   ✅ Activates buttons
   ✅ Toggles checkboxes
   ✅ Scrolls page

6. Arrow Keys:
   ✅ Navigate radio buttons
   ✅ Navigate dropdowns
   ✅ Navigate tabs
   ✅ Navigate menus

7. Escape Key:
   ✅ Closes modals
   ✅ Closes dropdowns
   ✅ Cancels actions

8. Home/End:
   ✅ Jump to first/last item in lists
   ✅ Jump to start/end of input

9. Skip Links:
   ✅ Tab shows "Skip to content"
   ✅ Enter jumps to main content

10. Forms:
    ✅ All inputs accessible
    ✅ Errors announced
    ✅ Validation on submit
*/
```

#### 4. Screen Reader Testing

**VoiceOver (Mac):**
```typescript
/*
How to use:

1. Enable: Cmd + F5
2. Basic commands:
   - VO + Right Arrow: Next element
   - VO + Left Arrow: Previous element
   - VO + U: Open rotor (navigation menu)
   - VO + A: Read from current position
   - VO + Shift + Down: Enter element (like a list)
   - Control: Stop reading

3. Test checklist:
   ✅ Page title announced
   ✅ Headings announced with level ("Heading level 1")
   ✅ Landmarks announced ("navigation landmark", "main landmark")
   ✅ Links descriptive (not "click here")
   ✅ Buttons labeled ("Close menu, button")
   ✅ Images have alt text
   ✅ Form inputs labeled
   ✅ Form errors announced
   ✅ Dynamic updates announced (aria-live)
   ✅ Modal focus trapped
   ✅ Skip links work

Common Issues:
❌ "Button" - Missing label
❌ "Image" - Missing alt text
❌ "Edit text" - Missing label
❌ Nothing announced - aria-hidden or display:none
❌ "Clickable" on div - Use button instead
*/
```

**NVDA (Windows - Free):**
```typescript
/*
Download: https://www.nvaccess.org/

Basic commands:
- Ctrl: Stop reading
- Insert + Down: Start reading
- Down Arrow: Next line
- H: Next heading
- B: Next button
- K: Next link
- F: Next form field
- D: Next landmark

Test same checklist as VoiceOver
*/
```

#### 5. Visual Testing

**Color Contrast:**
```typescript
/*
Tools:

1. Chrome DevTools:
   - Inspect element
   - See "Contrast ratio" in Styles pane
   - Shows AA/AAA pass/fail

2. WebAIM Contrast Checker:
   - https://webaim.org/resources/contrastchecker/
   - Enter foreground/background colors
   - See WCAG AA/AAA results

3. Lighthouse:
   - Automatically checks all text

Requirements:
✅ Normal text: 4.5:1 (AA), 7:1 (AAA)
✅ Large text (18pt+): 3:1 (AA), 4.5:1 (AAA)
✅ UI components: 3:1

Common failures:
❌ Gray text (#777) on white: 2.8:1 - FAIL
✅ Dark gray (#595959) on white: 4.6:1 - PASS AA
✅ Black on white: 21:1 - PASS AAA
*/
```

**Zoom Testing:**
```typescript
/*
Test at different zoom levels:

1. 200% zoom:
   ✅ Layout doesn't break
   ✅ Text readable
   ✅ No horizontal scroll
   ✅ Buttons still clickable
   ✅ Forms still usable

2. 400% zoom:
   ✅ Text still visible
   ✅ Single column layout
   ✅ Content reflows

3. Browser text size:
   - Settings → Increase text size
   ✅ Text scales
   ✅ Layout adapts
*/
```

**Color Blindness:**
```typescript
/*
Chrome DevTools Emulation:

1. DevTools → Rendering tab
2. "Emulate vision deficiencies"
3. Select:
   - Protanopia (red-blind)
   - Deuteranopia (green-blind)
   - Tritanopia (blue-blind)
   - Achromatopsia (no color)

Test:
✅ Information not conveyed by color alone
✅ Error states have icons/text
✅ Charts have patterns/labels
✅ Links distinguishable from text

Bad:
❌ "Required fields in red"
❌ "Green = good, Red = bad" without labels
❌ Links only distinguished by color

Good:
✅ "Required fields marked with *"
✅ Icons + color
✅ Links underlined + color
*/
```

#### 6. Continuous Integration

**Automated CI Tests:**
```typescript
// package.json
{
  "scripts": {
    "test:a11y": "jest --testPathPattern=a11y",
    "test:a11y:ci": "CI=true npm run test:a11y"
  }
}

// .github/workflows/a11y.yml
name: Accessibility Tests

on: [push, pull_request]

jobs:
  a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - run: npm ci
      - run: npm run test:a11y:ci
      - run: npm run build
      - run: npx lighthouse ${{ secrets.STAGING_URL }} --only-categories=accessibility --chrome-flags="--headless"
```

**Pre-commit Hook:**
```bash
# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npm run test:a11y
```

#### 7. Accessibility Testing in Cypress

**cypress-axe:**
```bash
npm install --save-dev cypress-axe axe-core
```

```typescript
// cypress/support/commands.ts
import 'cypress-axe';

// cypress/e2e/accessibility.cy.ts
describe('Accessibility', () => {
  beforeEach(() => {
    cy.visit('/');
    cy.injectAxe();
  });
  
  it('Has no accessibility violations on homepage', () => {
    cy.checkA11y();
  });
  
  it('Has no violations in modal', () => {
    cy.get('[data-testid="open-modal"]').click();
    cy.checkA11y('[role="dialog"]');
  });
  
  it('Has no color contrast violations', () => {
    cy.checkA11y(null, {
      rules: {
        'color-contrast': { enabled: true },
      },
    });
  });
  
  it('Form is accessible', () => {
    cy.visit('/contact');
    cy.checkA11y('form', {
      rules: {
        'label': { enabled: true },
        'aria-required-attr': { enabled: true },
      },
    });
  });
});
```

#### 8. Testing Checklist

**Complete A11y Testing:**
```typescript
/*
Comprehensive Accessibility Testing Checklist:

✅ AUTOMATED:
  ✅ jest-axe on all components
  ✅ Lighthouse score 100
  ✅ axe DevTools scan
  ✅ CI/CD integration

✅ KEYBOARD:
  ✅ Tab through entire app
  ✅ Focus visible
  ✅ Tab order logical
  ✅ Enter/Space activate
  ✅ Escape closes
  ✅ Arrow keys work
  ✅ No keyboard traps

✅ SCREEN READER:
  ✅ VoiceOver (Mac) test
  ✅ NVDA (Windows) test
  ✅ Page structure clear
  ✅ All content announced
  ✅ ARIA correct
  ✅ Landmarks used

✅ VISUAL:
  ✅ Color contrast 4.5:1+
  ✅ 200% zoom works
  ✅ Color blind friendly
  ✅ Text resizable
  ✅ No info by color only

✅ FORMS:
  ✅ All inputs labeled
  ✅ Errors announced
  ✅ Required marked
  ✅ Autocomplete hints

✅ CONTENT:
  ✅ Alt text on images
  ✅ Captions on videos
  ✅ Transcripts for audio
  ✅ Descriptive link text
  ✅ Logical heading order

✅ USER TESTING:
  ✅ Test with real users
  ✅ Test with assistive tech users
  ✅ Iterate based on feedback
*/
```

#### 9. Common Issues and Fixes

**Top 10 A11y Issues:**
```typescript
/*
1. Missing alt text
   ❌ <img src="photo.jpg" />
   ✅ <img src="photo.jpg" alt="Person smiling" />

2. Missing form labels
   ❌ <input type="text" placeholder="Name" />
   ✅ <label htmlFor="name">Name:</label>
      <input id="name" type="text" />

3. Low color contrast
   ❌ color: #777; (2.8:1)
   ✅ color: #595959; (4.6:1)

4. Non-semantic HTML
   ❌ <div onClick={click}>Click</div>
   ✅ <button onClick={click}>Click</button>

5. Missing heading hierarchy
   ❌ <h1> then <h3>
   ✅ <h1> then <h2> then <h3>

6. Keyboard trap
   ❌ Modal with no way to close via keyboard
   ✅ Modal with Escape key and Tab trap

7. Empty links/buttons
   ❌ <button><Icon /></button>
   ✅ <button aria-label="Close"><Icon /></button>

8. Missing focus indicators
   ❌ button:focus { outline: none; }
   ✅ button:focus-visible { outline: 2px solid blue; }

9. Incorrect ARIA usage
   ❌ <button role="link">
   ✅ <button> (already has role="button")

10. Not announcing dynamic updates
    ❌ <div>Loading complete</div>
    ✅ <div role="status" aria-live="polite">Loading complete</div>
*/
```

#### 10. Resources and Tools

**Essential Tools:**
```typescript
/*
Automated Testing:
- axe-core: https://github.com/dequelabs/axe-core
- jest-axe: https://github.com/nickcolley/jest-axe
- axe DevTools: Chrome extension
- Lighthouse: Built into Chrome
- Pa11y: CLI tool
- Accessibility Insights: Microsoft tool

Screen Readers:
- VoiceOver: Mac (free, built-in)
- NVDA: Windows (free)
- JAWS: Windows (commercial)
- TalkBack: Android (free)

Color Tools:
- WebAIM Contrast Checker
- Colorblind emulation (Chrome DevTools)
- Contrast Ratio tool by Lea Verou

Learning Resources:
- MDN Accessibility Guide
- WCAG 2.1 Guidelines
- WebAIM articles
- A11y Project
- Inclusive Components
*/
```

#### Summary

**Multi-Layered Testing Approach:**

1. **Automated** (30% coverage)
   - jest-axe in unit tests
   - Lighthouse in CI
   - axe DevTools during dev

2. **Manual** (50% coverage)
   - Keyboard testing
   - Screen reader testing
   - Visual inspection

3. **User Testing** (20% coverage)
   - Real users with disabilities
   - Feedback and iteration

**Key Principle:**
> "Automated tools catch 30-40% of issues. Manual testing is essential!"

**Golden Rule:**
> "Test early, test often, test with real users and assistive technologies!"

</details>

---

### 206. What is the React ARIA library?

<details>
<summary>View Answer</summary>

**React Aria** is a library of React Hooks by Adobe that provides accessible UI primitives for your design system. It handles complex accessibility, internationalization, and interactions so you can focus on your design.

#### 1. What is React Aria?

**Overview:**
```typescript
/*
React Aria by Adobe:
https://react-spectrum.adobe.com/react-aria/

What it provides:
✅ Accessible components via hooks
✅ Keyboard navigation
✅ Focus management
✅ ARIA attributes
✅ Screen reader support
✅ Internationalization (RTL, date/number formats)
✅ Mobile touch interactions
✅ Unstyled (bring your own styles)

Why use it:
✅ Accessibility is HARD to get right
✅ Save 100+ hours of development
✅ Battle-tested in Adobe products
✅ WCAG 2.1 Level AA compliant
✅ Tested with screen readers
✅ Handles edge cases you didn't know existed

Components:
- Buttons, Links
- Text fields, Select, Combobox
- Dialog, Modal, Popover
- Tabs, Accordion, Menu
- Calendar, DatePicker
- Table, List, Grid
- And 30+ more...
*/
```

#### 2. Installation

```bash
# Install React Aria hooks
npm install react-aria

# Or install individual packages
npm install @react-aria/button
npm install @react-aria/dialog
npm install @react-aria/menu
```

#### 3. Basic Button Example

**useButton Hook:**
```typescript
import { useButton } from '@react-aria/button';
import { useRef } from 'react';

function Button(props) {
  const ref = useRef<HTMLButtonElement>(null);
  const { buttonProps } = useButton(props, ref);
  
  return (
    <button
      {...buttonProps}
      ref={ref}
      className="my-button"
    >
      {props.children}
    </button>
  );
}

// Usage
<Button onPress={() => console.log('Pressed')}>
  Click me
</Button>

// useButton provides:
// ✅ onClick handler
// ✅ onKeyDown (Enter/Space)
// ✅ role="button" if needed
// ✅ aria-disabled if disabled
// ✅ Prevent double-click
// ✅ Touch events on mobile
```

**Why useButton over <button>?**
```typescript
/*
useButton handles:

1. Enter AND Space keys (native button only handles Space in some browsers)
2. Prevents text selection on double-click
3. Supports onPress instead of onClick (better for touch)
4. Prevents multiple activations
5. Works with any element (not just <button>)
6. Handles disabled state consistently
7. Touch events for mobile

Example: Press vs Click
- onClick: Fired on mouse up
- onPress: Fired only if mouse up is on same element
  (Better UX: user can drag away to cancel)
*/
```

#### 4. Dialog/Modal

**useDialog and useModal:**
```typescript
import { useDialog } from '@react-aria/dialog';
import { useModal, useOverlay } from '@react-aria/overlays';
import { useRef } from 'react';
import { OverlayContainer } from '@react-aria/overlays';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: React.ReactNode;
}

function Modal({ isOpen, onClose, title, children }: ModalProps) {
  const ref = useRef<HTMLDivElement>(null);
  
  // useModal: Focus management and ARIA
  const { modalProps } = useModal();
  
  // useOverlay: Click outside and Escape to close
  const { overlayProps, underlayProps } = useOverlay(
    {
      isOpen,
      onClose,
      isDismissable: true,  // Click outside to close
    },
    ref
  );
  
  // useDialog: Dialog ARIA attributes
  const { dialogProps, titleProps } = useDialog({}, ref);
  
  if (!isOpen) return null;
  
  return (
    <OverlayContainer>
      <div className="modal-underlay" {...underlayProps}>
        <div
          {...overlayProps}
          {...dialogProps}
          {...modalProps}
          ref={ref}
          className="modal"
        >
          <h2 {...titleProps}>{title}</h2>
          {children}
          <button onClick={onClose}>Close</button>
        </div>
      </div>
    </OverlayContainer>
  );
}

// React Aria automatically:
// ✅ Traps focus in modal
// ✅ Restores focus on close
// ✅ Closes on Escape
// ✅ Closes on click outside
// ✅ Sets aria-modal="true"
// ✅ Sets aria-labelledby
// ✅ Hides background content from screen readers
```

#### 5. Menu/Dropdown

**useMenu Hook:**
```typescript
import { useMenuTrigger } from '@react-aria/menu';
import { useButton } from '@react-aria/button';
import { useMenu, useMenuItem } from '@react-aria/menu';
import { useRef, useState } from 'react';
import { Item } from '@react-stately/collections';
import { useTreeState } from '@react-stately/tree';

function MenuButton() {
  const [isOpen, setIsOpen] = useState(false);
  
  const triggerRef = useRef<HTMLButtonElement>(null);
  const menuRef = useRef<HTMLUListElement>(null);
  
  // Menu state
  const state = useTreeState({
    selectionMode: 'none',
    children: [
      <Item key="new">New</Item>,
      <Item key="open">Open</Item>,
      <Item key="save">Save</Item>,
    ],
  });
  
  // Trigger button
  const { menuTriggerProps, menuProps } = useMenuTrigger(
    {},
    { isOpen, setOpen: setIsOpen },
    triggerRef
  );
  
  const { buttonProps } = useButton(menuTriggerProps, triggerRef);
  
  return (
    <div>
      <button {...buttonProps} ref={triggerRef}>
        Menu
      </button>
      
      {isOpen && (
        <ul {...menuProps} ref={menuRef} className="menu">
          {[...state.collection].map((item) => (
            <MenuItem key={item.key} item={item} state={state} />
          ))}
        </ul>
      )}
    </div>
  );
}

function MenuItem({ item, state }) {
  const ref = useRef<HTMLLIElement>(null);
  const { menuItemProps } = useMenuItem({ key: item.key }, state, ref);
  
  return (
    <li {...menuItemProps} ref={ref}>
      {item.rendered}
    </li>
  );
}

// React Aria automatically:
// ✅ Arrow key navigation
// ✅ Home/End keys
// ✅ Type-ahead search
// ✅ Enter/Space to select
// ✅ Escape to close
// ✅ Focus management
// ✅ ARIA roles and attributes
// ✅ Screen reader announcements
```

#### 6. Text Field

**useTextField Hook:**
```typescript
import { useTextField } from '@react-aria/textfield';
import { useRef } from 'react';

interface TextFieldProps {
  label: string;
  description?: string;
  errorMessage?: string;
  value: string;
  onChange: (value: string) => void;
  isRequired?: boolean;
  isDisabled?: boolean;
}

function TextField(props: TextFieldProps) {
  const { label, description, errorMessage } = props;
  const ref = useRef<HTMLInputElement>(null);
  
  const { labelProps, inputProps, descriptionProps, errorMessageProps } =
    useTextField(props, ref);
  
  return (
    <div className="text-field">
      <label {...labelProps}>{label}</label>
      <input
        {...inputProps}
        ref={ref}
        className="input"
      />
      {description && (
        <div {...descriptionProps} className="description">
          {description}
        </div>
      )}
      {errorMessage && (
        <div {...errorMessageProps} className="error">
          {errorMessage}
        </div>
      )}
    </div>
  );
}

// Usage
<TextField
  label="Email"
  description="We'll never share your email"
  errorMessage={errors.email}
  value={email}
  onChange={setEmail}
  isRequired
/>

// React Aria automatically:
// ✅ Associates label with input (htmlFor/id)
// ✅ aria-required if required
// ✅ aria-invalid if error
// ✅ aria-describedby for description/error
// ✅ Generates unique IDs
// ✅ Handles disabled state
```

#### 7. Tabs

**useTabs Hook:**
```typescript
import { useTabs, useTab, useTabList, useTabPanel } from '@react-aria/tabs';
import { useTabListState } from '@react-stately/tabs';
import { useRef } from 'react';
import { Item } from '@react-stately/collections';

function Tabs(props) {
  const state = useTabListState(props);
  const ref = useRef<HTMLDivElement>(null);
  const { tabListProps } = useTabList(props, state, ref);
  
  return (
    <div>
      <div {...tabListProps} ref={ref} className="tab-list">
        {[...state.collection].map((item) => (
          <Tab key={item.key} item={item} state={state} />
        ))}
      </div>
      <TabPanel key={state.selectedItem?.key} state={state} />
    </div>
  );
}

function Tab({ item, state }) {
  const ref = useRef<HTMLDivElement>(null);
  const { tabProps } = useTab({ key: item.key }, state, ref);
  
  return (
    <div {...tabProps} ref={ref} className="tab">
      {item.rendered}
    </div>
  );
}

function TabPanel({ state, ...props }) {
  const ref = useRef<HTMLDivElement>(null);
  const { tabPanelProps } = useTabPanel(props, state, ref);
  
  return (
    <div {...tabPanelProps} ref={ref} className="tab-panel">
      {state.selectedItem?.props.children}
    </div>
  );
}

// Usage
<Tabs>
  <Item key="tab1" title="Tab 1">
    <p>Content 1</p>
  </Item>
  <Item key="tab2" title="Tab 2">
    <p>Content 2</p>
  </Item>
</Tabs>

// React Aria automatically:
// ✅ Arrow key navigation
// ✅ Home/End keys
// ✅ Automatic activation or manual (configurable)
// ✅ Only selected tab in tab order
// ✅ aria-selected
// ✅ aria-controls linking tab to panel
// ✅ role="tablist", role="tab", role="tabpanel"
```

#### 8. Select/Combobox

**useSelect Hook:**
```typescript
import { useSelect } from '@react-aria/select';
import { useSelectState } from '@react-stately/select';
import { Item } from '@react-stately/collections';
import { useButton } from '@react-aria/button';
import { useListBox, useOption } from '@react-aria/listbox';

function Select(props) {
  const state = useSelectState(props);
  const ref = useRef<HTMLDivElement>(null);
  const { triggerProps, valueProps, menuProps } = useSelect(
    props,
    state,
    ref
  );
  
  const { buttonProps } = useButton(triggerProps, ref);
  
  return (
    <div>
      <label>{props.label}</label>
      <button {...buttonProps} ref={ref}>
        <span {...valueProps}>
          {state.selectedItem?.rendered || 'Select an option'}
        </span>
      </button>
      
      {state.isOpen && (
        <ListBox {...menuProps} state={state} />
      )}
    </div>
  );
}

function ListBox({ state, ...props }) {
  const ref = useRef<HTMLUListElement>(null);
  const { listBoxProps } = useListBox(props, state, ref);
  
  return (
    <ul {...listBoxProps} ref={ref}>
      {[...state.collection].map((item) => (
        <Option key={item.key} item={item} state={state} />
      ))}
    </ul>
  );
}

function Option({ item, state }) {
  const ref = useRef<HTMLLIElement>(null);
  const { optionProps, isSelected } = useOption(
    { key: item.key },
    state,
    ref
  );
  
  return (
    <li {...optionProps} ref={ref}>
      {item.rendered}
      {isSelected && ' ✓'}
    </li>
  );
}

// Usage
<Select label="Country">
  <Item key="us">United States</Item>
  <Item key="uk">United Kingdom</Item>
  <Item key="ca">Canada</Item>
</Select>

// React Aria automatically:
// ✅ Arrow keys navigate options
// ✅ Type-ahead search
// ✅ Enter/Space to select
// ✅ Escape to close
// ✅ Home/End keys
// ✅ Screen reader announcements
// ✅ Mobile-friendly
// ✅ ARIA attributes
```

#### 9. DatePicker

**useDatePicker Hook:**
```typescript
import { useDatePicker } from '@react-aria/datepicker';
import { useDatePickerState } from '@react-stately/datepicker';
import { parseDate } from '@internationalized/date';

function DatePicker(props) {
  const state = useDatePickerState(props);
  const ref = useRef<HTMLDivElement>(null);
  
  const { groupProps, labelProps, fieldProps, buttonProps, dialogProps } =
    useDatePicker(props, state, ref);
  
  return (
    <div>
      <label {...labelProps}>{props.label}</label>
      <div {...groupProps} ref={ref}>
        <DateField {...fieldProps} />
        <button {...buttonProps}>📅</button>
      </div>
      {state.isOpen && (
        <div {...dialogProps}>
          <Calendar state={state} />
        </div>
      )}
    </div>
  );
}

// Usage
<DatePicker
  label="Event date"
  value={parseDate('2025-01-01')}
  onChange={(date) => console.log(date)}
/>

// React Aria automatically:
// ✅ Keyboard navigation in calendar
// ✅ Date formatting (locale-aware)
// ✅ Validation
// ✅ Min/max dates
// ✅ Disabled dates
// ✅ Time zones
// ✅ Internationalization
// ✅ Screen reader support
```

#### 10. Benefits and Use Cases

**Why Use React Aria:**
```typescript
/*
Benefits:

1. Accessibility Out of the Box:
   ✅ WCAG 2.1 AA compliant
   ✅ Tested with screen readers
   ✅ Keyboard navigation
   ✅ Focus management
   ✅ ARIA attributes

2. Save Development Time:
   ✅ Don't reinvent the wheel
   ✅ 100+ hours saved per project
   ✅ Edge cases handled
   ✅ Battle-tested

3. Internationalization:
   ✅ RTL support
   ✅ Date/number formatting
   ✅ Translations
   ✅ Locale-aware behavior

4. Mobile Support:
   ✅ Touch interactions
   ✅ Virtual keyboard handling
   ✅ Responsive behavior

5. Unstyled:
   ✅ Bring your own styles
   ✅ Full design control
   ✅ No CSS to override

6. Production Ready:
   ✅ Used by Adobe products
   ✅ Well maintained
   ✅ Comprehensive docs
   ✅ TypeScript support

When to Use:
✅ Building a design system
✅ Complex interactive components
✅ Accessibility is critical
✅ International product
✅ Want to avoid common mistakes

When NOT to Use:
❌ Simple static sites
❌ Prototype/MVP
❌ Already using a component library (Material-UI, Chakra)
❌ Bundle size is critical concern (React Aria adds ~50KB)
*/
```

**Comparison:**
```typescript
/*
React Aria vs Other Solutions:

1. React Aria:
   ✅ Unstyled hooks
   ✅ Full control over markup/styles
   ✅ Accessibility-first
   ✅ ~50KB bundle
   ❌ More code to write
   ❌ Learning curve

2. Headless UI:
   ✅ Similar concept
   ✅ Tailwind CSS integration
   ✅ Simpler API
   ❌ Fewer components
   ❌ Less comprehensive

3. Radix UI:
   ✅ Unstyled components
   ✅ Good accessibility
   ✅ Simpler than React Aria
   ❌ Components not hooks

4. Material-UI:
   ✅ Full component library
   ✅ Quick to prototype
   ❌ Styled (Material Design)
   ❌ Harder to customize
   ❌ Larger bundle

5. Chakra UI:
   ✅ Accessible components
   ✅ Easy to customize
   ❌ Styled
   ❌ Less control

6. Build from scratch:
   ✅ Full control
   ❌ 100+ hours of work
   ❌ Many edge cases to handle
   ❌ Likely to miss accessibility issues
*/
```

#### Summary

**React Aria Overview:**

**What it is:**
> "A library of React Hooks that provides accessible UI primitives for your design system."

**Key Features:**
1. **Accessible** - WCAG 2.1 AA compliant
2. **Unstyled** - Full design control
3. **Internationalized** - RTL, locales, formatting
4. **Mobile-friendly** - Touch interactions
5. **Production-ready** - Battle-tested by Adobe

**Core Philosophy:**
> "Provide behavior, you provide the styles."

**When to Use:**
> "Building a design system or complex components where accessibility is critical!"

**Golden Rule:**
> "Don't build accessible components from scratch. Use React Aria and save 100+ hours!"

</details>

---

### 207. How do you handle screen reader compatibility?

<details>
<summary>View Answer</summary>

**Screen reader compatibility** ensures your React app is usable by blind and visually impaired users who rely on assistive technology to navigate the web. Approximately 2-3% of users rely on screen readers.

#### 1. How Screen Readers Work

**Understanding Screen Readers:**
```typescript
/*
What Screen Readers Do:

1. Read page content aloud
2. Announce element roles ("button", "heading level 1")
3. Announce states ("checked", "expanded")
4. Navigate by landmarks, headings, links
5. Form reading mode
6. Table navigation mode

Popular Screen Readers:

- JAWS (Windows): Most popular, commercial ($1000+)
- NVDA (Windows): Free, open source
- VoiceOver (Mac/iOS): Built-in, free
- TalkBack (Android): Built-in, free
- Narrator (Windows): Built-in

Market Share:
- JAWS: 40%
- NVDA: 30%
- VoiceOver: 20%
- Others: 10%

Test with at least:
✅ VoiceOver (if on Mac)
✅ NVDA (Windows, free)
*/
```

#### 2. Semantic HTML

**Use Proper Elements:**
```typescript
// ❌ BAD: Divs with click handlers
function BadNav() {
  return (
    <div className="nav">
      <div onClick={goHome}>Home</div>
      <div onClick={goAbout}>About</div>
    </div>
  );
}
// Screen reader: "clickable, clickable"
// No context, confusing!

// ✅ GOOD: Semantic HTML
function GoodNav() {
  return (
    <nav aria-label="Main navigation">
      <a href="/">Home</a>
      <a href="/about">About</a>
    </nav>
  );
}
// Screen reader: "navigation landmark, link Home, link About"
// Clear context and purpose!

// ❌ BAD: Heading styles without semantics
function BadHeading() {
  return (
    <div className="text-2xl font-bold">
      Page Title
    </div>
  );
}
// Screen reader: "Page Title"
// Not identified as heading, can't navigate to it

// ✅ GOOD: Proper heading
function GoodHeading() {
  return <h1>Page Title</h1>;
}
// Screen reader: "Heading level 1, Page Title"
// Can navigate by headings!
```

**Landmarks:**
```typescript
function App() {
  return (
    <div>
      {/* ✅ Header landmark */}
      <header>
        <h1>Site Title</h1>
      </header>
      
      {/* ✅ Navigation landmark */}
      <nav aria-label="Main navigation">
        <a href="/">Home</a>
        <a href="/about">About</a>
      </nav>
      
      {/* ✅ Main landmark (only one per page) */}
      <main>
        <article>
          <h2>Article Title</h2>
          <p>Content...</p>
        </article>
        
        {/* ✅ Complementary landmark */}
        <aside>
          <h2>Sidebar</h2>
        </aside>
      </main>
      
      {/* ✅ Footer landmark */}
      <footer>
        <p>&copy; 2025</p>
      </footer>
    </div>
  );
}

// Screen reader keyboard shortcuts:
// - NVDA: D key jumps to next landmark
// - VoiceOver: VO + U opens rotor, select Landmarks
// - User can quickly navigate to "main landmark"
```

#### 3. ARIA Live Regions

**Announcing Dynamic Updates:**
```typescript
// ✅ Polite announcements (wait for pause)
function StatusMessage({ message }: { message: string }) {
  return (
    <div
      role="status"
      aria-live="polite"
      aria-atomic="true"  // Read entire content
    >
      {message}
    </div>
  );
}

// Usage:
<StatusMessage message="5 items added to cart" />
// Screen reader waits for pause, then:
// "5 items added to cart"

// ✅ Assertive announcements (interrupt immediately)
function ErrorAlert({ error }: { error: string }) {
  return (
    <div
      role="alert"
      aria-live="assertive"
      aria-atomic="true"
    >
      {error}
    </div>
  );
}

// Usage:
<ErrorAlert error="Payment failed. Please try again." />
// Screen reader immediately announces:
// "Alert. Payment failed. Please try again."

// ✅ Loading states
function LoadingMessage({ isLoading }: { isLoading: boolean }) {
  return (
    <div
      role="status"
      aria-live="polite"
      aria-atomic="true"
    >
      {isLoading ? 'Loading...' : 'Content loaded'}
    </div>
  );
}
```

**Shopping Cart Example:**
```typescript
function ShoppingCart() {
  const [items, setItems] = useState<Item[]>([]);
  const [announcement, setAnnouncement] = useState('');
  
  const addItem = (item: Item) => {
    setItems([...items, item]);
    setAnnouncement(
      `${item.name} added to cart. ${items.length + 1} items total. Subtotal $${calculateTotal()}`
    );
    
    // Clear announcement after it's read
    setTimeout(() => setAnnouncement(''), 1000);
  };
  
  const removeItem = (itemId: string) => {
    const item = items.find(i => i.id === itemId);
    setItems(items.filter(i => i.id !== itemId));
    setAnnouncement(
      `${item?.name} removed from cart. ${items.length - 1} items remaining.`
    );
  };
  
  return (
    <div>
      {/* Visual cart */}
      <h2>Cart ({items.length})</h2>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name} - ${item.price}
            <button
              onClick={() => removeItem(item.id)}
              aria-label={`Remove ${item.name} from cart`}
            >
              Remove
            </button>
          </li>
        ))}
      </ul>
      
      {/* Screen reader announcements */}
      <div
        role="status"
        aria-live="polite"
        aria-atomic="true"
        className="sr-only"  // Visually hidden
      >
        {announcement}
      </div>
    </div>
  );
}

// CSS for screen-reader-only content:
/*
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}
*/
```

#### 4. Form Accessibility

**Accessible Forms:**
```typescript
function SignupForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState<Record<string, string>>({});
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const newErrors: Record<string, string> = {};
    
    if (!email) {
      newErrors.email = 'Email is required';
    }
    if (!password) {
      newErrors.password = 'Password is required';
    }
    
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      // Focus first error
      const firstErrorField = Object.keys(newErrors)[0];
      document.getElementById(firstErrorField)?.focus();
    }
  };
  
  return (
    <form onSubmit={handleSubmit} noValidate>
      {/* Email field */}
      <div>
        <label htmlFor="email">
          Email:
          <span aria-label="required">*</span>
        </label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          aria-required="true"
          aria-invalid={errors.email ? 'true' : 'false'}
          aria-describedby={errors.email ? 'email-error' : 'email-hint'}
        />
        <p id="email-hint" className="hint">
          We'll never share your email
        </p>
        {errors.email && (
          <p id="email-error" role="alert" className="error">
            {errors.email}
          </p>
        )}
      </div>
      
      {/* Password field */}
      <div>
        <label htmlFor="password">
          Password:
          <span aria-label="required">*</span>
        </label>
        <input
          id="password"
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          aria-required="true"
          aria-invalid={errors.password ? 'true' : 'false'}
          aria-describedby={errors.password ? 'password-error' : 'password-hint'}
        />
        <p id="password-hint" className="hint">
          Must be at least 8 characters
        </p>
        {errors.password && (
          <p id="password-error" role="alert" className="error">
            {errors.password}
          </p>
        )}
      </div>
      
      <button type="submit">Sign Up</button>
    </form>
  );
}

// Screen reader flow:
// 1. "Email, edit text, required"
// 2. "We'll never share your email" (hint)
// 3. If error: "Alert. Email is required" (immediately)
// 4. "Email, edit text, required, invalid"
```

#### 5. Images and Alt Text

**Alt Text Best Practices:**
```typescript
// ❌ BAD: Generic alt text
<img src="photo.jpg" alt="image" />
<img src="chart.png" alt="chart" />
<img src="person.jpg" alt="photo" />

// ✅ GOOD: Descriptive alt text
<img
  src="photo.jpg"
  alt="Woman giving presentation at tech conference"
/>
<img
  src="chart.png"
  alt="Bar chart showing 30% increase in sales from Q1 to Q2 2025"
/>
<img
  src="person.jpg"
  alt="John Smith, CEO of Acme Corp, wearing blue suit"
/>

// ✅ Decorative images (empty alt)
<img
  src="decorative-border.png"
  alt=""
  role="presentation"
/>
// Screen reader skips this image

// ✅ Complex images with long description
<figure>
  <img
    src="complex-chart.png"
    alt="Sales data chart"
    aria-describedby="chart-description"
  />
  <figcaption id="chart-description">
    This chart shows sales data from January to December 2025.
    Sales started at $100k in January, peaked at $250k in June,
    and ended at $180k in December.
  </figcaption>
</figure>

// ✅ Icon buttons
<button aria-label="Close menu">
  <XIcon aria-hidden="true" />  {/* Hide icon from screen reader */}
</button>
// Screen reader: "Close menu, button"
```

#### 6. Button and Link Labels

**Descriptive Labels:**
```typescript
// ❌ BAD: Non-descriptive
<a href="/about">Click here</a> to learn more.
<button onClick={submit}>Submit</button>  {/* Submit what? */}
<button><XIcon /></button>  {/* No label! */}

// ✅ GOOD: Descriptive
<a href="/about">Learn more about our company</a>
<button onClick={handleSubmitForm}>Submit registration form</button>
<button aria-label="Close dialog">
  <XIcon aria-hidden="true" />
</button>

// ✅ Context-specific labels
function DeleteButton({ itemName }: { itemName: string }) {
  return (
    <button
      onClick={handleDelete}
      aria-label={`Delete ${itemName}`}
    >
      Delete
    </button>
  );
}

// Usage:
<DeleteButton itemName="iPhone 15 Pro" />
// Screen reader: "Delete iPhone 15 Pro, button"
// Much clearer than just "Delete, button"!
```

#### 7. Loading States

**Accessible Loading:**
```typescript
function LoadingButton({
  isLoading,
  children,
  ...props
}: {
  isLoading: boolean;
  children: React.ReactNode;
}) {
  return (
    <button
      {...props}
      disabled={isLoading}
      aria-busy={isLoading}
      aria-live="polite"
    >
      {isLoading ? (
        <>
          <span className="spinner" aria-hidden="true" />
          <span className="sr-only">Loading...</span>
        </>
      ) : (
        children
      )}
    </button>
  );
}

// Screen reader announces:
// "Submit, button, busy" (when loading)
// "Loading..." (change announcement)
// "Submit, button" (when done)

function DataTable({ isLoading, data }) {
  if (isLoading) {
    return (
      <div
        role="status"
        aria-live="polite"
        aria-busy="true"
      >
        <span className="spinner" aria-hidden="true" />
        <span className="sr-only">Loading data...</span>
      </div>
    );
  }
  
  return (
    <table>
      {/* Table content */}
    </table>
  );
}
```

#### 8. Route Changes

**Announcing Navigation:**
```typescript
import { useEffect, useRef } from 'react';
import { useLocation } from 'react-router-dom';

function RouteAnnouncer() {
  const location = useLocation();
  const announcerRef = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    // Announce page change
    const pageTitle = document.title;
    if (announcerRef.current) {
      announcerRef.current.textContent = `Navigated to ${pageTitle}`;
    }
    
    // Focus main content
    const main = document.querySelector('main');
    if (main) {
      (main as HTMLElement).focus();
    }
  }, [location.pathname]);
  
  return (
    <div
      ref={announcerRef}
      role="status"
      aria-live="polite"
      aria-atomic="true"
      className="sr-only"
    />
  );
}

function App() {
  return (
    <Router>
      <RouteAnnouncer />
      <Routes>
        {/* Routes */}
      </Routes>
    </Router>
  );
}

// Screen reader announces:
// "Navigated to About Us - My Company"
```

#### 9. Testing with Screen Readers

**VoiceOver Testing (Mac):**
```typescript
/*
Enable VoiceOver: Cmd + F5

Basic Commands:
- VO + Right Arrow: Next element
- VO + Left Arrow: Previous element
- VO + A: Read from current position
- VO + U: Open rotor (navigation menu)
- VO + Space: Activate element
- Control: Stop reading
- VO + Shift + Down: Enter element (like form)
- VO + Shift + Up: Exit element

Rotor Navigation (VO + U):
- Headings: Navigate by h1, h2, etc.
- Landmarks: Navigate to header, nav, main, footer
- Links: List all links
- Form Controls: List all inputs, buttons

Test Checklist:
✅ Page title announced
✅ Headings have proper hierarchy
✅ Landmarks announced ("navigation", "main")
✅ Links are descriptive
✅ Buttons have labels
✅ Images have alt text
✅ Forms have labels
✅ Errors announced
✅ Dynamic updates announced
✅ Modal focus works
*/
```

**NVDA Testing (Windows):**
```typescript
/*
Download: https://www.nvaccess.org/ (FREE)

Basic Commands:
- Insert + Down: Start reading
- Down Arrow: Next line
- Up Arrow: Previous line
- Ctrl: Stop reading
- H: Next heading
- Shift + H: Previous heading
- K: Next link
- B: Next button
- F: Next form field
- D: Next landmark
- Insert + F7: Elements list

Test Same Checklist as VoiceOver
*/
```

#### 10. Common Mistakes

**Screen Reader Anti-Patterns:**
```typescript
// ❌ MISTAKE 1: aria-hidden on focusable elements
<button aria-hidden="true">Click</button>
// Focusable but hidden from screen reader - CONFUSING!

// ✅ FIX: Remove button or remove aria-hidden
<button>Click</button>

// ❌ MISTAKE 2: Empty aria-label
<button aria-label="">Submit</button>
// Screen reader: "button" (no label!)

// ✅ FIX: Provide descriptive label
<button aria-label="Submit form">Submit</button>

// ❌ MISTAKE 3: Relying on placeholder
<input type="text" placeholder="Email" />
// Placeholder disappears when typing!

// ✅ FIX: Use proper label
<label htmlFor="email">Email:</label>
<input id="email" type="text" placeholder="john@example.com" />

// ❌ MISTAKE 4: Click handler on div
<div onClick={submit}>Submit</div>
// Screen reader: "clickable" (not announced as button)

// ✅ FIX: Use proper button
<button onClick={submit}>Submit</button>

// ❌ MISTAKE 5: No announcement for updates
setData(newData);
// Screen reader doesn't know content changed!

// ✅ FIX: Use live region
<div role="status" aria-live="polite">
  {data.length} results found
</div>
```

#### Summary

**Screen Reader Compatibility Checklist:**

```typescript
/*
✅ Semantic HTML:
  - Use proper elements (<nav>, <main>, <button>)
  - Heading hierarchy (h1 → h2 → h3)
  - Landmarks for navigation

✅ Labels:
  - All inputs have labels
  - Icon buttons have aria-label
  - Links are descriptive
  - Images have alt text

✅ ARIA:
  - Live regions for updates
  - aria-expanded, aria-selected for state
  - aria-describedby for additional info
  - role="alert" for errors

✅ Focus:
  - Visible focus indicators
  - Focus moves to new content
  - Modal focus trapped
  - Restore focus on close

✅ Forms:
  - Labels with htmlFor
  - Errors with role="alert"
  - aria-required for required fields
  - aria-invalid for errors

✅ Testing:
  - Test with VoiceOver or NVDA
  - Navigate entire app
  - Verify all content announced
  - Check announcements are clear
*/
```

**Key Principles:**
1. **Semantic HTML first** - Proper elements provide context
2. **Label everything** - Screen readers need text
3. **Announce updates** - Use ARIA live regions
4. **Test with real screen readers** - Automated tests catch only 30%
5. **Focus management** - Always know where user is

**Golden Rule:**
> "If a screen reader can't understand it, it's not accessible!"

**Testing Rule:**
> "Turn on VoiceOver/NVDA and try to use your app. If it's confusing to you, it's confusing to users!"

</details>

---

### 208. What are semantic HTML elements and their importance?

<details>
<summary>View Answer</summary>

**Semantic HTML elements** clearly describe their meaning and purpose to both the browser and developers. Using semantic HTML is the foundation of web accessibility and SEO.

#### 1. What is Semantic HTML?

**Definition:**
```typescript
/*
Semantic HTML:
HTML elements that clearly describe their content and purpose.

Examples:
✅ <header> - Page or section header
✅ <nav> - Navigation links
✅ <main> - Main content
✅ <article> - Self-contained content
✅ <section> - Thematic grouping
✅ <aside> - Sidebar or complementary content
✅ <footer> - Page or section footer
✅ <button> - Interactive button
✅ <h1>-<h6> - Headings

Non-Semantic:
❌ <div> - Generic container (no meaning)
❌ <span> - Generic inline container

Why It Matters:
✅ Screen readers announce element roles
✅ Search engines understand content structure
✅ Browsers provide default functionality
✅ Developers understand code faster
✅ Better keyboard navigation
*/
```

#### 2. Document Structure

**Page Layout:**
```typescript
// ❌ BAD: Non-semantic divs
function BadLayout() {
  return (
    <div className="page">
      <div className="top">
        <div className="logo">Logo</div>
      </div>
      <div className="menu">
        <div className="link">Home</div>
        <div className="link">About</div>
      </div>
      <div className="content">
        <div className="post">
          <div className="title">Title</div>
          <div className="text">Content...</div>
        </div>
      </div>
      <div className="sidebar">
        <div className="widget">Related</div>
      </div>
      <div className="bottom">
        <div className="copyright">&copy; 2025</div>
      </div>
    </div>
  );
}

// Screen reader: Just reads content, no structure
// SEO: Search engines don't understand page structure
// Developer: Must read CSS classes to understand

// ✅ GOOD: Semantic HTML
function GoodLayout() {
  return (
    <div className="page">
      <header>
        <img src="/logo.png" alt="Company Logo" />
      </header>
      
      <nav aria-label="Main navigation">
        <a href="/">Home</a>
        <a href="/about">About</a>
      </nav>
      
      <main>
        <article>
          <h1>Article Title</h1>
          <p>Content...</p>
        </article>
      </main>
      
      <aside>
        <h2>Related Articles</h2>
        {/* Sidebar content */}
      </aside>
      
      <footer>
        <p>&copy; 2025 Company Name</p>
      </footer>
    </div>
  );
}

// Screen reader: Announces landmarks ("navigation", "main", etc.)
// SEO: Search engines understand page structure
// Developer: Instantly understands page layout
```

#### 3. Landmark Elements

**Main Landmarks:**
```typescript
function App() {
  return (
    <>
      {/* ✅ Header - Top of page/section */}
      <header>
        <h1>Site Name</h1>
        {/* Logo, tagline, site-wide header content */}
      </header>
      
      {/* ✅ Nav - Navigation links */}
      <nav aria-label="Main navigation">
        <a href="/">Home</a>
        <a href="/products">Products</a>
        <a href="/about">About</a>
      </nav>
      
      {/* ✅ Main - Primary content (only ONE per page!) */}
      <main>
        <h2>Welcome</h2>
        <p>Main page content...</p>
      </main>
      
      {/* ✅ Aside - Complementary content */}
      <aside>
        <h2>Related Links</h2>
        {/* Sidebar, related content */}
      </aside>
      
      {/* ✅ Footer - Bottom of page/section */}
      <footer>
        <p>&copy; 2025</p>
        {/* Copyright, links, contact info */}
      </footer>
    </>
  );
}

// Screen reader shortcuts:
// VoiceOver: VO + U → Landmarks
// NVDA: D key jumps to next landmark
// User can quickly navigate to "main landmark"
```

**Multiple Navigation:**
```typescript
function Page() {
  return (
    <div>
      {/* Primary navigation */}
      <nav aria-label="Main navigation">
        <a href="/">Home</a>
        <a href="/products">Products</a>
      </nav>
      
      <main>
        <article>
          <h1>Article Title</h1>
          
          {/* Article navigation - needs label! */}
          <nav aria-label="Article sections">
            <a href="#intro">Introduction</a>
            <a href="#methods">Methods</a>
            <a href="#results">Results</a>
          </nav>
          
          {/* Content */}
        </article>
      </main>
      
      {/* Footer navigation */}
      <footer>
        <nav aria-label="Footer navigation">
          <a href="/privacy">Privacy</a>
          <a href="/terms">Terms</a>
        </nav>
      </footer>
    </div>
  );
}

// Multiple <nav> elements need aria-label to distinguish them
```

#### 4. Content Sectioning

**Article vs Section:**
```typescript
// ✅ <article> - Self-contained, reusable content
// Use for: Blog posts, news articles, forum posts, comments
function BlogPost() {
  return (
    <article>
      <h2>How to Use Semantic HTML</h2>
      <p>Published on <time datetime="2025-12-28">December 28, 2025</time></p>
      <p>Content that makes sense on its own...</p>
      
      {/* Nested article (comment) */}
      <article>
        <h3>Comment by John</h3>
        <p>Great article!</p>
      </article>
    </article>
  );
}

// ✅ <section> - Thematic grouping of content
// Use for: Chapters, tabbed content, themed groups
function AboutPage() {
  return (
    <main>
      <h1>About Us</h1>
      
      <section>
        <h2>Our Mission</h2>
        <p>Mission statement...</p>
      </section>
      
      <section>
        <h2>Our Team</h2>
        <p>Team information...</p>
      </section>
      
      <section>
        <h2>Contact</h2>
        <p>Contact details...</p>
      </section>
    </main>
  );
}

// Rule of thumb:
// - Can this be syndicated/shared independently? → <article>
// - Is this a chapter/theme in larger content? → <section>
// - Neither? → <div>
```

#### 5. Headings

**Heading Hierarchy:**
```typescript
// ❌ BAD: Skipping levels, using for style
function BadHeadings() {
  return (
    <div>
      <h1>Page Title</h1>
      <h3>Subsection</h3>  {/* Skipped h2! */}
      <h5>Small text</h5>   {/* Used for styling, not structure */}
      <h2>Another section</h2>  {/* Out of order */}
    </div>
  );
}

// Screen reader: Confusing structure
// SEO: Poor content hierarchy

// ✅ GOOD: Logical hierarchy
function GoodHeadings() {
  return (
    <div>
      <h1>Page Title</h1>
      
      <h2>Section 1</h2>
      <p>Content...</p>
      
      <h3>Subsection 1.1</h3>
      <p>Content...</p>
      
      <h3>Subsection 1.2</h3>
      <p>Content...</p>
      
      <h2>Section 2</h2>
      <p>Content...</p>
    </div>
  );
}

// Screen reader: Can navigate by heading level
// SEO: Clear content structure
// Accessibility: Users can skip to sections

// ✅ Styling without breaking hierarchy
function StyledHeading() {
  return (
    <h3 className="text-sm font-normal">
      Small subheading
    </h3>
  );
}
// Use CSS classes for styling, not wrong heading level!
```

**Heading Guidelines:**
```typescript
/*
Heading Best Practices:

✅ One <h1> per page (page title)
✅ Don't skip levels (h1 → h2 → h3, not h1 → h3)
✅ Use headings for structure, not styling
✅ Every section should have a heading
✅ Headings should be descriptive

Screen reader shortcuts:
- VoiceOver: VO + Cmd + H (next heading)
- NVDA: H (next heading), 1-6 (specific level)
- User can navigate page structure by headings

Example page outline:
h1: Home Page
  h2: Featured Products
    h3: Product 1
    h3: Product 2
  h2: About Us
  h2: Contact
*/
```

#### 6. Interactive Elements

**Buttons vs Links:**
```typescript
// ✅ <button> - Performs action (submit, open, toggle)
function ActionButtons() {
  return (
    <>
      <button onClick={handleSubmit}>Submit Form</button>
      <button onClick={openModal}>Open Modal</button>
      <button onClick={toggleMenu}>Toggle Menu</button>
    </>
  );
}
// Keyboard: Enter and Space activate
// Screen reader: "Submit Form, button"

// ✅ <a> - Navigation to different page/location
function NavigationLinks() {
  return (
    <>
      <a href="/about">About Us</a>
      <a href="/contact">Contact</a>
      <a href="#section">Jump to Section</a>
    </>
  );
}
// Keyboard: Enter activates
// Screen reader: "About Us, link"
// Right-click: "Open in new tab" option

// ❌ BAD: Wrong element for purpose
function WrongElements() {
  return (
    <>
      {/* ❌ Button styled as link for navigation */}
      <button onClick={() => router.push('/about')}>
        About
      </button>
      {/* Use <a href="/about">About</a> instead! */}
      
      {/* ❌ Link with onClick for action */}
      <a href="#" onClick={handleSubmit}>
        Submit
      </a>
      {/* Use <button onClick={handleSubmit}>Submit</button> instead! */}
      
      {/* ❌ Div with click handler */}
      <div onClick={handleClick}>Click me</div>
      {/* Use <button onClick={handleClick}>Click me</button> instead! */}
    </>
  );
}

// Rule:
// - Changes URL or navigates? → <a>
// - Performs action on current page? → <button>
```

#### 7. Forms

**Semantic Form Elements:**
```typescript
function SemanticForm() {
  return (
    <form onSubmit={handleSubmit}>
      {/* ✅ Label with htmlFor */}
      <label htmlFor="name">Name:</label>
      <input id="name" type="text" required />
      
      {/* ✅ Fieldset for grouping */}
      <fieldset>
        <legend>Shipping Address</legend>
        
        <label htmlFor="street">Street:</label>
        <input id="street" type="text" />
        
        <label htmlFor="city">City:</label>
        <input id="city" type="text" />
      </fieldset>
      
      {/* ✅ Select with optgroup */}
      <label htmlFor="country">Country:</label>
      <select id="country">
        <optgroup label="North America">
          <option value="us">United States</option>
          <option value="ca">Canada</option>
        </optgroup>
        <optgroup label="Europe">
          <option value="uk">United Kingdom</option>
          <option value="de">Germany</option>
        </optgroup>
      </select>
      
      {/* ✅ Button with type */}
      <button type="submit">Submit</button>
      <button type="reset">Reset</button>
      <button type="button" onClick={cancel}>Cancel</button>
    </form>
  );
}

// Screen reader announces:
// "Name, edit text, required"
// "Shipping Address, group"
// "Street, edit text"
// "Submit, button"
```

#### 8. Text-Level Semantics

**Inline Elements:**
```typescript
function TextSemantics() {
  return (
    <article>
      {/* ✅ <strong> - Important/serious */}
      <p>
        <strong>Warning:</strong> This action cannot be undone.
      </p>
      
      {/* ✅ <em> - Emphasis */}
      <p>
        You <em>must</em> submit the form by Friday.
      </p>
      
      {/* ✅ <mark> - Highlighted */}
      <p>
        Search results for <mark>React</mark>.
      </p>
      
      {/* ✅ <time> - Date/time */}
      <p>
        Published on <time datetime="2025-12-28">December 28, 2025</time>
      </p>
      
      {/* ✅ <abbr> - Abbreviation */}
      <p>
        <abbr title="HyperText Markup Language">HTML</abbr>
      </p>
      
      {/* ✅ <code> - Code */}
      <p>
        Use <code>const</code> instead of <code>var</code>.
      </p>
      
      {/* ❌ <b> - Bold (no semantic meaning) */}
      {/* ❌ <i> - Italic (no semantic meaning) */}
      {/* Use <strong> and <em> instead! */}
    </article>
  );
}
```

#### 9. Lists

**Semantic Lists:**
```typescript
// ✅ <ul> - Unordered list
function UnorderedList() {
  return (
    <ul>
      <li>Item 1</li>
      <li>Item 2</li>
      <li>Item 3</li>
    </ul>
  );
}
// Screen reader: "List, 3 items. Item 1, 1 of 3..."

// ✅ <ol> - Ordered list
function OrderedList() {
  return (
    <ol>
      <li>First step</li>
      <li>Second step</li>
      <li>Third step</li>
    </ol>
  );
}
// Screen reader: "List, 3 items. 1. First step..."

// ✅ <dl> - Description list
function DescriptionList() {
  return (
    <dl>
      <dt>HTML</dt>
      <dd>HyperText Markup Language</dd>
      
      <dt>CSS</dt>
      <dd>Cascading Style Sheets</dd>
    </dl>
  );
}
// Use for glossaries, metadata, key-value pairs

// ❌ BAD: Divs instead of lists
function FakeList() {
  return (
    <div>
      <div>• Item 1</div>
      <div>• Item 2</div>
      <div>• Item 3</div>
    </div>
  );
}
// Screen reader: No indication this is a list!
```

#### 10. Benefits Summary

**Why Semantic HTML Matters:**
```typescript
/*
Benefits of Semantic HTML:

1. ACCESSIBILITY:
   ✅ Screen readers understand content structure
   ✅ Landmarks for navigation
   ✅ Default keyboard support
   ✅ Clear announcements ("button", "navigation", "heading level 2")

2. SEO:
   ✅ Search engines understand content hierarchy
   ✅ Better rankings
   ✅ Rich snippets in search results
   ✅ Featured in Google answers

3. MAINTAINABILITY:
   ✅ Self-documenting code
   ✅ Easier to understand
   ✅ Less CSS needed
   ✅ Faster development

4. BROWSER SUPPORT:
   ✅ Default functionality (buttons, forms)
   ✅ Right-click menus
   ✅ Browser extensions work better
   ✅ Reader mode

5. PERFORMANCE:
   ✅ Less CSS/JavaScript needed
   ✅ Browser optimizations
   ✅ Faster rendering

Semantic HTML Checklist:
✅ <header>, <nav>, <main>, <footer> for layout
✅ <article>, <section>, <aside> for content
✅ <h1>-<h6> for headings (proper hierarchy)
✅ <button> for actions, <a> for navigation
✅ <label> for form inputs
✅ <ul>/<ol>/<dl> for lists
✅ <table> for tabular data
✅ <figure>, <figcaption> for images
✅ <time>, <mark>, <strong>, <em> for inline semantics
*/
```

#### Summary

**Semantic vs Non-Semantic:**

```typescript
// Non-semantic (BAD):
<div class="header">
  <div class="nav">
    <div class="link">Home</div>
  </div>
</div>
<div class="main">
  <div class="post">
    <div class="title">Title</div>
  </div>
</div>

// Semantic (GOOD):
<header>
  <nav>
    <a href="/">Home</a>
  </nav>
</header>
<main>
  <article>
    <h1>Title</h1>
  </article>
</main>
```

**Key Principles:**
1. **Use semantic elements** - They describe their purpose
2. **Proper hierarchy** - h1 → h2 → h3, not h1 → h4
3. **Right element for the job** - Button for actions, links for navigation
4. **Landmarks** - header, nav, main, aside, footer
5. **Labels** - All form inputs need labels

**Golden Rule:**
> "If there's a semantic HTML element for it, use it! Don't use div/span when semantic elements exist."

**Impact:**
> "Semantic HTML is the foundation of accessibility. Get this right, and 50% of a11y work is done!"

</details>

---

### 209. How do you implement skip links in React?

<details>
<summary>View Answer</summary>

**Skip links** allow keyboard and screen reader users to bypass repetitive content (like navigation) and jump directly to the main content. They're essential for accessibility and required by WCAG 2.1.

#### 1. Why Skip Links?

**The Problem:**
```typescript
/*
Without Skip Links:

User visits your site with keyboard.
Page has 50 navigation links in header.
User must Tab through ALL 50 links to reach main content.
On every. Single. Page.

Time wasted:
- 50 links × 1 second each = 50 seconds per page
- 10 pages = 8+ minutes of just tabbing!

Screen reader users:
- Must listen to entire navigation menu
- On every page visit
- Extremely frustrating

WCAG Requirement:
"Bypass Blocks" (WCAG 2.1 Level A)
- Must provide a mechanism to skip repeated blocks
- Skip links are the most common solution
*/
```

#### 2. Basic Implementation

**Simple Skip Link:**
```typescript
function Layout({ children }) {
  return (
    <div>
      {/* Skip link - MUST be first focusable element */}
      <a href="#main-content" className="skip-link">
        Skip to main content
      </a>
      
      <header>
        <nav>
          {/* 50 navigation links */}
          <a href="/">Home</a>
          <a href="/about">About</a>
          {/* ... 48 more links */}
        </nav>
      </header>
      
      <main id="main-content" tabIndex={-1}>
        {children}
      </main>
    </div>
  );
}

// CSS: Hidden until focused
/*
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px 16px;
  text-decoration: none;
  z-index: 1000;
}

.skip-link:focus {
  top: 0;
}
*/

// How it works:
// 1. User presses Tab → Skip link appears
// 2. User presses Enter → Focus jumps to main content
// 3. User continues tabbing from main content
// 4. Navigation is bypassed!
```

#### 3. Enhanced Skip Link with Focus Management

**Focus Main Content:**
```typescript
import { useRef } from 'react';

function SkipLink() {
  const mainRef = useRef<HTMLElement | null>(null);
  
  const handleSkipClick = (e: React.MouseEvent<HTMLAnchorElement>) => {
    e.preventDefault();
    
    // Find main element
    const main = document.getElementById('main-content');
    if (main) {
      // Focus it
      main.focus();
      
      // Scroll to it (in case it's offscreen)
      main.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }
  };
  
  return (
    <a
      href="#main-content"
      className="skip-link"
      onClick={handleSkipClick}
    >
      Skip to main content
    </a>
  );
}

function Layout({ children }) {
  return (
    <div>
      <SkipLink />
      
      <header>
        <nav>{/* Navigation */}</nav>
      </header>
      
      <main
        id="main-content"
        tabIndex={-1}  // Make focusable programmatically
      >
        {children}
      </main>
    </div>
  );
}
```

**Why tabIndex={-1}?**
```typescript
/*
tabIndex={-1} on <main>:

✅ Makes element focusable programmatically
✅ NOT in normal tab order
✅ Can be focused with JavaScript
✅ Allows skip link to work

Without it:
- Skip link navigates to #main-content
- But focus doesn't move
- User still has to Tab from top!

With it:
- Skip link focuses main element
- Next Tab moves to first focusable element IN main
- Navigation is successfully bypassed!
*/
```

#### 4. Multiple Skip Links

**Skip to Different Sections:**
```typescript
function MultipleSkipLinks() {
  const skipTo = (id: string) => {
    const element = document.getElementById(id);
    if (element) {
      element.focus();
      element.scrollIntoView({ behavior: 'smooth' });
    }
  };
  
  return (
    <nav className="skip-links" aria-label="Skip links">
      <a
        href="#main-content"
        className="skip-link"
        onClick={(e) => {
          e.preventDefault();
          skipTo('main-content');
        }}
      >
        Skip to main content
      </a>
      
      <a
        href="#navigation"
        className="skip-link"
        onClick={(e) => {
          e.preventDefault();
          skipTo('navigation');
        }}
      >
        Skip to navigation
      </a>
      
      <a
        href="#search"
        className="skip-link"
        onClick={(e) => {
          e.preventDefault();
          skipTo('search');
        }}
      >
        Skip to search
      </a>
    </nav>
  );
}

function Layout({ children }) {
  return (
    <div>
      <MultipleSkipLinks />
      
      <header>
        <nav id="navigation" tabIndex={-1}>
          {/* Navigation */}
        </nav>
        
        <form id="search" tabIndex={-1}>
          {/* Search */}
        </form>
      </header>
      
      <main id="main-content" tabIndex={-1}>
        {children}
      </main>
    </div>
  );
}
```

#### 5. Skip Link Styling

**Visible on Focus:**
```css
/* Basic skip link */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px 16px;
  text-decoration: none;
  z-index: 1000;
  border-radius: 0 0 4px 0;
}

.skip-link:focus {
  top: 0;
}

/* Better styling */
.skip-link {
  position: absolute;
  left: 0;
  top: -100px;  /* Offscreen */
  background: #007bff;
  color: white;
  padding: 12px 24px;
  text-decoration: none;
  font-weight: 600;
  z-index: 10000;
  transition: top 0.2s;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

.skip-link:focus {
  top: 0;
  outline: 2px solid #fff;
  outline-offset: 2px;
}

/* Multiple skip links */
.skip-links {
  position: absolute;
  left: 0;
  top: -200px;
  display: flex;
  gap: 8px;
  z-index: 10000;
}

.skip-links:focus-within {
  top: 0;
}

/* High contrast mode */
@media (prefers-contrast: high) {
  .skip-link {
    border: 3px solid currentColor;
  }
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  .skip-link {
    background: #fff;
    color: #000;
  }
}
```

#### 6. React Router Integration

**Skip Links with Client-Side Routing:**
```typescript
import { useEffect, useRef } from 'react';
import { useLocation } from 'react-router-dom';

function Layout({ children }) {
  const location = useLocation();
  const mainRef = useRef<HTMLElement>(null);
  
  // Focus main content on route change
  useEffect(() => {
    // Wait for render to complete
    setTimeout(() => {
      mainRef.current?.focus();
    }, 0);
  }, [location.pathname]);
  
  const handleSkipClick = (e: React.MouseEvent) => {
    e.preventDefault();
    mainRef.current?.focus();
  };
  
  return (
    <div>
      <a
        href="#main-content"
        className="skip-link"
        onClick={handleSkipClick}
      >
        Skip to main content
      </a>
      
      <header>
        <nav>{/* Navigation */}</nav>
      </header>
      
      <main
        id="main-content"
        ref={mainRef}
        tabIndex={-1}
      >
        {children}
      </main>
    </div>
  );
}

// On route change:
// 1. Skip link still visible
// 2. Main content focused automatically
// 3. Screen reader announces new page
```

#### 7. Next.js Implementation

**Skip Link in Next.js:**
```typescript
// components/SkipLink.tsx
import { useRouter } from 'next/router';
import { useEffect, useRef } from 'react';

export function SkipLink() {
  const router = useRouter();
  const mainRef = useRef<HTMLElement | null>(null);
  
  useEffect(() => {
    const handleRouteChange = () => {
      const main = document.getElementById('main-content');
      if (main) {
        main.focus();
      }
    };
    
    router.events.on('routeChangeComplete', handleRouteChange);
    return () => {
      router.events.off('routeChangeComplete', handleRouteChange);
    };
  }, [router.events]);
  
  const handleClick = (e: React.MouseEvent) => {
    e.preventDefault();
    const main = document.getElementById('main-content');
    if (main) {
      main.focus();
      main.scrollIntoView();
    }
  };
  
  return (
    <a
      href="#main-content"
      className="skip-link"
      onClick={handleClick}
    >
      Skip to main content
    </a>
  );
}

// app/layout.tsx or pages/_app.tsx
import { SkipLink } from '@/components/SkipLink';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <SkipLink />
        
        <nav>{/* Navigation */}</nav>
        
        <main id="main-content" tabIndex={-1}>
          {children}
        </main>
      </body>
    </html>
  );
}
```

#### 8. Testing Skip Links

**Manual Testing:**
```typescript
/*
Skip Link Testing Checklist:

✅ Press Tab key
   - Skip link should be FIRST focusable element
   - Skip link should be VISIBLE when focused

✅ Press Enter on skip link
   - Focus should move to main content
   - Next Tab should focus first element IN main
   - Navigation should be bypassed

✅ Test with screen reader
   - VoiceOver/NVDA announces "Skip to main content, link"
   - After activation, announces main content

✅ Test on different pages
   - Skip link works on all pages
   - Focus management consistent

✅ Test with JavaScript disabled
   - Skip link still works (href="#main-content")
   - No onClick needed for basic functionality

Common Issues:

❌ Skip link not first element
   - Must be FIRST in DOM order
   - Before logo, navigation, everything

❌ Main content not focusable
   - Add tabIndex={-1} to main element
   - Without it, skip link doesn't work properly

❌ Skip link not visible on focus
   - Check CSS: top: 0 on :focus
   - z-index high enough (9999+)
   - Not covered by other elements

❌ Multiple skip links don't all show
   - Use :focus-within on container
   - Or make each individually visible
*/
```

**Automated Testing:**
```typescript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

test('Skip link is first focusable element', async () => {
  render(<Layout><p>Content</p></Layout>);
  
  const skipLink = screen.getByRole('link', { name: /skip to main content/i });
  
  // Tab once
  await userEvent.tab();
  
  // Skip link should be focused
  expect(skipLink).toHaveFocus();
});

test('Skip link focuses main content', async () => {
  render(
    <Layout>
      <button>First button in main</button>
    </Layout>
  );
  
  const skipLink = screen.getByRole('link', { name: /skip to main content/i });
  const mainButton = screen.getByRole('button', { name: /first button/i });
  
  // Focus skip link
  skipLink.focus();
  
  // Activate skip link
  await userEvent.keyboard('{Enter}');
  
  // Main element should be focused
  const main = document.getElementById('main-content');
  expect(main).toHaveFocus();
  
  // Next tab should focus first element in main
  await userEvent.tab();
  expect(mainButton).toHaveFocus();
});
```

#### 9. Alternatives to Skip Links

**Other Bypass Methods:**
```typescript
/*
WCAG "Bypass Blocks" can be satisfied by:

1. Skip Links (Most Common):
   ✅ Simple to implement
   ✅ Works everywhere
   ✅ User-controlled
   ❌ Requires user action

2. Landmark Navigation:
   ✅ Built into screen readers
   ✅ No implementation needed (if using semantic HTML)
   ❌ Only for screen reader users
   ❌ Not all users know about it

3. Headings:
   ✅ Screen readers can navigate by headings
   ✅ Proper h1-h6 hierarchy
   ❌ Only for screen reader users

4. ARIA Landmarks:
   ✅ role="navigation", role="main", etc.
   ✅ Same as semantic HTML
   ❌ Only for screen reader users

Best Practice:
Use ALL of them!
- Skip links for keyboard users
- Landmarks for screen reader users
- Proper headings for structure

Together they provide comprehensive bypass mechanisms.
*/
```

#### 10. Best Practices

**Skip Link Guidelines:**
```typescript
/*
Skip Link Best Practices:

✅ Position:
  - FIRST focusable element on page
  - Before logo, navigation, everything
  - In DOM order, not just visually

✅ Visibility:
  - Hidden by default (offscreen)
  - Visible on focus
  - High z-index (9999+)
  - Clear styling (high contrast)

✅ Target:
  - Skip to <main> element
  - Add tabIndex={-1} to main
  - Use unique ID

✅ Text:
  - Clear: "Skip to main content"
  - Not: "Skip navigation" (what if there's sidebar too?)
  - Descriptive and action-oriented

✅ Multiple Links:
  - "Skip to main content" (most important)
  - "Skip to navigation" (less common)
  - "Skip to search" (if prominent)
  - Don't overdo it (3 max)

✅ Focus Management:
  - Focus target element
  - Scroll to target
  - Works with client-side routing

✅ Testing:
  - Test with keyboard only
  - Test with screen reader
  - Test on all pages
  - Test that it's actually FIRST focus

❌ Avoid:
  - Skip link not first element
  - Not visible on focus
  - Target not focusable
  - Broken in SPA navigation
  - "Skip nav" with no other bypass method
*/
```

#### Summary

**Skip Link Implementation:**

**Basic Pattern:**
```typescript
<a href="#main-content" className="skip-link">
  Skip to main content
</a>

<header>
  <nav>{/* Many links */}</nav>
</header>

<main id="main-content" tabIndex={-1}>
  {children}
</main>
```

**CSS:**
```css
.skip-link {
  position: absolute;
  top: -40px;
}

.skip-link:focus {
  top: 0;
}
```

**Key Points:**
1. **First element** - Must be first focusable element
2. **Visible on focus** - Hidden until Tab is pressed
3. **Target main** - Skip to main content
4. **tabIndex={-1}** - Make main focusable
5. **Test it** - Actually try it with keyboard only

**Why It Matters:**
> "Skip links save keyboard users 30+ seconds per page. On a 10-page session, that's 5+ minutes saved!"

**Golden Rule:**
> "Every website needs a skip link. No exceptions!"

</details>

---

### 210. What is color contrast and why does it matter?

<details>
<summary>View Answer</summary>

**Color contrast** is the difference in luminance between foreground (text) and background colors. Sufficient contrast is essential for users with low vision, color blindness, and anyone viewing screens in bright sunlight.

#### 1. Why Color Contrast Matters

**The Problem:**
```typescript
/*
Who Needs Good Contrast:

1. Low Vision (8% of population):
   - Need high contrast to read text
   - Can't perceive subtle differences

2. Color Blindness (8% of men, 0.5% of women):
   - Can't distinguish certain colors
   - Red-green most common

3. Aging Eyes (Everyone over 40):
   - Contrast sensitivity decreases with age
   - 20-year-old vs 60-year-old: 50% reduction

4. Environmental Factors:
   - Bright sunlight on screen
   - Glossy screens with glare
   - Dim lighting conditions

5. Situational:
   - Tired eyes at end of day
   - Driving (quick glances at screen)
   - Multitasking (peripheral vision)

Impact of Poor Contrast:
❌ Text is invisible or hard to read
❌ Eye strain and headaches
❌ Slow reading speed
❌ Information missed
❌ Legal compliance issues (WCAG)
*/
```

#### 2. WCAG Standards

**Contrast Ratios:**
```typescript
/*
WCAG 2.1 Contrast Requirements:

Level AA (Minimum):
- Normal text: 4.5:1
- Large text (18pt+ or 14pt+ bold): 3:1
- UI components: 3:1

Level AAA (Enhanced):
- Normal text: 7:1
- Large text: 4.5:1

Contrast Ratio Formula:
(L1 + 0.05) / (L2 + 0.05)
Where L1 = lighter color luminance
      L2 = darker color luminance
Range: 1:1 (white on white) to 21:1 (black on white)

Examples:

❌ #777 on #fff: 2.8:1 - FAIL
⚠️ #767676 on #fff: 4.54:1 - PASS AA (barely)
✅ #595959 on #fff: 4.6:1 - PASS AA
✅ #333 on #fff: 12.6:1 - PASS AAA
✅ #000 on #fff: 21:1 - PASS AAA (maximum)

UI Components:
❌ Border color must have 3:1 contrast with background
❌ Focus indicator must have 3:1 contrast
❌ Icons and graphics: 3:1
*/
```

#### 3. Testing Contrast

**Chrome DevTools:**
```typescript
/*
Using Chrome DevTools:

1. Inspect element (F12)
2. Select text element
3. Styles pane shows "Contrast ratio"
4. Shows:
   - Current ratio (e.g., 4.54)
   - AA pass/fail ✓ or ✗
   - AAA pass/fail ✓ or ✗
5. Color picker shows AA/AAA lines
6. Adjust color to meet standards

Lighthouse:
1. Open DevTools → Lighthouse tab
2. Run accessibility audit
3. Shows "Background and foreground colors do not have sufficient contrast"
4. Lists all violations
5. Click to inspect element

Example output:
"9 elements with low contrast found:
 - .text-gray-400: 2.8:1 (requires 4.5:1)
 - .btn-secondary: 3.2:1 (requires 4.5:1)
 - ..."
*/
```

**Online Tools:**
```typescript
/*
WebAIM Contrast Checker:
https://webaim.org/resources/contrastchecker/

Usage:
1. Enter foreground color (text): #777777
2. Enter background color: #ffffff
3. See results:
   - Ratio: 2.8:1
   - Normal text AA: FAIL
   - Large text AA: FAIL
4. Adjust colors to pass

Color.review:
https://color.review/

Features:
- Compare two colors
- Shows ratio
- WCAG AA/AAA results
- Suggests better colors
- Simulator for color blindness

Contrast Ratio (Lea Verou):
https://contrast-ratio.com/

Features:
- Quick ratio check
- Drag to adjust colors
- Real-time updates
- Simple interface
*/
```

#### 4. Common Contrast Issues

**Typography Problems:**
```typescript
// ❌ FAIL: Light gray on white (2.8:1)
const styles = {
  color: '#777777',
  backgroundColor: '#ffffff',
};

// ✅ PASS AA: Darker gray (4.6:1)
const styles = {
  color: '#595959',
  backgroundColor: '#ffffff',
};

// ❌ FAIL: Light text on light background
function BadCard() {
  return (
    <div style={{ background: '#f0f0f0', color: '#ccc' }}>
      <p>Hard to read!</p>  {/* 1.6:1 - FAIL */}
    </div>
  );
}

// ✅ PASS AAA: Dark text on light background
function GoodCard() {
  return (
    <div style={{ background: '#f5f5f5', color: '#333' }}>
      <p>Easy to read!</p>  {/* 11.7:1 - PASS AAA */}
    </div>
  );
}

// ❌ FAIL: Placeholder text often too light
input::placeholder {
  color: #aaa;  /* 2.3:1 - FAIL */
}

// ✅ PASS: Darker placeholder
input::placeholder {
  color: #767676;  /* 4.5:1 - PASS AA */
}
```

**Button Contrast:**
```typescript
// ❌ FAIL: Low contrast button
function BadButton() {
  return (
    <button
      style={{
        background: '#e0e0e0',
        color: '#999',  // 2.4:1 - FAIL
        border: '1px solid #ddd',  // 1.2:1 with background - FAIL
      }}
    >
      Submit
    </button>
  );
}

// ✅ PASS: High contrast button
function GoodButton() {
  return (
    <button
      style={{
        background: '#007bff',
        color: '#fff',  // 4.5:1 - PASS AA
        border: '2px solid #0056b3',  // 3.2:1 - PASS for UI component
      }}
    >
      Submit
    </button>
  );
}
```

**Link Contrast:**
```typescript
// ❌ FAIL: Links only distinguishable by color
function BadLinks() {
  return (
    <p style={{ color: '#333' }}>
      Visit our <a href="/about" style={{ color: '#007bff' }}>about page</a>.
    </p>
  );
}
// Color blind users can't distinguish link from text!

// ✅ PASS: Links with underline + color
function GoodLinks() {
  return (
    <p style={{ color: '#333' }}>
      Visit our{' '}
      <a
        href="/about"
        style={{
          color: '#0056b3',
          textDecoration: 'underline',
        }}
      >
        about page
      </a>.
    </p>
  );
}
// Underline ensures links are distinguishable without color
```

#### 5. Dark Mode Considerations

**Dark Mode Contrast:**
```typescript
/*
Dark Mode Challenges:

1. Not just inverted colors:
   ❌ White text on black: 21:1 (too high, causes halation/glow)
   ✅ #e0e0e0 on #121212: 15.8:1 (better)

2. Reduce contrast slightly:
   - Light mode: Black on white (21:1)
   - Dark mode: Off-white on off-black (15:1)
   - Reason: Dark mode is for low-light, less contrast needed

3. Test both modes:
   - Colors that work in light mode may fail in dark
   - Re-test all contrast ratios
*/

function ThemedButton() {
  const isDark = useDarkMode();
  
  return (
    <button
      style={{
        // Light mode
        background: isDark ? '#1976d2' : '#007bff',
        color: isDark ? '#e0e0e0' : '#ffffff',
        
        // Both pass AA:
        // Light: 4.5:1
        // Dark: 7.2:1
      }}
    >
      Submit
    </button>
  );
}
```

#### 6. Focus Indicators

**Focus Contrast:**
```css
/* ❌ FAIL: Low contrast focus */
button:focus {
  outline: 2px solid #ddd;  /* 1.3:1 with white background */
}

/* ✅ PASS: High contrast focus */
button:focus-visible {
  outline: 2px solid #0066cc;  /* 4.5:1 */
  outline-offset: 2px;
}

/* ✅ PASS: System accent color (adapts to user preference) */
button:focus-visible {
  outline: 2px solid AccentColor;
  outline-offset: 2px;
}

/* ✅ PASS: High contrast mode */
@media (prefers-contrast: high) {
  button:focus-visible {
    outline: 3px solid currentColor;
    outline-offset: 3px;
  }
}
```

#### 7. State Indicators

**Interactive States:**
```typescript
// ❌ FAIL: State only shown by color change
function BadCheckbox({ checked }) {
  return (
    <div
      style={{
        width: 20,
        height: 20,
        background: checked ? '#4caf50' : '#ddd',  // Color only!
      }}
    />
  );
}
// Color blind users can't tell if checked!

// ✅ PASS: State shown by shape + color
function GoodCheckbox({ checked }) {
  return (
    <div
      style={{
        width: 20,
        height: 20,
        border: '2px solid #333',
        background: checked ? '#4caf50' : 'transparent',
      }}
    >
      {checked && <CheckIcon />}  {/* Visual indicator */}
    </div>
  );
}

// ✅ PASS: Error with icon + color + text
function FormError({ error }) {
  if (!error) return null;
  
  return (
    <div
      role="alert"
      style={{
        color: '#c62828',  // Red (enough contrast)
        display: 'flex',
        gap: '8px',
      }}
    >
      <ErrorIcon aria-hidden="true" />  {/* Visual indicator */}
      <span>{error}</span>  {/* Text explanation */}
    </div>
  );
}
```

#### 8. Color System

**Accessible Color Palette:**
```typescript
// Design system with accessible colors
const colors = {
  // Text colors (on white background)
  text: {
    primary: '#212121',    // 16.1:1 - AAA
    secondary: '#666666',  // 5.7:1 - AAA
    disabled: '#999999',   // 2.8:1 - FAIL (use with caution)
  },
  
  // Background colors
  background: {
    primary: '#ffffff',
    secondary: '#f5f5f5',
    dark: '#121212',
  },
  
  // Semantic colors (on white)
  semantic: {
    success: '#2e7d32',  // 4.5:1 - AA
    warning: '#ed6c02',  // 3.4:1 - FAIL for text, OK for large/UI
    error: '#d32f2f',    // 4.5:1 - AA
    info: '#0288d1',     // 4.5:1 - AA
  },
  
  // Button colors
  button: {
    primary: {
      bg: '#1976d2',
      text: '#ffffff',  // 4.5:1 - AA
    },
    secondary: {
      bg: '#f5f5f5',
      text: '#333333',  // 11.7:1 - AAA
    },
  },
};

// Usage
function Button({ variant = 'primary', children }) {
  const buttonColors = colors.button[variant];
  
  return (
    <button
      style={{
        background: buttonColors.bg,
        color: buttonColors.text,
      }}
    >
      {children}
    </button>
  );
}
```

#### 9. Testing Workflow

**Development Process:**
```typescript
/*
Contrast Testing Workflow:

1. DESIGN:
   ✅ Use contrast checker in design tool (Figma, etc.)
   ✅ All text meets 4.5:1 minimum
   ✅ UI components meet 3:1
   ✅ Test with color blindness simulator

2. DEVELOPMENT:
   ✅ Use CSS variables for colors
   ✅ Test with browser DevTools
   ✅ Run axe DevTools scan
   ✅ Check focus indicators

3. TESTING:
   ✅ Lighthouse audit (automated)
   ✅ Manual inspection of all pages
   ✅ Test in bright sunlight (if possible)
   ✅ Test with grayscale mode

4. CONTINUOUS:
   ✅ Add contrast checks to CI
   ✅ Lint colors against standards
   ✅ Monitor with automated tools
   ✅ User testing with low vision users
*/

// Example: CSS variable system
:root {
  /* Base colors */
  --color-text-primary: #212121;    /* 16.1:1 */
  --color-text-secondary: #666666;  /* 5.7:1 */
  --color-bg-primary: #ffffff;
  --color-bg-secondary: #f5f5f5;
  
  /* Semantic colors */
  --color-success: #2e7d32;  /* 4.5:1 */
  --color-error: #d32f2f;    /* 4.5:1 */
  
  /* All validated for contrast! */
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --color-text-primary: #e0e0e0;    /* 15.8:1 on #121212 */
    --color-text-secondary: #b3b3b3;  /* 8.6:1 */
    --color-bg-primary: #121212;
    --color-bg-secondary: #1e1e1e;
    
    /* Re-validated for dark mode! */
  }
}
```

#### 10. Common Mistakes

**Contrast Anti-Patterns:**
```typescript
/*
❌ MISTAKE 1: Gray text everywhere
#777 is popular but only 2.8:1 - FAILS AA
Use #595959 or darker (4.6:1+)

❌ MISTAKE 2: Light placeholder text
<input placeholder="..." />  /* Often #999 */
Use #767676 minimum (4.5:1)

❌ MISTAKE 3: Disabled state too light
Disabled buttons at 2:1 contrast
Still need 4.5:1 if text is important

❌ MISTAKE 4: Icon-only buttons
<button><Icon /></button>
Icon must have 3:1 contrast with background
Add aria-label for screen readers

❌ MISTAKE 5: Color-only information
"Required fields in red"
Also add asterisk (*) or "required" text

❌ MISTAKE 6: Thin fonts at low contrast
Light font weight + low contrast = unreadable
Use heavier weight or higher contrast

❌ MISTAKE 7: Focus outline removed
button:focus { outline: none; }
Always provide alternative focus indicator

❌ MISTAKE 8: Not testing in different modes
Works in light mode, fails in dark mode
Test all color schemes!
*/
```

#### Summary

**Color Contrast Requirements:**

**WCAG Standards:**
- **Normal text**: 4.5:1 (AA), 7:1 (AAA)
- **Large text** (18pt+): 3:1 (AA), 4.5:1 (AAA)
- **UI components**: 3:1

**Testing Tools:**
1. Chrome DevTools (built-in)
2. Lighthouse audits
3. WebAIM Contrast Checker
4. axe DevTools

**Best Practices:**
```typescript
/*
✅ Use dark text on light backgrounds (#333 on #fff)
✅ Test all color combinations
✅ Don't rely on color alone
✅ Provide text alternatives
✅ Test with color blindness simulator
✅ Test in bright light and dark mode
✅ Use system colors when possible
✅ Validate during design, not after
*/
```

**Quick Reference:**
```typescript
// Safe color combinations (on white #fff)
✅ #000000 (black): 21:1 - AAA
✅ #333333 (dark gray): 12.6:1 - AAA
✅ #595959 (medium gray): 4.6:1 - AA
⚠️ #767676 (light gray): 4.54:1 - AA (barely)
❌ #777777 (too light): 2.8:1 - FAIL
❌ #999999 (way too light): 2.8:1 - FAIL
```

**Key Principle:**
> "If users can't read it, it doesn't matter how beautiful your design is!"

**Golden Rule:**
> "4.5:1 for text, 3:1 for UI components. No exceptions!"

**Testing Rule:**
> "View your site in bright sunlight. If you can't read it, neither can your users!"

</details>

---

## Completion Status

Topic 21: Accessibility (a11y) - **COMPLETE** ✅

- Q201: Making React apps accessible ✅
- Q202: ARIA in React ✅
- Q203: Keyboard navigation ✅
- Q204: Focus management ✅
- Q205: Testing accessibility ✅
- Q206: React ARIA library ✅
- Q207: Screen reader compatibility ✅
- Q208: Semantic HTML elements ✅
- Q209: Skip links implementation ✅
- Q210: Color contrast ✅

**All 10 questions completed!** 🎉
