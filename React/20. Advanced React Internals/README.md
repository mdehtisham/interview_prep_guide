# Advanced React Internals

> Expert / Architect Level (5+ years)

---

## Questions

191. How does React Fiber architecture work?
192. What is the difference between Stack and Fiber reconcilers?
193. How does React's scheduling algorithm work?
194. What is time slicing in React?
195. How does React prioritize updates?
196. What is the lane model in React?
197. How does React's commit phase work?
198. What is tearing in concurrent rendering?
199. How does React handle Suspense internally?
200. What is the reconciliation process in detail?

---

## Detailed Answers

### 191. How does React Fiber architecture work?

<details>
<summary>View Answer</summary>

**React Fiber** is a complete rewrite of React's reconciliation engine introduced in React 16. It's the foundation for concurrent features and enables React to pause, abort, or resume work, making rendering interruptible and prioritizable.

#### What is Fiber?

**Fiber = Unit of Work**

A Fiber is a JavaScript object that represents a **unit of work**. Each React element (component) has a corresponding Fiber node that tracks:
- Component type and props
- State and hooks
- Child, sibling, and parent relationships
- Work priority
- Effects to perform

```javascript
// Simplified Fiber structure
const Fiber = {
  // Identity
  type: 'div' | Component,
  key: 'unique-key',
  
  // Relationships (Linked List Structure)
  return: parentFiber,        // Parent
  child: firstChildFiber,     // First child
  sibling: nextSiblingFiber,  // Next sibling
  
  // State
  memoizedState: currentState,
  memoizedProps: currentProps,
  pendingProps: newProps,
  
  // Effects
  flags: UpdateEffect | PlacementEffect,
  
  // Alternate (Double Buffering)
  alternate: workInProgressFiber,
  
  // Priority
  lanes: SyncLane | DefaultLane,
};
```

---

#### Why Fiber Was Created

**Problems with Old Stack Reconciler:**

```javascript
// Old Stack Reconciler (React 15)
function reconcile(element) {
  // Recursive, synchronous
  const children = element.props.children;
  
  children.forEach(child => {
    reconcile(child); // Can't pause or interrupt
  });
  
  // Once started, must complete entire tree
  // Blocks main thread until done
  // No prioritization possible
}

// Problem:
// Large component tree (10,000 nodes)
// Takes 100ms to process
// UI frozen for 100ms (no user input, animations janky)
```

**Fiber Solutions:**

```javascript
// Fiber Reconciler (React 16+)
function workLoop(deadline) {
  while (nextUnitOfWork && deadline.timeRemaining() > 1) {
    // Work on one Fiber at a time
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    
    // Can pause here if time runs out
  }
  
  if (nextUnitOfWork) {
    // More work to do, schedule continuation
    requestIdleCallback(workLoop);
  } else {
    // All work done, commit to DOM
    commitRoot();
  }
}

// Benefits:
// - Interruptible (pause/resume)
// - Prioritizable (urgent work first)
// - Non-blocking (yield to browser)
```

---

#### Fiber Tree Structure

**Component Tree:**
```jsx
function App() {
  return (
    <div>
      <Header />
      <Main>
        <Article />
        <Sidebar />
      </Main>
    </div>
  );
}
```

**Fiber Linked List:**
```
App (Fiber)
 |
 child
 |
Div (Fiber)
 |
 child
 |
Header (Fiber) --sibling--> Main (Fiber)
                               |
                              child
                               |
                            Article (Fiber) --sibling--> Sidebar (Fiber)
```

**Why Linked List?**
```javascript
// Allows easy pause/resume
let currentFiber = rootFiber;

while (currentFiber) {
  // Process current fiber
  workOnFiber(currentFiber);
  
  // Can save currentFiber and resume later
  if (shouldPause()) {
    saveFiber(currentFiber);
    break;
  }
  
  // Traverse: child -> sibling -> return
  if (currentFiber.child) {
    currentFiber = currentFiber.child;
  } else if (currentFiber.sibling) {
    currentFiber = currentFiber.sibling;
  } else {
    currentFiber = currentFiber.return;
  }
}
```

---

#### Two-Phase Rendering

**Phase 1: Render Phase (Interruptible)**

```javascript
function renderPhase() {
  // Build work-in-progress fiber tree
  // Can be paused, aborted, restarted
  
  let fiber = rootFiber;
  
  while (fiber) {
    // 1. Call component function / render method
    const elements = fiber.type(fiber.props);
    
    // 2. Create/update child fibers
    reconcileChildren(fiber, elements);
    
    // 3. Mark effects needed
    if (propsChanged(fiber)) {
      fiber.flags |= Update;
    }
    
    // 4. Check if should yield to browser
    if (shouldYield()) {
      return fiber; // Resume later
    }
    
    fiber = getNextFiber(fiber);
  }
  
  // All fibers processed, ready to commit
  return null;
}
```

**Phase 2: Commit Phase (Synchronous, Uninterruptible)**

```javascript
function commitPhase() {
  // Apply changes to DOM
  // Must complete without interruption
  
  const finishedWork = workInProgressRoot;
  
  // 1. Before mutation effects (getSnapshotBeforeUpdate)
  commitBeforeMutationEffects(finishedWork);
  
  // 2. Mutation effects (DOM updates)
  commitMutationEffects(finishedWork);
  
  // 3. Switch current tree
  root.current = finishedWork;
  
  // 4. Layout effects (useLayoutEffect)
  commitLayoutEffects(finishedWork);
  
  // 5. Passive effects (useEffect) - scheduled after paint
  schedulePassiveEffects(finishedWork);
}
```

---

#### Double Buffering

**Two Trees: Current and Work-in-Progress**

```javascript
// Current tree (on screen)
const currentTree = {
  fiber: { type: App, props: { count: 1 } },
};

// Work-in-progress tree (being built)
const wipTree = {
  fiber: { type: App, props: { count: 2 } },
};

// Each fiber points to its alternate
currentFiber.alternate = wipFiber;
wipFiber.alternate = currentFiber;

// During render phase: Build wip tree
// During commit phase: Swap trees
root.current = wipTree; // Atomic swap

// Old current becomes new wip for next update
```

**Benefits:**
```javascript
// 1. Abort work without affecting UI
if (higherPriorityUpdate) {
  discardWipTree(); // Current tree unchanged
}

// 2. Reuse work from previous render
if (componentDidntChange) {
  wipFiber = cloneFiber(currentFiber); // Reuse
}

// 3. Compare with previous state
const oldProps = currentFiber.memoizedProps;
const newProps = wipFiber.pendingProps;
if (shallowEqual(oldProps, newProps)) {
  // Skip update
}
```

---

#### Work Loop

**Cooperative Scheduling:**

```javascript
let nextUnitOfWork = null;
let workInProgressRoot = null;

function workLoop(deadline) {
  let shouldYield = false;
  
  while (nextUnitOfWork && !shouldYield) {
    // Process one fiber
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    
    // Check if browser needs main thread
    shouldYield = deadline.timeRemaining() < 1;
  }
  
  if (!nextUnitOfWork && workInProgressRoot) {
    // Render phase complete, commit
    commitRoot();
  }
  
  // Continue work in next frame
  requestIdleCallback(workLoop);
}

function performUnitOfWork(fiber) {
  // 1. Begin work (render component)
  const children = fiber.type(fiber.props);
  
  // 2. Reconcile children
  reconcileChildren(fiber, children);
  
  // 3. Return next unit of work
  if (fiber.child) return fiber.child;
  if (fiber.sibling) return fiber.sibling;
  
  let nextFiber = fiber.return;
  while (nextFiber) {
    if (nextFiber.sibling) return nextFiber.sibling;
    nextFiber = nextFiber.return;
  }
  
  return null; // All work done
}
```

---

#### Priority-Based Scheduling

**Lane Model (React 18+):**

```javascript
// Different priorities represented as binary lanes
const SyncLane = 0b0000000000000000000000000000001;       // Highest
const InputContinuousLane = 0b0000000000000000000000000000100;
const DefaultLane = 0b0000000000000000000000000010000;     // Normal
const TransitionLane = 0b0000000000000000001000000000000;  // Low
const IdleLane = 0b0100000000000000000000000000000;        // Lowest

function scheduleUpdate(fiber, lane) {
  // Mark fiber with priority lane
  fiber.lanes |= lane;
  
  let parent = fiber.return;
  while (parent) {
    // Bubble priority up to root
    parent.childLanes |= lane;
    parent = parent.return;
  }
  
  // Schedule work at this priority
  scheduleUpdateOnFiber(root, lane);
}

// Example: User input (high priority)
function handleClick() {
  scheduleUpdate(fiber, InputContinuousLane);
}

// Example: Data fetch (low priority)
function fetchData() {
  scheduleUpdate(fiber, TransitionLane);
}
```

---

#### Concurrent Features Enabled by Fiber

**1. Time Slicing:**
```javascript
// Break work into chunks
function workLoop() {
  while (workInProgress && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
  // Yield to browser for input/paint
}
```

**2. Suspense:**
```javascript
function Component() {
  const data = use(promise); // Throws promise
  return <div>{data}</div>;
}

// Fiber catches thrown promise
try {
  renderComponent(fiber);
} catch (thrown) {
  if (isPromise(thrown)) {
    // Mark fiber as suspended
    fiber.flags |= SuspenseEffect;
    // Resume when promise resolves
    thrown.then(() => scheduleUpdate(fiber));
  }
}
```

**3. Concurrent Rendering:**
```jsx
import { startTransition } from 'react';

function App() {
  const [text, setText] = useState('');
  const [items, setItems] = useState([]);
  
  const handleChange = (e) => {
    // High priority: Update input immediately
    setText(e.target.value);
    
    // Low priority: Filter list (can be interrupted)
    startTransition(() => {
      const filtered = largeList.filter(item =>
        item.includes(e.target.value)
      );
      setItems(filtered);
    });
  };
  
  return (
    <>
      <input value={text} onChange={handleChange} />
      <List items={items} />
    </>
  );
}

// Input stays responsive even while filtering large list
```

---

#### Complete Fiber Lifecycle

**Update Cycle:**

```javascript
// 1. Trigger update
setState(newValue);

// 2. Schedule work
scheduleUpdateOnFiber(fiber, lane);

// 3. Render phase (interruptible)
while (nextUnitOfWork) {
  // Build work-in-progress tree
  nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
  
  if (shouldYield()) {
    // Save state, yield to browser
    return;
  }
}

// 4. Commit phase (synchronous)
commitRoot();
  // 4a. Before mutation (getSnapshotBeforeUpdate)
  // 4b. Mutation (DOM updates)
  // 4c. Layout effects (useLayoutEffect)
  // 4d. Schedule passive effects (useEffect)

// 5. Paint
// Browser paints changes to screen

// 6. Passive effects
// Run useEffect callbacks
```

---

#### Effects Tracking

**Effect Flags:**

```javascript
const NoEffect = 0b00000000000000;
const Placement = 0b00000000000010;  // Insert into DOM
const Update = 0b00000000000100;     // Update props/state
const Deletion = 0b00000000001000;   // Remove from DOM
const Ref = 0b00000000100000;        // Update ref
const Passive = 0b00000001000000;    // useEffect
const Layout = 0b00000100000000;     // useLayoutEffect

// During render phase: Mark effects
function updateComponent(fiber) {
  const oldProps = fiber.alternate?.memoizedProps;
  const newProps = fiber.pendingProps;
  
  if (!shallowEqual(oldProps, newProps)) {
    fiber.flags |= Update;
  }
  
  if (fiber.ref) {
    fiber.flags |= Ref;
  }
}

// During commit phase: Process effects
function commitEffects(fiber) {
  if (fiber.flags & Placement) {
    commitPlacement(fiber);
  }
  if (fiber.flags & Update) {
    commitUpdate(fiber);
  }
  if (fiber.flags & Deletion) {
    commitDeletion(fiber);
  }
}
```

---

#### Practical Example

**Large List Update:**

```jsx
function LargeList({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

function App() {
  const [items, setItems] = useState(generateItems(10000));
  
  const handleUpdate = () => {
    // In React 15: UI freezes for ~100ms
    // In React 16+ with Fiber: Smooth, interruptible
    setItems(generateItems(10000));
  };
  
  return (
    <>
      <button onClick={handleUpdate}>Update</button>
      <LargeList items={items} />
    </>
  );
}

// Fiber breaks work into chunks:
// Frame 1: Process fibers 1-100 (16ms)
// Frame 2: Process fibers 101-200 (16ms)
// ...
// UI remains responsive throughout
```

---

#### Key Benefits

**✅ Interruptible Rendering:**
```javascript
// Can pause work to handle urgent updates
if (higherPriorityUpdate) {
  pauseCurrentWork();
  startHighPriorityWork();
}
```

**✅ Prioritization:**
```javascript
// User input > data fetching > background work
scheduleUpdate(fiber, InputContinuousLane); // Urgent
scheduleUpdate(fiber, TransitionLane);      // Can wait
```

**✅ Better Performance:**
```javascript
// 60 FPS = 16ms per frame
// Fiber yields to browser every ~5ms
// Animations stay smooth
```

**✅ Concurrent Features:**
```javascript
// Suspense, Transitions, Streaming SSR
// All built on Fiber foundation
```

---

#### Interview Tips

✅ **Key Points:**
- Fiber is a unit of work, each component has a Fiber node
- Rendering is interruptible (pause/resume/abort)
- Two-phase: Render (interruptible) + Commit (synchronous)
- Double buffering: Current and work-in-progress trees
- Linked list structure enables pausable traversal
- Lane model for priority-based scheduling
- Foundation for concurrent features

✅ **When to Mention:**
- Concurrent rendering discussions
- Performance optimization questions
- React internals deep dive
- Suspense/Transitions implementation
- Scheduling and prioritization

✅ **Common Follow-ups:**
- "What's the difference between Stack and Fiber?"
- "How does Fiber enable time slicing?"
- "What are lanes in React?"
- "How does double buffering work?"
- "What's the commit phase?"

✅ **Perfect Answer Structure:**
1. Definition: Fiber = unit of work, rewrite of reconciler
2. Why: Old Stack reconciler was blocking and non-interruptible
3. How: Linked list structure, two-phase rendering, double buffering
4. Benefits: Interruptible, prioritizable, enables concurrent features
5. Example: Show work loop with pause/resume
6. Real Impact: Smooth UI even with heavy updates

</details>

---

### 192. What is the difference between Stack and Fiber reconcilers?

<details>
<summary>View Answer</summary>

**Stack vs Fiber reconcilers** represent React's evolution from a **synchronous, blocking** rendering engine to an **asynchronous, interruptible** one. The Stack reconciler (React ≤15) used recursive calls that couldn't be interrupted, while Fiber (React 16+) uses a linked list structure that enables pausable, prioritizable rendering.

#### Stack Reconciler (React ≤15)

**How It Works:**

```javascript
// Stack Reconciler (Simplified)
function reconcile(element, container) {
  // Recursive calls - uses JavaScript call stack
  const instance = element.type(element.props);
  
  // Process children recursively
  if (instance.props.children) {
    instance.props.children.forEach(child => {
      reconcile(child, instance); // Recursive call
    });
  }
  
  // Update DOM
  container.appendChild(instance);
  
  // Can't pause or interrupt here!
  // Must complete entire tree before returning
}

// Problem visualization:
// reconcile(App)           | <-- Stack frame
//   reconcile(Header)      | <-- Stack frame
//     reconcile(Nav)       | <-- Stack frame
//       reconcile(Link)    | <-- Stack frame (deepest)
//       return             |
//     return               |
//   reconcile(Main)        |
//     ... (continue)       |
// Entire stack must unwind before control returns to browser
```

**Characteristics:**

```
❌ Synchronous: Once started, runs to completion
❌ Blocking: Locks main thread until done
❌ Recursive: Uses JavaScript call stack
❌ Non-interruptible: Can't pause or abort
❌ No prioritization: All updates equal
❌ Linear execution: Process tree top to bottom
```

**Performance Issue:**

```javascript
// Large component tree
function App() {
  return (
    <div>
      {Array(10000).fill(0).map((_, i) => (
        <ComplexComponent key={i} />
      ))}
    </div>
  );
}

// Stack Reconciler:
// - Processes all 10,000 components
// - Takes ~100ms (example)
// - Main thread blocked entire time
// - User input ignored
// - Animations janky
// - UI frozen

// Result: Poor user experience
```

---

#### Fiber Reconciler (React 16+)

**How It Works:**

```javascript
// Fiber Reconciler (Simplified)
function workLoop(deadline) {
  // Non-recursive - uses linked list
  while (nextUnitOfWork && deadline.timeRemaining() > 1) {
    // Process ONE unit of work
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    
    // Check if we should yield to browser
    if (deadline.timeRemaining() < 1) {
      // Pause here, resume later
      break;
    }
  }
  
  if (nextUnitOfWork) {
    // More work to do, schedule continuation
    requestIdleCallback(workLoop);
  } else {
    // All work done, commit changes
    commitRoot();
  }
}

function performUnitOfWork(fiber) {
  // Work on one fiber node
  const children = fiber.type(fiber.props);
  reconcileChildren(fiber, children);
  
  // Return next fiber (linked list traversal)
  if (fiber.child) return fiber.child;
  if (fiber.sibling) return fiber.sibling;
  return getNextFiberInTree(fiber);
}
```

**Characteristics:**

```
✅ Asynchronous: Can pause and resume
✅ Non-blocking: Yields to browser regularly
✅ Iterative: Uses linked list, not recursion
✅ Interruptible: Can pause, abort, or restart
✅ Prioritizable: High-priority work first
✅ Concurrent: Multiple renders in progress
```

---

#### Side-by-Side Comparison

**Architecture:**

```javascript
// STACK RECONCILER
// Uses JavaScript call stack (recursive)
function reconcileStack(element) {
  // Call stack grows with depth
  const children = element.children;
  
  children.forEach(child => {
    reconcileStack(child); // Recursive
  });
  
  // Stack unwinds when done
}

// FIBER RECONCILER  
// Uses linked list (iterative)
function reconcileFiber(fiber) {
  // No recursion, manual traversal
  let current = fiber;
  
  while (current) {
    // Process current node
    processNode(current);
    
    // Navigate via pointers
    current = getNextNode(current);
    
    // Can break out anytime
    if (shouldPause()) break;
  }
}
```

