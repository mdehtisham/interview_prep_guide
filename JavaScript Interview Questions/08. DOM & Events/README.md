## DOM & Events

---

### 71. What is the DOM?

**Answer:**

The DOM (Document Object Model) is a programming interface that represents HTML and XML documents as a tree structure of objects. It provides a structured representation of the document and defines methods for accessing and manipulating the document's content, structure, and styles. The DOM serves as a bridge between web pages and programming languages like JavaScript.

#### **DOM Structure**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>My Page</title>
  </head>
  <body>
    <div id="container">
      <h1 class="title">Hello World</h1>
      <p>This is a paragraph.</p>
    </div>
  </body>
</html>
```

**DOM Tree Representation:**
```
Document
  └─ html (document.documentElement)
      ├─ head
      │   └─ title
      │       └─ "My Page" (text node)
      └─ body (document.body)
          └─ div#container
              ├─ h1.title
              │   └─ "Hello World" (text node)
              └─ p
                  └─ "This is a paragraph." (text node)
```

#### **DOM Node Types**

```javascript
// Different node types in the DOM
console.log(Node.ELEMENT_NODE);                // 1
console.log(Node.TEXT_NODE);                   // 3
console.log(Node.COMMENT_NODE);                // 8
console.log(Node.DOCUMENT_NODE);               // 9
console.log(Node.DOCUMENT_FRAGMENT_NODE);      // 11

// Accessing node types
const element = document.querySelector('h1');
console.log(element.nodeType);                 // 1 (ELEMENT_NODE)
console.log(element.nodeName);                 // "H1"
console.log(element.nodeValue);                // null (elements don't have nodeValue)

const textNode = element.firstChild;
console.log(textNode.nodeType);                // 3 (TEXT_NODE)
console.log(textNode.nodeName);                // "#text"
console.log(textNode.nodeValue);               // "Hello World"

// Document node
console.log(document.nodeType);                // 9 (DOCUMENT_NODE)
console.log(document.nodeName);                // "#document"
```

#### **Accessing DOM Elements**

```javascript
// 1. By ID (returns single element or null)
const container = document.getElementById('container');

// 2. By class name (returns HTMLCollection)
const titles = document.getElementsByClassName('title');
console.log(titles[0]);  // First element with class 'title'

// 3. By tag name (returns HTMLCollection)
const paragraphs = document.getElementsByTagName('p');
console.log(paragraphs.length);

// 4. Query selector (returns first match or null)
const firstTitle = document.querySelector('.title');
const firstPara = document.querySelector('p');
const divPara = document.querySelector('div p');  // CSS selectors

// 5. Query selector all (returns NodeList)
const allTitles = document.querySelectorAll('.title');
const allParas = document.querySelectorAll('p');

// Convert to array for array methods
const parasArray = Array.from(allParas);
parasArray.forEach(p => console.log(p.textContent));

// 6. Special selectors
const bodyElement = document.body;             // <body>
const htmlElement = document.documentElement;  // <html>
const headElement = document.head;             // <head>
```

#### **DOM Traversal**

```javascript
const container = document.getElementById('container');

// Parent navigation
console.log(container.parentNode);           // Parent node
console.log(container.parentElement);        // Parent element (same for elements)

// Child navigation
console.log(container.childNodes);           // All child nodes (including text nodes)
console.log(container.children);             // Only element children (HTMLCollection)
console.log(container.firstChild);           // First child node (might be text)
console.log(container.firstElementChild);    // First child element
console.log(container.lastChild);            // Last child node
console.log(container.lastElementChild);     // Last child element
console.log(container.hasChildNodes());      // true/false

// Sibling navigation
const h1 = document.querySelector('h1');
console.log(h1.nextSibling);                 // Next sibling node
console.log(h1.nextElementSibling);          // Next sibling element
console.log(h1.previousSibling);             // Previous sibling node
console.log(h1.previousElementSibling);      // Previous sibling element

// Ancestor checking
const p = document.querySelector('p');
console.log(container.contains(p));          // true (p is inside container)

// Closest ancestor matching selector
const closestDiv = p.closest('div');         // Finds nearest ancestor div
console.log(closestDiv.id);                  // 'container'
```

#### **Manipulating DOM Content**

```javascript
const element = document.querySelector('h1');

// 1. Text content (plain text only, safe from XSS)
console.log(element.textContent);            // "Hello World"
element.textContent = 'New Text';            // Sets text, HTML is escaped

// 2. innerHTML (can parse HTML, XSS risk if not sanitized)
console.log(element.innerHTML);              // "Hello World"
element.innerHTML = '<span>Formatted</span> Text';  // Parses HTML

// 3. innerText (similar to textContent, but respects CSS styling)
console.log(element.innerText);              // Visible text only
element.innerText = 'Visible Text';

// 4. outerHTML (includes the element itself)
console.log(element.outerHTML);              // "<h1>...</h1>"
element.outerHTML = '<h2>New Header</h2>';   // Replaces entire element

// Creating text nodes
const textNode = document.createTextNode('Pure text');
element.appendChild(textNode);
```

#### **Creating and Modifying Elements**

```javascript
// Creating elements
const newDiv = document.createElement('div');
const newPara = document.createElement('p');
const newText = document.createTextNode('Hello');

// Setting attributes
newDiv.id = 'new-container';
newDiv.className = 'container-class';
newDiv.setAttribute('data-id', '123');
newDiv.setAttribute('aria-label', 'Container');

// Setting content
newPara.textContent = 'This is a new paragraph';
newPara.innerHTML = '<strong>Bold text</strong> and normal text';

// Building structure
newDiv.appendChild(newPara);

// Inserting into DOM
document.body.appendChild(newDiv);           // Append to end of body
container.insertBefore(newDiv, container.firstChild);  // Insert at beginning

// Modern insertion methods
container.append(newDiv);                    // Append (can take multiple nodes/strings)
container.prepend(newDiv);                   // Prepend to start
newPara.before(newDiv);                      // Insert before element
newPara.after(newDiv);                       // Insert after element
newPara.replaceWith(newDiv);                 // Replace element

// Cloning elements
const clone = newDiv.cloneNode(false);       // Shallow clone (no children)
const deepClone = newDiv.cloneNode(true);    // Deep clone (with children)

// Removing elements
newDiv.remove();                             // Remove from DOM (modern)
container.removeChild(newDiv);               // Old method
```

#### **Working with Attributes**

```javascript
const element = document.querySelector('h1');

// Getting attributes
const id = element.getAttribute('id');
const className = element.getAttribute('class');
const dataValue = element.getAttribute('data-value');

// Setting attributes
element.setAttribute('id', 'main-title');
element.setAttribute('class', 'title primary');
element.setAttribute('data-index', '1');

// Removing attributes
element.removeAttribute('data-index');

// Checking attributes
console.log(element.hasAttribute('id'));     // true/false

// Direct property access (common attributes)
element.id = 'new-id';
element.className = 'new-class';
element.title = 'Tooltip text';

// Data attributes
element.dataset.userId = '123';              // Sets data-user-id
element.dataset.itemCount = '5';             // Sets data-item-count
console.log(element.dataset.userId);         // '123'

// Getting all attributes
const attrs = element.attributes;            // NamedNodeMap
for (let attr of attrs) {
  console.log(`${attr.name}: ${attr.value}`);
}
```

#### **Modifying Styles**

```javascript
const element = document.querySelector('h1');

// Inline styles
element.style.color = 'red';
element.style.fontSize = '24px';
element.style.backgroundColor = 'blue';      // camelCase for CSS properties
element.style.margin = '10px 20px';

// Getting computed styles (including CSS rules)
const styles = window.getComputedStyle(element);
console.log(styles.color);                   // 'rgb(255, 0, 0)'
console.log(styles.fontSize);                // '24px'
console.log(styles.marginTop);               // Computed margin

// Setting multiple styles
Object.assign(element.style, {
  color: 'blue',
  fontSize: '20px',
  padding: '10px'
});

// Removing inline styles
element.style.removeProperty('color');
element.style.color = '';                    // Alternative

// CSS Text
element.style.cssText = 'color: red; font-size: 24px; margin: 10px;';
console.log(element.style.cssText);
```

#### **Working with Classes**

```javascript
const element = document.querySelector('h1');

// className (string)
element.className = 'title primary';         // Sets all classes
console.log(element.className);              // 'title primary'

// classList (modern, recommended)
element.classList.add('active');             // Add class
element.classList.add('highlight', 'bold');  // Add multiple
element.classList.remove('primary');         // Remove class
element.classList.toggle('visible');         // Toggle (add if absent, remove if present)
element.classList.toggle('dark', true);      // Force add
element.classList.replace('title', 'heading');  // Replace class

// Checking classes
console.log(element.classList.contains('active'));  // true/false

// Iterate over classes
element.classList.forEach(className => {
  console.log(className);
});

// Convert to array
const classArray = [...element.classList];
```

#### **DOM Events (Basic)**

```javascript
const button = document.querySelector('button');

// Adding event listeners
button.addEventListener('click', function(event) {
  console.log('Button clicked!');
  console.log('Event type:', event.type);
  console.log('Target:', event.target);
});

// Named function for event handler
function handleClick(event) {
  console.log('Clicked:', event.target);
}
button.addEventListener('click', handleClick);

// Removing event listeners (must be same function reference)
button.removeEventListener('click', handleClick);

// Event options
button.addEventListener('click', handler, {
  once: true,      // Auto-remove after first trigger
  capture: false,  // Use bubbling phase (default)
  passive: true    // Won't call preventDefault()
});

// Multiple events
['mouseenter', 'mouseleave'].forEach(eventType => {
  button.addEventListener(eventType, event => {
    console.log(eventType);
  });
});
```

#### **DocumentFragment for Performance**

```javascript
// Creating many elements efficiently
function createList(items) {
  // Use DocumentFragment to minimize reflows
  const fragment = document.createDocumentFragment();
  
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item;
    fragment.appendChild(li);  // Append to fragment (in memory)
  });
  
  // Single DOM update
  document.querySelector('ul').appendChild(fragment);
}

createList(['Item 1', 'Item 2', 'Item 3', 'Item 4']);

// Without fragment (causes multiple reflows - slower)
function createListSlow(items) {
  const ul = document.querySelector('ul');
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item;
    ul.appendChild(li);  // Each append causes reflow
  });
}
```

#### **Practical Examples**

**1. Dynamic List Creation:**
```javascript
function renderUserList(users) {
  const container = document.getElementById('user-list');
  container.innerHTML = '';  // Clear existing
  
  users.forEach(user => {
    const userCard = document.createElement('div');
    userCard.className = 'user-card';
    userCard.innerHTML = `
      <h3>${user.name}</h3>
      <p>${user.email}</p>
      <button data-user-id="${user.id}">View Profile</button>
    `;
    container.appendChild(userCard);
  });
}

const users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' }
];

renderUserList(users);
```

**2. Form Handling:**
```javascript
const form = document.querySelector('form');
const nameInput = document.getElementById('name');
const emailInput = document.getElementById('email');

form.addEventListener('submit', event => {
  event.preventDefault();  // Prevent default form submission
  
  const formData = {
    name: nameInput.value,
    email: emailInput.value
  };
  
  console.log('Form submitted:', formData);
  
  // Validate
  if (!formData.name) {
    nameInput.classList.add('error');
    return;
  }
  
  // Process form data
});
```

**3. Dynamic Content Update:**
```javascript
async function loadContent(url) {
  const container = document.getElementById('content');
  container.innerHTML = '<p>Loading...</p>';
  
  try {
    const response = await fetch(url);
    const data = await response.json();
    
    container.innerHTML = `
      <h2>${data.title}</h2>
      <p>${data.description}</p>
    `;
  } catch (error) {
    container.innerHTML = '<p class="error">Failed to load content</p>';
  }
}
```

#### **DOM Performance Considerations**

```javascript
// ❌ Bad: Multiple reflows
function inefficientUpdate() {
  const element = document.getElementById('box');
  element.style.width = '100px';   // Reflow
  element.style.height = '100px';  // Reflow
  element.style.margin = '10px';   // Reflow
}

// ✅ Good: Batch style changes
function efficientUpdate() {
  const element = document.getElementById('box');
  element.style.cssText = 'width: 100px; height: 100px; margin: 10px;';  // Single reflow
}

// ✅ Good: Use CSS classes
function bestUpdate() {
  const element = document.getElementById('box');
  element.classList.add('box-sized');  // CSS handles all styles
}

// ❌ Bad: Reading and writing in loop
const elements = document.querySelectorAll('.item');
elements.forEach(el => {
  const height = el.offsetHeight;  // Triggers reflow (read)
  el.style.height = height + 10 + 'px';  // Triggers reflow (write)
});

// ✅ Good: Separate reads and writes
const heights = [];
elements.forEach(el => {
  heights.push(el.offsetHeight);  // Batch reads
});
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + 'px';  // Batch writes
});
```

#### **Key Takeaways**

- **DOM is a tree structure** representing HTML documents
- **Nodes** include elements, text, comments, and documents
- **Access elements** via getElementById, querySelector, etc.
- **Navigate** with parentNode, children, nextSibling, etc.
- **Manipulate content** with textContent (safe) or innerHTML (flexible)
- **Create elements** with createElement, append with appendChild
- **Modern methods** (append, prepend, before, after) are cleaner
- **Attributes** accessed via getAttribute/setAttribute or direct properties
- **Styles** modified via element.style or classList
- **classList** preferred over className for class manipulation
- **DocumentFragment** improves performance for batch operations
- **Minimize reflows** by batching DOM updates
- **Event listeners** attach with addEventListener
- **DOM manipulation is expensive** - optimize for performance
- **XSS risk** with innerHTML - sanitize user input

---

### 72. What is event bubbling?

**Answer:**

Event bubbling is the process where an event triggered on a deeply nested element propagates upward through its ancestors in the DOM tree, triggering event handlers on parent elements. The event starts at the target element and "bubbles up" to the document root, allowing parent elements to respond to events from their children.

#### **Bubbling Mechanism**

```html
<div id="outer">
  <div id="middle">
    <button id="inner">Click Me</button>
  </div>
</div>
```

```javascript
// Event bubbling demonstration
const outer = document.getElementById('outer');
const middle = document.getElementById('middle');
const inner = document.getElementById('inner');

outer.addEventListener('click', () => {
  console.log('3. Outer div clicked');
});

middle.addEventListener('click', () => {
  console.log('2. Middle div clicked');
});

inner.addEventListener('click', () => {
  console.log('1. Button clicked');
});

// When button is clicked, console output:
// 1. Button clicked
// 2. Middle div clicked
// 3. Outer div clicked

// Event travels: button → middle div → outer div → document
```

#### **Event Flow Phases**

```javascript
// Three phases of event propagation:
// 1. CAPTURING PHASE (top-down)
// 2. TARGET PHASE (at target element)
// 3. BUBBLING PHASE (bottom-up)

const container = document.getElementById('container');
const button = document.querySelector('button');

// Capture phase listener (useCapture = true)
container.addEventListener('click', () => {
  console.log('Capture: Container');
}, true);

// Bubbling phase listener (useCapture = false, default)
container.addEventListener('click', () => {
  console.log('Bubble: Container');
}, false);

button.addEventListener('click', () => {
  console.log('Target: Button');
});

// Click button, output:
// Capture: Container  (capturing down)
// Target: Button      (at target)
// Bubble: Container   (bubbling up)
```

#### **Event Object Properties**

```javascript
button.addEventListener('click', function(event) {
  // Target: element that triggered the event
  console.log('event.target:', event.target);  // <button>
  
  // CurrentTarget: element with the listener attached
  console.log('event.currentTarget:', event.currentTarget);  // Same as 'this'
  
  // Event phase
  console.log('event.eventPhase:', event.eventPhase);
  // 1 = CAPTURING_PHASE, 2 = AT_TARGET, 3 = BUBBLING_PHASE
  
  // Bubbles flag
  console.log('event.bubbles:', event.bubbles);  // true
});

// Parent handler showing difference
container.addEventListener('click', function(event) {
  console.log('event.target:', event.target);          // <button> (original target)
  console.log('event.currentTarget:', event.currentTarget);  // <div> (current handler)
  console.log('this:', this);                          // <div> (same as currentTarget)
});
```

#### **Stopping Event Propagation**

```javascript
// 1. stopPropagation() - stops bubbling to parents
inner.addEventListener('click', function(event) {
  console.log('Button clicked');
  event.stopPropagation();  // Stops here, parents won't receive event
});

middle.addEventListener('click', () => {
  console.log('This will NOT run if stopPropagation is called');
});

// 2. stopImmediatePropagation() - stops bubbling AND other listeners on same element
inner.addEventListener('click', function(event) {
  console.log('First listener');
  event.stopImmediatePropagation();
});

inner.addEventListener('click', () => {
  console.log('This will NOT run');  // Skipped
});

// 3. Checking if propagation was stopped
middle.addEventListener('click', function(event) {
  if (event.cancelBubble) {  // Same as checking if stopPropagation was called
    console.log('Propagation was stopped');
  }
});
```

#### **Practical Use Cases**

**1. Event Delegation:**
```javascript
// Instead of attaching listener to each item
// Attach ONE listener to parent and use bubbling

const list = document.getElementById('todo-list');

// ❌ Inefficient: Multiple listeners
document.querySelectorAll('.todo-item').forEach(item => {
  item.addEventListener('click', handleClick);
});

// ✅ Efficient: Single listener with delegation
list.addEventListener('click', function(event) {
  // Check if clicked element is a todo item
  const todoItem = event.target.closest('.todo-item');
  
  if (todoItem) {
    console.log('Todo clicked:', todoItem.textContent);
    
    // Check specific actions
    if (event.target.classList.contains('delete-btn')) {
      deleteTodo(todoItem);
    } else if (event.target.classList.contains('complete-btn')) {
      toggleComplete(todoItem);
    }
  }
});

// Works for dynamically added items too!
function addTodoItem(text) {
  const li = document.createElement('li');
  li.className = 'todo-item';
  li.innerHTML = `
    ${text}
    <button class="complete-btn">✓</button>
    <button class="delete-btn">✗</button>
  `;
  list.appendChild(li);
  // No need to add event listeners!
}
```

**2. Modal/Overlay Closing:**
```javascript
// Close modal when clicking outside
const modal = document.getElementById('modal');
const modalContent = document.querySelector('.modal-content');

modal.addEventListener('click', function(event) {
  // Close if clicked on overlay (not content)
  if (event.target === modal) {
    closeModal();
  }
});

// Prevent closing when clicking inside modal content
modalContent.addEventListener('click', function(event) {
  event.stopPropagation();  // Don't bubble to modal overlay
});

// Alternative without stopPropagation
modal.addEventListener('click', function(event) {
  if (!modalContent.contains(event.target)) {
    closeModal();
  }
});
```

**3. Form Validation Bubbling:**
```javascript
const form = document.querySelector('form');

// Validate all inputs via bubbling
form.addEventListener('blur', function(event) {
  const input = event.target;
  
  if (input.tagName === 'INPUT' || input.tagName === 'TEXTAREA') {
    if (!input.value.trim()) {
      input.classList.add('error');
      showError(input, 'This field is required');
    } else {
      input.classList.remove('error');
      clearError(input);
    }
  }
}, true);  // Use capture to handle before individual handlers
```

**4. Table Row Actions:**
```javascript
const table = document.querySelector('table');

table.addEventListener('click', function(event) {
  const row = event.target.closest('tr');
  
  if (!row || !table.contains(row)) return;
  
  // Handle different button clicks
  if (event.target.matches('.edit-btn')) {
    editRow(row);
  } else if (event.target.matches('.delete-btn')) {
    deleteRow(row);
  } else if (event.target.matches('.view-btn')) {
    viewRowDetails(row);
  }
});
```

**5. Nested Component Communication:**
```javascript
// Parent component can listen to events from nested children
const app = document.getElementById('app');

app.addEventListener('custom-action', function(event) {
  console.log('Action from:', event.target);
  console.log('Action data:', event.detail);
});

// Child component dispatches custom event
const childButton = document.querySelector('.child-button');
childButton.addEventListener('click', function() {
  // Create and dispatch custom event (bubbles by default)
  const customEvent = new CustomEvent('custom-action', {
    bubbles: true,  // Enable bubbling
    detail: { action: 'buttonClick', value: 123 }
  });
  
  this.dispatchEvent(customEvent);
});
```

#### **Events That Don't Bubble**

```javascript
// Some events don't bubble by default:

// Focus events (non-bubbling)
input.addEventListener('focus', handler);    // Doesn't bubble
input.addEventListener('blur', handler);     // Doesn't bubble

// Use bubbling versions instead:
input.addEventListener('focusin', handler);  // Bubbles
input.addEventListener('focusout', handler); // Bubbles

// Other non-bubbling events:
// - load, unload
// - error
// - scroll (in some cases)
// - mouseenter, mouseleave (use mouseover, mouseout instead)

// Check if event bubbles
element.addEventListener('someEvent', function(event) {
  console.log('Bubbles:', event.bubbles);  // true/false
});
```

#### **Bubbling vs Capturing**

```javascript
const parent = document.getElementById('parent');
const child = document.getElementById('child');

// Capturing (top-down, parent first)
parent.addEventListener('click', () => {
  console.log('Parent - Capturing');
}, true);

child.addEventListener('click', () => {
  console.log('Child - Capturing');
}, true);

// Bubbling (bottom-up, child first)
child.addEventListener('click', () => {
  console.log('Child - Bubbling');
}, false);

parent.addEventListener('click', () => {
  console.log('Parent - Bubbling');
}, false);

// Click child, output:
// Parent - Capturing
// Child - Capturing
// Child - Bubbling
// Parent - Bubbling
```

#### **Common Pitfalls**

```javascript
// 1. Forgetting event.target vs event.currentTarget
parent.addEventListener('click', function(event) {
  console.log(event.target);          // Might be child element
  console.log(event.currentTarget);   // Always parent
  console.log(this);                  // Always parent (same as currentTarget)
});

// 2. Stopping propagation unnecessarily
button.addEventListener('click', function(event) {
  event.stopPropagation();  // Might break parent handlers
  // Be careful - this can break event delegation!
});

// 3. Not checking event.target in delegation
list.addEventListener('click', function(event) {
  // ❌ Assumes all clicks are on list items
  deleteItem(event.target);
  
  // ✅ Check if target is correct element
  if (event.target.matches('.list-item')) {
    deleteItem(event.target);
  }
});

// 4. Memory leaks with many listeners
// ❌ Creating thousands of listeners
items.forEach(item => {
  item.addEventListener('click', handler);
});

// ✅ Use delegation with one listener
container.addEventListener('click', function(event) {
  if (event.target.matches('.item')) {
    handler(event);
  }
});
```

#### **Performance Benefits**

```javascript
// Event delegation performance comparison

// Scenario: 1000 list items

// ❌ Individual listeners (1000 listeners)
function attachIndividualListeners() {
  const items = document.querySelectorAll('.item');
  items.forEach(item => {
    item.addEventListener('click', handleClick);
    item.addEventListener('mouseover', handleHover);
  });
  // Memory: High (2000 listeners)
  // Performance: Slower with many items
}

// ✅ Event delegation (1-2 listeners)
function attachDelegatedListeners() {
  const container = document.querySelector('.container');
  
  container.addEventListener('click', function(event) {
    if (event.target.matches('.item')) {
      handleClick(event);
    }
  });
  
  container.addEventListener('mouseover', function(event) {
    if (event.target.matches('.item')) {
      handleHover(event);
    }
  });
  // Memory: Low (2 listeners)
  // Performance: Fast, scales well
}
```

#### **Key Takeaways**

- **Event bubbling** propagates events from target to ancestors
- **Three phases**: capturing (down) → target → bubbling (up)
- **event.target** is the element that triggered the event
- **event.currentTarget** is the element with the listener
- **stopPropagation()** prevents bubbling to parents
- **stopImmediatePropagation()** stops all handlers
- **Event delegation** uses bubbling for efficient event handling
- **One listener on parent** better than many on children
- **Works with dynamic content** (elements added later)
- **Not all events bubble** (focus, blur, load, mouseenter/leave)
- **Bubbling enables** powerful patterns like event delegation
- **Performance benefit** with fewer listeners
- **Check event.target** in delegated handlers
- **Be careful with stopPropagation** - can break delegation
- Default for addEventListener is **bubbling phase** (useCapture=false)

---

### 73. What is event capturing?

**Answer:**

Event capturing (also called trickling) is the first phase of event propagation where an event travels from the document root down through ancestors to the target element. It's the opposite of bubbling - the event "captures" down the DOM tree before reaching the target. Capturing happens before the bubbling phase.

#### **Capturing Phase Flow**

```html
<div id="grandparent">
  <div id="parent">
    <button id="child">Click Me</button>
  </div>
</div>
```

```javascript
// Enabling capture phase with useCapture = true (third parameter)
const grandparent = document.getElementById('grandparent');
const parent = document.getElementById('parent');
const child = document.getElementById('child');

// Capture phase listeners (execute top-down)
grandparent.addEventListener('click', () => {
  console.log('1. Grandparent - Capturing');
}, true);  // true = capture phase

parent.addEventListener('click', () => {
  console.log('2. Parent - Capturing');
}, true);

child.addEventListener('click', () => {
  console.log('3. Child - Capturing');
}, true);

// When button is clicked:
// 1. Grandparent - Capturing (top level first)
// 2. Parent - Capturing
// 3. Child - Capturing (target last in capture phase)
```

#### **Complete Event Flow: Capture → Target → Bubble**

```javascript
const outer = document.getElementById('outer');
const inner = document.getElementById('inner');

// CAPTURE PHASE (true = useCapture)
outer.addEventListener('click', () => {
  console.log('1. Outer - CAPTURE (going down)');
}, true);

inner.addEventListener('click', () => {
  console.log('2. Inner - CAPTURE (reached target)');
}, true);

// TARGET PHASE (at the target element)
inner.addEventListener('click', () => {
  console.log('3. Inner - TARGET (at target)');
}, false);

// BUBBLING PHASE (false = useCapture, default)
inner.addEventListener('click', () => {
  console.log('4. Inner - BUBBLE (going up)');
}, false);

outer.addEventListener('click', () => {
  console.log('5. Outer - BUBBLE (back to top)');
}, false);