**Data Structure:**

```javascript
// STACK RECONCILER
// Tree structure (implicit via call stack)
const element = {
  type: 'div',
  props: {
    children: [
      { type: 'h1', props: {} },
      { type: 'p', props: {} },
    ],
  },
};
// Traversal via recursion

// FIBER RECONCILER
// Linked list structure (explicit pointers)
const fiber = {
  type: 'div',
  
  // Explicit relationship pointers
  child: h1Fiber,      // First child
  sibling: null,       // Next sibling
  return: parentFiber, // Parent
  
  alternate: currentFiber, // Double buffering
};
// Traversal via pointers
```

**Execution:**

```javascript
// STACK RECONCILER
function updateComponent(component) {
  // Synchronous, all-or-nothing
  const rendered = component.render();
  
  rendered.children.forEach(child => {
    updateComponent(child); // Must complete
  });
  
  updateDOM(component); // Can't interrupt
}

// Entire tree updated in single, uninterruptible pass

// FIBER RECONCILER
function workLoop() {
  while (nextUnitOfWork) {
    // Process incrementally
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    
    // Can pause between units
    if (shouldYield()) {
      return; // Resume later
    }
  }
  
  commitWork(); // Commit when ready
}

// Tree updated in small, interruptible chunks
```

---

#### Feature Comparison Table

| Feature | Stack Reconciler | Fiber Reconciler |
|---------|-----------------|------------------|
| **React Version** | ≤15 | 16+ |
| **Rendering** | Synchronous | Asynchronous |
| **Interruptible** | ❌ No | ✅ Yes |
| **Data Structure** | Tree (call stack) | Linked list |
| **Traversal** | Recursive | Iterative |
| **Pausing** | ❌ Impossible | ✅ Built-in |
| **Prioritization** | ❌ No | ✅ Lane-based |
| **Concurrent Mode** | ❌ No | ✅ Yes |
| **Time Slicing** | ❌ No | ✅ Yes |
| **Suspense** | ❌ No | ✅ Yes |
| **Error Boundaries** | Basic | Advanced |
| **Main Thread** | Blocked | Cooperative |

---

#### Performance Comparison

**Stack Reconciler (Blocking):**

```javascript
function LargeList() {
  const items = Array(5000).fill(0);
  
  return (
    <div>
      {items.map((_, i) => (
        <ExpensiveComponent key={i} />
      ))}
    </div>
  );
}

// React 15 (Stack):
// ┌─────────────────────────────────────┐
// │  Reconcile + Update (100ms)         │ <-- UI Frozen
// └─────────────────────────────────────┘
//                                        │
//                                    Paint
//
// User Experience:
// - Click button
// - Wait 100ms (frozen)
// - UI updates
// - Janky, unresponsive
```

**Fiber Reconciler (Non-blocking):**

```javascript
// Same component

// React 16+ (Fiber):
// ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
// │Chunk1│ │Chunk2│ │Chunk3│ │Chunk4│ ... Commit
// └──────┘ └──────┘ └──────┘ └──────┘
//    5ms      5ms      5ms      5ms
//    ↓        ↓        ↓        ↓
//  Yield    Yield    Yield    Yield
//
// User Experience:
// - Click button
// - Small delay, but UI responsive
// - Animations smooth
// - Can click other buttons during update
// - Feels fast and responsive
```

---

#### Real-World Impact

**Example: Search Input + Large List**

```jsx
function SearchApp() {
  const [query, setQuery] = useState('');
  const [items, setItems] = useState(generateItems(10000));
  
  const handleSearch = (e) => {
    const value = e.target.value;
    setQuery(value);
    
    // Filter large list
    const filtered = items.filter(item =>
      item.name.toLowerCase().includes(value.toLowerCase())
    );
    setItems(filtered);
  };
  
  return (
    <>
      <input value={query} onChange={handleSearch} />
      <List items={items} />
    </>
  );
}
```

**Stack Reconciler (React 15):**
```
User types: "a"
→ Trigger re-render
→ Process 10,000 items synchronously
→ UI frozen for 80ms
→ Input feels laggy
→ Each keystroke causes freeze
→ Poor typing experience
```

**Fiber Reconciler (React 16+):**
```
User types: "a"
→ Trigger re-render
→ Process items in chunks
→ Yield to browser every 5ms
→ Input stays responsive
→ Smooth typing
→ List updates progressively
```

**With Concurrent Features (React 18+):**
```jsx
import { startTransition } from 'react';

const handleSearch = (e) => {
  const value = e.target.value;
  
  // High priority: Update input immediately
  setQuery(value);
  
  // Low priority: Filter list (interruptible)
  startTransition(() => {
    const filtered = items.filter(item =>
      item.name.toLowerCase().includes(value.toLowerCase())
    );
    setItems(filtered);
  });
};

// Input updates instantly
// List updates when possible
// Perfect user experience
```

---

#### Migration Impact

**Breaking Changes (React 15 → 16):**

```javascript
// React 15: componentWillMount could be called once
componentWillMount() {
  this.fetchData(); // Called once before mount
}

// React 16+: Render phase lifecycles can be called multiple times
componentWillMount() {
  this.fetchData(); // Could be called MULTIPLE times!
  // If render is paused/aborted/restarted
}

// Solution: Use commit phase lifecycles
componentDidMount() {
  this.fetchData(); // Always called once after commit
}
```

**Deprecated Lifecycles:**
```javascript
// Deprecated (unsafe with Fiber)
componentWillMount()
componentWillReceiveProps()
componentWillUpdate()

// Reason: Called during render phase (interruptible)
// Could be called multiple times
// Could cause side effects

// New safe alternatives
static getDerivedStateFromProps() // Render phase, pure
componentDidMount()                // Commit phase, once
componentDidUpdate()               // Commit phase, once
```

---

#### Implementation Differences

**Stack: Recursive Rendering**

```javascript
function mountComponent(element) {
  const instance = new element.type(element.props);
  
  // Call lifecycle
  instance.componentWillMount();
  
  // Render (recursive)
  const renderedElement = instance.render();
  
  // Mount children (recursive)
  const renderedComponent = instantiateReactComponent(renderedElement);
  renderedComponent.mountComponent(); // Recursion
  
  // Update DOM immediately
  const markup = renderedComponent.getHostNode();
  return markup;
}
```

**Fiber: Iterative Rendering**

```javascript
function performUnitOfWork(fiber) {
  // No recursion - iterative
  
  // 1. Begin work on this fiber
  const shouldUpdate = beginWork(fiber);
  
  if (shouldUpdate) {
    // 2. Render component
    const children = fiber.type(fiber.props);
    
    // 3. Reconcile children (create child fibers)
    reconcileChildren(fiber, children);
  }
  
  // 4. Complete work on this fiber
  completeWork(fiber);
  
  // 5. Return next unit of work (linked list traversal)
  if (fiber.child) return fiber.child;
  if (fiber.sibling) return fiber.sibling;
  
  // Traverse up to find next sibling
  let parent = fiber.return;
  while (parent) {
    if (parent.sibling) return parent.sibling;
    parent = parent.return;
  }
  
  return null; // All work done
}
```

---

#### Visual Comparison

**Stack Reconciler Timeline:**
```
Time (ms): 0────20────40────60────80────100
           |═════════════════════════════| Render + Update (Blocked)
                                          └─ Paint

Main Thread: ████████████████████████████ (Busy)
User Input:  XXXXXXXXXXXXXXXXXXXXXXXXXXXX (Ignored)
Animations:  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ (Janky)
```

**Fiber Reconciler Timeline:**
```
Time (ms): 0────20────40────60────80────100
           |════|  |════|  |════|  |════|  └─ Commit + Paint
           Chunk1  Chunk2  Chunk3  Chunk4

Main Thread: ████░░░████░░░████░░░████░░░ (Cooperative)
User Input:  ✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓✓ (Responsive)
Animations:  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~ (Smooth)
```

---

#### Interview Tips

✅ **Key Points:**
- Stack: Synchronous, recursive, blocking, non-interruptible (React ≤15)
- Fiber: Asynchronous, iterative, non-blocking, interruptible (React 16+)
- Stack uses call stack, Fiber uses linked list
- Fiber enables time slicing, concurrent mode, Suspense
- Performance: Fiber yields to browser, Stack blocks until done
- Migration required deprecating render phase lifecycles

✅ **When to Mention:**
- React version differences questions
- Performance optimization discussions
- Concurrent rendering explanations
- React internals deep dive
- Why certain lifecycles were deprecated

✅ **Common Follow-ups:**
- "Why was Fiber created?"
- "What problems did Stack reconciler have?"
- "How does Fiber enable concurrent mode?"
- "What breaking changes came with Fiber?"
- "Why were lifecycles like componentWillMount deprecated?"

✅ **Perfect Answer Structure:**
1. Stack: Synchronous, recursive, blocking (React ≤15)
2. Problem: Long renders freeze UI, no prioritization
3. Fiber: Asynchronous, iterative, interruptible (React 16+)
4. How: Linked list structure enables pause/resume
5. Benefits: Time slicing, concurrent mode, better UX
6. Example: Show blocking vs non-blocking behavior
7. Impact: Smooth UI even with heavy updates

</details>

---

### 193. How does React's scheduling algorithm work?

<details>
<summary>View Answer</summary>

**React's scheduling algorithm** determines **when** and **in what order** to process updates. It uses a **priority-based, cooperative scheduling** system that breaks work into chunks, yields to the browser for high-priority tasks, and ensures smooth user experiences even during heavy rendering.

#### Core Concepts

**The Scheduler Package:**

```javascript
// React uses the 'scheduler' package
import { 
  scheduleCallback,
  cancelCallback,
  shouldYield,
  ImmediatePriority,
  UserBlockingPriority,
  NormalPriority,
  LowPriority,
  IdlePriority,
} from 'scheduler';

// Schedule work with priority
const taskId = scheduleCallback(
  UserBlockingPriority,
  () => {
    // Do work
  }
);

// Cancel if needed
cancelCallback(taskId);
```

---

#### Priority Levels

**Five Priority Levels:**

```javascript
// Priority levels (highest to lowest)
const ImmediatePriority = 1;      // Sync, must run now
const UserBlockingPriority = 2;   // User input, must be responsive
const NormalPriority = 3;          // Default, network responses
const LowPriority = 4;             // Can be delayed
const IdlePriority = 5;            // Run when idle

// Timeout values (how long before expiration)
const timeouts = {
  [ImmediatePriority]: -1,          // Already expired
  [UserBlockingPriority]: 250,      // 250ms
  [NormalPriority]: 5000,           // 5s
  [LowPriority]: 10000,             // 10s
  [IdlePriority]: maxSigned31BitInt, // Never expires
};
```

**Priority Examples:**

```jsx
import { startTransition, useTransition } from 'react';

function SearchApp() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleChange = (e) => {
    // HIGH PRIORITY: Update input (UserBlockingPriority)
    // User expects immediate feedback
    setQuery(e.target.value);
    
    // LOW PRIORITY: Update results (NormalPriority or lower)
    // Can be interrupted if user keeps typing
    startTransition(() => {
      const filtered = largeDataset.filter(item =>
        item.includes(e.target.value)
      );
      setResults(filtered);
    });
  };
  
  return (
    <>
      <input value={query} onChange={handleChange} />
      <Results data={results} />
    </>
  );
}
```

---

#### Lane Model (React 18+)

**Lanes Replace Expiration Times:**

```javascript
// Old system (React 16-17): Expiration times
const expirationTime = currentTime + timeout;
if (currentTime >= expirationTime) {
  // Task expired, must run
}

// New system (React 18+): Lanes (binary representation)
const SyncLane =              0b0000000000000000000000000000001;
const InputContinuousLane =   0b0000000000000000000000000000100;
const DefaultLane =           0b0000000000000000000000000010000;
const TransitionLane1 =       0b0000000000000000000000001000000;
const TransitionLane2 =       0b0000000000000000000000010000000;
// ... more transition lanes
const IdleLane =              0b0100000000000000000000000000000;

// Benefits:
// 1. Multiple priorities can coexist (bitwise OR)
// 2. Fast priority checks (bitwise AND)
// 3. Easy to merge/separate work
```

**Lane Operations:**

```javascript
// Mark work with lane
function scheduleUpdate(fiber, lane) {
  fiber.lanes |= lane; // Add lane to fiber
  
  // Bubble up to root
  let parent = fiber.return;
  while (parent) {
    parent.childLanes |= lane;
    parent = parent.return;
  }
}

// Check if lane has work
function hasLaneWork(fiber, lane) {
  return (fiber.lanes & lane) !== 0;
}

// Get highest priority lane
function getHighestPriorityLane(lanes) {
  // Returns rightmost (highest priority) set bit
  return lanes & -lanes;
}

// Remove lane after processing
function removeLane(lanes, lane) {
  return lanes & ~lane;
}
```

---

#### Task Queue System

**Two Queues:**

```javascript
const taskQueue = [];        // Immediate tasks (expired)
const timerQueue = [];       // Delayed tasks (not yet expired)

// Schedule a task
function scheduleCallback(priorityLevel, callback) {
  const currentTime = getCurrentTime();
  
  // Calculate start time
  const startTime = currentTime;
  
  // Calculate timeout based on priority
  const timeout = timeouts[priorityLevel];
  const expirationTime = startTime + timeout;
  
  // Create task
  const newTask = {
    id: taskIdCounter++,
    callback,
    priorityLevel,
    startTime,
    expirationTime,
    sortIndex: expirationTime, // For min-heap sorting
  };
  
  // Add to appropriate queue
  if (startTime > currentTime) {
    // Delayed task
    newTask.sortIndex = startTime;
    push(timerQueue, newTask);
    
    // Set timer to move to taskQueue when ready
    if (peek(taskQueue) === null) {
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } else {
    // Immediate task
    push(taskQueue, newTask);
    
    // Start work loop if not already running
    if (!isHostCallbackScheduled && !isPerformingWork) {
      isHostCallbackScheduled = true;
      requestHostCallback(flushWork);
    }
  }
  
  return newTask;
}
```

**Min-Heap Priority Queue:**

```javascript
// Tasks stored in min-heap (binary tree)
// Smallest expirationTime at root (highest priority)

function push(heap, node) {
  const index = heap.length;
  heap.push(node);
  siftUp(heap, node, index);
}

function peek(heap) {
  return heap.length === 0 ? null : heap[0];
}

function pop(heap) {
  if (heap.length === 0) return null;
  
  const first = heap[0];
  const last = heap.pop();
  
  if (last !== first) {
    heap[0] = last;
    siftDown(heap, last, 0);
  }
  
  return first;
}

// Example heap:
//       Task A (exp: 100)
//      /                \
// Task B (exp: 150)   Task C (exp: 200)
//
// Task A has earliest expiration = highest priority
```

---

#### Work Loop

**Main Scheduling Loop:**

```javascript
function workLoop(hasTimeRemaining, initialTime) {
  let currentTime = initialTime;
  
  // Move expired timers to task queue
  advanceTimers(currentTime);
  
  // Get highest priority task
  currentTask = peek(taskQueue);
  
  while (currentTask !== null) {
    // Check if task expired or should yield
    if (
      currentTask.expirationTime > currentTime &&
      (!hasTimeRemaining || shouldYield())
    ) {
      // Task not expired and we should yield
      break;
    }
    
    // Execute task callback
    const callback = currentTask.callback;
    
    if (typeof callback === 'function') {
      currentTask.callback = null;
      currentPriorityLevel = currentTask.priorityLevel;
      
      const didUserCallbackTimeout = 
        currentTask.expirationTime <= currentTime;
      
      // Run the callback
      const continuationCallback = callback(didUserCallbackTimeout);
      
      currentTime = getCurrentTime();
      
      // Check if callback wants to continue
      if (typeof continuationCallback === 'function') {
        // More work to do, update callback
        currentTask.callback = continuationCallback;
      } else {
        // Task complete, remove from queue
        if (currentTask === peek(taskQueue)) {
          pop(taskQueue);
        }
      }
      
      advanceTimers(currentTime);
    } else {
      // No callback, remove task
      pop(taskQueue);
    }
    
    // Get next task
    currentTask = peek(taskQueue);
  }
  
  // Return whether there's more work
  if (currentTask !== null) {
    return true; // More work
  } else {
    // Check timer queue
    const firstTimer = peek(timerQueue);
    if (firstTimer !== null) {
      requestHostTimeout(handleTimeout, firstTimer.startTime - currentTime);
    }
    return false; // No more work
  }
}
```

---

#### Yielding to Browser

**shouldYield() Function:**

```javascript
// Check if we should pause work
function shouldYield() {
  const currentTime = getCurrentTime();
  
  // Yield if frame deadline passed
  return currentTime >= deadline;
}

// Browser provides time budget
let deadline = 0;

function setDeadline(timeRemaining) {
  deadline = getCurrentTime() + timeRemaining;
}

// Typical frame budget: ~5ms
// 60 FPS = 16.67ms per frame
// React uses ~5ms, leaves 11.67ms for browser
```

**MessageChannel for Scheduling:**

```javascript
// React uses MessageChannel (not setTimeout)
const channel = new MessageChannel();
const port = channel.port2;

channel.port1.onmessage = () => {
  // This runs after current task completes
  // But before next event loop iteration
  performWorkUntilDeadline();
};

function requestHostCallback(callback) {
  scheduledHostCallback = callback;
  
  if (!isMessageLoopRunning) {
    isMessageLoopRunning = true;
    port.postMessage(null); // Schedule work
  }
}

function performWorkUntilDeadline() {
  if (scheduledHostCallback !== null) {
    const currentTime = getCurrentTime();
    
    // Yield after 5ms
    deadline = currentTime + yieldInterval;
    
    const hasTimeRemaining = true;
    let hasMoreWork = true;
    
    try {
      hasMoreWork = scheduledHostCallback(
        hasTimeRemaining,
        currentTime
      );
    } finally {
      if (hasMoreWork) {
        // More work, schedule continuation
        port.postMessage(null);
      } else {
        isMessageLoopRunning = false;
        scheduledHostCallback = null;
      }
    }
  }
}
```

---

#### Starvation Prevention

**Expiration Times Prevent Low Priority Starvation:**