// Click inner button, output:
// 1. Outer - CAPTURE (going down)
// 2. Inner - CAPTURE (reached target)
// 3. Inner - TARGET (at target)
// 4. Inner - BUBBLE (going up)
// 5. Outer - BUBBLE (back to top)
```

#### **Capture with Options Object**

```javascript
const element = document.getElementById('element');

// Modern syntax with options object
element.addEventListener('click', handler, {
  capture: true,   // Enable capturing phase
  once: false,     // Can trigger multiple times
  passive: false   // Can call preventDefault()
});

// Equivalent to old syntax
element.addEventListener('click', handler, true);

// Bubbling phase (default)
element.addEventListener('click', handler, {
  capture: false  // Or just omit options entirely
});
```

#### **Event Phase Detection**

```javascript
function logPhase(event) {
  const phases = {
    0: 'NONE',
    1: 'CAPTURING_PHASE',
    2: 'AT_TARGET',
    3: 'BUBBLING_PHASE'
  };
  
  console.log(`
    Phase: ${phases[event.eventPhase]}
    Target: ${event.target.id}
    CurrentTarget: ${event.currentTarget.id}
  `);
}

parent.addEventListener('click', logPhase, true);   // Capture
child.addEventListener('click', logPhase, false);   // Bubble

// Click child:
// Phase: CAPTURING_PHASE, Target: child, CurrentTarget: parent
// Phase: AT_TARGET, Target: child, CurrentTarget: child
// Phase: BUBBLING_PHASE, Target: child, CurrentTarget: parent
```

#### **Stopping Propagation in Capture Phase**

```javascript
// Stop during capture phase
grandparent.addEventListener('click', function(event) {
  console.log('Grandparent captured event');
  event.stopPropagation();  // Stops here, won't reach child or bubble back
}, true);

parent.addEventListener('click', () => {
  console.log('This will NOT run');
}, true);

child.addEventListener('click', () => {
  console.log('This will NOT run either');
});

// Only "Grandparent captured event" will log
```

#### **Practical Use Cases for Capturing**

**1. Global Event Interception:**
```javascript
// Intercept all clicks before they reach targets
document.addEventListener('click', function(event) {
  console.log('Global click intercepted:', event.target);
  
  // Conditional stopping
  if (event.target.matches('.disabled')) {
    event.stopPropagation();  // Prevent any further handling
    event.preventDefault();
    console.log('Clicked on disabled element - blocked');
  }
}, true);  // Capture phase ensures this runs first
```

**2. Form Validation Before Submission:**
```javascript
const form = document.querySelector('form');

// Validate in capture phase (before individual field handlers)
form.addEventListener('submit', function(event) {
  const invalidFields = form.querySelectorAll(':invalid');
  
  if (invalidFields.length > 0) {
    event.stopPropagation();  // Stop event propagation
    event.preventDefault();    // Prevent form submission
    
    console.log('Form validation failed');
    invalidFields.forEach(field => {
      field.classList.add('error');
    });
  }
}, true);  // Capture phase
```

**3. Analytics/Logging:**
```javascript
// Log all interactions before they're processed
document.body.addEventListener('click', function(event) {
  // Analytics tracking in capture phase
  const analytics = {
    timestamp: Date.now(),
    target: event.target.tagName,
    id: event.target.id,
    classes: event.target.className
  };
  
  console.log('Analytics:', analytics);
  // Send to analytics service
  
  // Don't stop propagation - let event continue
}, true);
```

**4. Access Control/Permission Checking:**
```javascript
// Check permissions before allowing interactions
const restrictedArea = document.getElementById('admin-panel');

restrictedArea.addEventListener('click', function(event) {
  if (!userHasPermission()) {
    event.stopPropagation();
    event.preventDefault();
    
    showNotification('Access denied: Insufficient permissions');
    console.log('Blocked interaction with:', event.target);
  }
}, true);  // Capture ensures this check happens first
```

**5. Focus Management:**
```javascript
// Handle focus events in capture phase
document.addEventListener('focusin', function(event) {
  console.log('Element gaining focus:', event.target);
  
  // Close dropdowns when focus moves outside
  if (!dropdown.contains(event.target)) {
    closeDropdown();
  }
  
  // Track focus for accessibility
  updateFocusIndicator(event.target);
}, true);  // Use capture for focusin
```

**6. Preventing Default Actions Early:**
```javascript
// Prevent link navigation in certain conditions
document.addEventListener('click', function(event) {
  const link = event.target.closest('a');
  
  if (link && link.hostname !== window.location.hostname) {
    // External link
    if (!confirmExternalNavigation()) {
      event.preventDefault();
      event.stopPropagation();
      console.log('External navigation cancelled');
    }
  }
}, true);  // Capture phase intercepts before link navigation
```

#### **Capture vs Bubble Comparison**

| Feature | Capturing | Bubbling |
|---------|-----------|----------|
| **Direction** | Top-down (document → target) | Bottom-up (target → document) |
| **Phase order** | First (phase 1) | Last (phase 3) |
| **Default** | No (must explicitly enable) | Yes (default behavior) |
| **Use case** | Intercept before target | Delegate from children |
| **Common usage** | Less common | More common |
| **Enable with** | `true` or `{capture: true}` | `false` or omit |
| **stopPropagation** | Stops descending and bubbling | Stops ascending only |
| **Best for** | Global handlers, validation | Event delegation |

```javascript
// Visual comparison
element.addEventListener('click', captureHandler, true);   // Capture
element.addEventListener('click', bubbleHandler, false);   // Bubble (default)
element.addEventListener('click', bubbleHandler);          // Bubble (default)
```

#### **Removing Capture Listeners**

```javascript
function handler(event) {
  console.log('Event handled');
}

// Adding capture listener
element.addEventListener('click', handler, true);

// Removing - must match capture flag!
element.removeEventListener('click', handler, true);  // ✅ Correct

// This won't work:
element.removeEventListener('click', handler, false);  // ❌ Wrong - doesn't match
element.removeEventListener('click', handler);          // ❌ Wrong - doesn't match
```

#### **Mixed Capture and Bubble Handlers**

```javascript
const container = document.getElementById('container');
const button = document.querySelector('button');

// Same element can have both capture and bubble handlers
container.addEventListener('click', () => {
  console.log('Container - Capture (going down)');
}, true);

container.addEventListener('click', () => {
  console.log('Container - Bubble (going up)');
}, false);

button.addEventListener('click', () => {
  console.log('Button clicked');
});

// Click button, output:
// Container - Capture (going down)  ← Capture phase
// Button clicked                     ← Target phase
// Container - Bubble (going up)     ← Bubble phase
```

#### **Real-World Patterns**

**1. Modal Trap Focus:**
```javascript
// Trap focus within modal during capture
modal.addEventListener('focusin', function(event) {
  if (!modal.contains(event.target)) {
    event.stopPropagation();
    // Return focus to modal
    modal.querySelector('[tabindex="0"]').focus();
  }
}, true);
```

**2. Keyboard Shortcuts:**
```javascript
// Global keyboard shortcuts in capture
document.addEventListener('keydown', function(event) {
  // Ctrl+S to save
  if (event.ctrlKey && event.key === 's') {
    event.preventDefault();
    event.stopPropagation();
    saveDocument();
  }
  
  // ESC to close modals
  if (event.key === 'Escape') {
    closeAllModals();
  }
}, true);  // Capture ensures this runs before other handlers
```

**3. Drag and Drop Guard:**
```javascript
// Prevent accidental dragging in capture phase
document.addEventListener('dragstart', function(event) {
  if (!event.target.matches('[draggable="true"]')) {
    event.preventDefault();
    event.stopPropagation();
    console.log('Drag prevented for:', event.target);
  }
}, true);
```

#### **When to Use Capturing**

**Use Capturing when:**
- Need to intercept events before they reach targets
- Implementing global event handlers (analytics, logging)
- Validating before processing
- Enforcing access control
- Preventing default actions early
- Want to handle events before bubbling phase

**Use Bubbling when:**
- Implementing event delegation
- Handling events from children
- Most standard event handling
- Working with dynamic content
- Default behavior is sufficient

```javascript
// ✅ Good use of capturing: Global validation
form.addEventListener('input', function(event) {
  // Validate all inputs before individual handlers
  validateInput(event.target);
}, true);

// ✅ Good use of bubbling: Event delegation
list.addEventListener('click', function(event) {
  if (event.target.matches('.item')) {
    handleItemClick(event.target);
  }
}, false);
```

#### **Key Takeaways**

- **Capturing** is the first phase of event propagation (top-down)
- **Opposite of bubbling** - travels from document to target
- **Enabled with** `true` or `{capture: true}` parameter
- **Less common** than bubbling but useful for interception
- **Runs before** target and bubbling phases
- **stopPropagation()** in capture prevents target and bubbling
- **Both phases can coexist** on same element
- **Must match capture flag** when removing listeners
- **Use for** global handlers, validation, access control
- **Perfect for** intercepting events before processing
- **Event phase** detectable via event.eventPhase property
- **Bubbling is default** - must explicitly enable capturing
- **Capturing handlers** execute in document → target order
- Understanding both phases enables **powerful event patterns**
- **Rarely needed** in simple applications but crucial for complex UIs


---

### 74. What is event delegation?

**Answer:**

Event delegation is a technique where instead of attaching event listeners to individual child elements, you attach a single event listener to a parent element and use event bubbling to handle events from child elements. This pattern leverages event propagation to handle events efficiently, especially for dynamic content.

#### **Basic Concept**

```javascript
// WITHOUT Event Delegation (inefficient)
const buttons = document.querySelectorAll('.button');
buttons.forEach(button => {
  button.addEventListener('click', function() {
    console.log('Button clicked:', this.textContent);
  });
});
// Problem: Creates N listeners for N buttons
// Problem: Doesn't work for dynamically added buttons

// WITH Event Delegation (efficient)
const container = document.querySelector('.button-container');
container.addEventListener('click', function(event) {
  // Check if clicked element is a button
  if (event.target.matches('.button')) {
    console.log('Button clicked:', event.target.textContent);
  }
});
// Solution: Only 1 listener
// Solution: Works for dynamically added buttons
```

#### **How Event Delegation Works**

```html
<ul id="todo-list">
  <li class="todo-item">Task 1 <button class="delete">Delete</button></li>
  <li class="todo-item">Task 2 <button class="delete">Delete</button></li>
  <li class="todo-item">Task 3 <button class="delete">Delete</button></li>
</ul>
```

```javascript
// Delegate to parent
const todoList = document.getElementById('todo-list');

todoList.addEventListener('click', function(event) {
  // event.target = the actual element clicked
  // event.currentTarget = the element with the listener (todoList)
  
  console.log('Clicked on:', event.target);
  console.log('Listener on:', event.currentTarget);
  
  // Check what was clicked
  if (event.target.classList.contains('delete')) {
    const todoItem = event.target.closest('.todo-item');
    todoItem.remove();
    console.log('Todo deleted');
  }
});

// Add new todo dynamically - delegation still works!
function addTodo(text) {
  const li = document.createElement('li');
  li.className = 'todo-item';
  li.innerHTML = `${text} <button class="delete">Delete</button>`;
  todoList.appendChild(li);
  // No need to attach new listener!
}

addTodo('Task 4');  // Delete button works automatically
```

#### **Matching Target Elements**

```javascript
const container = document.getElementById('container');

container.addEventListener('click', function(event) {
  const target = event.target;
  
  // 1. Using matches() - checks if element matches selector
  if (target.matches('.button')) {
    console.log('Button clicked');
  }
  
  // 2. Using classList.contains() - checks for specific class
  if (target.classList.contains('delete-btn')) {
    console.log('Delete button clicked');
  }
  
  // 3. Using closest() - finds nearest ancestor matching selector
  const listItem = target.closest('.list-item');
  if (listItem) {
    console.log('Clicked inside list item:', listItem);
  }
  
  // 4. Using tagName - check element type
  if (target.tagName === 'BUTTON') {
    console.log('Any button clicked');
  }
  
  // 5. Using dataset - check data attributes
  if (target.dataset.action === 'delete') {
    console.log('Delete action triggered');
  }
  
  // 6. Multiple conditions
  if (target.matches('button.primary') && !target.disabled) {
    console.log('Active primary button clicked');
  }
});
```

#### **Practical Examples**

**1. Todo List with Multiple Actions:**
```javascript
const todoList = document.getElementById('todos');

todoList.addEventListener('click', function(event) {
  const target = event.target;
  const todoItem = target.closest('.todo-item');
  
  if (!todoItem) return;  // Clicked outside todo items
  
  // Handle different button clicks
  if (target.classList.contains('complete-btn')) {
    todoItem.classList.toggle('completed');
    console.log('Todo marked as', todoItem.classList.contains('completed') ? 'complete' : 'incomplete');
  }
  else if (target.classList.contains('delete-btn')) {
    todoItem.remove();
    console.log('Todo deleted');
  }
  else if (target.classList.contains('edit-btn')) {
    const textSpan = todoItem.querySelector('.todo-text');
    const newText = prompt('Edit todo:', textSpan.textContent);
    if (newText) {
      textSpan.textContent = newText;
    }
  }
  else if (target.classList.contains('priority-btn')) {
    todoItem.dataset.priority = todoItem.dataset.priority === 'high' ? 'normal' : 'high';
    console.log('Priority changed to:', todoItem.dataset.priority);
  }
});

// Adding todos dynamically
function addTodo(text) {
  const li = document.createElement('li');
  li.className = 'todo-item';
  li.innerHTML = `
    <span class="todo-text">${text}</span>
    <button class="complete-btn">✓</button>
    <button class="edit-btn">✎</button>
    <button class="priority-btn">!</button>
    <button class="delete-btn">✗</button>
  `;
  todoList.appendChild(li);
}
```

**2. Data Table with Row Actions:**
```javascript
const table = document.querySelector('.data-table');

table.addEventListener('click', function(event) {
  const target = event.target;
  const row = target.closest('tr');
  
  if (!row || row.parentElement.tagName === 'THEAD') return;
  
  // Get row data
  const rowId = row.dataset.id;
  
  if (target.classList.contains('view-btn')) {
    viewDetails(rowId);
  }
  else if (target.classList.contains('edit-btn')) {
    editRow(rowId);
  }
  else if (target.classList.contains('delete-btn')) {
    if (confirm('Delete this row?')) {
      deleteRow(rowId);
      row.remove();
    }
  }
  else if (target.type === 'checkbox') {
    handleRowSelection(row, target.checked);
  }
  else {
    // Clicking anywhere else on row selects it
    selectRow(row);
  }
});

function viewDetails(id) {
  console.log('Viewing details for:', id);
  // Load and display details
}

function editRow(id) {
  console.log('Editing row:', id);
  // Open edit modal
}

function deleteRow(id) {
  console.log('Deleting row:', id);
  // API call to delete
}
```

**3. Navigation Menu with Submenus:**
```javascript
const nav = document.querySelector('.navigation');

nav.addEventListener('click', function(event) {
  const link = event.target.closest('a');
  
  if (!link) return;
  
  // Handle submenu toggles
  if (link.classList.contains('has-submenu')) {
    event.preventDefault();
    const submenu = link.nextElementSibling;
    submenu.classList.toggle('open');
    link.classList.toggle('active');
  }
  // Handle regular navigation
  else if (!link.classList.contains('disabled')) {
    console.log('Navigating to:', link.href);
    // Navigation happens naturally unless prevented
  }
});

// Close submenus when clicking outside
document.addEventListener('click', function(event) {
  if (!nav.contains(event.target)) {
    nav.querySelectorAll('.submenu.open').forEach(submenu => {
      submenu.classList.remove('open');
    });
  }
});
```

**4. Image Gallery with Lightbox:**
```javascript
const gallery = document.querySelector('.gallery');

gallery.addEventListener('click', function(event) {
  const thumbnail = event.target.closest('.thumbnail');
  
  if (thumbnail) {
    event.preventDefault();
    const fullImageUrl = thumbnail.dataset.fullsize;
    openLightbox(fullImageUrl);
  }
});

function openLightbox(imageUrl) {
  const lightbox = document.createElement('div');
  lightbox.className = 'lightbox';
  lightbox.innerHTML = `
    <div class="lightbox-content">
      <img src="${imageUrl}" alt="Full size">
      <button class="close-btn">✕</button>
    </div>
  `;
  
  document.body.appendChild(lightbox);
  
  // Delegate close action
  lightbox.addEventListener('click', function(event) {
    if (event.target === lightbox || event.target.classList.contains('close-btn')) {
      lightbox.remove();
    }
  });
}
```

**5. Form with Dynamic Fields:**
```javascript
const form = document.querySelector('.dynamic-form');

// Delegate all form interactions
form.addEventListener('click', function(event) {
  const target = event.target;
  
  // Add field button
  if (target.classList.contains('add-field')) {
    const fieldGroup = target.closest('.field-group');
    const newField = fieldGroup.cloneNode(true);
    newField.querySelector('input').value = '';
    fieldGroup.parentElement.appendChild(newField);
  }
  
  // Remove field button
  else if (target.classList.contains('remove-field')) {
    const fieldGroup = target.closest('.field-group');
    if (form.querySelectorAll('.field-group').length > 1) {
      fieldGroup.remove();
    }
  }
});

// Delegate validation
form.addEventListener('blur', function(event) {
  const input = event.target;
  
  if (input.tagName === 'INPUT' || input.tagName === 'TEXTAREA') {
    validateField(input);
  }
}, true);  // Use capture for blur events

function validateField(field) {
  if (field.required && !field.value.trim()) {
    field.classList.add('error');
    showError(field, 'This field is required');
  } else {
    field.classList.remove('error');
    clearError(field);
  }
}
```

**6. Card Grid with Filters:**
```javascript
const cardGrid = document.querySelector('.card-grid');
const filterBar = document.querySelector('.filter-bar');

// Delegate card interactions
cardGrid.addEventListener('click', function(event) {
  const card = event.target.closest('.card');
  
  if (!card) return;
  
  if (event.target.classList.contains('like-btn')) {
    card.classList.toggle('liked');
    updateLikeCount(card);
  }
  else if (event.target.classList.contains('share-btn')) {
    shareCard(card.dataset.id);
  }
  else if (event.target.classList.contains('bookmark-btn')) {
    card.classList.toggle('bookmarked');
  }
  else {
    // Clicking card itself opens details
    openCardDetails(card.dataset.id);
  }
});

// Delegate filter clicks
filterBar.addEventListener('click', function(event) {
  const filterBtn = event.target.closest('.filter-btn');
  
  if (filterBtn) {
    // Remove active from all
    filterBar.querySelectorAll('.filter-btn').forEach(btn => {
      btn.classList.remove('active');
    });
    
    // Add active to clicked
    filterBtn.classList.add('active');
    
    // Filter cards
    const category = filterBtn.dataset.category;
    filterCards(category);
  }
});
```

#### **Benefits of Event Delegation**

```javascript
// 1. MEMORY EFFICIENCY
// Without delegation: 1000 buttons = 1000 listeners
const buttons = document.querySelectorAll('.button');  // 1000 buttons
buttons.forEach(button => {
  button.addEventListener('click', handler);  // 1000 listeners
});

// With delegation: 1000 buttons = 1 listener
const container = document.querySelector('.container');
container.addEventListener('click', function(event) {
  if (event.target.matches('.button')) {
    handler(event);
  }
});  // Only 1 listener!

// 2. DYNAMIC CONTENT
// Works automatically with elements added later
function addButton(text) {
  const btn = document.createElement('button');
  btn.className = 'button';
  btn.textContent = text;
  container.appendChild(btn);
  // No need to attach listener - delegation handles it!
}

// 3. SIMPLIFIED CODE
// Single handler instead of multiple identical handlers
// Easier to maintain and update logic

// 4. PERFORMANCE
// Faster initialization (fewer listeners to attach)
// Better memory usage (fewer listener objects)
```

#### **Advanced Techniques**

**1. Event Delegation with Event Types:**
```javascript
const container = document.getElementById('container');

// Delegate multiple event types
['click', 'mouseover', 'mouseout'].forEach(eventType => {
  container.addEventListener(eventType, function(event) {
    const item = event.target.closest('.item');
    
    if (!item) return;
    
    switch(event.type) {
      case 'click':
        console.log('Item clicked');
        break;
      case 'mouseover':
        item.classList.add('hover');
        break;
      case 'mouseout':
        item.classList.remove('hover');
        break;
    }
  });
});
```

**2. Delegation with Custom Events:**
```javascript
const app = document.getElementById('app');

// Listen for custom events via delegation
app.addEventListener('item-selected', function(event) {
  console.log('Item selected:', event.detail);
  updateUI(event.detail);
});

// Child components can dispatch custom events
function selectItem(itemId) {
  const customEvent = new CustomEvent('item-selected', {
    bubbles: true,  // Must bubble for delegation
    detail: { id: itemId, timestamp: Date.now() }
  });
  
  this.dispatchEvent(customEvent);
}
```

**3. Delegation with Namespace Pattern:**
```javascript
const page = {
  init() {
    const container = document.getElementById('main');
    container.addEventListener('click', this.handleClick.bind(this));
  },
  
  handleClick(event) {
    const target = event.target;
    
    // Route to specific handlers
    if (target.matches('[data-action]')) {
      const action = target.dataset.action;
      const handler = this.actions[action];
      
      if (handler) {
        handler.call(this, event, target);
      }
    }
  },
  
  actions: {
    delete(event, target) {
      console.log('Delete action');
      const item = target.closest('.item');
      item.remove();
    },
    
    edit(event, target) {
      console.log('Edit action');
      const item = target.closest('.item');
      this.editItem(item);
    },
    
    share(event, target) {
      console.log('Share action');
      this.shareItem(target.dataset.id);
    }
  }
};

page.init();
```

**4. Preventing Delegation in Specific Cases:**
```javascript
container.addEventListener('click', function(event) {
  const target = event.target;
  
  // Skip if clicked on disabled elements
  if (target.disabled || target.classList.contains('disabled')) {
    return;
  }
  
  // Skip if in a "no-delegate" zone
  if (target.closest('.no-delegate')) {
    return;
  }
  
  // Process delegation
  if (target.matches('.action-btn')) {
    handleAction(target);
  }
});
```

#### **Common Pitfalls**

```javascript
// 1. Not checking if target exists
container.addEventListener('click', function(event) {
  // ❌ Might be null
  const item = event.target.closest('.item');
  item.classList.add('selected');  // Error if null!
  
  // ✅ Check first
  const itemSafe = event.target.closest('.item');
  if (itemSafe) {
    itemSafe.classList.add('selected');
  }
});

// 2. Using stopPropagation in child (breaks delegation)
button.addEventListener('click', function(event) {
  event.stopPropagation();  // ❌ Prevents delegation from working
});

// 3. Forgetting to check target type
container.addEventListener('click', function(event) {
  // ❌ Assumes all clicks are on buttons
  console.log(event.target.textContent);
  
  // ✅ Check before accessing
  if (event.target.matches('button')) {
    console.log(event.target.textContent);
  }
});

// 4. Attaching to wrong parent
document.body.addEventListener('click', function(event) {
  // ❌ Too broad - handles ALL clicks on page
  if (event.target.matches('.menu-item')) {
    // This will catch ALL .menu-item clicks everywhere
  }
});

// ✅ Attach to specific container
menu.addEventListener('click', function(event) {
  if (event.target.matches('.menu-item')) {
    // Only handles clicks in this menu
  }
});

// 5. Not considering event.target vs event.currentTarget
container.addEventListener('click', function(event) {
  console.log(this);                   // Always container (currentTarget)
  console.log(event.currentTarget);    // Always container
  console.log(event.target);           // Varies (actual clicked element)
});
```

#### **Performance Comparison**

```javascript
// Test with 1000 items

// METHOD 1: Individual listeners
console.time('Individual listeners');
const items1 = document.querySelectorAll('.item-individual');
items1.forEach(item => {
  item.addEventListener('click', function() {
    console.log('Clicked');
  });
});
console.timeEnd('Individual listeners');  // Slower, more memory

// METHOD 2: Event delegation
console.time('Event delegation');
const container = document.querySelector('.container-delegate');
container.addEventListener('click', function(event) {
  if (event.target.classList.contains('item-delegate')) {
    console.log('Clicked');
  }
});
console.timeEnd('Event delegation');  // Faster, less memory

// Memory usage comparison:
// Individual: ~1000 listener objects
// Delegation: 1 listener object
```

#### **When to Use Event Delegation**

**Use Event Delegation when:**
- Working with many similar elements
- Elements are added/removed dynamically
- Want to reduce memory usage
- Elements share similar event handling logic
- Building interactive lists, tables, or grids

**Don't Use Event Delegation when:**
- Only a few static elements
- Each element needs unique complex logic
- Performance of single listener matters
- Events don't bubble (focus, blur - use focusin/focusout instead)

```javascript
// ✅ Good for delegation
<ul id="menu">
  <li><a href="#">Item 1</a></li>
  <li><a href="#">Item 2</a></li>
  <!-- ... 100 more items ... -->
</ul>

menu.addEventListener('click', delegateHandler);

// ❌ Not ideal for delegation (too few elements)
<div id="header">
  <button id="login">Login</button>
  <button id="signup">Signup</button>
</div>

// Just attach directly
document.getElementById('login').addEventListener('click', loginHandler);
document.getElementById('signup').addEventListener('click', signupHandler);
```

#### **Key Takeaways**

- **Single listener on parent** handles events from children
- **Leverages event bubbling** to capture child events
- **Works with dynamic content** added after page load
- **Memory efficient** - fewer listeners needed
- **Use event.target** to identify clicked element
- **Use .closest()** to find ancestor matching selector
- **Check if target exists** before accessing properties
- **Don't stopPropagation** in child elements
- **Perfect for lists, tables, grids** with many items
- **Simplifies code** for similar event handlers
- **Better performance** with large numbers of elements
- **Common pattern** in modern JavaScript applications
- **Essential for SPAs** with dynamic content
- **Works with** click, mouseover, keydown, etc. (bubbling events)
- **Framework-agnostic** - works in vanilla JS and frameworks

---

### 75. What is the difference between event.preventDefault() and event.stopPropagation()?

**Answer:**

`event.preventDefault()` prevents the browser's default action for an event (like form submission or link navigation), while `event.stopPropagation()` stops the event from bubbling up to parent elements. They serve different purposes and can be used independently or together.

#### **event.preventDefault()**

Prevents the browser's default behavior for an event:

```javascript
// 1. Preventing form submission
const form = document.querySelector('form');
form.addEventListener('submit', function(event) {
  event.preventDefault();  // Stops form from submitting
  
  console.log('Form submission prevented');
  // Handle form with JavaScript instead
  const formData = new FormData(form);
  submitViaAjax(formData);
});