```javascript
// Problem: High priority updates keep coming
// Low priority updates never run

// Solution: Tasks expire and become urgent
function scheduleUpdate(fiber, lane) {
  const currentTime = getCurrentTime();
  const timeout = getTimeoutForLane(lane);
  const expirationTime = currentTime + timeout;
  
  fiber.expirationTime = expirationTime;
  
  // Example:
  // Low priority task: timeout = 10,000ms
  // After 10 seconds, becomes expired
  // Expired tasks run immediately (ImmediatePriority)
}

// This ensures low priority work eventually completes
```

---

#### Complete Example

**React Component to Scheduler Flow:**

```jsx
// 1. User interaction
function App() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    // 2. Trigger update
    setCount(c => c + 1);
  };
  
  return <button onClick={handleClick}>{count}</button>;
}

// 3. React's update flow:
function dispatchSetState(fiber, queue, action) {
  // Create update object
  const update = {
    lane: requestUpdateLane(fiber), // Get priority
    action,
    next: null,
  };
  
  // Add to update queue
  enqueueUpdate(fiber, queue, update);
  
  // 4. Schedule work
  scheduleUpdateOnFiber(fiber, lane);
}

function scheduleUpdateOnFiber(fiber, lane) {
  // Mark fiber tree with lane
  markUpdateLaneFromFiberToRoot(fiber, lane);
  
  // 5. Ensure root is scheduled
  ensureRootIsScheduled(root);
}

function ensureRootIsScheduled(root) {
  // Get highest priority lane
  const nextLanes = getNextLanes(root);
  const newCallbackPriority = getHighestPriorityLane(nextLanes);
  
  // Convert lane to scheduler priority
  let schedulerPriorityLevel;
  switch (lanesToEventPriority(nextLanes)) {
    case DiscreteEventPriority:
      schedulerPriorityLevel = ImmediatePriority;
      break;
    case ContinuousEventPriority:
      schedulerPriorityLevel = UserBlockingPriority;
      break;
    case DefaultEventPriority:
      schedulerPriorityLevel = NormalPriority;
      break;
    case IdleEventPriority:
      schedulerPriorityLevel = IdlePriority;
      break;
    default:
      schedulerPriorityLevel = NormalPriority;
  }
  
  // 6. Schedule callback with scheduler
  newCallbackNode = scheduleCallback(
    schedulerPriorityLevel,
    performConcurrentWorkOnRoot.bind(null, root)
  );
  
  root.callbackPriority = newCallbackPriority;
  root.callbackNode = newCallbackNode;
}

function performConcurrentWorkOnRoot(root, didTimeout) {
  // 7. Render phase (work loop)
  const lanes = getNextLanes(root);
  
  renderRootConcurrent(root, lanes);
  
  // 8. Commit phase
  if (workInProgressRootExitStatus === RootCompleted) {
    commitRoot(root);
  }
  
  // 9. Check for more work
  ensureRootIsScheduled(root);
  
  // 10. Return continuation if needed
  if (root.callbackNode === originalCallbackNode) {
    return performConcurrentWorkOnRoot.bind(null, root);
  }
  
  return null;
}
```

---

#### Batching Updates

**Automatic Batching (React 18+):**

```jsx
function handleClick() {
  // All updates batched automatically
  setCount(c => c + 1);    // \
  setFlag(f => !f);        //  > Batched
  setName('New Name');     // /
  
  // Only one render triggered
}

// Even in promises/timeouts (React 18+)
fetch('/api').then(() => {
  setCount(c => c + 1);    // \
  setFlag(f => !f);        //  > Batched
  setName('New Name');     // /
});

setTimeout(() => {
  setCount(c => c + 1);    // \
  setFlag(f => !f);        //  > Batched
  setName('New Name');     // /
}, 1000);
```

**Manual Batching (React 17):**

```jsx
import { unstable_batchedUpdates } from 'react-dom';

fetch('/api').then(() => {
  unstable_batchedUpdates(() => {
    setCount(c => c + 1);    // \
    setFlag(f => !f);        //  > Batched
    setName('New Name');     // /
  });
});
```

---

#### Priority Inheritance

**Context Updates Inherit Priority:**

```jsx
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    // High priority (user interaction)
    setTheme(t => t === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={toggleTheme}>Toggle</button>
      <DeepChild />
    </ThemeContext.Provider>
  );
}

function DeepChild() {
  // This update inherits priority from setTheme
  const theme = useContext(ThemeContext);
  return <div className={theme}>Content</div>;
}

// All consumers update at same priority as provider
```

---

#### Debug Visualization

**React DevTools Profiler:**

```jsx
import { Profiler } from 'react';

function App() {
  return (
    <Profiler
      id="App"
      onRender={(id, phase, actualDuration, baseDuration) => {
        console.log({
          component: id,
          phase, // 'mount' or 'update'
          actualDuration, // Time spent rendering
          baseDuration,   // Estimated time without memoization
        });
      }}
    >
      <MyComponent />
    </Profiler>
  );
}
```

---

#### Interview Tips

✅ **Key Points:**
- Priority-based scheduling: 5 levels (Immediate to Idle)
- Lane model: Binary representation for efficient priority management
- Task queues: Min-heap for expired tasks, timer queue for delayed
- Cooperative scheduling: Yields to browser every ~5ms
- Starvation prevention: Tasks expire and become urgent
- Automatic batching: Multiple updates rendered once (React 18+)
- MessageChannel: Better than setTimeout for scheduling

✅ **When to Mention:**
- Concurrent rendering discussions
- Performance optimization questions
- startTransition/useTransition explanations
- Update batching behavior
- React 18 features

✅ **Common Follow-ups:**
- "What are lanes in React?"
- "How does shouldYield work?"
- "What's the difference between priorities?"
- "How does automatic batching work?"
- "Why MessageChannel instead of setTimeout?"

✅ **Perfect Answer Structure:**
1. Purpose: Determine when and in what order to process updates
2. Priority Levels: 5 levels from Immediate to Idle
3. Lane Model: Binary representation for efficient priority checks
4. Task Queues: Min-heap for prioritization
5. Yielding: Cooperative scheduling, ~5ms chunks
6. Starvation Prevention: Tasks expire and become urgent
7. Example: Show user input prioritized over data updates

</details>

---

### 194. What is time slicing in React?

<details>
<summary>View Answer</summary>

**Time slicing** is React's ability to **split rendering work into chunks** and spread it across multiple frames, yielding to the browser between chunks to keep the UI responsive. It prevents long-running renders from blocking user input, animations, and other high-priority tasks.

#### The Problem Time Slicing Solves

**Without Time Slicing (React 15):**

```jsx
function App() {
  const [items, setItems] = useState([]);
  
  const handleUpdate = () => {
    // Generate 10,000 items
    const newItems = Array(10000).fill(0).map((_, i) => ({
      id: i,
      name: `Item ${i}`,
      description: `Description for item ${i}`,
    }));
    
    setItems(newItems);
  };
  
  return (
    <>
      <button onClick={handleUpdate}>Update</button>
      <List items={items} />
    </>
  );
}

// React 15 behavior:
// ┌───────────────────────────────────────┐
// │  Render 10,000 items (100ms blocked)    │
// └───────────────────────────────────────┘
//
// During these 100ms:
// ❌ User clicks are ignored
// ❌ Animations freeze
// ❌ Scrolling is janky
// ❌ Input fields don't update
// ❌ Page feels frozen
```

**With Time Slicing (React 18+):**

```jsx
// Same component, but in React 18 with concurrent features

import { startTransition } from 'react';

function App() {
  const [items, setItems] = useState([]);
  
  const handleUpdate = () => {
    const newItems = Array(10000).fill(0).map((_, i) => ({
      id: i,
      name: `Item ${i}`,
      description: `Description for item ${i}`,
    }));
    
    // Wrap in transition to enable time slicing
    startTransition(() => {
      setItems(newItems);
    });
  };
  
  return (
    <>
      <button onClick={handleUpdate}>Update</button>
      <List items={items} />
    </>
  );
}

// React 18 behavior with time slicing:
// ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
// │Chunk1│ │Chunk2│ │Chunk3│ │Chunk4│ ...
// └──────┘ └──────┘ └──────┘ └──────┘
//   5ms      5ms      5ms      5ms
//    ↓        ↓        ↓        ↓
//  Yield    Yield    Yield    Yield
//
// Between chunks:
// ✅ User clicks processed
// ✅ Animations continue smoothly
// ✅ Scrolling remains smooth
// ✅ Input fields responsive
// ✅ Page feels fast
```

---

#### How Time Slicing Works

**1. Break Work into Units:**

```javascript
// Fiber architecture enables unit-based work
function performUnitOfWork(fiber) {
  // Process ONE component/element
  beginWork(fiber);
  completeWork(fiber);
  
  // Return next unit of work
  return getNextUnitOfWork(fiber);
}

// React can pause between any two units
```

**2. Check Time Budget:**

```javascript
function workLoopConcurrent() {
  // Process units while time available
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
}

function shouldYield() {
  // Check if frame time expired
  const currentTime = getCurrentTime();
  return currentTime >= deadline;
}

// Typical budget: ~5ms per chunk
// 60 FPS = 16.67ms per frame
// React uses 5ms, leaves 11.67ms for browser
```

**3. Yield to Browser:**

```javascript
function performConcurrentWorkOnRoot(root) {
  // Try to render
  let exitStatus = renderRootConcurrent(root, lanes);
  
  if (exitStatus === RootInProgress) {
    // More work to do
    // Yield to browser, resume later
    return performConcurrentWorkOnRoot.bind(null, root);
  }
  
  if (exitStatus === RootCompleted) {
    // All work done
    commitRoot(root);
  }
  
  return null;
}

// Browser processes:
// - User input events
// - Animations
// - Layout
// - Paint

// Then React resumes
```

---

#### Real-World Example

**Search with Large Dataset:**

```jsx
import { useState, startTransition, useDeferredValue } from 'react';

function SearchApp() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  // Large dataset
  const allItems = useMemo(() => {
    return Array(50000).fill(0).map((_, i) => ({
      id: i,
      name: `Product ${i}`,
      description: `Description ${i}`,
      price: Math.random() * 100,
    }));
  }, []);
  
  const handleSearch = (e) => {
    const value = e.target.value;
    
    // HIGH PRIORITY: Update input immediately
    setQuery(value);
    
    // LOW PRIORITY: Filter results (time-sliced)
    startTransition(() => {
      const filtered = allItems.filter(item =>
        item.name.toLowerCase().includes(value.toLowerCase())
      );
      setResults(filtered);
    });
  };
  
  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={handleSearch}
        placeholder="Search..."
      />
      
      {isPending && <Spinner />}
      
      <Results items={results} />
    </div>
  );
}

function Results({ items }) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <h3>{item.name}</h3>
          <p>{item.description}</p>
          <span>${item.price.toFixed(2)}</span>
        </li>
      ))}
    </ul>
  );
}

// User experience:
// 1. User types "pro"
// 2. Input updates INSTANTLY (high priority)
// 3. Results filter happens in background (time-sliced)
// 4. If user types again, filter is interrupted and restarted
// 5. Smooth, responsive experience
```

---

#### Time Slicing Timeline

**Frame-by-Frame Breakdown:**

```
Frame 1 (16.67ms):
  ┌────────────────────────────────────────────┐
  │ React Work (5ms) | Browser (11.67ms)       │
  │ Process fibers  | Input, Layout, Paint    │
  │ 0-500 of 5000   | Animations, Scroll      │
  └────────────────────────────────────────────┘

Frame 2 (16.67ms):
  ┌────────────────────────────────────────────┐
  │ React Work (5ms) | Browser (11.67ms)       │
  │ Process fibers  | Smooth animations       │
  │ 501-1000        | Process user clicks     │
  └────────────────────────────────────────────┘

Frame 3 (16.67ms):
  ┌────────────────────────────────────────────┐
  │ React Work (5ms) | Browser (11.67ms)       │
  │ Process fibers  | UI remains responsive   │
  │ 1001-1500       | No jank                 │
  └────────────────────────────────────────────┘

... continues until all 5000 fibers processed

Result: Smooth 60 FPS maintained throughout
```

---

#### Enabling Time Slicing

**1. Concurrent Rendering:**

```jsx
import { createRoot } from 'react-dom/client';

// React 18: createRoot enables concurrent features
const root = createRoot(document.getElementById('root'));
root.render(<App />);

// React 17: Legacy root, no time slicing
ReactDOM.render(<App />, document.getElementById('root'));
```

**2. startTransition:**

```jsx
import { startTransition } from 'react';

function App() {
  const [tab, setTab] = useState('home');
  
  const handleTabChange = (newTab) => {
    // Mark as non-urgent, enable time slicing
    startTransition(() => {
      setTab(newTab);
    });
  };
  
  return (
    <>
      <button onClick={() => handleTabChange('home')}>Home</button>
      <button onClick={() => handleTabChange('posts')}>Posts</button>
      <TabContent tab={tab} />
    </>
  );
}
```

**3. useTransition:**

```jsx
import { useTransition } from 'react';

function App() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();
  
  const handleTabChange = (newTab) => {
    startTransition(() => {
      setTab(newTab);
    });
  };
  
  return (
    <>
      <button onClick={() => handleTabChange('home')}>Home</button>
      <button onClick={() => handleTabChange('posts')}>
        Posts {isPending && '...'}
      </button>
      <TabContent tab={tab} />
    </>
  );
}
```

**4. useDeferredValue:**

```jsx
import { useDeferredValue } from 'react';

function SearchResults({ query }) {
  // Deferred value updates with lower priority
  const deferredQuery = useDeferredValue(query);
  
  const results = useMemo(() => {
    // Expensive computation
    return largeDataset.filter(item =>
      item.name.includes(deferredQuery)
    );
  }, [deferredQuery]);
  
  return (
    <ul>
      {results.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

---

#### Interruption & Restart

**Higher Priority Interrupts Lower Priority:**

```jsx
function App() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleSearch = (e) => {
    const value = e.target.value;
    
    // High priority
    setQuery(value);
    
    // Low priority (can be interrupted)
    startTransition(() => {
      const filtered = expensiveFilter(value);
      setResults(filtered);
    });
  };
  
  return (
    <>
      <input value={query} onChange={handleSearch} />
      <Results items={results} />
    </>
  );
}

// Timeline:
// 0ms: User types "a"
//   → High priority: Update input
//   → Start low priority: Filter for "a"
// 5ms: Yield to browser
// 10ms: Resume filtering
// 15ms: User types "ab" (new input!)
//   → INTERRUPT current filter
//   → High priority: Update input to "ab"
//   → DISCARD work for "a"
//   → START NEW filter for "ab"
// Result: No wasted work, input stays responsive
```

---

#### Visual Indicators

**Show Pending State:**

```jsx
function App() {
  const [query, setQuery] = useState('');
  const [items, setItems] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleSearch = (e) => {
    setQuery(e.target.value);
    
    startTransition(() => {
      const results = searchItems(e.target.value);
      setItems(results);
    });
  };
  
  return (
    <div>
      <input value={query} onChange={handleSearch} />
      
      {/* Show loading state during transition */}
      {isPending && <Spinner />}
      
      {/* Optionally dim content */}
      <div style={{ opacity: isPending ? 0.5 : 1 }}>
        <Results items={items} />
      </div>
    </div>
  );
}
```

---

#### Performance Comparison

**Measuring Impact:**

```jsx
import { Profiler } from 'react';

function App() {
  const handleRender = (id, phase, actualDuration) => {
    console.log({
      component: id,
      phase,
      renderTime: actualDuration,
    });
  };
  
  return (
    <Profiler id="App" onRender={handleRender}>
      <LargeList />
    </Profiler>
  );
}

// Without time slicing:
// { component: 'App', phase: 'update', renderTime: 120 }
// Single 120ms render (blocks UI)

// With time slicing:
// { component: 'App', phase: 'update', renderTime: 30 } (frame 1)
// { component: 'App', phase: 'update', renderTime: 30 } (frame 2)
// { component: 'App', phase: 'update', renderTime: 30 } (frame 3)
// { component: 'App', phase: 'update', renderTime: 30 } (frame 4)
// Total: 120ms, but spread across frames (UI responsive)
```

---

#### Limitations

**When Time Slicing Doesn't Help:**

```jsx
// 1. Commit phase is synchronous (can't be interrupted)
function App() {
  const [items, setItems] = useState([]);
  
  useEffect(() => {
    // This runs synchronously during commit
    items.forEach(item => {
      document.getElementById(item.id).scrollIntoView();
    });
  }, [items]);
  
  // Long commit phase = UI frozen
}

// 2. Expensive pure computations
function Component() {
  // This computation happens during render
  // Time slicing helps, but computation itself is still slow
  const result = expensiveCalculation(); // 100ms
  
  return <div>{result}</div>;
}

// Solution: Move to Web Worker
const worker = new Worker('worker.js');
worker.postMessage({ type: 'calculate' });

// 3. Synchronous state updates
function handleClick() {
  // Both updates are synchronous (high priority)
  setCount(c => c + 1);
  setName('New');
  
  // No time slicing (intentionally)
}
```

---

#### Interview Tips

✅ **Key Points:**
- Time slicing splits rendering into chunks across frames
- Prevents long renders from blocking user input/animations
- Enabled by Fiber architecture and concurrent rendering
- Typical chunk: ~5ms, yields to browser between chunks
- Use startTransition/useTransition to enable for updates
- Higher priority updates can interrupt lower priority ones
- Commit phase still synchronous (safety requirement)

✅ **When to Mention:**
- Concurrent rendering discussions
- Performance optimization questions
- startTransition/useTransition usage
- React 18 features
- User experience improvements

✅ **Common Follow-ups:**
- "How is time slicing different from code splitting?"
- "What's the chunk size?"
- "Can the commit phase be time-sliced?"
- "How do you enable time slicing?"
- "What happens if a higher priority update comes in?"

✅ **Perfect Answer Structure:**
1. Definition: Splitting rendering work into chunks across frames
2. Problem: Long renders block UI, poor UX
3. Solution: Process ~5ms chunks, yield to browser
4. How: Fiber's linked list enables pause/resume
5. Enable: startTransition, useTransition, useDeferredValue
6. Benefits: Smooth UI, responsive input, interruptible
7. Example: Search input stays responsive during filtering

</details>

---

### 195. How does React prioritize updates?

<details>
<summary>View Answer</summary>

**React prioritizes updates** based on their **source** and **urgency**, processing high-priority updates (user input) immediately while deferring low-priority updates (data fetching) to prevent blocking the UI. This ensures smooth, responsive user experiences even during heavy rendering.

#### Priority Categories

**Event-Based Priority Assignment:**

```javascript
// React automatically assigns priorities based on event types