// 2. Preventing link navigation
const link = document.querySelector('a');
link.addEventListener('click', function(event) {
  event.preventDefault();  // Stops navigation to href
  
  console.log('Link clicked but not followed');
  // Custom navigation logic
  customNavigate(this.href);
});

// 3. Preventing checkbox toggle
const checkbox = document.querySelector('input[type="checkbox"]');
checkbox.addEventListener('click', function(event) {
  if (!userHasPermission()) {
    event.preventDefault();  // Prevents checking/unchecking
    alert('You do not have permission');
  }
});

// 4. Preventing text selection
const text = document.querySelector('.no-select');
text.addEventListener('mousedown', function(event) {
  event.preventDefault();  // Prevents text selection
});

// 5. Preventing context menu
document.addEventListener('contextmenu', function(event) {
  event.preventDefault();  // Prevents right-click menu
  showCustomContextMenu(event.pageX, event.pageY);
});
```

#### **event.stopPropagation()**

Stops the event from bubbling up to parent elements:

```javascript
// DOM structure
// <div id="outer">
//   <div id="middle">
//     <button id="inner">Click</button>
//   </div>
// </div>

const outer = document.getElementById('outer');
const middle = document.getElementById('middle');
const inner = document.getElementById('inner');

outer.addEventListener('click', () => {
  console.log('Outer clicked');
});

middle.addEventListener('click', () => {
  console.log('Middle clicked');
});

inner.addEventListener('click', (event) => {
  console.log('Inner clicked');
  event.stopPropagation();  // Stops bubbling to middle and outer
});

// Clicking button outputs only:
// "Inner clicked"
// (middle and outer handlers don't fire)
```

#### **Visual Comparison**

```javascript
// WITHOUT stopPropagation
document.getElementById('parent').addEventListener('click', () => {
  console.log('1. Parent');
});

document.getElementById('child').addEventListener('click', () => {
  console.log('2. Child');
});

// Click child, output:
// 2. Child
// 1. Parent (bubbled up)

// WITH stopPropagation
document.getElementById('parent').addEventListener('click', () => {
  console.log('1. Parent');
});

document.getElementById('child').addEventListener('click', (event) => {
  console.log('2. Child');
  event.stopPropagation();  // Stop here
});

// Click child, output:
// 2. Child
// (no "1. Parent" - stopped)
```

#### **Detailed Comparison**

| Feature | preventDefault() | stopPropagation() |
|---------|------------------|-------------------|
| **Purpose** | Prevent default browser action | Stop event bubbling |
| **Affects** | Browser behavior | Event propagation |
| **Examples** | Form submit, link navigation | Parent event handlers |
| **Event flow** | Continues normally | Stops at current element |
| **Target** | Browser default action | Parent/ancestor handlers |
| **Use case** | Custom behavior instead of default | Isolate event handling |
| **Bubbling** | Continues | Stops |
| **Default action** | Prevented | Still happens |
| **Check if used** | `event.defaultPrevented` | No built-in check |

#### **Practical Examples**

**1. Form Validation:**
```javascript
const form = document.querySelector('form');

form.addEventListener('submit', function(event) {
  event.preventDefault();  // Prevent form submission
  
  const email = document.getElementById('email').value;
  
  if (!isValidEmail(email)) {
    showError('Invalid email address');
    return;
  }
  
  // Submit via AJAX
  submitForm(new FormData(form));
});

function isValidEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

**2. Custom Dropdown:**
```javascript
const dropdown = document.querySelector('.dropdown');
const dropdownButton = dropdown.querySelector('.dropdown-button');
const dropdownMenu = dropdown.querySelector('.dropdown-menu');

// Open dropdown - stop propagation to prevent document close handler
dropdownButton.addEventListener('click', function(event) {
  event.stopPropagation();  // Don't trigger document click
  dropdownMenu.classList.toggle('open');
});

// Keep dropdown open when clicking inside
dropdownMenu.addEventListener('click', function(event) {
  event.stopPropagation();  // Don't trigger document click
});

// Close dropdown when clicking outside
document.addEventListener('click', function() {
  dropdownMenu.classList.remove('open');
});

// Prevent default for links in dropdown
dropdownMenu.addEventListener('click', function(event) {
  if (event.target.tagName === 'A') {
    event.preventDefault();  // Prevent navigation
    const action = event.target.dataset.action;
    handleAction(action);
    dropdownMenu.classList.remove('open');
  }
});
```

**3. Drag and Drop:**
```javascript
const draggable = document.querySelector('.draggable');

draggable.addEventListener('dragstart', function(event) {
  event.dataTransfer.setData('text/plain', this.id);
  // No preventDefault here - we want default drag behavior
});

const dropzone = document.querySelector('.dropzone');

dropzone.addEventListener('dragover', function(event) {
  event.preventDefault();  // Required to allow drop
  this.classList.add('drag-over');
});

dropzone.addEventListener('drop', function(event) {
  event.preventDefault();  // Prevent default (open as link)
  event.stopPropagation();  // Don't bubble to parent dropzones
  
  this.classList.remove('drag-over');
  const id = event.dataTransfer.getData('text/plain');
  const element = document.getElementById(id);
  this.appendChild(element);
});
```

**4. Modal Dialog:**
```javascript
const modal = document.querySelector('.modal');
const modalContent = document.querySelector('.modal-content');
const closeButton = document.querySelector('.close-modal');

// Close modal when clicking overlay
modal.addEventListener('click', function(event) {
  // Close if clicked directly on modal (not content)
  if (event.target === modal) {
    closeModal();
  }
});

// Prevent modal content clicks from closing
modalContent.addEventListener('click', function(event) {
  event.stopPropagation();  // Don't bubble to modal overlay
});

// Close button prevents default if it's a link
closeButton.addEventListener('click', function(event) {
  event.preventDefault();  // If it's an <a> tag
  closeModal();
});

function closeModal() {
  modal.classList.remove('open');
}
```

**5. Nested Clickable Elements:**
```javascript
// Card with multiple interactive elements
const card = document.querySelector('.card');
const likeButton = card.querySelector('.like-button');
const shareButton = card.querySelector('.share-button');

// Clicking card opens details
card.addEventListener('click', function() {
  openCardDetails(this.dataset.id);
});

// Like button stops propagation to prevent card click
likeButton.addEventListener('click', function(event) {
  event.stopPropagation();  // Don't open card
  toggleLike(card.dataset.id);
});

// Share button stops propagation and might prevent default
shareButton.addEventListener('click', function(event) {
  event.preventDefault();      // If it's a link
  event.stopPropagation();     // Don't open card
  shareCard(card.dataset.id);
});
```

**6. Custom Context Menu:**
```javascript
document.addEventListener('contextmenu', function(event) {
  event.preventDefault();  // Prevent browser context menu
  
  showCustomContextMenu(event.pageX, event.pageY);
});

const customMenu = document.querySelector('.custom-context-menu');

// Prevent menu clicks from closing menu
customMenu.addEventListener('click', function(event) {
  event.stopPropagation();  // Don't trigger document click
  
  // Handle menu item clicks
  if (event.target.classList.contains('menu-item')) {
    const action = event.target.dataset.action;
    handleContextAction(action);
    hideCustomContextMenu();
  }
});

// Close menu when clicking elsewhere
document.addEventListener('click', function() {
  hideCustomContextMenu();
});
```

#### **Using Both Together**

```javascript
// Common pattern: prevent default AND stop propagation
const link = document.querySelector('.special-link');

link.addEventListener('click', function(event) {
  event.preventDefault();      // Don't follow link
  event.stopPropagation();     // Don't trigger parent handlers
  
  // Custom handling
  console.log('Special link clicked');
  customAction();
});

// Example: Button inside clickable card
const cardContainer = document.querySelector('.card-container');
const actionButton = document.querySelector('.action-button');

cardContainer.addEventListener('click', function() {
  console.log('Card clicked');
});

actionButton.addEventListener('click', function(event) {
  event.preventDefault();      // If it's a link/submit button
  event.stopPropagation();     // Don't trigger card click
  
  performAction();
});
```

#### **Checking if Default was Prevented**

```javascript
document.addEventListener('click', function(event) {
  if (event.defaultPrevented) {
    console.log('Default action was prevented');
    // React accordingly
  }
});

const link = document.querySelector('a');
link.addEventListener('click', function(event) {
  event.preventDefault();
});

link.addEventListener('click', function(event) {
  console.log(event.defaultPrevented);  // true
});
```

#### **stopImmediatePropagation()**

Stops propagation AND prevents other handlers on same element:

```javascript
const button = document.querySelector('button');

button.addEventListener('click', function(event) {
  console.log('First handler');
  event.stopImmediatePropagation();  // Stop everything
});

button.addEventListener('click', function() {
  console.log('Second handler - will NOT run');
});

// Parent won't receive event either
document.body.addEventListener('click', function() {
  console.log('Body handler - will NOT run');
});

// Only outputs: "First handler"
```

#### **Common Mistakes**

```javascript
// 1. Using stopPropagation when not needed
button.addEventListener('click', function(event) {
  event.stopPropagation();  // ❌ Might break event delegation
  // Only use if you specifically need to prevent bubbling
});

// 2. Forgetting to check if preventDefault is needed
link.addEventListener('click', function(event) {
  // ❌ Doesn't prevent navigation
  console.log('Link clicked');
  customAction();
});

// ✅ Prevent navigation
link.addEventListener('click', function(event) {
  event.preventDefault();
  console.log('Link clicked');
  customAction();
});

// 3. Calling on wrong event object
function handleClick(e) {
  // ❌ Wrong variable
  event.preventDefault();
  
  // ✅ Correct
  e.preventDefault();
}

// 4. Trying to prevent non-cancelable events
window.addEventListener('scroll', function(event) {
  event.preventDefault();  // ❌ Does nothing - scroll isn't cancelable
});

// 5. Not understanding they're independent
form.addEventListener('submit', function(event) {
  event.stopPropagation();  // ❌ Doesn't prevent form submission!
  // Form still submits because default wasn't prevented
});

// ✅ Correct
form.addEventListener('submit', function(event) {
  event.preventDefault();  // Prevents submission
});
```

#### **Key Takeaways**

- **preventDefault()** stops browser's default action
- **stopPropagation()** stops event bubbling to parents
- **Different purposes** - can be used independently
- **Often used together** in interactive elements
- **preventDefault()** affects browser behavior
- **stopPropagation()** affects event flow
- **Check defaultPrevented** property to see if preventDefault was called
- **stopImmediatePropagation()** stops everything
- **Don't overuse stopPropagation** - can break event delegation
- **Always use preventDefault** to prevent default actions
- **Use stopPropagation** only when needed
- **Both are methods** on the event object
- **Neither affects** the other's functionality
- Understanding both is **essential** for event handling
- **Common in** forms, links, modals, dropdowns

---

### 76. What is the difference between innerHTML and textContent?

**Answer:**

`innerHTML` gets or sets the HTML markup inside an element, parsing HTML tags and rendering them, while `textContent` gets or sets only the text content, treating everything as plain text. `innerHTML` can execute scripts and is vulnerable to XSS attacks if not sanitized, whereas `textContent` is safe from XSS.

#### **Basic Differences**

```javascript
const div = document.querySelector('div');

// innerHTML - parses HTML tags
div.innerHTML = '<strong>Bold</strong> text';
// Result: Bold text (strong tag is rendered)
console.log(div.innerHTML);  // "<strong>Bold</strong> text"

// textContent - treats everything as plain text
div.textContent = '<strong>Bold</strong> text';
// Result: <strong>Bold</strong> text (tags shown as text)
console.log(div.textContent);  // "<strong>Bold</strong> text"
```

#### **Getting Content**

```html
<div id="example">
  <p>Hello <strong>World</strong></p>
  <span style="display: none;">Hidden</span>
</div>
```

```javascript
const example = document.getElementById('example');

// innerHTML - returns HTML markup
console.log(example.innerHTML);
// Output: "\n  <p>Hello <strong>World</strong></p>\n  <span style="display: none;">Hidden</span>\n"

// textContent - returns all text (including hidden)
console.log(example.textContent);
// Output: "Hello World Hidden" (including hidden span)

// innerText - returns visible text only (respects CSS)
console.log(example.innerText);
// Output: "Hello World" (hidden span excluded)
```

#### **Setting Content**

```javascript
const container = document.getElementById('container');

// 1. innerHTML - creates DOM elements
container.innerHTML = `
  <h2>Title</h2>
  <p>Paragraph text</p>
  <button>Click Me</button>
`;
// Creates actual HTML elements

// 2. textContent - escapes HTML
container.textContent = `
  <h2>Title</h2>
  <p>Paragraph text</p>
`;
// Shows literal text: "<h2>Title</h2><p>Paragraph text</p>"

// 3. Appending vs replacing
container.innerHTML += '<p>Added paragraph</p>';  // Adds to existing
container.textContent += 'Added text';  // Replaces all with text
```

#### **Security: XSS Vulnerability**

```javascript
// ❌ DANGEROUS with innerHTML (XSS vulnerability)
const userInput = '<img src=x onerror="alert(\'XSS Attack!\')">';
div.innerHTML = userInput;  // Script executes! ⚠️

// ✅ SAFE with textContent
div.textContent = userInput;
// Shows literal text: <img src=x onerror="alert('XSS Attack!')">

// ❌ DANGEROUS: User-generated content with innerHTML
function displayComment(comment) {
  const commentDiv = document.getElementById('comments');
  commentDiv.innerHTML = comment;  // ⚠️ User can inject scripts!
}

displayComment('<script>stealCookies()</script>');  // Executes!

// ✅ SAFE: User-generated content with textContent
function displayCommentSafe(comment) {
  const commentDiv = document.getElementById('comments');
  commentDiv.textContent = comment;  // Safe - no script execution
}

displayCommentSafe('<script>stealCookies()</script>');  // Shows as text

// ✅ SAFE: Sanitize if you must use innerHTML
function displayCommentSanitized(comment) {
  const commentDiv = document.getElementById('comments');
  commentDiv.innerHTML = DOMPurify.sanitize(comment);  // Use library
}
```

#### **Performance Comparison**

```javascript
const container = document.getElementById('container');

// innerHTML - slower (parses HTML, creates DOM)
console.time('innerHTML');
container.innerHTML = 'Simple text';
console.timeEnd('innerHTML');  // Slower

// textContent - faster (just sets text)
console.time('textContent');
container.textContent = 'Simple text';
console.timeEnd('textContent');  // Faster

// Performance difference with large content
const largeText = 'Text '.repeat(1000);

console.time('innerHTML large');
container.innerHTML = largeText;
console.timeEnd('innerHTML large');  // Much slower

console.time('textContent large');
container.textContent = largeText;
console.timeEnd('textContent large');  // Much faster
```

#### **Detailed Comparison Table**

| Feature | innerHTML | textContent | innerText |
|---------|-----------|-------------|-----------|
| **Content type** | HTML markup | Plain text | Visible text |
| **HTML parsing** | Yes | No | No |
| **Script execution** | Yes ⚠️ | No ✅ | No ✅ |
| **XSS risk** | High ⚠️ | None ✅ | None ✅ |
| **Performance** | Slower | Faster | Slower (triggers reflow) |
| **Hidden elements** | Included | Included | Excluded (respects CSS) |
| **Whitespace** | Preserved (in HTML) | All included | Collapsed |
| **Returns** | HTML string | All text | Rendered text |
| **Sets** | Parses HTML | Escapes HTML | Escapes HTML |
| **Use case** | Rendering HTML | Safe text display | User-visible text |
| **Browser support** | All | All | All (IE9+) |

#### **Practical Examples**

**1. Displaying User-Generated Content:**
```javascript
// ❌ NEVER do this with user input
function displayUserComment(comment) {
  document.getElementById('comment').innerHTML = comment;
  // User can inject: <img src=x onerror="alert('XSS')">
}

// ✅ SAFE: Use textContent
function displayUserCommentSafe(comment) {
  document.getElementById('comment').textContent = comment;
  // All HTML is escaped and shown as plain text
}

// ✅ SAFE: Create elements programmatically
function displayUserCommentBest(comment) {
  const commentDiv = document.getElementById('comment');
  commentDiv.textContent = '';  // Clear
  
  const p = document.createElement('p');
  p.textContent = comment;  // Safe
  commentDiv.appendChild(p);
}
```

**2. Template Rendering:**
```javascript
// innerHTML for trusted templates
function renderProduct(product) {
  const container = document.getElementById('product');
  
  // ✅ Safe: trusted template
  container.innerHTML = `
    <div class="product-card">
      <img src="${escapeHtml(product.image)}" alt="Product">
      <h3>${escapeHtml(product.name)}</h3>
      <p>${escapeHtml(product.description)}</p>
      <span class="price">$${product.price}</span>
    </div>
  `;
}

// Helper to escape HTML in data
function escapeHtml(text) {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
}
```

**3. Clearing Content:**
```javascript
const container = document.getElementById('container');

// Both work for clearing
container.innerHTML = '';   // Clears all HTML
container.textContent = '';  // Clears all text (slightly faster)

// Performance comparison for clearing
console.time('Clear with innerHTML');
container.innerHTML = '';
console.timeEnd('Clear with innerHTML');

console.time('Clear with textContent');
container.textContent = '';
console.timeEnd('Clear with textContent');  // Usually faster
```

**4. Extracting Text:**
```javascript
const article = document.querySelector('article');

// Get all text including hidden elements
const allText = article.textContent;
console.log(allText);  // Includes hidden content

// Get only visible text
const visibleText = article.innerText;
console.log(visibleText);  // Only visible content

// Get HTML structure
const htmlContent = article.innerHTML;
console.log(htmlContent);  // HTML markup
```

**5. Building Lists:**
```javascript
const list = document.getElementById('list');

// ❌ Inefficient with innerHTML (recreates all elements)
function addItemsInefficient(items) {
  items.forEach(item => {
    list.innerHTML += `<li>${item}</li>`;  // Recreates entire list each time!
  });
}

// ✅ Better: Build string then set once
function addItemsBetter(items) {
  const html = items.map(item => `<li>${escapeHtml(item)}</li>`).join('');
  list.innerHTML = html;  // Set once
}

// ✅ Best: Use textContent with createElement
function addItemsBest(items) {
  const fragment = document.createDocumentFragment();
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item;  // Safe from XSS
    fragment.appendChild(li);
  });
  list.appendChild(fragment);
}
```

**6. Form Feedback:**
```javascript
const errorMessage = document.getElementById('error');

// Safe to use textContent for simple messages
function showError(message) {
  errorMessage.textContent = message;
  errorMessage.classList.add('visible');
}

// If you need formatting, use innerHTML with caution
function showFormattedError(message) {
  // ✅ Safe: trusted content only
  errorMessage.innerHTML = `
    <span class="icon">⚠️</span>
    <span class="message">${escapeHtml(message)}</span>
  `;
  errorMessage.classList.add('visible');
}
```

#### **innerText vs textContent**

```html
<div id="example">
  Hello <span style="display: none;">Hidden</span> World
  <style>body { color: black; }</style>
</div>
```

```javascript
const example = document.getElementById('example');

// textContent - returns ALL text (including hidden and <style>)
console.log(example.textContent);
// "Hello Hidden World body { color: black; }"

// innerText - returns only VISIBLE text
console.log(example.innerText);
// "Hello World"

// innerText is slower (triggers reflow to check visibility)
console.time('textContent');
for (let i = 0; i < 1000; i++) {
  const text = example.textContent;
}
console.timeEnd('textContent');  // Faster

console.time('innerText');
for (let i = 0; i < 1000; i++) {
  const text = example.innerText;
}
console.timeEnd('innerText');  // Slower
```

#### **When to Use Each**

**Use innerHTML when:**
- Rendering trusted HTML templates
- Need to create multiple elements at once
- Working with HTML from your own code
- Building complex DOM structures
- Content is sanitized or from trusted source

**Use textContent when:**
- Displaying user-generated content
- Security is a concern
- Only need plain text
- Performance matters
- Want to prevent any HTML rendering

**Use innerText when:**
- Need only visible text (respect CSS)
- Extracting display text
- Don't need hidden content
- Matching user perception of content

```javascript
// ✅ innerHTML: Trusted template
productDiv.innerHTML = `<h3>${safeName}</h3><p>${safeDesc}</p>`;

// ✅ textContent: User input
commentDiv.textContent = userComment;  // Safe from XSS

// ✅ innerText: Copy visible text
copyButton.addEventListener('click', () => {
  navigator.clipboard.writeText(article.innerText);
});
```

#### **Common Pitfalls**

```javascript
// 1. Using innerHTML with user input
// ❌ DANGEROUS
userDiv.innerHTML = userInput;  // XSS vulnerability!

// 2. Slow innerHTML appending
// ❌ Inefficient
for (let i = 0; i < 1000; i++) {
  list.innerHTML += `<li>Item ${i}</li>`;  // Recreates DOM 1000 times!
}

// 3. Not escaping data in templates
// ❌ VULNERABLE
div.innerHTML = `<p>${userData}</p>`;  // userData could contain scripts

// ✅ Escape user data
div.innerHTML = `<p>${escapeHtml(userData)}</p>`;

// 4. Using innerText when textContent is sufficient
// ❌ Slower
const text = element.innerText;  // Triggers reflow

// ✅ Faster
const text = element.textContent;  // No reflow

// 5. Forgetting innerHTML replaces all content
div.innerHTML = '<p>New content</p>';  // Removes all existing content
```

#### **Key Takeaways**

- **innerHTML** parses HTML, **textContent** treats as plain text
- **textContent** is **safer** - no XSS risk
- **innerHTML** can execute scripts - **never use with user input**
- **textContent** is **faster** for plain text
- **innerText** respects CSS visibility
- **innerHTML** useful for trusted templates
- **Always escape user data** before using in innerHTML
- **textContent** is preferred for security
- **innerHTML** replaces all existing content
- **textContent** includes hidden elements
- **innerText** triggers reflow (slower)
- Use **DOMPurify** or similar for sanitizing innerHTML
- **createElement + textContent** safest for dynamic content
- **Performance**: textContent > innerHTML for plain text
- **Security**: textContent > innerHTML for user content


---

### 77. What is the difference between window and document?

**Answer:**

`window` is the global object representing the browser window and provides access to browser features (timers, location, history, localStorage), while `document` is a property of `window` representing the HTML document and providing access to the DOM tree. The window is the container; the document is the content inside it.

#### **Basic Relationship**

```javascript
// document is a property of window
console.log(window.document === document);  // true

// In global scope, 'document' is actually 'window.document'
console.log(document);         // Document object
console.log(window.document);  // Same Document object

// window is the global object
console.log(window.window === window);  // true
console.log(this === window);           // true (in global scope, non-strict)

// Global variables become properties of window
var globalVar = 'test';
console.log(window.globalVar);  // 'test'

// But document is not global in the same way
var document = 'test';  // ❌ Bad idea - shadows window.document
```

#### **Hierarchy**

```javascript
// Browser Object Model (BOM) hierarchy
// window (Browser Window)
//   ├─ document (HTML Document)
//   ├─ navigator (Browser info)
//   ├─ location (URL info)
//   ├─ history (Browser history)
//   ├─ screen (Screen properties)
//   ├─ localStorage (Storage)
//   └─ sessionStorage (Storage)

// document is part of window
console.log(window.document);        // Document
console.log(window.navigator);       // Navigator
console.log(window.location);        // Location

// document contains the DOM
console.log(document.documentElement);  // <html>
console.log(document.body);             // <body>
console.log(document.head);             // <head>
```

#### **Window Object Properties and Methods**

```javascript
// BROWSER WINDOW PROPERTIES
console.log(window.innerWidth);    // Viewport width (px)
console.log(window.innerHeight);   // Viewport height (px)
console.log(window.outerWidth);    // Browser window width
console.log(window.outerHeight);   // Browser window height

// BROWSER INFORMATION
console.log(window.navigator.userAgent);   // Browser/OS info
console.log(window.navigator.language);    // 'en-US'
console.log(window.navigator.onLine);      // true/false

// URL AND NAVIGATION
console.log(window.location.href);         // Full URL
console.log(window.location.hostname);     // 'example.com'
console.log(window.location.pathname);     // '/path/to/page'
window.location.href = 'https://google.com';  // Navigate

// HISTORY
window.history.back();                     // Go back
window.history.forward();                  // Go forward
window.history.go(-2);                     // Go back 2 pages

// SCREEN
console.log(window.screen.width);          // Screen width
console.log(window.screen.height);         // Screen height
console.log(window.screen.availWidth);     // Available width
console.log(window.screen.colorDepth);     // Color depth

// STORAGE
window.localStorage.setItem('key', 'value');
window.sessionStorage.setItem('key', 'value');

// TIMERS
const timeoutId = window.setTimeout(() => {
  console.log('Delayed');
}, 1000);

const intervalId = window.setInterval(() => {
  console.log('Repeating');
}, 1000);

window.clearTimeout(timeoutId);
window.clearInterval(intervalId);

// DIALOGS
window.alert('Alert message');
const confirmed = window.confirm('Are you sure?');
const userInput = window.prompt('Enter name:', 'Default');

// WINDOW CONTROL
window.open('https://google.com', '_blank');  // Open new window
window.close();                                // Close window
window.focus();                                // Focus window
window.blur();                                 // Unfocus window

// SCROLLING
window.scrollTo(0, 500);                       // Scroll to position
window.scrollBy(0, 100);                       // Scroll by amount
window.scroll(0, 0);                           // Scroll to top

// REQUEST ANIMATION FRAME
window.requestAnimationFrame(callback);
window.cancelAnimationFrame(id);

// FETCH API
window.fetch('https://api.example.com/data')
  .then(response => response.json())
  .then(data => console.log(data));
```

#### **Document Object Properties and Methods**