const DiscreteEventPriority = SyncLane;        // Highest
const ContinuousEventPriority = InputContinuousLane;
const DefaultEventPriority = DefaultLane;
const IdleEventPriority = IdleLane;            // Lowest

// Event type → Priority mapping
const eventPriorities = {
  // Discrete events (user actions)
  click: DiscreteEventPriority,
  keydown: DiscreteEventPriority,
  keyup: DiscreteEventPriority,
  input: DiscreteEventPriority,
  submit: DiscreteEventPriority,
  
  // Continuous events
  mousemove: ContinuousEventPriority,
  scroll: ContinuousEventPriority,
  drag: ContinuousEventPriority,
  wheel: ContinuousEventPriority,
  
  // Default (programmatic updates)
  setTimeout: DefaultEventPriority,
  promise: DefaultEventPriority,
  
  // Idle
  requestIdleCallback: IdleEventPriority,
};
```

---

#### Priority Levels in Practice

**1. Synchronous (Immediate) Priority:**

```jsx
function App() {
  const [value, setValue] = useState('');
  
  // User input → Immediate priority
  const handleInput = (e) => {
    setValue(e.target.value);
    // Updates SYNCHRONOUSLY
    // React blocks to ensure input feels instant
  };
  
  return <input value={value} onChange={handleInput} />;
}

// Why immediate?
// Users expect instant feedback when typing
// Any delay feels broken/laggy
```

**2. User-Blocking Priority:**

```jsx
function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  // Mouse move → User-blocking priority
  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
    // High priority, but can skip frames if needed
  };
  
  return (
    <div onMouseMove={handleMouseMove}>
      <Cursor x={position.x} y={position.y} />
    </div>
  );
}

// Why user-blocking?
// Needs to feel responsive
// But can afford minor delays
```

**3. Default Priority:**

```jsx
function App() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    // Network response → Default priority
    fetch('/api/data')
      .then(res => res.json())
      .then(data => {
        setData(data);
        // Default priority
        // Not urgent, can wait
      });
  }, []);
  
  return <div>{data && <DataDisplay data={data} />}</div>;
}

// Why default?
// User isn't waiting for this specific update
// Can be interrupted by user actions
```

**4. Transition Priority (Low):**

```jsx
import { startTransition } from 'react';

function App() {
  const [tab, setTab] = useState('home');
  
  const switchTab = (newTab) => {
    // Mark as low priority
    startTransition(() => {
      setTab(newTab);
    });
  };
  
  return (
    <>
      <button onClick={() => switchTab('home')}>Home</button>
      <button onClick={() => switchTab('profile')}>Profile</button>
      <TabContent tab={tab} />
    </>
  );
}

// Why low priority?
// Navigation can show loading state
// OK to take time for smooth experience
// Can be interrupted by urgent updates
```

**5. Idle Priority:**

```jsx
function App() {
  const [analytics, setAnalytics] = useState([]);
  
  useEffect(() => {
    // Analytics → Idle priority
    requestIdleCallback(() => {
      const data = processAnalytics();
      setAnalytics(data);
      // Lowest priority
      // Only runs when browser is idle
    });
  }, []);
  
  return <div>Main content</div>;
}

// Why idle?
// Not visible to user
// Can happen anytime
// Should never block important work
```

---

#### Real-World Example: Search with Prioritization

**Without Prioritization:**

```jsx
function SearchApp() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleSearch = (e) => {
    const value = e.target.value;
    
    // Both updates same priority
    setQuery(value);
    
    // Expensive filtering
    const filtered = largeDataset.filter(item =>
      item.name.toLowerCase().includes(value.toLowerCase())
    );
    setResults(filtered);
  };
  
  return (
    <>
      <input value={query} onChange={handleSearch} />
      <Results items={results} />
    </>
  );
}

// Problem:
// Input feels laggy (both updates processed together)
// User sees delay when typing
// Poor experience
```

**With Prioritization:**

```jsx
import { startTransition } from 'react';

function SearchApp() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const handleSearch = (e) => {
    const value = e.target.value;
    
    // HIGH PRIORITY: Update input immediately
    setQuery(value);
    
    // LOW PRIORITY: Filter results (interruptible)
    startTransition(() => {
      const filtered = largeDataset.filter(item =>
        item.name.toLowerCase().includes(value.toLowerCase())
      );
      setResults(filtered);
    });
  };
  
  return (
    <>
      <input value={query} onChange={handleSearch} />
      <Results items={results} />
    </>
  );
}

// Benefits:
// Input updates instantly (high priority)
// Filtering happens in background (low priority)
// If user types again, filtering is interrupted
// Smooth, responsive experience
```

---

#### Priority Assignment Process

**1. Determine Event Priority:**

```javascript
function dispatchEvent(event) {
  // Get priority based on event type
  const priority = getEventPriority(event.type);
  
  // Examples:
  // click → DiscreteEventPriority (sync)
  // mousemove → ContinuousEventPriority
  // timeout → DefaultEventPriority
  
  return priority;
}

function getEventPriority(type) {
  switch (type) {
    case 'click':
    case 'keydown':
    case 'input':
      return DiscreteEventPriority;
    
    case 'mousemove':
    case 'scroll':
      return ContinuousEventPriority;
    
    default:
      return DefaultEventPriority;
  }
}
```

**2. Convert Priority to Lane:**

```javascript
function requestUpdateLane(fiber) {
  // Get current event priority
  const eventPriority = getCurrentUpdatePriority();
  
  // Check if inside transition
  if (isInsideTransition) {
    return TransitionLane;
  }
  
  // Convert event priority to lane
  switch (eventPriority) {
    case DiscreteEventPriority:
      return SyncLane;
    
    case ContinuousEventPriority:
      return InputContinuousLane;
    
    case DefaultEventPriority:
      return DefaultLane;
    
    case IdleEventPriority:
      return IdleLane;
    
    default:
      return DefaultLane;
  }
}
```

**3. Schedule Update with Lane:**

```javascript
function scheduleUpdateOnFiber(fiber, lane) {
  // Mark fiber with lane
  fiber.lanes |= lane;
  
  // Bubble priority up to root
  let parent = fiber.return;
  while (parent !== null) {
    parent.childLanes |= lane;
    parent = parent.return;
  }
  
  // Schedule root with this priority
  ensureRootIsScheduled(root, lane);
}
```

---

#### Priority Inversion Handling

**Problem: Low Priority Update Blocks High Priority:**

```jsx
function App() {
  const [count, setCount] = useState(0);
  const [heavy, setHeavy] = useState([]);
  
  const handleClick = () => {
    // Low priority: Heavy computation
    startTransition(() => {
      const result = expensiveComputation();
      setHeavy(result);
    });
    
    // High priority: Simple counter
    // But must wait for heavy computation in same component!
    setCount(c => c + 1);
  };
  
  return (
    <>
      <button onClick={handleClick}>Click: {count}</button>
      <HeavyComponent data={heavy} />
    </>
  );
}

// Solution: Split components
function App() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        Click: {count}
      </button>
      <HeavyComponentWrapper />
    </>
  );
}

function HeavyComponentWrapper() {
  const [heavy, setHeavy] = useState([]);
  
  const handleUpdate = () => {
    startTransition(() => {
      setHeavy(expensiveComputation());
    });
  };
  
  return <HeavyComponent data={heavy} onUpdate={handleUpdate} />;
}

// Counter updates independently (high priority)
// Heavy computation separate (low priority)
```

---

#### Priority Inheritance

**Context Updates Inherit Parent Priority:**

```jsx
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    // High priority click event
    setTheme(t => t === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={toggleTheme}>Toggle Theme</button>
      <DeepChild />
    </ThemeContext.Provider>
  );
}

function DeepChild() {
  // This update inherits high priority from parent
  const theme = useContext(ThemeContext);
  return <div className={theme}>Content</div>;
}

// All consumers update with same priority as producer
```

---

#### Batching and Priority

**Same Priority Updates Batch Together:**

```jsx
function App() {
  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);
  const [name, setName] = useState('');
  
  const handleClick = () => {
    // All same priority (DiscreteEvent)
    setCount(c => c + 1);
    setFlag(f => !f);
    setName('Updated');
    
    // Batched into single render
    // All processed together
  };
  
  return <button onClick={handleClick}>Update</button>;
}

// Result: 1 render, not 3
```

**Different Priority Updates Don't Batch:**

```jsx
function App() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);
  
  const handleClick = () => {
    // High priority
    setCount(c => c + 1);
    
    // Low priority
    startTransition(() => {
      setItems(generateItems());
    });
  };
  
  return (
    <>
      <div>Count: {count}</div>
      <List items={items} />
    </>
  );
}

// Result: 2 separate renders
// 1. High priority: count updates immediately
// 2. Low priority: items update later (interruptible)
```

---

#### Priority Starvation Prevention

**Expiration Escalates Priority:**

```javascript
function scheduleUpdate(fiber, lane) {
  const currentTime = getCurrentTime();
  
  // Calculate expiration based on lane
  const timeout = getTimeoutForLane(lane);
  const expirationTime = currentTime + timeout;
  
  fiber.expirationTime = expirationTime;
  
  // Lane-based timeouts:
  // SyncLane: Immediate (no timeout)
  // InputContinuousLane: 250ms
  // DefaultLane: 5000ms
  // TransitionLane: 10000ms
  // IdleLane: Never expires
}

function shouldForceSync(fiber) {
  const currentTime = getCurrentTime();
  
  // If update expired, force to sync priority
  if (currentTime >= fiber.expirationTime) {
    return true; // Escalate to sync
  }
  
  return false;
}

// Example:
// 1. Low priority update scheduled (10s timeout)
// 2. High priority updates keep coming
// 3. After 10 seconds, low priority expires
// 4. Escalated to sync priority
// 5. Must be processed immediately
// Result: Prevents starvation
```

---

#### useTransition Hook

**Mark Updates as Non-Urgent:**

```jsx
import { useTransition } from 'react';

function TabContainer() {
  const [activeTab, setActiveTab] = useState('home');
  const [isPending, startTransition] = useTransition();
  
  const selectTab = (tab) => {
    startTransition(() => {
      // This update is low priority
      setActiveTab(tab);
    });
  };
  
  return (
    <>
      <button onClick={() => selectTab('home')}>
        Home
      </button>
      <button onClick={() => selectTab('profile')}>
        Profile {isPending && '...'}
      </button>
      
      {/* Tab content can show loading state */}
      <div style={{ opacity: isPending ? 0.5 : 1 }}>
        <TabContent tab={activeTab} />
      </div>
    </>
  );
}
```

---

#### useDeferredValue Hook

**Defer Expensive Computations:**

```jsx
import { useDeferredValue, useMemo } from 'react';

function SearchResults({ query }) {
  // Deferred value updates with low priority
  const deferredQuery = useDeferredValue(query);
  
  // Expensive computation uses deferred value
  const results = useMemo(() => {
    return largeDataset.filter(item =>
      item.name.includes(deferredQuery)
    );
  }, [deferredQuery]);
  
  return (
    <div>
      {/* Show stale results while deferredQuery updates */}
      {results.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}

function App() {
  const [query, setQuery] = useState('');
  
  return (
    <>
      {/* Input updates immediately (high priority) */}
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
      />
      
      {/* Results update with low priority */}
      <SearchResults query={query} />
    </>
  );
}
```

---

#### Priority Decision Tree

```javascript
function determinePriority(update) {
  // 1. Inside startTransition?
  if (isInsideTransition) {
    return TransitionPriority; // Low
  }
  
  // 2. User interaction?
  if (isUserInteraction(update)) {
    if (isDiscrete(update)) {
      return SyncPriority; // Immediate
    }
    return ContinuousEventPriority; // High
  }
  
  // 3. Async update (fetch, timeout)?
  if (isAsyncUpdate(update)) {
    return DefaultPriority; // Normal
  }
  
  // 4. Background work?
  if (isIdleCallback(update)) {
    return IdlePriority; // Lowest
  }
  
  // Default
  return DefaultPriority;
}
```

---

#### Debugging Priority Issues

**React DevTools Profiler:**

```jsx
import { Profiler } from 'react';

function App() {
  return (
    <Profiler
      id="App"
      onRender={(id, phase, actualDuration, baseDuration, startTime) => {
        console.log({
          component: id,
          phase,
          duration: actualDuration,
          startTime,
        });
      }}
    >
      <MyComponent />
    </Profiler>
  );
}

// Check render timing and frequency
// Identify slow updates
// Verify priority is correct
```

---

#### Interview Tips

✅ **Key Points:**
- React assigns priority based on event source (click, scroll, fetch)
- 5 priority levels: Sync, User-blocking, Default, Transition, Idle
- User input gets highest priority (immediate/sync)
- startTransition marks updates as low priority
- Priority prevents low-priority work from blocking urgent updates
- Automatic batching for same-priority updates
- Expiration prevents starvation

✅ **When to Mention:**
- Concurrent rendering discussions
- Performance optimization questions
- startTransition/useTransition usage
- User experience improvements
- Update batching behavior

✅ **Common Follow-ups:**
- "What determines an update's priority?"
- "How do you change update priority?"
- "What's priority starvation and how is it prevented?"
- "Do same-priority updates batch?"
- "How does startTransition affect priority?"

✅ **Perfect Answer Structure:**
1. Purpose: Ensure responsive UI by processing urgent updates first
2. Assignment: Based on event source (click=high, fetch=normal)
3. Levels: Sync, User-blocking, Default, Transition, Idle
4. Control: Use startTransition to mark updates as low priority
5. Starvation: Expired updates escalate to sync priority
6. Example: Input updates immediately, filtering deferred
7. Benefit: Smooth UX even during heavy rendering

</details>

---

### 196. What is the lane model in React?

<details>
<summary>View Answer</summary>

**The lane model** is React's system for representing and managing **work priorities** using **binary lanes** (bit flags). It replaced the expiration time model in React 18, providing more efficient priority handling, better batching, and enabling concurrent features like transitions and Suspense.

#### What Are Lanes?

**Binary Representation of Priority:**

```javascript
// Each lane is a bit position (31 lanes total)
const SyncLane =                0b0000000000000000000000000000001; // Lane 0
const InputContinuousLane =     0b0000000000000000000000000000100; // Lane 2
const DefaultLane =             0b0000000000000000000000000010000; // Lane 4
const TransitionLane1 =         0b0000000000000000000000001000000; // Lane 6
const TransitionLane2 =         0b0000000000000000000000010000000; // Lane 7
// ... more lanes
const IdleLane =                0b0100000000000000000000000000000; // Lane 30

// Multiple lanes can be represented in a single number
const multipleLanes = SyncLane | DefaultLane;
// 0b0000000000000000000000000010001
//  Both bit 0 and bit 4 are set
```

**Why Binary?**

```javascript
// Extremely fast operations using bitwise operators

// Check if lane has work: O(1)
const hasWork = (lanes & SyncLane) !== 0;

// Add lane: O(1)
lanes |= DefaultLane;

// Remove lane: O(1)
lanes &= ~DefaultLane;

// Get highest priority lane: O(1)
const highestLane = lanes & -lanes;

// Merge lanes: O(1)
const combined = lanesA | lanesB;

// Compare with old system:
// Expiration times required sorting, comparisons O(log n)
// Lanes use bit operations O(1)
```

---

#### Lane Hierarchy

**Priority from Highest to Lowest:**

```javascript
// Synchronous (immediate)
const SyncLane = 0b0000000000000000000000000000001;

// Continuous input (hover, drag)
const InputContinuousHydrationLane = 0b0000000000000000000000000000010;
const InputContinuousLane =          0b0000000000000000000000000000100;

// Default hydration and updates
const DefaultHydrationLane = 0b0000000000000000000000000001000;
const DefaultLane =          0b0000000000000000000000000010000;

// Transitions (non-urgent updates)
const TransitionHydrationLane = 0b0000000000000000000000000100000;
const TransitionLanes = [
  0b0000000000000000000000001000000, // TransitionLane1
  0b0000000000000000000000010000000, // TransitionLane2
  0b0000000000000000000000100000000, // TransitionLane3
  // ... up to TransitionLane16
];

// Retry lanes (for Suspense boundaries)
const RetryLanes = [
  0b0000000000000010000000000000000,
  0b0000000000000100000000000000000,
  0b0000000000001000000000000000000,
  0b0000000000010000000000000000000,
  0b0000000000100000000000000000000,
];

// Selection (text selection updates)
const SelectiveHydrationLane = 0b0000000001000000000000000000000;

// Idle (lowest priority)
const IdleHydrationLane = 0b0000000010000000000000000000000;
const IdleLane =          0b0100000000000000000000000000000;

// Offscreen (hidden content)
const OffscreenLane = 0b1000000000000000000000000000000;
```

---

#### Lane Operations

**Adding Lanes:**

```javascript
function scheduleUpdate(fiber, lane) {
  // Add lane to fiber
  fiber.lanes |= lane;
  
  // Bubble up to root
  let parent = fiber.return;
  while (parent !== null) {
    parent.childLanes |= lane;
    parent = parent.return;
  }
}

// Example:
const fiber = { lanes: 0b0000 };

// Add SyncLane (0b0001)
fiber.lanes |= SyncLane;
// fiber.lanes = 0b0001

// Add DefaultLane (0b0100)
fiber.lanes |= DefaultLane;
// fiber.lanes = 0b0101 (both lanes present)
```

**Checking Lanes:**

```javascript
function hasLane(lanes, lane) {
  return (lanes & lane) !== 0;
}

// Example:
const lanes = 0b0101; // Has SyncLane and DefaultLane

hasLane(lanes, SyncLane);    // true (0b0101 & 0b0001 = 0b0001)
hasLane(lanes, DefaultLane); // true (0b0101 & 0b0100 = 0b0100)
hasLane(lanes, IdleLane);    // false (0b0101 & 0b0010 = 0b0000)
```

**Removing Lanes:**

```javascript
function removeLane(lanes, lane) {
  return lanes & ~lane;
}

// Example:
let lanes = 0b0101; // Has SyncLane and DefaultLane

// Remove SyncLane
lanes = removeLane(lanes, SyncLane);
// lanes = 0b0100 (only DefaultLane remains)

// Bitwise NOT (~) flips all bits:
// SyncLane =  0b0001
// ~SyncLane = 0b1110 (all bits except bit 0)
// 0b0101 & 0b1110 = 0b0100
```

**Getting Highest Priority Lane:**

```javascript
function getHighestPriorityLane(lanes) {
  // Two's complement trick: isolates rightmost set bit
  return lanes & -lanes;
}

// Example:
const lanes = 0b0110; // Lanes 1 and 2 set

// How it works:
// lanes =  0b0110
// -lanes = 0b1010 (two's complement)
// lanes & -lanes = 0b0010 (rightmost bit)

const highest = getHighestPriorityLane(lanes);
// highest = 0b0010 (Lane 1, highest priority in this set)
```

**Including Lanes:**

```javascript
function includesNonIdleWork(lanes) {
  return (lanes & NonIdleLanes) !== 0;
}

function includesBlockingLane(lanes) {
  return (
    (lanes & SyncLane) !== 0 ||
    (lanes & InputContinuousLane) !== 0
  );
}

function includesTransitionLane(lanes) {
  return (lanes & TransitionLanes) !== 0;
}
```

---

#### Lane Sets

**Predefined Lane Groups:**

```javascript
// No lanes
const NoLanes = 0b0000000000000000000000000000000;

// All non-idle work
const NonIdleLanes = 0b0001111111111111111111111111111;

// All transition lanes
const TransitionLanes = 0b0000000000000001111111110000000;

// All retry lanes (Suspense)
const RetryLanes = 0b0000000111110000000000000000000;

// Blocking lanes (must run soon)
const BlockingLanes = SyncLane | InputContinuousLane | DefaultLane;

// Example usage:
if ((lanes & TransitionLanes) !== 0) {
  // Has at least one transition lane
  console.log('Transition work pending');
}
```

---

#### Priority Mapping

**Event Priority to Lane:**

```javascript
function eventPriorityToLane(priority) {
  switch (priority) {
    case DiscreteEventPriority:
      return SyncLane;
    
    case ContinuousEventPriority:
      return InputContinuousLane;
    
    case DefaultEventPriority:
      return DefaultLane;
    
    case IdleEventPriority:
      return IdleLane;
    
    default:
      return DefaultLane;
  }
}

// Usage:
const lane = eventPriorityToLane(DiscreteEventPriority);
// lane = SyncLane (0b0001)
```

**Lane to Scheduler Priority:**

```javascript
function lanesToSchedulerPriority(lanes) {
  const highestLane = getHighestPriorityLane(lanes);
  
  if ((highestLane & SyncLane) !== 0) {
    return ImmediatePriority;
  }
  
  if ((highestLane & InputContinuousLane) !== 0) {
    return UserBlockingPriority;
  }
  
  if ((highestLane & (DefaultLane | TransitionLanes)) !== 0) {
    return NormalPriority;
  }
  
  return IdlePriority;
}
```

---

#### Transition Lanes

**Multiple Concurrent Transitions:**

```javascript
// React maintains a pool of transition lanes
const transitionLanePool = [
  TransitionLane1,  // 0b0000000000000000000000001000000
  TransitionLane2,  // 0b0000000000000000000000010000000
  TransitionLane3,  // 0b0000000000000000000000100000000
  // ... up to TransitionLane16
];

let nextTransitionLane = 0;

function requestTransitionLane() {
  // Get next available transition lane
  const lane = transitionLanePool[nextTransitionLane];
  nextTransitionLane = (nextTransitionLane + 1) % transitionLanePool.length;
  return lane;
}

// Why multiple transition lanes?
// Allows multiple concurrent transitions
// Each can be interrupted/resumed independently
```

**Example with Multiple Transitions:**

```jsx
function App() {
  const [tab1, setTab1] = useState('home');
  const [tab2, setTab2] = useState('profile');
  
  const switchTab1 = (tab) => {
    startTransition(() => {
      setTab1(tab); // Uses TransitionLane1
    });
  };
  
  const switchTab2 = (tab) => {
    startTransition(() => {
      setTab2(tab); // Uses TransitionLane2
    });
  };
  
  // Both transitions can progress independently
  // If user clicks switchTab1 again, only Tab1 transition restarts
  // Tab2 transition continues unaffected
}
```

---

#### Lane Entanglement

**Tracking Dependencies Between Updates:**

```javascript
// When Suspense boundary suspends, entangle lanes
function markRootEntangled(root, suspendedLanes) {
  // Any work in suspendedLanes now depends on these lanes
  root.entangledLanes |= suspendedLanes;
  
  // Future updates must include entangled lanes
}

// Example:
// 1. Transition starts (TransitionLane1)
// 2. Component suspends (data loading)
// 3. Mark TransitionLane1 as entangled
// 4. Other lanes must wait for TransitionLane1 to complete
```

---

#### Complete Update Flow

**From Event to Lane:**

```jsx
function App() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    setCount(c => c + 1);
  };
  
  return <button onClick={handleClick}>Count: {count}</button>;
}

// Flow:

// 1. User clicks button
handleClick();

// 2. Determine event priority
const priority = DiscreteEventPriority; // Click event

// 3. Convert to lane
const lane = eventPriorityToLane(priority);
// lane = SyncLane (0b0001)

// 4. Schedule update
scheduleUpdate(fiber, lane);

// 5. Mark fiber with lane
fiber.lanes |= lane;
// fiber.lanes = 0b0001

// 6. Bubble to root
let parent = fiber.return;
while (parent) {
  parent.childLanes |= lane;
  parent = parent.return;
}

// 7. Ensure root scheduled
ensureRootIsScheduled(root);

// 8. Process lanes
const nextLanes = getNextLanes(root);
// Returns: SyncLane (highest priority)

// 9. Render
renderRootSync(root, nextLanes);

// 10. Commit
commitRoot(root);

// 11. Clear lanes
fiber.lanes = removeLane(fiber.lanes, lane);
// fiber.lanes = 0b0000
```

---

#### Suspense and Retry Lanes

**Exponential Backoff with Lanes:**

```javascript
const RetryLanes = [
  RetryLane1,  // Retry after ~100ms
  RetryLane2,  // Retry after ~200ms
  RetryLane3,  // Retry after ~400ms
  RetryLane4,  // Retry after ~800ms
  RetryLane5,  // Retry after ~1600ms
];

let currentRetryLane = 0;

function getSuspenseRetryLane() {
  const lane = RetryLanes[currentRetryLane];
  currentRetryLane = Math.min(
    currentRetryLane + 1,
    RetryLanes.length - 1
  );
  return lane;
}

// Example:
// 1st suspend: RetryLane1 (~100ms)
// 2nd suspend: RetryLane2 (~200ms)
// 3rd suspend: RetryLane3 (~400ms)
// Exponential backoff prevents excessive retries
```

---

#### Debugging Lanes

**Lane Names for Logging:**

```javascript
function getLaneName(lane) {
  if (lane === SyncLane) return 'SyncLane';
  if (lane === InputContinuousLane) return 'InputContinuousLane';
  if (lane === DefaultLane) return 'DefaultLane';
  if ((lane & TransitionLanes) !== 0) {
    const index = Math.log2(lane & TransitionLanes) - 6;
    return `TransitionLane${index + 1}`;
  }
  if (lane === IdleLane) return 'IdleLane';
  return `Lane(0b${lane.toString(2)})`;
}

// Usage:
console.log('Processing lanes:', getLaneName(fiber.lanes));
// Output: "Processing lanes: SyncLane"
```

**Visualize Lane State:**

```javascript
function logLanes(lanes) {
  console.log('Lanes:', lanes.toString(2).padStart(31, '0'));
  console.log('Has SyncLane:', hasLane(lanes, SyncLane));
  console.log('Has DefaultLane:', hasLane(lanes, DefaultLane));
  console.log('Has TransitionLane:', hasLane(lanes, TransitionLanes));
  console.log('Highest priority:', getLaneName(getHighestPriorityLane(lanes)));
}

// Example output:
// Lanes: 0000000000000000000000000010001
// Has SyncLane: true
// Has DefaultLane: true
// Has TransitionLane: false
// Highest priority: SyncLane
```

---

#### Benefits Over Expiration Times

**Old System (React 16-17): Expiration Times**

```javascript
// Problems with expiration times:
const expirationTime1 = 1000;
const expirationTime2 = 1500;
const expirationTime3 = 2000;

// 1. Required sorting
const sorted = [expirationTime1, expirationTime2, expirationTime3].sort();

// 2. Hard to batch
if (expirationTime1 === expirationTime2) {
  // Can batch
}

// 3. Limited granularity
// Only ~30 different expiration buckets

// 4. Difficult to check multiple priorities
if (expirationTime <= currentTime) {
  // Must process
}
```

**New System (React 18+): Lanes**

```javascript
// Benefits:
const lanes = SyncLane | DefaultLane | TransitionLane1;

// 1. No sorting needed
const highest = lanes & -lanes; // O(1)

// 2. Easy batching
const combined = lanesA | lanesB; // O(1)

// 3. 31 distinct priorities
// Much more granular

// 4. Fast multi-priority checks
if ((lanes & (SyncLane | InputContinuousLane)) !== 0) {
  // Has high priority work
}

// 5. Independent transitions
// Each transition lane can progress separately
```

---

#### Interview Tips

✅ **Key Points:**
- Lanes are binary representation of work priorities
- 31 lanes total, each lane is a bit position
- Bitwise operations (OR, AND, NOT) are O(1) and extremely fast
- Replaced expiration time model in React 18
- Multiple transition lanes allow concurrent transitions
- Enables efficient batching and priority management
- Foundation for concurrent features

✅ **When to Mention:**
- React 18+ internals discussions
- Priority and scheduling questions
- Performance optimization deep dives
- Concurrent rendering architecture
- Transition and Suspense implementation

✅ **Common Follow-ups:**
- "Why use binary lanes instead of numbers?"
- "How many lanes does React have?"
- "What's the difference between lanes and expiration times?"
- "How do transition lanes work?"
- "How does React determine which lane to use?"

✅ **Perfect Answer Structure:**
1. Definition: Binary representation of work priorities
2. Why Binary: Bitwise operations are O(1) and very fast
3. Structure: 31 lanes, each bit is a priority level
4. Operations: OR to add, AND to check, & -lanes for highest
5. Benefits: Faster than expiration times, better batching
6. Transitions: Multiple lanes allow concurrent transitions
7. Example: Show lane operations with binary numbers

</details>

---

### 197. How does React's commit phase work?

<details>
<summary>View Answer</summary>

**The commit phase** is the second phase of React's rendering process where React applies changes to the DOM. It's **synchronous and uninterruptible**, ensuring all DOM updates happen atomically to maintain consistency. The commit phase consists of three sub-phases: before mutation, mutation, and layout.

#### Two-Phase Rendering

**Overview:**

```javascript
// PHASE 1: RENDER PHASE (Interruptible)
// - Build work-in-progress fiber tree
// - Calculate what needs to change
// - Mark effects on fibers
// - Can be paused, aborted, restarted
// - No side effects allowed

function renderPhase() {
  workInProgress = createWorkInProgress(current);
  
  while (workInProgress !== null && !shouldYield()) {
    workInProgress = performUnitOfWork(workInProgress);
  }
  
  // Result: Complete fiber tree with effect flags
}

// PHASE 2: COMMIT PHASE (Synchronous, Uninterruptible)
// - Apply changes to DOM
// - Run lifecycle methods
// - Execute effects
// - Must complete without interruption
// - Side effects happen here

function commitPhase() {
  commitBeforeMutationEffects();  // Pre-mutation
  commitMutationEffects();        // Mutation (DOM updates)
  // Switch trees here
  commitLayoutEffects();          // Post-mutation (layout)
  // Schedule passive effects (useEffect)
}
```

---

#### Why Commit Phase Must Be Synchronous

**Consistency Requirements:**

```jsx
function Component() {
  const [count, setCount] = useState(0);
  const ref = useRef();
  
  useLayoutEffect(() => {
    // Must read consistent DOM state
    const width = ref.current.offsetWidth;
    console.log('Width:', width);
  });
  
  return <div ref={ref}>{count}</div>;
}

// If commit phase were interruptible:
// 1. Update DOM: count = 1
// 2. Pause (interrupted)
// 3. Another update: count = 2
// 4. Resume useLayoutEffect
// 5. Reads: count = 2 but expected 1
// Result: Inconsistent state!

// Solution: Commit phase is atomic
// All DOM updates + effects run together
// No interruptions = consistent state
```

---

#### Three Sub-Phases

**1. Before Mutation Phase:**

```javascript
function commitBeforeMutationEffects(root, firstChild) {
  // Traverse fiber tree with effects
  let fiber = firstChild;
  
  while (fiber !== null) {
    // Check for snapshot flag (getSnapshotBeforeUpdate)
    if (fiber.flags & Snapshot) {
      // Class components only
      const instance = fiber.stateNode;
      const snapshot = instance.getSnapshotBeforeUpdate(
        fiber.memoizedProps,
        fiber.memoizedState
      );
      instance.__reactInternalSnapshotBeforeUpdate = snapshot;
    }
    
    // Process children
    if (fiber.child !== null) {
      fiber = fiber.child;
      continue;
    }
    
    // Move to next fiber
    fiber = getNextFiber(fiber);
  }
}

// Example usage:
class ScrollList extends Component {
  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Capture scroll position BEFORE DOM updates
    if (prevProps.items.length < this.props.items.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }
  
  componentDidUpdate(prevProps, prevState, snapshot) {
    // Use snapshot AFTER DOM updates
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }
}
```

**2. Mutation Phase (DOM Updates):**

```javascript
function commitMutationEffects(root, firstChild) {
  let fiber = firstChild;
  
  while (fiber !== null) {
    const flags = fiber.flags;
    
    // ContentReset: Clear text content
    if (flags & ContentReset) {
      commitResetTextContent(fiber);
    }
    
    // Ref: Detach old refs
    if (flags & Ref) {
      const current = fiber.alternate;
      if (current !== null) {
        commitDetachRef(current);
      }
    }
    
    // Primary effect: Placement, Update, or Deletion
    const primaryFlags = flags & (Placement | Update | Deletion);
    
    switch (primaryFlags) {
      case Placement: {
        // Insert new node into DOM
        commitPlacement(fiber);
        fiber.flags &= ~Placement;
        break;
      }
      
      case Update: {
        // Update existing node
        const current = fiber.alternate;
        commitWork(current, fiber);
        break;
      }
      
      case Deletion: {
        // Remove node from DOM
        commitDeletion(root, fiber);
        break;
      }
    }
    
    fiber = getNextFiber(fiber);
  }
}
```

**Placement (Insert Node):**

```javascript
function commitPlacement(fiber) {
  // Find parent DOM node
  const parentFiber = getHostParentFiber(fiber);
  const parentDOM = parentFiber.stateNode;
  
  // Get DOM node to insert
  const node = fiber.stateNode;
  
  // Find correct position (before sibling)
  const before = getHostSibling(fiber);
  
  if (before) {
    // Insert before sibling
    parentDOM.insertBefore(node, before);
  } else {
    // Append to end
    parentDOM.appendChild(node);
  }
}

// Example:
// Before: <div><span>A</span><span>C</span></div>
// Insert B between A and C
// After:  <div><span>A</span><span>B</span><span>C</span></div>
```

**Update (Modify Node):**

```javascript
function commitWork(current, fiber) {
  switch (fiber.tag) {
    case HostComponent: {
      // Update DOM element
      const instance = fiber.stateNode;
      const newProps = fiber.memoizedProps;
      const oldProps = current !== null ? current.memoizedProps : newProps;
      
      // Update properties
      updateDOMProperties(instance, oldProps, newProps);
      break;
    }
    
    case HostText: {
      // Update text content
      const textInstance = fiber.stateNode;
      const newText = fiber.memoizedProps;
      textInstance.nodeValue = newText;
      break;
    }
  }
}

function updateDOMProperties(domElement, oldProps, newProps) {
  // Remove old properties
  for (let propName in oldProps) {
    if (!(propName in newProps)) {
      // Property removed
      if (propName === 'style') {
        domElement.style = '';
      } else if (propName.startsWith('on')) {
        // Remove event listener
        const eventName = propName.toLowerCase().substring(2);
        domElement.removeEventListener(eventName, oldProps[propName]);
      } else {
        domElement.removeAttribute(propName);
      }
    }
  }
  
  // Add/update new properties
  for (let propName in newProps) {
    const newValue = newProps[propName];
    const oldValue = oldProps[propName];
    
    if (newValue !== oldValue) {
      if (propName === 'style') {
        // Update styles
        Object.assign(domElement.style, newValue);
      } else if (propName.startsWith('on')) {
        // Add event listener
        const eventName = propName.toLowerCase().substring(2);
        domElement.addEventListener(eventName, newValue);
      } else if (propName === 'children') {
        // Text children
        if (typeof newValue === 'string' || typeof newValue === 'number') {
          domElement.textContent = newValue;
        }
      } else {
        // Regular attribute
        domElement.setAttribute(propName, newValue);
      }
    }
  }
}
```

**Deletion (Remove Node):**

```javascript
function commitDeletion(root, fiber) {
  // Recursively unmount
  unmountHostComponents(fiber);
  
  // Detach from parent
  const parent = getHostParentFiber(fiber);
  const parentDOM = parent.stateNode;
  parentDOM.removeChild(fiber.stateNode);
}