```javascript
// DOCUMENT PROPERTIES
console.log(document.title);              // Page title
console.log(document.URL);                // Document URL
console.log(document.domain);             // Domain name
console.log(document.referrer);           // Referring URL
console.log(document.lastModified);       // Last modified date
console.log(document.readyState);         // 'loading', 'interactive', 'complete'
console.log(document.characterSet);       // 'UTF-8'

// DOM TREE ACCESS
console.log(document.documentElement);    // <html> element
console.log(document.head);               // <head> element
console.log(document.body);               // <body> element

// SELECTING ELEMENTS
const el1 = document.getElementById('id');
const el2 = document.querySelector('.class');
const els = document.querySelectorAll('div');
const byClass = document.getElementsByClassName('class');
const byTag = document.getElementsByTagName('p');

// CREATING ELEMENTS
const div = document.createElement('div');
const text = document.createTextNode('Text');
const fragment = document.createDocumentFragment();
const comment = document.createComment('Comment');

// DOCUMENT MANIPULATION
document.title = 'New Title';             // Change title
document.body.style.background = 'blue';  // Style body

// FORMS
console.log(document.forms);              // All forms
console.log(document.images);             // All images
console.log(document.links);              // All links
console.log(document.scripts);            // All scripts

// DOCUMENT EVENTS
document.addEventListener('DOMContentLoaded', () => {
  console.log('DOM ready');
});

document.addEventListener('click', (e) => {
  console.log('Document clicked');
});

// COOKIES
document.cookie = 'name=value';
console.log(document.cookie);             // All cookies as string

// ACTIVE ELEMENT
console.log(document.activeElement);      // Currently focused element
```

#### **Comparison Table**

| Feature | window | document |
|---------|--------|----------|
| **Type** | Global object (BOM) | DOM object |
| **Represents** | Browser window | HTML document |
| **Scope** | Entire browser window | Document content only |
| **Parent** | Top-level global object | Property of window |
| **Access** | `window` or implicit | `document` or `window.document` |
| **Purpose** | Browser features/control | DOM manipulation |
| **Properties** | innerWidth, location, history | body, title, forms |
| **Methods** | setTimeout, alert, fetch | getElementById, createElement |
| **Events** | resize, scroll, load | DOMContentLoaded, click |
| **Global vars** | Attached here | Not attached |
| **Exists when** | Browser window open | Document loaded |

#### **Practical Examples**

**1. Window Events vs Document Events:**
```javascript
// Window events (browser-level)
window.addEventListener('resize', () => {
  console.log('Window resized:', window.innerWidth);
});

window.addEventListener('scroll', () => {
  console.log('Window scrolled:', window.scrollY);
});

window.addEventListener('load', () => {
  console.log('Everything loaded (images, CSS, etc.)');
});

// Document events (content-level)
document.addEventListener('DOMContentLoaded', () => {
  console.log('DOM ready (HTML parsed, scripts executed)');
  // Safe to manipulate DOM here
});

document.addEventListener('click', (e) => {
  console.log('Clicked on:', e.target);
});

document.addEventListener('keydown', (e) => {
  console.log('Key pressed:', e.key);
});
```

**2. Page Information:**
```javascript
// Using window for browser info
function getBrowserInfo() {
  return {
    windowSize: {
      width: window.innerWidth,
      height: window.innerHeight
    },
    screen: {
      width: window.screen.width,
      height: window.screen.height
    },
    url: window.location.href,
    userAgent: window.navigator.userAgent,
    online: window.navigator.onLine
  };
}

// Using document for content info
function getDocumentInfo() {
  return {
    title: document.title,
    url: document.URL,
    domain: document.domain,
    lastModified: document.lastModified,
    readyState: document.readyState,
    elementCount: document.querySelectorAll('*').length,
    images: document.images.length,
    links: document.links.length,
    forms: document.forms.length
  };
}

console.log(getBrowserInfo());
console.log(getDocumentInfo());
```

**3. Responsive Design:**
```javascript
// window for viewport dimensions
function checkViewport() {
  const width = window.innerWidth;
  
  if (width < 768) {
    console.log('Mobile view');
    document.body.classList.add('mobile');
  } else if (width < 1024) {
    console.log('Tablet view');
    document.body.classList.add('tablet');
  } else {
    console.log('Desktop view');
    document.body.classList.add('desktop');
  }
}

window.addEventListener('resize', checkViewport);
checkViewport();  // Initial check

// document for DOM manipulation
function adjustContent() {
  const isMobile = document.body.classList.contains('mobile');
  const nav = document.querySelector('.navigation');
  
  if (isMobile) {
    nav.classList.add('mobile-nav');
  } else {
    nav.classList.remove('mobile-nav');
  }
}
```

**4. Scroll Position:**
```javascript
// window for scroll position
function getScrollPosition() {
  return {
    x: window.scrollX || window.pageXOffset,  // Horizontal scroll
    y: window.scrollY || window.pageYOffset   // Vertical scroll
  };
}

// Scroll to top button
const scrollTopBtn = document.getElementById('scroll-top');

window.addEventListener('scroll', () => {
  const scrollPos = getScrollPosition();
  
  if (scrollPos.y > 300) {
    scrollTopBtn.style.display = 'block';
  } else {
    scrollTopBtn.style.display = 'none';
  }
});

scrollTopBtn.addEventListener('click', () => {
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  });
});
```

**5. Page Navigation:**
```javascript
// window.location for navigation
function navigateTo(url) {
  window.location.href = url;  // Navigate to URL
}

function reloadPage() {
  window.location.reload();    // Reload page
}

function goBack() {
  window.history.back();       // Go back in history
}

// document for internal navigation
function scrollToElement(id) {
  const element = document.getElementById(id);
  if (element) {
    element.scrollIntoView({ behavior: 'smooth' });
  }
}

// Hash navigation
window.addEventListener('hashchange', () => {
  const hash = window.location.hash.slice(1);  // Remove #
  const element = document.getElementById(hash);
  if (element) {
    element.scrollIntoView();
  }
});
```

**6. Fullscreen API:**
```javascript
// window for browser control
function toggleFullscreen() {
  if (!document.fullscreenElement) {
    // Enter fullscreen - use document method
    document.documentElement.requestFullscreen();
  } else {
    // Exit fullscreen - use document method
    document.exitFullscreen();
  }
}

// Listen on document for fullscreen events
document.addEventListener('fullscreenchange', () => {
  if (document.fullscreenElement) {
    console.log('Entered fullscreen');
  } else {
    console.log('Exited fullscreen');
  }
});

const fullscreenBtn = document.getElementById('fullscreen-btn');
fullscreenBtn.addEventListener('click', toggleFullscreen);
```

**7. Storage and Cookies:**
```javascript
// window for storage (persistent)
function saveToStorage(key, value) {
  window.localStorage.setItem(key, value);
}

function getFromStorage(key) {
  return window.localStorage.getItem(key);
}

// document for cookies (sent to server)
function setCookie(name, value, days) {
  const date = new Date();
  date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
  document.cookie = `${name}=${value};expires=${date.toUTCString()};path=/`;
}

function getCookie(name) {
  const cookies = document.cookie.split(';');
  for (let cookie of cookies) {
    const [key, value] = cookie.trim().split('=');
    if (key === name) return value;
  }
  return null;
}

// Usage
saveToStorage('theme', 'dark');
console.log(getFromStorage('theme'));  // 'dark'

setCookie('user', 'john', 7);
console.log(getCookie('user'));        // 'john'
```

**8. Modal with Window Dimensions:**
```javascript
// Use window for sizing, document for DOM
function centerModal() {
  const modal = document.getElementById('modal');
  const modalWidth = modal.offsetWidth;
  const modalHeight = modal.offsetHeight;
  
  // Use window dimensions to center
  const left = (window.innerWidth - modalWidth) / 2;
  const top = (window.innerHeight - modalHeight) / 2;
  
  modal.style.left = left + 'px';
  modal.style.top = top + 'px';
}

function showModal() {
  const modal = document.getElementById('modal');
  modal.style.display = 'block';
  centerModal();
}

// Recenter on window resize
window.addEventListener('resize', () => {
  const modal = document.getElementById('modal');
  if (modal.style.display === 'block') {
    centerModal();
  }
});
```

#### **Global Scope Differences**

```javascript
// window is the global object
var globalVar = 'test';
console.log(window.globalVar);  // 'test'

function globalFunc() {
  return 'function';
}
console.log(window.globalFunc());  // 'function'

// let/const are not added to window
let letVar = 'test';
const constVar = 'test';
console.log(window.letVar);     // undefined
console.log(window.constVar);   // undefined

// document is a property of window, not a scope
document.customProp = 'value';  // Can add properties
console.log(document.customProp);  // 'value'
console.log(window.document.customProp);  // 'value'
```

#### **Timing Differences**

```javascript
// window.onload - fires when everything is loaded
window.onload = function() {
  console.log('Window loaded (images, CSS, scripts)');
  // All resources loaded
};

// Or with addEventListener
window.addEventListener('load', () => {
  console.log('Everything loaded');
});

// document.DOMContentLoaded - fires when DOM is ready
document.addEventListener('DOMContentLoaded', () => {
  console.log('DOM ready (HTML parsed)');
  // DOM can be manipulated, but images may not be loaded
});

// Typical execution order:
// 1. DOMContentLoaded (DOM ready)
// 2. load (everything ready)
```

#### **Common Use Cases**

```javascript
// USE WINDOW FOR:
// - Browser dimensions
const width = window.innerWidth;

// - Timers
window.setTimeout(() => {}, 1000);

// - Navigation
window.location.href = '/page';

// - Browser info
const userAgent = window.navigator.userAgent;

// - Storage
window.localStorage.setItem('key', 'value');

// - Global functions
window.alert('Message');

// - AJAX/Fetch
window.fetch('/api/data');

// USE DOCUMENT FOR:
// - DOM access
const element = document.getElementById('id');

// - DOM creation
const div = document.createElement('div');

// - Document info
const title = document.title;

// - Cookies
document.cookie = 'name=value';

// - Active element
const focused = document.activeElement;

// - Forms/images
const forms = document.forms;

// - Document events
document.addEventListener('click', handler);
```

#### **Common Mistakes**

```javascript
// 1. Confusing window.onload with DOMContentLoaded
// ❌ Waiting too long
window.onload = function() {
  // Waits for all images/resources
  const element = document.getElementById('id');
};

// ✅ Use DOMContentLoaded for DOM manipulation
document.addEventListener('DOMContentLoaded', () => {
  // DOM ready, can manipulate immediately
  const element = document.getElementById('id');
});

// 2. Accessing document before it's ready
console.log(document.body);  // null if script in <head>

// ✅ Wait for DOM
document.addEventListener('DOMContentLoaded', () => {
  console.log(document.body);  // Now available
});

// 3. Using window methods on document
// ❌ Wrong
document.setTimeout(() => {}, 1000);  // TypeError

// ✅ Correct
window.setTimeout(() => {}, 1000);
setTimeout(() => {}, 1000);  // window is implicit

// 4. Trying to select elements from window
// ❌ Wrong
const el = window.getElementById('id');  // TypeError

// ✅ Correct
const el = document.getElementById('id');

// 5. Confusing window events with document events
// Window resize
window.addEventListener('resize', handler);  // ✅
document.addEventListener('resize', handler); // ❌ Won't work

// Document clicks
document.addEventListener('click', handler);  // ✅
window.addEventListener('click', handler);    // ✅ Also works (bubbles up)
```

#### **Modern API Considerations**

```javascript
// Some APIs are on window
console.log(window.fetch);            // Function
console.log(window.localStorage);     // Storage
console.log(window.sessionStorage);   // Storage
console.log(window.requestAnimationFrame);  // Function

// Some are on document
console.log(document.getElementById);      // Function
console.log(document.querySelector);       // Function
console.log(document.createElement);       // Function
console.log(document.cookie);             // String

// Some are on both (but behave differently)
window.addEventListener('load', handler);      // Window load
document.addEventListener('DOMContentLoaded', handler);  // DOM ready

// Console is typically on window
console.log(window.console === console);  // true
```

#### **Key Takeaways**

- **window** is the global browser object (BOM)
- **document** is a property of window (DOM)
- **window** represents the browser window itself
- **document** represents the HTML document content
- **Use window** for browser features (timers, navigation, storage)
- **Use document** for DOM manipulation (elements, content)
- **window is global** - all global variables attached here
- **document is not global** - it's a window property
- **window events**: resize, scroll, load
- **document events**: DOMContentLoaded, click, keydown
- **window.onload** fires after everything loads
- **DOMContentLoaded** fires when DOM is ready
- **document is part of window** (`window.document === document`)
- Both are **essential** for web development
- Understanding the difference enables **proper API usage**

---

### 78. What are different ways to select DOM elements?

**Answer:**

JavaScript provides multiple methods for selecting DOM elements, ranging from traditional methods like `getElementById()` to modern CSS-selector-based methods like `querySelector()` and `querySelectorAll()`. Each method has different use cases, performance characteristics, and return types.

#### **1. getElementById()**

Returns a single element with the specified ID (or null if not found):

```javascript
// HTML: <div id="header">Header</div>
const header = document.getElementById('header');
console.log(header);  // <div id="header">...</div>

// Case-sensitive
const el1 = document.getElementById('Header');  // null (wrong case)
const el2 = document.getElementById('header');  // Found

// Returns null if not found
const notFound = document.getElementById('nonexistent');
console.log(notFound);  // null

// Fastest selection method
// Only works on document, not on elements
const wrong = header.getElementById('other');  // TypeError
```

#### **2. getElementsByClassName()**