function unmountHostComponents(fiber) {
  // Call lifecycle methods
  if (fiber.tag === ClassComponent) {
    fiber.stateNode.componentWillUnmount();
  }
  
  // Cleanup refs
  if (fiber.ref !== null) {
    fiber.ref.current = null;
  }
  
  // Cleanup effects
  if (fiber.flags & Passive) {
    // Schedule useEffect cleanup
    schedulePassiveEffectCleanup(fiber);
  }
  
  // Recurse to children
  let child = fiber.child;
  while (child !== null) {
    unmountHostComponents(child);
    child = child.sibling;
  }
}
```

**3. Layout Phase:**

```javascript
function commitLayoutEffects(root, committedLanes) {
  let fiber = root.current.child;
  
  while (fiber !== null) {
    const flags = fiber.flags;
    
    // Attach refs
    if (flags & Ref) {
      commitAttachRef(fiber);
    }
    
    // Call lifecycle methods and layout effects
    if (flags & (Update | Callback)) {
      switch (fiber.tag) {
        case ClassComponent: {
          const instance = fiber.stateNode;
          
          if (fiber.flags & Update) {
            if (fiber.alternate === null) {
              // Mount
              instance.componentDidMount();
            } else {
              // Update
              const prevProps = fiber.alternate.memoizedProps;
              const prevState = fiber.alternate.memoizedState;
              const snapshot = instance.__reactInternalSnapshotBeforeUpdate;
              instance.componentDidUpdate(prevProps, prevState, snapshot);
            }
          }
          
          // Process update queue (setState callbacks)
          if (fiber.flags & Callback) {
            processUpdateQueue(fiber);
          }
          break;
        }
        
        case FunctionComponent: {
          // useLayoutEffect
          if (fiber.flags & Layout) {
            commitHookEffectListMount(HookLayout, fiber);
          }
          break;
        }
      }
    }
    
    fiber = getNextFiber(fiber);
  }
}

function commitAttachRef(fiber) {
  const ref = fiber.ref;
  if (ref !== null) {
    const instance = fiber.stateNode;
    
    // Set ref
    if (typeof ref === 'function') {
      ref(instance);
    } else {
      ref.current = instance;
    }
  }
}
```

---

#### Tree Switching

**Double Buffering:**

```javascript
// After mutation phase, before layout phase
function commitRoot(root) {
  // ... before mutation
  commitBeforeMutationEffects(root, finishedWork);
  
  // ... mutation (DOM updates)
  commitMutationEffects(root, finishedWork);
  
  // SWITCH TREES (atomic pointer swap)
  root.current = finishedWork;
  // Now: finishedWork is current
  //      old current becomes work-in-progress
  
  // ... layout
  commitLayoutEffects(root, lanes);
  
  // Schedule passive effects
  schedulePassiveEffects(root, finishedWork);
}

// Why switch here?
// - DOM is updated
// - Layout effects need to read NEW tree
// - Refs need to point to NEW tree
// - componentDidMount/Update see NEW state
```

---

#### Passive Effects (useEffect)

**Scheduled After Paint:**

```javascript
function commitRootImpl(root, lanes) {
  // ... before mutation, mutation, layout
  
  // Check if there are passive effects
  if (
    (finishedWork.flags & Passive) !== NoFlags ||
    (finishedWork.subtreeFlags & Passive) !== NoFlags
  ) {
    // Schedule passive effects
    if (!rootDoesHavePassiveEffects) {
      rootDoesHavePassiveEffects = true;
      
      // Schedule after paint
      scheduleCallback(NormalPriority, () => {
        flushPassiveEffects();
        return null;
      });
    }
  }
  
  // Commit phase complete
}

function flushPassiveEffects() {
  // 1. Run cleanup functions (old effects)
  commitPassiveUnmountEffects(root.current);
  
  // 2. Run effect functions (new effects)
  commitPassiveMountEffects(root, root.current);
}

// Timeline:
// 1. Render phase
// 2. Commit phase (before mutation, mutation, layout)
// 3. Browser paint
// 4. Passive effects (useEffect)
```

**Example:**

```jsx
function Component() {
  const [count, setCount] = useState(0);
  
  useLayoutEffect(() => {
    console.log('1. Layout effect (sync, before paint)');
    return () => console.log('1. Layout cleanup');
  });
  
  useEffect(() => {
    console.log('2. Passive effect (async, after paint)');
    return () => console.log('2. Passive cleanup');
  });
  
  return <div>{count}</div>;
}

// Execution order on mount:
// - Render phase
// - Commit: before mutation
// - Commit: mutation (DOM updated)
// - Commit: layout
//   → "1. Layout effect (sync, before paint)"
// - Browser paints
// - Passive effects
//   → "2. Passive effect (async, after paint)"

// On unmount:
// - "1. Layout cleanup" (during commit)
// - "2. Passive cleanup" (after paint)
```

---

#### Effect Flags

**Types of Effects:**

```javascript
const NoFlags = 0b000000000000000000;
const Placement = 0b000000000000000010;    // New node
const Update = 0b000000000000000100;       // Props/state changed
const Deletion = 0b000000000000001000;     // Node removed
const Snapshot = 0b000000000000100000;     // getSnapshotBeforeUpdate
const Ref = 0b000000000001000000;          // Ref attached/detached
const Callback = 0b000000000010000000;     // setState callback
const Passive = 0b000000001000000000;      // useEffect
const Layout = 0b000000010000000000;       // useLayoutEffect
const LayoutStatic = 0b000000100000000000; // useLayoutEffect (deps [])
const PassiveStatic = 0b000001000000000000;// useEffect (deps [])

// During render phase, effects are marked:
function updateFunctionComponent(fiber) {
  // ...
  
  // If has useLayoutEffect
  fiber.flags |= Layout;
  
  // If has useEffect
  fiber.flags |= Passive;
  
  // If props changed
  fiber.flags |= Update;
}

// During commit phase, effects are processed:
if (fiber.flags & Layout) {
  commitLayoutEffects(fiber);
}
if (fiber.flags & Passive) {
  schedulePassiveEffects(fiber);
}
```

---

#### Complete Commit Flow

**Full Timeline:**

```javascript
function performSyncWorkOnRoot(root) {
  // RENDER PHASE (interruptible)
  renderRootSync(root, lanes);
  
  const finishedWork = root.current.alternate;
  root.finishedWork = finishedWork;
  
  // COMMIT PHASE (synchronous, uninterruptible)
  commitRoot(root);
}

function commitRoot(root) {
  const finishedWork = root.finishedWork;
  
  // 1. BEFORE MUTATION
  commitBeforeMutationEffects(root, finishedWork);
  // - getSnapshotBeforeUpdate (class components)
  
  // 2. MUTATION
  commitMutationEffects(root, finishedWork);
  // - Detach refs
  // - Insert new nodes (Placement)
  // - Update existing nodes (Update)
  // - Remove old nodes (Deletion)
  
  // 3. SWITCH TREES (atomic)
  root.current = finishedWork;
  
  // 4. LAYOUT
  commitLayoutEffects(root, finishedWork);
  // - Attach refs
  // - componentDidMount/Update (class)
  // - useLayoutEffect (function)
  // - setState callbacks
  
  // 5. SCHEDULE PASSIVE EFFECTS
  if (hasPassiveEffects) {
    scheduleCallback(NormalPriority, flushPassiveEffects);
  }
  
  // 6. CLEANUP
  cleanupEffectPools();
}

// After commit, browser paints
// Then passive effects run
function flushPassiveEffects() {
  // 1. Cleanup old effects
  commitPassiveUnmountEffects(root.current);
  
  // 2. Run new effects
  commitPassiveMountEffects(root, root.current);
}
```

---

#### Real-World Example

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const inputRef = useRef();
  
  useLayoutEffect(() => {
    // Runs during layout phase (sync)
    console.log('Layout: Input width =', inputRef.current.offsetWidth);
  });
  
  useEffect(() => {
    // Runs after paint (async)
    console.log('Effect: Todos count =', todos.length);
    
    return () => {
      console.log('Cleanup: Old count was', todos.length);
    };
  }, [todos]);
  
  const addTodo = () => {
    setTodos([...todos, { id: Date.now(), text: 'New Todo' }]);
  };
  
  return (
    <div>
      <input ref={inputRef} />
      <button onClick={addTodo}>Add</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}

// When addTodo is clicked:

// 1. RENDER PHASE (interruptible)
//    - Create new fiber tree
//    - Mark effects: Update, Layout, Passive

// 2. COMMIT PHASE (synchronous)
//    a. Before Mutation: (nothing here)
//    
//    b. Mutation:
//       - Create new <li> DOM nodes
//       - Insert into <ul>
//    
//    c. Switch Trees:
//       - root.current = finishedWork
//    
//    d. Layout:
//       - "Layout: Input width = 200"
//       - (useLayoutEffect runs, DOM is updated)

// 3. BROWSER PAINTS
//    - User sees new todo item

// 4. PASSIVE EFFECTS
//    - "Cleanup: Old count was 0"
//    - "Effect: Todos count = 1"
```

---

#### Error Boundaries

**Error Handling During Commit:**

```jsx
class ErrorBoundary extends Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, info) {
    // Runs during commit phase
    console.error('Error caught:', error);
  }
  
  render() {
    if (this.state.hasError) {
      return <div>Something went wrong</div>;
    }
    return this.props.children;
  }
}

// If error during commit:
function commitMutationEffects(root, fiber) {
  try {
    // ... process effects
  } catch (error) {
    // Capture error
    captureCommitPhaseError(fiber, error);
    
    // Find nearest error boundary
    const errorBoundary = findNearestErrorBoundary(fiber);
    
    // Schedule re-render with error state
    scheduleErrorBoundaryUpdate(errorBoundary, error);
  }
}
```

---

#### Interview Tips

✅ **Key Points:**
- Commit phase is synchronous and uninterruptible
- Three sub-phases: before mutation, mutation, layout
- Before mutation: getSnapshotBeforeUpdate
- Mutation: DOM updates (insert, update, delete)
- Layout: refs, componentDidMount/Update, useLayoutEffect
- Tree switch happens between mutation and layout
- useEffect runs after paint (passive effects)
- Must be atomic to maintain consistency

✅ **When to Mention:**
- Lifecycle method discussions
- useEffect vs useLayoutEffect
- Ref timing questions
- Performance optimization
- React rendering internals

✅ **Common Follow-ups:**
- "Why is commit phase synchronous?"
- "What's the difference between layout and passive effects?"
- "When do refs get attached?"
- "What happens during the mutation phase?"
- "Why does useEffect run after paint?"

✅ **Perfect Answer Structure:**
1. Purpose: Apply changes to DOM synchronously
2. Why Sync: Must be atomic for consistency
3. Three Sub-phases: Before mutation, mutation, layout
4. Before Mutation: getSnapshotBeforeUpdate
5. Mutation: DOM updates (placement, update, deletion)
6. Layout: Refs, lifecycles, useLayoutEffect
7. Passive: useEffect scheduled after paint
8. Example: Show execution order with useEffect/useLayoutEffect

</details>

---

### 198. What is tearing in concurrent rendering?

<details>
<summary>View Answer</summary>

**Tearing** is a visual inconsistency that occurs when different parts of the UI display different versions of the same data. In concurrent rendering, tearing can happen when an external store updates while React is rendering, causing some components to see the old value and others to see the new value within the same frame.

#### The Problem

**Without Tearing (Consistent):**

```jsx
// React state (automatically prevents tearing)
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <Display1 value={count} />
      <Display2 value={count} />
      <Display3 value={count} />
    </>
  );
}

// All three displays show SAME value
// Even if count updates during render
// React ensures consistency
```

**With Tearing (Inconsistent):**

```jsx
// External store (can cause tearing)
const store = {
  count: 0,
  listeners: [],
  subscribe(listener) {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  },
  getCount() {
    return this.count;
  },
  setCount(value) {
    this.count = value;
    this.listeners.forEach(listener => listener());
  },
};

function Display1() {
  const [count, setCount] = useState(store.getCount());
  
  useEffect(() => {
    return store.subscribe(() => {
      setCount(store.getCount());
    });
  }, []);
  
  return <div>Display 1: {count}</div>;
}

function Display2() {
  const [count, setCount] = useState(store.getCount());
  
  useEffect(() => {
    return store.subscribe(() => {
      setCount(store.getCount());
    });
  }, []);
  
  return <div>Display 2: {count}</div>;
}

function App() {
  return (
    <>
      <Display1 />
      <Display2 />
    </>
  );
}

// Problem:
// 1. React starts rendering (concurrent mode)
// 2. Display1 renders: reads count = 5
// 3. External update: store.count = 6
// 4. Display2 renders: reads count = 6
// 5. UI shows: Display1 = 5, Display2 = 6
// Result: TEARING! Inconsistent UI!
```

---

#### When Tearing Occurs

**Timeline of Tearing:**

```
Time  | React Render          | Store State | Display1 | Display2
------|-----------------------|-------------|----------|----------
0ms   | Start render          | count = 5   | -        | -
5ms   | Render Display1       | count = 5   | reads 5  | -
10ms  | [External update]     | count = 6   | -        | -
15ms  | Render Display2       | count = 6   | -        | reads 6
20ms  | Commit to DOM         | count = 6   | shows 5  | shows 6

Result: TORN UI - Display1 shows 5, Display2 shows 6
```

**Why Only in Concurrent Mode?**

```javascript
// Legacy Mode (React 17 and earlier)
// Rendering is synchronous and uninterruptible
function renderSync() {
  renderDisplay1(); // reads count = 5
  renderDisplay2(); // reads count = 5
  commit();         // both show 5
  // External updates happen AFTER commit
  // No chance for tearing
}

// Concurrent Mode (React 18+)
// Rendering is interruptible
function renderConcurrent() {
  renderDisplay1();     // reads count = 5
  yield();              // Pause for browser
  // External update here: count = 6
  renderDisplay2();     // reads count = 6
  commit();             // Display1=5, Display2=6
  // TEARING!
}
```

---

#### Real-World Example

**Redux Store Tearing:**

```jsx
// Redux store (external)
const store = createStore(reducer);

// Naive implementation (VULNERABLE TO TEARING)
function useSelector(selector) {
  const [state, setState] = useState(() => selector(store.getState()));
  
  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(selector(store.getState()));
    });
    return unsubscribe;
  }, [selector]);
  
  return state;
}

function UserName() {
  const name = useSelector(state => state.user.name);
  return <div>Name: {name}</div>;
}

function UserAge() {
  const age = useSelector(state => state.user.age);
  return <div>Age: {age}</div>;
}

function App() {
  return (
    <>
      <UserName />
      <UserAge />
    </>
  );
}

// Scenario:
// 1. User = { name: 'Alice', age: 25 }
// 2. React starts rendering UserName (concurrent)
// 3. UserName reads: name = 'Alice'
// 4. External dispatch: updateUser({ name: 'Bob', age: 26 })
// 5. React resumes, renders UserAge
// 6. UserAge reads: age = 26
// 7. UI shows: Name: Alice, Age: 26
// Result: TEARING! Alice is not 26 years old!
```

---

#### Solution: useSyncExternalStore

**React 18 Hook to Prevent Tearing:**

```jsx
import { useSyncExternalStore } from 'react';

// External store
const store = {
  count: 0,
  listeners: [],
  getCount() {
    return this.count;
  },
  subscribe(listener) {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  },
  setCount(value) {
    this.count = value;
    this.listeners.forEach(l => l());
  },
};

// Safe hook using useSyncExternalStore
function useCount() {
  return useSyncExternalStore(
    store.subscribe.bind(store),  // subscribe
    store.getCount.bind(store),   // getSnapshot
    store.getCount.bind(store)    // getServerSnapshot (SSR)
  );
}

function Display1() {
  const count = useCount();
  return <div>Display 1: {count}</div>;
}

function Display2() {
  const count = useCount();
  return <div>Display 2: {count}</div>;
}

// How useSyncExternalStore prevents tearing:
// 1. Tracks snapshot version
// 2. If store updates during render, detects mismatch
// 3. Restarts render from beginning
// 4. All components read same version
// 5. Consistent UI!
```

---

#### How useSyncExternalStore Works

**Internal Implementation (Simplified):**

```javascript
function useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot) {
  // Read initial snapshot
  const snapshot = getSnapshot();
  
  // Store snapshot in state
  const [{ inst }, forceUpdate] = useState({ inst: { snapshot } });
  
  // Subscribe to store
  useEffect(() => {
    inst.getSnapshot = getSnapshot;
    inst.subscribe = subscribe;
    
    // Check if snapshot changed while rendering
    if (checkIfSnapshotChanged(inst)) {
      forceUpdate({ inst });
    }
    
    const handleStoreChange = () => {
      // Store updated, check for changes
      if (checkIfSnapshotChanged(inst)) {
        forceUpdate({ inst });
      }
    };
    
    return subscribe(handleStoreChange);
  }, [subscribe, getSnapshot]);
  
  // During render, track snapshot
  useEffect(() => {
    // If snapshot changed during render
    if (checkIfSnapshotChanged(inst)) {
      // Force synchronous re-render
      forceUpdate({ inst });
    }
  });
  
  return snapshot;
}

function checkIfSnapshotChanged(inst) {
  const latestSnapshot = inst.getSnapshot();
  return !Object.is(latestSnapshot, inst.snapshot);
}

// Key mechanism:
// If store changes during render:
// 1. Detect mismatch (checkIfSnapshotChanged)
// 2. Force SYNCHRONOUS re-render
// 3. All components read new snapshot
// 4. Consistent UI
```

---

#### Redux Example with useSyncExternalStore

**React-Redux v8+ (Tearing-Safe):**

```jsx
import { useSyncExternalStore } from 'react';
import { createStore } from 'redux';

// Redux store
const store = createStore(reducer);

// Tearing-safe selector
function useSelector(selector) {
  return useSyncExternalStore(
    store.subscribe,
    () => selector(store.getState()),
    () => selector(store.getState())
  );
}

function UserProfile() {
  const user = useSelector(state => state.user);
  
  return (
    <div>
      <div>Name: {user.name}</div>
      <div>Age: {user.age}</div>
    </div>
  );
}

// Safe from tearing!
// If store updates during render:
// - useSyncExternalStore detects it
// - Forces synchronous re-render
// - Both name and age consistent
```

---

#### Detecting Tearing

**Test for Tearing:**

```jsx
function TearingTest() {
  // Read value multiple times
  const value1 = useExternalValue();
  const value2 = useExternalValue();
  const value3 = useExternalValue();
  
  // If concurrent rendering + external updates
  // These might differ (tearing)
  useEffect(() => {
    if (value1 !== value2 || value2 !== value3) {
      console.error('TEARING DETECTED!', { value1, value2, value3 });
    }
  });
  
  return (
    <div>
      <div>Value 1: {value1}</div>
      <div>Value 2: {value2}</div>
      <div>Value 3: {value3}</div>
    </div>
  );
}

// With proper useSyncExternalStore: Always same
// Without: Can differ during concurrent rendering
```

---

#### Migration Guide

**Old Pattern (Vulnerable):**

```jsx
// ❌ Don't do this in React 18+
function useExternalValue() {
  const [value, setValue] = useState(externalStore.getValue());
  
  useEffect(() => {
    return externalStore.subscribe(() => {
      setValue(externalStore.getValue());
    });
  }, []);
  
  return value;
}
```

**New Pattern (Safe):**

```jsx
import { useSyncExternalStore } from 'react';

// ✅ Tearing-safe
function useExternalValue() {
  return useSyncExternalStore(
    externalStore.subscribe,
    externalStore.getValue,
    externalStore.getValue // SSR
  );
}
```

---

#### Library Examples

**Zustand (v4+):**

```jsx
import create from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

// Internally uses useSyncExternalStore
// Automatically safe from tearing

function Counter() {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);
  
  return (
    <button onClick={increment}>{count}</button>
  );
}
```

**Jotai (v1+):**

```jsx
import { atom, useAtom } from 'jotai';

const countAtom = atom(0);

// Internally uses useSyncExternalStore
// Safe from tearing

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  
  return (
    <button onClick={() => setCount(c => c + 1)}>{count}</button>
  );
}
```

**MobX (v6+):**

```jsx
import { makeAutoObservable } from 'mobx';
import { observer } from 'mobx-react-lite';

class Store {
  count = 0;
  
  constructor() {
    makeAutoObservable(this);
  }
  
  increment() {
    this.count++;
  }
}

const store = new Store();

// observer() uses useSyncExternalStore internally
const Counter = observer(() => {
  return (
    <button onClick={() => store.increment()}>
      {store.count}
    </button>
  );
});
```

---

#### SSR Considerations

**Server Snapshot:**

```jsx
function useSyncExternalStore(
  subscribe,
  getSnapshot,
  getServerSnapshot // Important for SSR!
) {
  // Client: uses getSnapshot
  // Server: uses getServerSnapshot
  
  return useSyncExternalStore(
    subscribe,
    getSnapshot,
    getServerSnapshot || getSnapshot
  );
}

// Example: localStorage (client-only)
function useLocalStorage(key) {
  return useSyncExternalStore(
    (callback) => {
      window.addEventListener('storage', callback);
      return () => window.removeEventListener('storage', callback);
    },
    () => localStorage.getItem(key),
    () => null // Server: no localStorage
  );
}
```

---

#### Performance Considerations

**Synchronous Re-renders:**

```jsx
// If store updates frequently during render
// useSyncExternalStore may trigger many synchronous re-renders
// This is by design to prevent tearing

// Optimization: Memoize selectors
const selectUser = (state) => state.user;
const memoizedSelector = useMemo(() => selectUser, []);

const user = useSyncExternalStore(
  store.subscribe,
  () => memoizedSelector(store.getState())
);

// Or use selector with shallow equality
function useShallowSelector(selector) {
  const selected = useSyncExternalStore(
    store.subscribe,
    () => selector(store.getState())
  );
  
  // Avoid re-renders if shallow equal
  return useMemo(() => selected, [JSON.stringify(selected)]);
}
```

---

#### Interview Tips

✅ **Key Points:**
- Tearing = UI showing inconsistent versions of same data
- Occurs in concurrent rendering when external store updates mid-render
- Different components see different values of same data
- useSyncExternalStore prevents tearing by detecting mismatches
- Forces synchronous re-render if store changes during render
- All modern state libraries (Redux, Zustand, Jotai, MobX) use it
- Critical for React 18+ concurrent features

✅ **When to Mention:**
- React 18 concurrent features discussions
- External state management questions
- Redux/Zustand/MobX integration
- useSyncExternalStore usage
- Visual consistency issues

✅ **Common Follow-ups:**
- "Why doesn't React state have tearing?"
- "How does useSyncExternalStore prevent it?"
- "When do you need to use useSyncExternalStore?"
- "Does Redux cause tearing?"
- "What's the performance cost of preventing tearing?"

✅ **Perfect Answer Structure:**
1. Definition: UI showing different versions of same data
2. Cause: External store updates during concurrent render
3. Example: Display1 shows count=5, Display2 shows count=6
4. Why Problem: Breaks consistency, confusing UX
5. Solution: useSyncExternalStore detects mismatches
6. How: Forces synchronous re-render if store changes
7. Libraries: Redux, Zustand, Jotai all use it internally

</details>

---

### 199. How does React handle Suspense internally?

<details>
<summary>View Answer</summary>

**Suspense** works by catching promises thrown during rendering. When a component suspends (throws a promise), React pauses rendering that subtree, shows a fallback UI, and resumes rendering when the promise resolves. This enables declarative loading states and coordinated data fetching.

#### Basic Mechanism

**Component Throws Promise:**

```jsx
// Component that suspends
function User({ userId }) {
  const user = use(fetchUser(userId)); // Throws promise if not ready
  return <div>{user.name}</div>;
}

// Internal implementation
function use(promise) {
  // Check if promise is resolved
  if (promise.status === 'fulfilled') {
    return promise.value;
  }
  
  if (promise.status === 'rejected') {
    throw promise.reason;
  }
  
  // Promise pending: Throw it (suspend)
  throw promise;
}

// React catches the thrown promise
try {
  renderComponent(User);
} catch (thrownValue) {
  if (isPromise(thrownValue)) {
    // This is a suspension!
    handleSuspense(thrownValue);
  } else {
    // Regular error
    throw thrownValue;
  }
}
```

---

#### Suspense Boundary

**Catching Suspended Components:**

```jsx
import { Suspense } from 'react';

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <User userId={1} />
    </Suspense>
  );
}

// Internal boundary structure
const SuspenseBoundary = {
  type: Suspense,
  fallback: <Loading />,
  children: <User userId={1} />,
  
  // Internal state
  state: {
    dehydrated: false,
    retryLane: NoLane,
    suspendedChildren: [],
  },
};
```

---

#### Rendering Flow

**1. Initial Render:**

```javascript
function renderRoot(root) {
  try {
    // Start rendering from root
    workInProgress = root.current.alternate;
    renderRootImpl(root);
  } catch (error) {
    // Handle errors/suspensions
  }
}

function renderComponent(fiber) {
  // Render component
  const Component = fiber.type;
  const props = fiber.pendingProps;
  
  try {
    const result = Component(props);
    return result;
  } catch (thrownValue) {
    // Check if suspension
    if (isPromise(thrownValue)) {
      handleSuspension(fiber, thrownValue);
    } else {
      throw thrownValue; // Re-throw errors
    }
  }
}
```

**2. Handle Suspension:**

```javascript
function handleSuspension(fiber, promise) {
  // Mark fiber as suspended
  fiber.flags |= ShouldCapture;
  
  // Find nearest Suspense boundary
  const suspenseBoundary = findNearestSuspenseBoundary(fiber);
  
  if (suspenseBoundary) {
    // Mark boundary as suspended
    suspenseBoundary.flags |= DidCapture;
    
    // Track promise
    attachPingListener(promise, suspenseBoundary);
    
    // Schedule retry when promise resolves
    promise.then(
      () => pingRetry(suspenseBoundary),
      () => pingRetry(suspenseBoundary)
    );
    
    // Stop rendering this subtree
    throwException(fiber, promise);
  } else {
    // No boundary, throw to parent
    throw promise;
  }
}

function findNearestSuspenseBoundary(fiber) {
  let current = fiber.return;
  
  while (current !== null) {
    if (current.tag === SuspenseComponent) {
      return current;
    }
    current = current.return;
  }
  
  return null;
}
```

**3. Show Fallback:**

```javascript
function updateSuspenseComponent(current, workInProgress) {
  const nextProps = workInProgress.pendingProps;
  
  // Check if suspended
  if (workInProgress.flags & DidCapture) {
    // Suspended: Render fallback
    workInProgress.flags &= ~DidCapture;
    
    const fallback = nextProps.fallback;
    return mountSuspenseFallbackChildren(
      workInProgress,
      nextProps.children,
      fallback
    );
  } else {
    // Not suspended: Render children
    return mountSuspensePrimaryChildren(
      workInProgress,
      nextProps.children
    );
  }
}

function mountSuspenseFallbackChildren(workInProgress, children, fallback) {
  // Create fallback fiber
  const fallbackFiber = createFiberFromFragment(fallback);
  fallbackFiber.return = workInProgress;
  
  // Hide primary children (offscreen)
  const primaryChildFragment = createFiberFromFragment(children);
  primaryChildFragment.flags |= Visibility;
  primaryChildFragment.return = workInProgress;
  
  workInProgress.child = fallbackFiber;
  fallbackFiber.sibling = primaryChildFragment;
  
  return fallbackFiber;
}
```

**4. Resolve and Retry:**

```javascript
function pingRetry(suspenseBoundary) {
  // Promise resolved, schedule retry
  const root = getRootForFiber(suspenseBoundary);
  const retryLane = getRetryLane(suspenseBoundary);
  
  // Mark for retry
  suspenseBoundary.lanes |= retryLane;
  
  // Schedule update
  scheduleUpdateOnFiber(root, suspenseBoundary, retryLane);
}

// On retry, render primary children again
function retrySuspenseComponent(workInProgress) {
  // Clear suspended flag
  workInProgress.flags &= ~DidCapture;
  
  // Try rendering primary children
  const primaryChildren = workInProgress.pendingProps.children;
  
  try {
    return mountSuspensePrimaryChildren(workInProgress, primaryChildren);
  } catch (thrownValue) {
    if (isPromise(thrownValue)) {
      // Still suspended, show fallback
      handleSuspension(workInProgress, thrownValue);
    }
  }
}
```

---

#### Complete Example

```jsx
import { Suspense, use } from 'react';

// Resource cache
const cache = new Map();

function fetchUser(id) {
  if (!cache.has(id)) {
    const promise = fetch(`/api/users/${id}`)
      .then(res => res.json())
      .then(data => {
        promise.status = 'fulfilled';
        promise.value = data;
        return data;
      })
      .catch(error => {
        promise.status = 'rejected';
        promise.reason = error;
        throw error;
      });
    
    promise.status = 'pending';
    cache.set(id, promise);
  }
  
  return cache.get(id);
}

function User({ userId }) {
  // This will throw a promise if data not ready
  const user = use(fetchUser(userId));
  return <div>{user.name}</div>;
}

function App() {
  return (
    <Suspense fallback={<div>Loading user...</div>}>
      <User userId={1} />
    </Suspense>
  );
}

// Execution flow:
// 1. Render App
// 2. Render Suspense boundary
// 3. Try render User
// 4. use() throws promise (data not ready)
// 5. React catches promise at Suspense boundary
// 6. Show fallback: "Loading user..."
// 7. Promise resolves
// 8. React retries rendering User
// 9. use() returns data
// 10. Show User content
```

---

#### Nested Suspense

**Multiple Boundaries:**

```jsx
function App() {
  return (
    <Suspense fallback={<div>Loading app...</div>}>
      <Header />
      <Suspense fallback={<div>Loading content...</div>}>
        <Content />
        <Suspense fallback={<div>Loading sidebar...</div>}>
          <Sidebar />
        </Suspense>
      </Suspense>
    </Suspense>
  );
}

// If Sidebar suspends:
// - Inner Suspense shows "Loading sidebar..."
// - Content and Header remain visible
// - Granular loading states

// If Content suspends:
// - Middle Suspense shows "Loading content..."
// - Header remains visible
// - Sidebar hidden

// If Header suspends:
// - Outer Suspense shows "Loading app..."
// - Everything hidden
```

---

#### SuspenseList (Coordination)

**Control Reveal Order:**

```jsx
import { SuspenseList, Suspense } from 'react';

function Feed() {
  return (
    <SuspenseList revealOrder="forwards">
      <Suspense fallback={<Loading />}>
        <Post id={1} />
      </Suspense>
      <Suspense fallback={<Loading />}>
        <Post id={2} />
      </Suspense>
      <Suspense fallback={<Loading />}>
        <Post id={3} />
      </Suspense>
    </SuspenseList>
  );
}

// revealOrder="forwards":
// - Posts appear in order (1, 2, 3)
// - Even if Post 3 loads before Post 2
// - Wait for earlier posts

// revealOrder="backwards":
// - Posts appear in reverse (3, 2, 1)

// revealOrder="together":
// - All posts appear at once
// - Wait for all to load
```

**Internal Implementation:**

```javascript
function updateSuspenseListComponent(current, workInProgress) {
  const revealOrder = workInProgress.pendingProps.revealOrder;
  const children = workInProgress.pendingProps.children;
  
  // Track child suspense states
  const suspenseStates = [];
  let child = workInProgress.child;
  
  while (child !== null) {
    if (child.tag === SuspenseComponent) {
      suspenseStates.push({
        fiber: child,
        resolved: (child.flags & DidCapture) === NoFlags,
      });
    }
    child = child.sibling;
  }
  
  // Apply reveal order logic
  switch (revealOrder) {
    case 'forwards': {
      // Hide children after first unresolved
      let firstUnresolved = suspenseStates.findIndex(s => !s.resolved);
      if (firstUnresolved !== -1) {
        for (let i = firstUnresolved + 1; i < suspenseStates.length; i++) {
          suspenseStates[i].fiber.flags |= Visibility;
        }
      }
      break;
    }
    
    case 'backwards': {
      // Hide children before last unresolved
      let lastUnresolved = -1;
      for (let i = suspenseStates.length - 1; i >= 0; i--) {
        if (!suspenseStates[i].resolved) {
          lastUnresolved = i;
          break;
        }
      }
      if (lastUnresolved !== -1) {
        for (let i = 0; i < lastUnresolved; i++) {
          suspenseStates[i].fiber.flags |= Visibility;
        }
      }
      break;
    }
    
    case 'together': {
      // Hide all if any unresolved
      const allResolved = suspenseStates.every(s => s.resolved);
      if (!allResolved) {
        suspenseStates.forEach(s => {
          s.fiber.flags |= Visibility;
        });
      }
      break;
    }
  }
  
  return workInProgress.child;
}
```

---

#### Offscreen Rendering

**Primary Children While Suspended:**

```javascript
// When showing fallback, primary children rendered offscreen
function mountSuspenseFallbackChildren(workInProgress, primaryChildren, fallback) {
  // Fallback fiber (visible)
  const fallbackFiber = createFiberFromFragment(fallback);
  
  // Primary children fiber (hidden, continues rendering)
  const primaryChildFragment = createFiberFromOffscreen(primaryChildren);
  primaryChildFragment.mode |= ConcurrentMode;
  primaryChildFragment.flags |= Visibility; // Hidden
  
  // Keep rendering primary children in background
  // When data loads, can immediately show
  // No re-render needed!
  
  workInProgress.child = fallbackFiber;
  fallbackFiber.sibling = primaryChildFragment;
  
  return fallbackFiber;
}

// Benefits:
// - Primary children continue rendering in background
// - When data loads, instantly swap to primary
// - Smooth transition
```

---

#### Transition Integration

**startTransition + Suspense:**

```jsx
import { startTransition, Suspense } from 'react';

function App() {
  const [userId, setUserId] = useState(1);
  
  const selectUser = (id) => {
    startTransition(() => {
      setUserId(id);
    });
  };
  
  return (
    <div>
      <button onClick={() => selectUser(2)}>User 2</button>
      <Suspense fallback={<Loading />}>
        <User userId={userId} />
      </Suspense>
    </div>
  );
}

// Behavior:
// 1. Click button
// 2. startTransition marks update as non-urgent
// 3. User component suspends
// 4. Instead of showing fallback immediately:
//    - Keep showing old content (User 1)
//    - Render new content (User 2) offscreen
//    - Show loading indicator (isPending)
// 5. When data loads, swap to User 2
// Result: No jarring fallback, smooth transition
```

**Internal Transition Handling:**

```javascript
function updateSuspenseComponent(current, workInProgress) {
  const isTransition = isInsideTransition();
  
  if (workInProgress.flags & DidCapture) {
    // Suspended
    
    if (isTransition) {
      // Transition: Keep showing old content
      // Don't show fallback immediately
      // Mark as pending
      workInProgress.lanes |= TransitionLane;
      
      // Continue rendering new content offscreen
      return updateSuspensePrimaryChildren(
        current,
        workInProgress,
        workInProgress.pendingProps.children
      );
    } else {
      // Non-transition: Show fallback immediately
      return mountSuspenseFallbackChildren(
        workInProgress,
        workInProgress.pendingProps.children,
        workInProgress.pendingProps.fallback
      );
    }
  }
  
  // Not suspended, show primary children
  return mountSuspensePrimaryChildren(
    workInProgress,
    workInProgress.pendingProps.children
  );
}
```

---

#### Retry Lanes

**Exponential Backoff:**

```javascript
const RetryLanes = [
  RetryLane1,  // ~100ms
  RetryLane2,  // ~200ms
  RetryLane3,  // ~400ms
  RetryLane4,  // ~800ms
  RetryLane5,  // ~1600ms
];

let currentRetryLane = 0;

function getRetryLane(suspenseBoundary) {
  // Get current retry lane
  const lane = RetryLanes[currentRetryLane];
  
  // Increment for next retry (exponential backoff)
  currentRetryLane = Math.min(
    currentRetryLane + 1,
    RetryLanes.length - 1
  );
  
  return lane;
}

// Example:
// 1st suspend: Retry with RetryLane1 (~100ms)
// 2nd suspend: Retry with RetryLane2 (~200ms)
// 3rd suspend: Retry with RetryLane3 (~400ms)
// Prevents excessive retries
```

---

#### Error Boundaries + Suspense

**Handling Both:**

```jsx
class ErrorBoundary extends Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  render() {
    if (this.state.hasError) {
      return <div>Error occurred</div>;
    }
    return this.props.children;
  }
}

function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<Loading />}>
        <User userId={1} />
      </Suspense>
    </ErrorBoundary>
  );
}

// If User suspends: Suspense catches (shows Loading)
// If User throws error: ErrorBoundary catches (shows Error)
// If fetch fails: Promise rejects, use() throws error
//                 → ErrorBoundary catches
```

---

#### Streaming SSR

**Server-Side Suspense:**