Returns a live HTMLCollection of elements with the specified class:

```javascript
// HTML: <div class="card">Card 1</div>
//       <div class="card">Card 2</div>
const cards = document.getElementsByClassName('card');
console.log(cards);  // HTMLCollection(2) [div.card, div.card]

// Access by index
console.log(cards[0]);  // First card
console.log(cards[1]);  // Second card

// Length property
console.log(cards.length);  // 2

// Live collection - updates automatically
console.log(cards.length);  // 2
const newCard = document.createElement('div');
newCard.className = 'card';
document.body.appendChild(newCard);
console.log(cards.length);  // 3 (automatically updated)

// Multiple classes (space-separated)
const special = document.getElementsByClassName('card special');

// Can be called on elements
const container = document.getElementById('container');
const containerCards = container.getElementsByClassName('card');

// Convert to array for array methods
const cardsArray = Array.from(cards);
cardsArray.forEach(card => console.log(card));
```

#### **3. getElementsByTagName()**

Returns a live HTMLCollection of elements with the specified tag name:

```javascript
// Get all paragraphs
const paragraphs = document.getElementsByTagName('p');
console.log(paragraphs);  // HTMLCollection of <p> elements

// Get all divs
const divs = document.getElementsByTagName('div');

// Get all links
const links = document.getElementsByTagName('a');

// Case-insensitive
const P = document.getElementsByTagName('P');  // Works
const p = document.getElementsByTagName('p');  // Same result

// Get all elements with '*'
const allElements = document.getElementsByTagName('*');
console.log(allElements.length);  // Total element count

// Live collection
console.log(divs.length);  // e.g., 5
const newDiv = document.createElement('div');
document.body.appendChild(newDiv);
console.log(divs.length);  // 6 (updated)

// Can be called on elements
const container = document.getElementById('container');
const containerDivs = container.getElementsByTagName('div');
```

#### **4. querySelector()**

Returns the first element matching a CSS selector (or null):

```javascript
// By ID (like getElementById, but with CSS selector)
const header = document.querySelector('#header');

// By class
const card = document.querySelector('.card');  // First .card

// By tag
const paragraph = document.querySelector('p');  // First <p>

// Complex selectors
const firstLink = document.querySelector('div a');  // First <a> inside <div>
const navLink = document.querySelector('.nav > a');  // Direct child
const activeItem = document.querySelector('.item.active');  // Multiple classes
const emailInput = document.querySelector('input[type="email"]');  // Attribute

// Pseudo-classes
const firstChild = document.querySelector('li:first-child');
const lastChild = document.querySelector('li:last-child');
const nthChild = document.querySelector('li:nth-child(3)');
const checked = document.querySelector('input:checked');
const disabled = document.querySelector('button:disabled');

// Returns null if not found
const notFound = document.querySelector('.nonexistent');
console.log(notFound);  // null

// Can be called on elements (scoped selection)
const container = document.querySelector('.container');
const cardInContainer = container.querySelector('.card');

// Combining selectors
const complex = document.querySelector('div.card:not(.disabled) > h2');
```

#### **5. querySelectorAll()**

Returns a static NodeList of all matching elements:

```javascript
// Get all elements with a class
const cards = document.querySelectorAll('.card');
console.log(cards);  // NodeList(2) [div.card, div.card]

// Access by index
console.log(cards[0]);
console.log(cards[1]);

// Length
console.log(cards.length);

// Static collection - does NOT update
console.log(cards.length);  // 2
const newCard = document.createElement('div');
newCard.className = 'card';
document.body.appendChild(newCard);
console.log(cards.length);  // Still 2 (not updated)

// forEach (NodeList has forEach)
cards.forEach(card => {
  console.log(card.textContent);
});

// Array methods (convert first)
const cardsArray = Array.from(cards);
const mapped = cardsArray.map(card => card.textContent);

// Or spread operator
const cardsArray2 = [...cards];

// Complex selectors
const links = document.querySelectorAll('nav a');
const activeItems = document.querySelectorAll('.item.active');
const inputs = document.querySelectorAll('input[type="text"]');
const visibleCards = document.querySelectorAll('.card:not(.hidden)');

// Returns empty NodeList if none found
const none = document.querySelectorAll('.nonexistent');
console.log(none);  // NodeList(0) []
console.log(none.length);  // 0

// Can be called on elements
const container = document.querySelector('.container');
const cardsInContainer = container.querySelectorAll('.card');
```

#### **6. getElementsByName()**

Returns a live NodeList of elements with the specified name attribute:

```javascript
// HTML: <input name="email" />
//       <input name="email" />
const emailInputs = document.getElementsByName('email');
console.log(emailInputs);  // NodeList(2)

// Commonly used for radio buttons
// HTML: <input type="radio" name="gender" value="male">
//       <input type="radio" name="gender" value="female">
const genderRadios = document.getElementsByName('gender');

// Find checked radio
const checked = Array.from(genderRadios).find(radio => radio.checked);
console.log(checked?.value);  // 'male' or 'female'

// Live NodeList
console.log(genderRadios.length);  // 2
```

#### **7. Special Selectors**

```javascript
// document.body - <body> element
const body = document.body;

// document.head - <head> element
const head = document.head;

// document.documentElement - <html> element
const html = document.documentElement;

// document.forms - All forms (HTMLCollection)
const forms = document.forms;
console.log(forms[0]);        // First form
console.log(forms['myForm']); // Form with name="myForm"

// document.images - All images
const images = document.images;

// document.links - All <a> with href
const links = document.links;

// document.scripts - All scripts
const scripts = document.scripts;

// document.activeElement - Currently focused element
const focused = document.activeElement;
console.log(focused.tagName);  // e.g., 'INPUT'
```

#### **8. Modern Methods**

```javascript
// closest() - find nearest ancestor matching selector
const button = document.querySelector('.delete-btn');
const card = button.closest('.card');  // Nearest .card ancestor
const container = button.closest('#container');

// matches() - check if element matches selector
const element = document.querySelector('.item');
console.log(element.matches('.item'));      // true
console.log(element.matches('.active'));    // false

if (element.matches('.active')) {
  console.log('Element is active');
}

// Element relationships
const parent = element.parentElement;           // Parent element
const children = element.children;              // Child elements (HTMLCollection)
const firstChild = element.firstElementChild;   // First child element
const lastChild = element.lastElementChild;     // Last child element
const nextSibling = element.nextElementSibling; // Next sibling
const prevSibling = element.previousElementSibling;  // Previous sibling
```

#### **Comparison Table**

| Method | Returns | Live/Static | CSS Selectors | Performance | Use Case |
|--------|---------|-------------|---------------|-------------|----------|
| **getElementById** | Element or null | N/A | No | Fastest | Single element by ID |
| **getElementsByClassName** | HTMLCollection | Live | No | Fast | Multiple by class |
| **getElementsByTagName** | HTMLCollection | Live | No | Fast | Multiple by tag |
| **querySelector** | Element or null | N/A | Yes | Moderate | First match, complex selector |
| **querySelectorAll** | NodeList | Static | Yes | Slower | All matches, complex selector |
| **getElementsByName** | NodeList | Live | No | Fast | By name attribute |

#### **Practical Examples**

**1. Form Validation:**
```javascript
// Select form and inputs
const form = document.querySelector('#registration-form');
const emailInput = form.querySelector('input[name="email"]');
const passwordInput = form.querySelector('input[type="password"]');
const submitButton = form.querySelector('button[type="submit"]');

// All required fields
const requiredFields = form.querySelectorAll('[required]');

requiredFields.forEach(field => {
  field.addEventListener('blur', () => {
    if (!field.value.trim()) {
      field.classList.add('error');
    }
  });
});
```

**2. Navigation Menu:**
```javascript
// Select navigation elements
const nav = document.querySelector('.navigation');
const navLinks = nav.querySelectorAll('a');
const activeLink = nav.querySelector('a.active');
const dropdowns = nav.querySelectorAll('.has-dropdown');

// Add event listeners
navLinks.forEach(link => {
  link.addEventListener('click', function(e) {
    // Remove active from all
    navLinks.forEach(l => l.classList.remove('active'));
    // Add to clicked
    this.classList.add('active');
  });
});
```

**3. Image Gallery:**
```javascript
// Select gallery elements
const gallery = document.getElementById('gallery');
const thumbnails = gallery.querySelectorAll('.thumbnail');
const lightbox = document.querySelector('.lightbox');
const lightboxImage = lightbox.querySelector('img');
const closeButton = lightbox.querySelector('.close');

thumbnails.forEach(thumb => {
  thumb.addEventListener('click', () => {
    const fullSrc = thumb.dataset.fullsize;
    lightboxImage.src = fullSrc;
    lightbox.classList.add('open');
  });
});
```

**4. Todo List:**
```javascript
// Select todo elements
const todoList = document.getElementById('todo-list');
const todoItems = todoList.querySelectorAll('.todo-item');
const addButton = document.querySelector('#add-todo');
const inputField = document.querySelector('#todo-input');

// Select by state
const completedTodos = todoList.querySelectorAll('.todo-item.completed');
const activeTodos = todoList.querySelectorAll('.todo-item:not(.completed)');

console.log(`Active: ${activeTodos.length}, Completed: ${completedTodos.length}`);
```

**5. Table Operations:**
```javascript
// Select table elements
const table = document.querySelector('.data-table');
const headers = table.querySelectorAll('thead th');
const rows = table.querySelectorAll('tbody tr');
const cells = table.querySelectorAll('tbody td');

// Select specific rows
const evenRows = table.querySelectorAll('tbody tr:nth-child(even)');
const oddRows = table.querySelectorAll('tbody tr:nth-child(odd)');

// Add zebra striping
evenRows.forEach(row => row.classList.add('even'));
oddRows.forEach(row => row.classList.add('odd'));

// Select by data attribute
const highPriorityRows = table.querySelectorAll('tr[data-priority="high"]');
```

**6. Dynamic Content:**
```javascript
// Select container
const container = document.getElementById('content');

// Add content dynamically
function addCard(title, content) {
  const card = document.createElement('div');
  card.className = 'card';
  card.innerHTML = `<h3>${title}</h3><p>${content}</p>`;
  container.appendChild(card);
}

addCard('Card 1', 'Content 1');
addCard('Card 2', 'Content 2');

// Select newly added cards
const cards = container.querySelectorAll('.card');  // Static snapshot
const cardsLive = container.getElementsByClassName('card');  // Live collection

console.log(cards.length);      // 2
console.log(cardsLive.length);  // 2

addCard('Card 3', 'Content 3');

console.log(cards.length);      // Still 2 (static)
console.log(cardsLive.length);  // 3 (live, updated)
```

#### **Performance Considerations**

```javascript
// FASTEST: getElementById
const el1 = document.getElementById('header');

// FAST: getElementsByClassName, getElementsByTagName
const els2 = document.getElementsByClassName('card');
const els3 = document.getElementsByTagName('div');

// MODERATE: querySelector (stops at first match)
const el4 = document.querySelector('.card');

// SLOWER: querySelectorAll (checks all elements)
const els5 = document.querySelectorAll('.card');

// For multiple selections with simple selectors, prefer:
// getElementsByClassName over querySelectorAll
const cards1 = document.getElementsByClassName('card');  // Faster
const cards2 = document.querySelectorAll('.card');       // Slower

// But querySelector* allows complex selectors:
const complex = document.querySelectorAll('div.card:not(.disabled) > h2');
// This flexibility often outweighs performance difference
```

#### **Best Practices**

```javascript
// 1. Use getElementById for unique IDs
const header = document.getElementById('header');  // ✅ Fastest

// 2. Use querySelector for complex selectors
const firstActiveCard = document.querySelector('.card.active');  // ✅

// 3. Use querySelectorAll for multiple matches
const allCards = document.querySelectorAll('.card');  // ✅

// 4. Cache selectors
// ❌ Bad: Selecting multiple times
for (let i = 0; i < 10; i++) {
  const cards = document.querySelectorAll('.card');  // Inefficient
}

// ✅ Good: Select once, reuse
const cards = document.querySelectorAll('.card');
for (let i = 0; i < 10; i++) {
  // Use cards
}

// 5. Scope selections to containers
// ❌ Less efficient
const buttons = document.querySelectorAll('.modal .button');

// ✅ More efficient
const modal = document.querySelector('.modal');
const buttons = modal.querySelectorAll('.button');

// 6. Convert HTMLCollection/NodeList for array methods
const cards = document.getElementsByClassName('card');
const cardsArray = Array.from(cards);
cardsArray.map(card => card.textContent);

// Or spread operator
const cardsArray2 = [...cards];
```

#### **Key Takeaways**

- **getElementById** is fastest for single element by ID
- **querySelector** uses CSS selectors, very flexible
- **querySelectorAll** returns static NodeList
- **getElementsByClassName/TagName** return live HTMLCollection
- **Live collections** update automatically
- **Static collections** are snapshots
- **querySelector** stops at first match (faster)
- **querySelectorAll** finds all matches
- **Cache selectors** for better performance
- **Scope selections** to containers when possible
- **Modern preference**: querySelector/querySelectorAll for flexibility
- **Use getElementById** when selecting by ID only
- **Convert collections** to arrays for array methods
- **NodeList has forEach**, HTMLCollection doesn't
- Understanding **return types** is crucial