```jsx
// Server
import { renderToPipeableStream } from 'react-dom/server';

function App() {
  return (
    <html>
      <body>
        <Header />
        <Suspense fallback={<Loading />}>
          <SlowComponent />
        </Suspense>
        <Footer />
      </body>
    </html>
  );
}

const { pipe } = renderToPipeableStream(<App />, {
  onShellReady() {
    // Send initial HTML (Header, Loading, Footer)
    pipe(response);
  },
  onAllReady() {
    // All suspense boundaries resolved
  },
});

// Timeline:
// 1. Server starts rendering
// 2. Header renders immediately
// 3. SlowComponent suspends
// 4. Server sends: <Header />, <Loading />, <Footer />
// 5. Client sees page with loading state
// 6. SlowComponent finishes on server
// 7. Server streams replacement HTML
// 8. Client replaces loading with content
// Result: Fast initial paint, progressive enhancement
```

---

#### Practical Patterns

**1. Parallel Data Fetching:**

```jsx
function Profile({ userId }) {
  return (
    <Suspense fallback={<Loading />}>
      <UserInfo userId={userId} />
      <UserPosts userId={userId} />
      <UserFollowers userId={userId} />
    </Suspense>
  );
}

// All three fetch in parallel
// Single loading state
// All appear together
```

**2. Waterfall Prevention:**

```jsx
// Bad: Waterfall
function Profile({ userId }) {
  const user = use(fetchUser(userId));
  // Must wait for user before fetching posts
  const posts = use(fetchPosts(user.id));
  return <div>...</div>;
}

// Good: Parallel
function Profile({ userId }) {
  // Start fetching immediately
  const userPromise = fetchUser(userId);
  const postsPromise = fetchPosts(userId);
  
  const user = use(userPromise);
  const posts = use(postsPromise);
  
  return <div>...</div>;
}
```

**3. Granular Loading:**

```jsx
function Dashboard() {
  return (
    <div>
      <Suspense fallback={<HeaderSkeleton />}>
        <Header />
      </Suspense>
      <Suspense fallback={<ChartsSkeleton />}>
        <Charts />
      </Suspense>
      <Suspense fallback={<TableSkeleton />}>
        <DataTable />
      </Suspense>
    </div>
  );
}

// Each section loads independently
// Progressive appearance
// Better perceived performance
```

---

#### Interview Tips

✅ **Key Points:**
- Suspense catches thrown promises during rendering
- Shows fallback while waiting, retries when promise resolves
- Nearest Suspense boundary catches suspension
- Primary children render offscreen while showing fallback
- Integrates with transitions for smooth updates
- SuspenseList coordinates multiple boundaries
- Enables streaming SSR with progressive hydration
- Different from error boundaries (catches promises, not errors)

✅ **When to Mention:**
- Data fetching discussions
- Loading state management
- SSR/streaming rendering
- React 18 concurrent features
- Code splitting with lazy()

✅ **Common Follow-ups:**
- "How does React know to catch promises?"
- "What's the difference between Suspense and error boundaries?"
- "How does Suspense work with transitions?"
- "Can you explain SuspenseList?"
- "How does streaming SSR use Suspense?"

✅ **Perfect Answer Structure:**
1. Mechanism: Components throw promises when data not ready
2. Boundary: Nearest Suspense catches, shows fallback
3. Retry: Promise resolves, React retries rendering
4. Offscreen: Primary children continue rendering hidden
5. Transitions: Can avoid showing fallback for smooth UX
6. Coordination: SuspenseList controls reveal order
7. Example: Show component suspending and resolving

</details>

---

### 200. What is the reconciliation process in detail?

<details>
<summary>View Answer</summary>

**Reconciliation** is React's algorithm for determining what changed between renders and efficiently updating the DOM. It compares the new virtual DOM tree with the previous one (diffing), identifies minimal changes needed, and updates only what changed. This process is the core of React's performance.

#### Overview

**The Problem:**

```javascript
// Previous render
const oldTree = <div><span>Hello</span><p>World</p></div>;

// New render
const newTree = <div><span>Hi</span><p>World</p></div>;

// Question: What's the minimal set of DOM operations to transform oldTree to newTree?
// Answer: Update <span>'s text content from "Hello" to "Hi"

// Naive approach: Replace entire tree O(n³)
// React's approach: Diffing algorithm O(n)
```

---

#### Key Principles

**1. Different Types = Replace:**

```jsx
// Old
<div>Content</div>

// New
<span>Content</span>

// Result: Destroy <div>, create <span>
// Even though content is same
// Different element types = complete replacement
```

**2. Same Type = Update:**

```jsx
// Old
<div className="old">Content</div>

// New
<div className="new">Content</div>

// Result: Keep <div>, update className
// Same element type = update attributes
```

**3. Keys for Lists:**

```jsx
// Without keys: O(n²) worst case
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
</ul>

// With keys: O(n) with map lookup
<ul>
  <li key="1">Item 1</li>
  <li key="2">Item 2</li>
</ul>
```

---

#### Diffing Algorithm

**Level-by-Level Comparison:**

```javascript
function reconcileChildren(current, workInProgress, nextChildren) {
  if (current === null) {
    // Mount: No previous children
    mountChildFibers(workInProgress, nextChildren);
  } else {
    // Update: Reconcile with previous children
    reconcileChildFibers(current, workInProgress, nextChildren);
  }
}

function reconcileChildFibers(returnFiber, currentFirstChild, newChild) {
  // Handle different types of children
  
  if (typeof newChild === 'object' && newChild !== null) {
    switch (newChild.$$typeof) {
      case REACT_ELEMENT_TYPE:
        return reconcileSingleElement(
          returnFiber,
          currentFirstChild,
          newChild
        );
      
      case REACT_PORTAL_TYPE:
        return reconcileSinglePortal(
          returnFiber,
          currentFirstChild,
          newChild
        );
    }
    
    if (isArray(newChild)) {
      return reconcileChildrenArray(
        returnFiber,
        currentFirstChild,
        newChild
      );
    }
  }
  
  if (typeof newChild === 'string' || typeof newChild === 'number') {
    return reconcileSingleTextNode(
      returnFiber,
      currentFirstChild,
      '' + newChild
    );
  }
  
  // Delete remaining children
  return deleteRemainingChildren(returnFiber, currentFirstChild);
}
```

---

#### Single Element Reconciliation

**Same Type:**

```javascript
function reconcileSingleElement(
  returnFiber,
  currentFirstChild,
  element
) {
  const key = element.key;
  let child = currentFirstChild;
  
  // Check if can reuse existing fiber
  while (child !== null) {
    // Compare keys
    if (child.key === key) {
      // Compare types
      if (child.elementType === element.type) {
        // CAN REUSE!
        // Delete siblings (if any)
        deleteRemainingChildren(returnFiber, child.sibling);
        
        // Clone fiber with new props
        const existing = useFiber(child, element.props);
        existing.return = returnFiber;
        return existing;
      }
      
      // Key matches but type differs: Can't reuse
      // Delete this and remaining siblings
      deleteRemainingChildren(returnFiber, child);
      break;
    } else {
      // Key doesn't match: Delete this child
      deleteChild(returnFiber, child);
    }
    
    child = child.sibling;
  }
  
  // Can't reuse: Create new fiber
  const created = createFiberFromElement(element);
  created.return = returnFiber;
  return created;
}

// Example:
// Old: <div key="a">Old</div>
// New: <div key="a">New</div>
// Result: Reuse fiber, update props

// Old: <div key="a">Content</div>
// New: <span key="a">Content</span>
// Result: Delete div, create span
```

---

#### Array Reconciliation

**Most Complex Case:**

```javascript
function reconcileChildrenArray(
  returnFiber,
  currentFirstChild,
  newChildren
) {
  let resultingFirstChild = null;
  let previousNewFiber = null;
  let oldFiber = currentFirstChild;
  let lastPlacedIndex = 0;
  let newIdx = 0;
  
  // PASS 1: Update common prefix
  // While keys match, update in place
  for (; oldFiber !== null && newIdx < newChildren.length; newIdx++) {
    const newChild = newChildren[newIdx];
    
    if (oldFiber.key !== newChild.key) {
      // Keys don't match, break
      break;
    }
    
    // Keys match: Update
    const newFiber = updateSlot(
      returnFiber,
      oldFiber,
      newChild
    );
    
    newFiber.return = returnFiber;
    
    if (previousNewFiber === null) {
      resultingFirstChild = newFiber;
    } else {
      previousNewFiber.sibling = newFiber;
    }
    
    previousNewFiber = newFiber;
    oldFiber = oldFiber.sibling;
  }
  
  // All new children processed?
  if (newIdx === newChildren.length) {
    // Delete remaining old children
    deleteRemainingChildren(returnFiber, oldFiber);
    return resultingFirstChild;
  }
  
  // All old children processed?
  if (oldFiber === null) {
    // Add remaining new children
    for (; newIdx < newChildren.length; newIdx++) {
      const newFiber = createChild(returnFiber, newChildren[newIdx]);
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
    }
    return resultingFirstChild;
  }
  
  // PASS 2: Handle moves/additions/deletions
  // Create map of remaining old children
  const existingChildren = mapRemainingChildren(returnFiber, oldFiber);
  
  // Process remaining new children
  for (; newIdx < newChildren.length; newIdx++) {
    const newChild = newChildren[newIdx];
    
    // Try to find in existing children
    const newFiber = updateFromMap(
      existingChildren,
      returnFiber,
      newIdx,
      newChild
    );
    
    if (newFiber !== null) {
      if (newFiber.alternate !== null) {
        // Matched existing child: Remove from map
        existingChildren.delete(
          newFiber.key === null ? newIdx : newFiber.key
        );
      }
      
      // Check if moved
      lastPlacedIndex = placeChild(newFiber, lastPlacedIndex, newIdx);
      
      if (previousNewFiber === null) {
        resultingFirstChild = newFiber;
      } else {
        previousNewFiber.sibling = newFiber;
      }
      previousNewFiber = newFiber;
    }
  }
  
  // Delete remaining old children (not in new array)
  existingChildren.forEach(child => deleteChild(returnFiber, child));
  
  return resultingFirstChild;
}

function mapRemainingChildren(returnFiber, currentFirstChild) {
  const existingChildren = new Map();
  let existingChild = currentFirstChild;
  
  while (existingChild !== null) {
    const key = existingChild.key !== null ? existingChild.key : existingChild.index;
    existingChildren.set(key, existingChild);
    existingChild = existingChild.sibling;
  }
  
  return existingChildren;
}
```

---

#### Placement Detection

**Tracking Moves:**

```javascript
function placeChild(newFiber, lastPlacedIndex, newIndex) {
  newFiber.index = newIndex;
  
  const current = newFiber.alternate;
  
  if (current !== null) {
    // Update: Check if moved
    const oldIndex = current.index;
    
    if (oldIndex < lastPlacedIndex) {
      // Moved forward: Mark for placement
      newFiber.flags |= Placement;
      return lastPlacedIndex;
    } else {
      // Stayed in place or moved backward
      return oldIndex;
    }
  } else {
    // Mount: New child
    newFiber.flags |= Placement;
    return lastPlacedIndex;
  }
}

// Example:
// Old: [A, B, C, D]
// New: [B, A, C, D]

// Process B: oldIndex=1, lastPlaced=0 → update lastPlaced=1
// Process A: oldIndex=0, lastPlaced=1 → oldIndex < lastPlaced → MOVED
// Mark A with Placement flag
```

---

#### Examples

**1. Simple Update:**

```jsx
// Old
<div className="old">Hello</div>

// New
<div className="new">Hello</div>

// Reconciliation:
// 1. Same type (div)
// 2. Reuse fiber
// 3. Update props: className: "old" → "new"
// 4. No children changed
// Result: One prop update
```

**2. Type Change:**

```jsx
// Old
<div>Content</div>

// New  
<span>Content</span>

// Reconciliation:
// 1. Different types (div → span)
// 2. Delete div fiber
// 3. Create span fiber
// 4. Mark div for deletion
// 5. Mark span for placement
// Result: Full replacement
```

**3. List Update:**

```jsx
// Old
<ul>
  <li key="a">A</li>
  <li key="b">B</li>
  <li key="c">C</li>
</ul>

// New
<ul>
  <li key="b">B</li>
  <li key="a">A</li>
  <li key="c">C</li>
</ul>

// Reconciliation:
// Pass 1:
// - Compare first child: key "a" ≠ key "b" → break

// Pass 2:
// - Map: {a: fiberA, b: fiberB, c: fiberC}
// - Process "b": Found in map (oldIndex=1), reuse
// - Process "a": Found in map (oldIndex=0), oldIndex < lastPlaced → MOVED
// - Process "c": Found in map (oldIndex=2), reuse
// - Empty map

// Result:
// - Reuse all fibers
// - Move A after B
// - 1 DOM move operation
```

**4. Add/Remove:**

```jsx
// Old
<ul>
  <li key="a">A</li>
  <li key="b">B</li>
</ul>

// New
<ul>
  <li key="a">A</li>
  <li key="c">C</li>
  <li key="b">B</li>
</ul>

// Reconciliation:
// Pass 1:
// - key="a" matches → reuse
// - key="c" ≠ key="b" → break

// Pass 2:
// - Map: {b: fiberB}
// - Process "c": Not in map → create new
// - Process "b": In map → reuse, remove from map
// - Empty map

// Result:
// - Reuse A and B
// - Create C
// - Insert C between A and B
```

---

#### Component Reconciliation

**Function Components:**

```javascript
function updateFunctionComponent(
  current,
  workInProgress,
  Component,
  nextProps
) {
  // Execute component function
  const nextChildren = Component(nextProps);
  
  // Reconcile children
  reconcileChildren(
    current,
    workInProgress,
    nextChildren
  );
  
  return workInProgress.child;
}

// Example:
function Counter({ count }) {
  return <div>{count}</div>;
}

// Old: <Counter count={1} />
// New: <Counter count={2} />

// Reconciliation:
// 1. Same component type → reuse fiber
// 2. Call Counter(2)
// 3. Returns <div>2</div>
// 4. Reconcile children: text "1" → "2"
// 5. Update text node
```

**Class Components:**

```javascript
function updateClassComponent(
  current,
  workInProgress,
  Component,
  nextProps
) {
  // Get or create instance
  const instance = workInProgress.stateNode;
  
  if (instance === null) {
    // Mount
    constructClassInstance(workInProgress, Component, nextProps);
    mountClassInstance(workInProgress, Component, nextProps);
  } else {
    // Update
    updateClassInstance(current, workInProgress, Component, nextProps);
  }
  
  // Call render()
  const nextChildren = instance.render();
  
  // Reconcile children
  reconcileChildren(
    current,
    workInProgress,
    nextChildren
  );
  
  return workInProgress.child;
}
```

---

#### Bailout Optimization

**Skip Unchanged Subtrees:**

```javascript
function beginWork(current, workInProgress) {
  // Check if can bailout (skip work)
  if (current !== null) {
    const oldProps = current.memoizedProps;
    const newProps = workInProgress.pendingProps;
    
    if (
      oldProps === newProps &&
      !hasContextChanged() &&
      !hasScheduledUpdate(workInProgress)
    ) {
      // Props unchanged, context unchanged, no updates
      // BAILOUT: Skip this subtree
      return bailoutOnAlreadyFinishedWork(
        current,
        workInProgress
      );
    }
  }
  
  // Can't bailout, do work
  switch (workInProgress.tag) {
    case FunctionComponent:
      return updateFunctionComponent(
        current,
        workInProgress,
        workInProgress.type,
        workInProgress.pendingProps
      );
    // ...
  }
}

function bailoutOnAlreadyFinishedWork(current, workInProgress) {
  // Clone child fibers (reuse structure)
  cloneChildFibers(current, workInProgress);
  return workInProgress.child;
}

// Example:
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <ExpensiveChild value="static" />
    </div>
  );
}

// When count updates:
// - Parent re-renders
// - ExpensiveChild props unchanged ("static")
// - Bailout: Skip ExpensiveChild reconciliation
// - Performance win!
```

---

#### Complete Reconciliation Flow

```javascript
// 1. Trigger update
setState(newValue);

// 2. Schedule work
scheduleUpdateOnFiber(fiber, lane);

// 3. Begin work on root
function performUnitOfWork(fiber) {
  // RECONCILIATION STARTS HERE
  const next = beginWork(fiber.alternate, fiber);
  
  if (next !== null) {
    // Has children, continue with child
    return next;
  }
  
  // No children, complete this unit
  completeUnitOfWork(fiber);
}

// 4. Begin work (reconciliation)
function beginWork(current, workInProgress) {
  switch (workInProgress.tag) {
    case FunctionComponent:
      // Call function, get children
      const children = workInProgress.type(workInProgress.pendingProps);
      
      // RECONCILE CHILDREN
      reconcileChildren(current, workInProgress, children);
      
      return workInProgress.child;
  }
}

// 5. Reconcile children (diffing)
function reconcileChildren(current, workInProgress, nextChildren) {
  if (current === null) {
    // Mount
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren
    );
  } else {
    // Update: DIFF ALGORITHM
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren
    );
  }
}

// 6. Mark effects
// During reconciliation, fibers are marked:
// - Placement: New node
// - Update: Props changed
// - Deletion: Node removed

// 7. Complete work
function completeWork(fiber) {
  // Create/update DOM node
  // Bubble up effects
}

// 8. Commit phase
// Apply marked effects to DOM
function commitRoot(root) {
  commitMutationEffects(root);
  // Insert, update, delete DOM nodes
}
```

---

#### Interview Tips

✅ **Key Points:**
- Reconciliation = diffing algorithm to find minimal changes
- O(n) time complexity (vs O(n³) naive approach)
- Three principles: Different types replace, same type updates, keys for lists
- Two-pass algorithm for arrays: common prefix, then map lookup
- Placement detection tracks moved elements
- Bailout optimization skips unchanged subtrees
- Marks effects (Placement, Update, Deletion) during reconciliation
- Commit phase applies effects to DOM

✅ **When to Mention:**
- React performance questions
- Virtual DOM explanations
- List rendering and keys
- React.memo/PureComponent
- Rendering optimization

✅ **Common Follow-ups:**
- "Why do we need keys in lists?"
- "What's the time complexity?"
- "How does React know what changed?"
- "What's the difference between mount and update?"
- "How does bailout work?"

✅ **Perfect Answer Structure:**
1. Purpose: Find minimal changes between renders
2. Algorithm: O(n) diffing with three principles
3. Principles: Different types replace, same updates, keys
4. Arrays: Two-pass with map lookup for efficiency
5. Effects: Mark Placement/Update/Deletion during reconciliation
6. Bailout: Skip unchanged subtrees for performance
7. Example: Show list reconciliation with keys

</details>
