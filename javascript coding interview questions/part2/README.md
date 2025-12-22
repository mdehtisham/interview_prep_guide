### **Data Structures**

61. Implement a Stack class

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Array-Based Stack**
```javascript
/**
 * Stack implementation using array
 * LIFO (Last In First Out)
 * Time Complexity: O(1) for all operations
 * Space Complexity: O(n)
 */
class Stack {
  constructor() {
    this.items = [];
  }
  
  // Add element to top of stack
  push(element) {
    this.items.push(element);
    return this.size();
  }
  
  // Remove and return top element
  pop() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items.pop();
  }
  
  // View top element without removing
  peek() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.items.length - 1];
  }
  
  // Check if stack is empty
  isEmpty() {
    return this.items.length === 0;
  }
  
  // Get stack size
  size() {
    return this.items.length;
  }
  
  // Clear all elements
  clear() {
    this.items = [];
  }
  
  // Convert to array
  toArray() {
    return [...this.items];
  }
  
  // Print stack
  print() {
    console.log(this.items.toString());
  }
}

// Usage
const stack = new Stack();
stack.push(1);
stack.push(2);
stack.push(3);
console.log(stack.peek()); // 3
console.log(stack.pop());  // 3
console.log(stack.size()); // 2
console.log(stack.isEmpty()); // false

// Test examples
console.log('\n=== Array-Based Stack ===');
const stack1 = new Stack();
stack1.push(10);
stack1.push(20);
stack1.push(30);
console.log('Stack:', stack1.toArray()); // [10, 20, 30]
console.log('Pop:', stack1.pop()); // 30
console.log('Peek:', stack1.peek()); // 20
console.log('Size:', stack1.size()); // 2
```

### **Approach 2: Linked List-Based Stack**
```javascript
/**
 * Stack implementation using linked list
 * More memory efficient for large stacks
 */
class Node {
  constructor(value) {
    this.value = value;
    this.next = null;
  }
}

class LinkedListStack {
  constructor() {
    this.top = null;
    this.length = 0;
  }
  
  push(value) {
    const newNode = new Node(value);
    newNode.next = this.top;
    this.top = newNode;
    this.length++;
    return this.length;
  }
  
  pop() {
    if (this.isEmpty()) {
      return undefined;
    }
    
    const value = this.top.value;
    this.top = this.top.next;
    this.length--;
    return value;
  }
  
  peek() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.top.value;
  }
  
  isEmpty() {
    return this.length === 0;
  }
  
  size() {
    return this.length;
  }
  
  clear() {
    this.top = null;
    this.length = 0;
  }
  
  toArray() {
    const result = [];
    let current = this.top;
    
    while (current) {
      result.push(current.value);
      current = current.next;
    }
    
    return result.reverse(); // Reverse to show bottom-to-top
  }
  
  print() {
    console.log(this.toArray().toString());
  }
}

// Usage
console.log('\n=== Linked List Stack ===');
const stack2 = new LinkedListStack();
stack2.push('A');
stack2.push('B');
stack2.push('C');
console.log('Stack:', stack2.toArray()); // ['A', 'B', 'C']
console.log('Pop:', stack2.pop()); // 'C'
console.log('Peek:', stack2.peek()); // 'B'
```

### **Approach 3: Stack with Min/Max Tracking**
```javascript
/**
 * Stack that tracks minimum and maximum values
 * Useful for problems requiring O(1) min/max access
 */
class MinMaxStack {
  constructor() {
    this.items = [];
    this.minStack = [];
    this.maxStack = [];
  }
  
  push(value) {
    this.items.push(value);
    
    // Track minimum
    if (this.minStack.length === 0 || value <= this.getMin()) {
      this.minStack.push(value);
    }
    
    // Track maximum
    if (this.maxStack.length === 0 || value >= this.getMax()) {
      this.maxStack.push(value);
    }
    
    return this.size();
  }
  
  pop() {
    if (this.isEmpty()) {
      return undefined;
    }
    
    const value = this.items.pop();
    
    // Update min stack
    if (value === this.getMin()) {
      this.minStack.pop();
    }
    
    // Update max stack
    if (value === this.getMax()) {
      this.maxStack.pop();
    }
    
    return value;
  }
  
  peek() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.items.length - 1];
  }
  
  getMin() {
    if (this.minStack.length === 0) {
      return undefined;
    }
    return this.minStack[this.minStack.length - 1];
  }
  
  getMax() {
    if (this.maxStack.length === 0) {
      return undefined;
    }
    return this.maxStack[this.maxStack.length - 1];
  }
  
  isEmpty() {
    return this.items.length === 0;
  }
  
  size() {
    return this.items.length;
  }
  
  clear() {
    this.items = [];
    this.minStack = [];
    this.maxStack = [];
  }
}

// Usage
console.log('\n=== Min/Max Stack ===');
const stack3 = new MinMaxStack();
stack3.push(5);
stack3.push(2);
stack3.push(8);
stack3.push(1);
console.log('Min:', stack3.getMin()); // 1
console.log('Max:', stack3.getMax()); // 8
stack3.pop();
console.log('Min after pop:', stack3.getMin()); // 2
```

### **Bonus: Stack with Capacity Limit**
```javascript
/**
 * Fixed-size stack with capacity limit
 */
class BoundedStack {
  constructor(capacity) {
    this.capacity = capacity;
    this.items = [];
  }
  
  push(value) {
    if (this.isFull()) {
      throw new Error('Stack overflow: capacity exceeded');
    }
    this.items.push(value);
    return this.size();
  }
  
  pop() {
    if (this.isEmpty()) {
      throw new Error('Stack underflow: stack is empty');
    }
    return this.items.pop();
  }
  
  peek() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.items.length - 1];
  }
  
  isEmpty() {
    return this.items.length === 0;
  }
  
  isFull() {
    return this.items.length >= this.capacity;
  }
  
  size() {
    return this.items.length;
  }
  
  remainingCapacity() {
    return this.capacity - this.items.length;
  }
}

// Usage
console.log('\n=== Bounded Stack ===');
const stack4 = new BoundedStack(3);
stack4.push(1);
stack4.push(2);
stack4.push(3);
console.log('Full?', stack4.isFull()); // true
console.log('Remaining:', stack4.remainingCapacity()); // 0
// stack4.push(4); // Throws error
```

### **Common Stack Applications**
```javascript
// 1. Reverse a string
function reverseString(str) {
  const stack = new Stack();
  
  for (let char of str) {
    stack.push(char);
  }
  
  let reversed = '';
  while (!stack.isEmpty()) {
    reversed += stack.pop();
  }
  
  return reversed;
}

console.log('\n=== Applications ===');
console.log('Reversed:', reverseString('hello')); // 'olleh'

// 2. Check balanced parentheses
function isBalanced(str) {
  const stack = new Stack();
  const pairs = { '(': ')', '{': '}', '[': ']' };
  
  for (let char of str) {
    if (char in pairs) {
      stack.push(char);
    } else if (Object.values(pairs).includes(char)) {
      if (stack.isEmpty() || pairs[stack.pop()] !== char) {
        return false;
      }
    }
  }
  
  return stack.isEmpty();
}

console.log('Balanced:', isBalanced('{[()]}')); // true
console.log('Balanced:', isBalanced('{[(])}')); // false

// 3. Evaluate postfix expression
function evaluatePostfix(expression) {
  const stack = new Stack();
  const tokens = expression.split(' ');
  
  for (let token of tokens) {
    if (!isNaN(token)) {
      stack.push(Number(token));
    } else {
      const b = stack.pop();
      const a = stack.pop();
      
      switch (token) {
        case '+': stack.push(a + b); break;
        case '-': stack.push(a - b); break;
        case '*': stack.push(a * b); break;
        case '/': stack.push(a / b); break;
      }
    }
  }
  
  return stack.pop();
}

console.log('Postfix result:', evaluatePostfix('3 4 + 2 *')); // 14

// 4. Undo/Redo functionality
class UndoRedoManager {
  constructor() {
    this.undoStack = new Stack();
    this.redoStack = new Stack();
  }
  
  execute(action) {
    action.execute();
    this.undoStack.push(action);
    this.redoStack.clear(); // Clear redo on new action
  }
  
  undo() {
    if (!this.undoStack.isEmpty()) {
      const action = this.undoStack.pop();
      action.undo();
      this.redoStack.push(action);
    }
  }
  
  redo() {
    if (!this.redoStack.isEmpty()) {
      const action = this.redoStack.pop();
      action.execute();
      this.undoStack.push(action);
    }
  }
}
```

**Interview Tips:**
- Stack: LIFO (Last In First Out) data structure
- Core operations: push (add), pop (remove), peek (view top), O(1) time
- Implementation: array (simple) or linked list (memory efficient)
- Use cases: function call stack, undo/redo, backtracking, expression evaluation
- Common problems: balanced parentheses, reverse string, postfix evaluation
- Min/Max stack: track min/max in O(1) using auxiliary stacks
- Space complexity: O(n) where n is number of elements
- Variants: bounded stack (fixed size), double stack (two stacks in one array)
- Consider: overflow (capacity) and underflow (empty) conditions

</details>

62. Implement a Queue class

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Array-Based Queue**
```javascript
/**
 * Queue implementation using array
 * FIFO (First In First Out)
 * Time Complexity: enqueue O(1), dequeue O(n) due to shift
 * Space Complexity: O(n)
 */
class Queue {
  constructor() {
    this.items = [];
  }
  
  // Add element to rear
  enqueue(element) {
    this.items.push(element);
    return this.size();
  }
  
  // Remove and return front element
  dequeue() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items.shift();
  }
  
  // View front element without removing
  front() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[0];
  }
  
  // View rear element
  rear() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.items.length - 1];
  }
  
  // Check if queue is empty
  isEmpty() {
    return this.items.length === 0;
  }
  
  // Get queue size
  size() {
    return this.items.length;
  }
  
  // Clear all elements
  clear() {
    this.items = [];
  }
  
  // Convert to array
  toArray() {
    return [...this.items];
  }
  
  // Print queue
  print() {
    console.log(this.items.toString());
  }
}

// Usage
const queue = new Queue();
queue.enqueue(1);
queue.enqueue(2);
queue.enqueue(3);
console.log(queue.front()); // 1
console.log(queue.dequeue()); // 1
console.log(queue.size()); // 2

// Test examples
console.log('=== Array-Based Queue ===');
const queue1 = new Queue();
queue1.enqueue('A');
queue1.enqueue('B');
queue1.enqueue('C');
console.log('Queue:', queue1.toArray()); // ['A', 'B', 'C']
console.log('Dequeue:', queue1.dequeue()); // 'A'
console.log('Front:', queue1.front()); // 'B'
console.log('Rear:', queue1.rear()); // 'C'
```

### **Approach 2: Optimized Queue with Index Pointers**
```javascript
/**
 * Optimized queue using index pointers
 * Avoids O(n) shift operation by tracking front index
 * Time Complexity: O(1) for all operations
 */
class OptimizedQueue {
  constructor() {
    this.items = {};
    this.frontIndex = 0;
    this.rearIndex = 0;
  }
  
  enqueue(element) {
    this.items[this.rearIndex] = element;
    this.rearIndex++;
    return this.size();
  }
  
  dequeue() {
    if (this.isEmpty()) {
      return undefined;
    }
    
    const item = this.items[this.frontIndex];
    delete this.items[this.frontIndex];
    this.frontIndex++;
    
    // Reset indices when queue is empty
    if (this.frontIndex === this.rearIndex) {
      this.frontIndex = 0;
      this.rearIndex = 0;
    }
    
    return item;
  }
  
  front() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.frontIndex];
  }
  
  rear() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.rearIndex - 1];
  }
  
  isEmpty() {
    return this.size() === 0;
  }
  
  size() {
    return this.rearIndex - this.frontIndex;
  }
  
  clear() {
    this.items = {};
    this.frontIndex = 0;
    this.rearIndex = 0;
  }
  
  toArray() {
    const result = [];
    for (let i = this.frontIndex; i < this.rearIndex; i++) {
      result.push(this.items[i]);
    }
    return result;
  }
  
  print() {
    console.log(this.toArray().toString());
  }
}

// Usage
console.log('\n=== Optimized Queue ===');
const queue2 = new OptimizedQueue();
queue2.enqueue(10);
queue2.enqueue(20);
queue2.enqueue(30);
console.log('Queue:', queue2.toArray()); // [10, 20, 30]
console.log('Dequeue:', queue2.dequeue()); // 10
console.log('Size:', queue2.size()); // 2
```

### **Approach 3: Linked List-Based Queue**
```javascript
/**
 * Queue implementation using linked list
 * Most efficient for large queues
 */
class QueueNode {
  constructor(value) {
    this.value = value;
    this.next = null;
  }
}

class LinkedListQueue {
  constructor() {
    this.head = null; // Front
    this.tail = null; // Rear
    this.length = 0;
  }
  
  enqueue(value) {
    const newNode = new QueueNode(value);
    
    if (this.isEmpty()) {
      this.head = newNode;
      this.tail = newNode;
    } else {
      this.tail.next = newNode;
      this.tail = newNode;
    }
    
    this.length++;
    return this.length;
  }
  
  dequeue() {
    if (this.isEmpty()) {
      return undefined;
    }
    
    const value = this.head.value;
    this.head = this.head.next;
    
    if (this.head === null) {
      this.tail = null;
    }
    
    this.length--;
    return value;
  }
  
  front() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.head.value;
  }
  
  rear() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.tail.value;
  }
  
  isEmpty() {
    return this.length === 0;
  }
  
  size() {
    return this.length;
  }
  
  clear() {
    this.head = null;
    this.tail = null;
    this.length = 0;
  }
  
  toArray() {
    const result = [];
    let current = this.head;
    
    while (current) {
      result.push(current.value);
      current = current.next;
    }
    
    return result;
  }
  
  print() {
    console.log(this.toArray().toString());
  }
}

// Usage
console.log('\n=== Linked List Queue ===');
const queue3 = new LinkedListQueue();
queue3.enqueue('X');
queue3.enqueue('Y');
queue3.enqueue('Z');
console.log('Queue:', queue3.toArray()); // ['X', 'Y', 'Z']
console.log('Dequeue:', queue3.dequeue()); // 'X'
console.log('Front:', queue3.front()); // 'Y'
```

### **Bonus: Priority Queue**
```javascript
/**
 * Priority Queue (Min Heap based)
 * Elements with higher priority are dequeued first
 */
class PriorityQueue {
  constructor() {
    this.items = [];
  }
  
  enqueue(element, priority = 0) {
    const queueElement = { element, priority };
    let added = false;
    
    // Insert based on priority (lower number = higher priority)
    for (let i = 0; i < this.items.length; i++) {
      if (queueElement.priority < this.items[i].priority) {
        this.items.splice(i, 0, queueElement);
        added = true;
        break;
      }
    }
    
    if (!added) {
      this.items.push(queueElement);
    }
    
    return this.size();
  }
  
  dequeue() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items.shift().element;
  }
  
  front() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[0].element;
  }
  
  isEmpty() {
    return this.items.length === 0;
  }
  
  size() {
    return this.items.length;
  }
  
  clear() {
    this.items = [];
  }
  
  toArray() {
    return this.items.map(item => `${item.element}(${item.priority})`);
  }
}

// Usage
console.log('\n=== Priority Queue ===');
const pq = new PriorityQueue();
pq.enqueue('Low priority', 3);
pq.enqueue('High priority', 1);
pq.enqueue('Medium priority', 2);
console.log('Priority Queue:', pq.toArray()); // ['High priority(1)', 'Medium priority(2)', 'Low priority(3)']
console.log('Dequeue:', pq.dequeue()); // 'High priority'
```

### **Bonus: Circular Queue**
```javascript
/**
 * Circular Queue (Ring Buffer)
 * Fixed size queue with wraparound
 */
class CircularQueue {
  constructor(capacity) {
    this.capacity = capacity;
    this.items = new Array(capacity);
    this.front = -1;
    this.rear = -1;
    this.length = 0;
  }
  
  enqueue(element) {
    if (this.isFull()) {
      throw new Error('Queue is full');
    }
    
    if (this.isEmpty()) {
      this.front = 0;
    }
    
    this.rear = (this.rear + 1) % this.capacity;
    this.items[this.rear] = element;
    this.length++;
    
    return this.length;
  }
  
  dequeue() {
    if (this.isEmpty()) {
      return undefined;
    }
    
    const element = this.items[this.front];
    
    if (this.front === this.rear) {
      // Queue becomes empty
      this.front = -1;
      this.rear = -1;
    } else {
      this.front = (this.front + 1) % this.capacity;
    }
    
    this.length--;
    return element;
  }
  
  getFront() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.front];
  }
  
  getRear() {
    if (this.isEmpty()) {
      return undefined;
    }
    return this.items[this.rear];
  }
  
  isEmpty() {
    return this.length === 0;
  }
  
  isFull() {
    return this.length === this.capacity;
  }
  
  size() {
    return this.length;
  }
  
  clear() {
    this.items = new Array(this.capacity);
    this.front = -1;
    this.rear = -1;
    this.length = 0;
  }
  
  toArray() {
    if (this.isEmpty()) return [];
    
    const result = [];
    let i = this.front;
    
    do {
      result.push(this.items[i]);
      i = (i + 1) % this.capacity;
    } while (i !== (this.rear + 1) % this.capacity);
    
    return result;
  }
}

// Usage
console.log('\n=== Circular Queue ===');
const cq = new CircularQueue(3);
cq.enqueue(1);
cq.enqueue(2);
cq.enqueue(3);
console.log('Full?', cq.isFull()); // true
console.log('Dequeue:', cq.dequeue()); // 1
cq.enqueue(4); // Wraps around
console.log('Queue:', cq.toArray()); // [2, 3, 4]
```

### **Common Queue Applications**
```javascript
// 1. BFS (Breadth-First Search)
function bfs(graph, start) {
  const queue = new Queue();
  const visited = new Set();
  const result = [];
  
  queue.enqueue(start);
  visited.add(start);
  
  while (!queue.isEmpty()) {
    const node = queue.dequeue();
    result.push(node);
    
    for (let neighbor of graph[node] || []) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.enqueue(neighbor);
      }
    }
  }
  
  return result;
}

console.log('\n=== Applications ===');
const graph = {
  A: ['B', 'C'],
  B: ['D', 'E'],
  C: ['F'],
  D: [], E: [], F: []
};
console.log('BFS:', bfs(graph, 'A')); // ['A', 'B', 'C', 'D', 'E', 'F']

// 2. Task Scheduler
class TaskScheduler {
  constructor() {
    this.queue = new Queue();
  }
  
  addTask(task) {
    this.queue.enqueue(task);
  }
  
  processNext() {
    if (!this.queue.isEmpty()) {
      const task = this.queue.dequeue();
      console.log(`Processing: ${task}`);
      return task;
    }
  }
  
  processAll() {
    while (!this.queue.isEmpty()) {
      this.processNext();
    }
  }
}

// 3. Sliding Window Maximum
function slidingWindowMax(nums, k) {
  const result = [];
  const queue = new OptimizedQueue(); // Stores indices
  
  for (let i = 0; i < nums.length; i++) {
    // Remove elements outside window
    while (!queue.isEmpty() && queue.front() < i - k + 1) {
      queue.dequeue();
    }
    
    // Remove smaller elements (not useful)
    while (!queue.isEmpty() && nums[queue.rear()] < nums[i]) {
      // Remove from rear (would need deque)
    }
    
    queue.enqueue(i);
    
    if (i >= k - 1) {
      result.push(nums[queue.front()]);
    }
  }
  
  return result;
}
```

**Interview Tips:**
- Queue: FIFO (First In First Out) data structure
- Core operations: enqueue (add to rear), dequeue (remove from front), O(1) time
- Implementation: array (simple but shift is O(n)), object with pointers (O(1)), linked list (best)
- Use cases: BFS, task scheduling, request handling, buffering
- Priority Queue: elements dequeued by priority, not insertion order
- Circular Queue: fixed-size queue with wraparound, efficient for buffers
- Deque (double-ended queue): can add/remove from both ends
- Common problems: recent calls, moving average, sliding window
- Space complexity: O(n) where n is number of elements

</details>

63. Implement a Linked List with add, remove, and find methods

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Singly Linked List**
```javascript
/**
 * Singly Linked List implementation
 * Each node points to next node
 * Time Complexity: 
 *   - add: O(1) at head, O(n) at tail
 *   - remove: O(n)
 *   - find: O(n)
 * Space Complexity: O(n)
 */
class ListNode {
  constructor(value) {
    this.value = value;
    this.next = null;
  }
}

class LinkedList {
  constructor() {
    this.head = null;
    this.tail = null;
    this.length = 0;
  }
  
  // Add to end
  add(value) {
    const newNode = new ListNode(value);
    
    if (!this.head) {
      this.head = newNode;
      this.tail = newNode;
    } else {
      this.tail.next = newNode;
      this.tail = newNode;
    }
    
    this.length++;
    return this;
  }
  
  // Add to beginning
  addFirst(value) {
    const newNode = new ListNode(value);
    newNode.next = this.head;
    this.head = newNode;
    
    if (!this.tail) {
      this.tail = newNode;
    }
    
    this.length++;
    return this;
  }
  
  // Add at specific index
  addAt(index, value) {
    if (index < 0 || index > this.length) {
      throw new Error('Index out of bounds');
    }
    
    if (index === 0) {
      return this.addFirst(value);
    }
    
    if (index === this.length) {
      return this.add(value);
    }
    
    const newNode = new ListNode(value);
    let current = this.head;
    
    for (let i = 0; i < index - 1; i++) {
      current = current.next;
    }
    
    newNode.next = current.next;
    current.next = newNode;
    this.length++;
    
    return this;
  }
  
  // Remove by value (first occurrence)
  remove(value) {
    if (!this.head) {
      return null;
    }
    
    // Remove from head
    if (this.head.value === value) {
      const removed = this.head;
      this.head = this.head.next;
      
      if (!this.head) {
        this.tail = null;
      }
      
      this.length--;
      return removed.value;
    }
    
    // Remove from middle or end
    let current = this.head;
    
    while (current.next) {
      if (current.next.value === value) {
        const removed = current.next;
        current.next = current.next.next;
        
        if (!current.next) {
          this.tail = current;
        }
        
        this.length--;
        return removed.value;
      }
      current = current.next;
    }
    
    return null;
  }
  
  // Remove from beginning
  removeFirst() {
    if (!this.head) {
      return null;
    }
    
    const value = this.head.value;
    this.head = this.head.next;
    
    if (!this.head) {
      this.tail = null;
    }
    
    this.length--;
    return value;
  }
  
  // Remove from end
  removeLast() {
    if (!this.head) {
      return null;
    }
    
    if (this.head === this.tail) {
      const value = this.head.value;
      this.head = null;
      this.tail = null;
      this.length--;
      return value;
    }
    
    let current = this.head;
    
    while (current.next !== this.tail) {
      current = current.next;
    }
    
    const value = this.tail.value;
    current.next = null;
    this.tail = current;
    this.length--;
    
    return value;
  }
  
  // Remove at specific index
  removeAt(index) {
    if (index < 0 || index >= this.length) {
      return null;
    }
    
    if (index === 0) {
      return this.removeFirst();
    }
    
    let current = this.head;
    
    for (let i = 0; i < index - 1; i++) {
      current = current.next;
    }
    
    const value = current.next.value;
    current.next = current.next.next;
    
    if (!current.next) {
      this.tail = current;
    }
    
    this.length--;
    return value;
  }
  
  // Find by value
  find(value) {
    let current = this.head;
    let index = 0;
    
    while (current) {
      if (current.value === value) {
        return { value: current.value, index };
      }
      current = current.next;
      index++;
    }
    
    return null;
  }
  
  // Find by index
  get(index) {
    if (index < 0 || index >= this.length) {
      return null;
    }
    
    let current = this.head;
    
    for (let i = 0; i < index; i++) {
      current = current.next;
    }
    
    return current.value;
  }
  
  // Check if value exists
  contains(value) {
    return this.find(value) !== null;
  }
  
  // Get size
  size() {
    return this.length;
  }
  
  // Check if empty
  isEmpty() {
    return this.length === 0;
  }
  
  // Clear all nodes
  clear() {
    this.head = null;
    this.tail = null;
    this.length = 0;
  }
  
  // Convert to array
  toArray() {
    const result = [];
    let current = this.head;
    
    while (current) {
      result.push(current.value);
      current = current.next;
    }
    
    return result;
  }
  
  // Print list
  print() {
    console.log(this.toArray().join(' -> '));
  }
  
  // Reverse the list
  reverse() {
    let prev = null;
    let current = this.head;
    this.tail = this.head;
    
    while (current) {
      const next = current.next;
      current.next = prev;
      prev = current;
      current = next;
    }
    
    this.head = prev;
    return this;
  }
}

// Usage
console.log('=== Singly Linked List ===');
const list = new LinkedList();
list.add(1).add(2).add(3).add(4);
console.log('List:', list.toArray()); // [1, 2, 3, 4]

list.addFirst(0);
console.log('After addFirst(0):', list.toArray()); // [0, 1, 2, 3, 4]

list.addAt(2, 1.5);
console.log('After addAt(2, 1.5):', list.toArray()); // [0, 1, 1.5, 2, 3, 4]

console.log('Find 3:', list.find(3)); // { value: 3, index: 4 }

list.remove(1.5);
console.log('After remove(1.5):', list.toArray()); // [0, 1, 2, 3, 4]

console.log('Get at index 2:', list.get(2)); // 2

list.reverse();
console.log('After reverse:', list.toArray()); // [4, 3, 2, 1, 0]
```

### **Approach 2: Doubly Linked List**
```javascript
/**
 * Doubly Linked List implementation
 * Each node has prev and next pointers
 * Better for bidirectional traversal
 */
class DoublyListNode {
  constructor(value) {
    this.value = value;
    this.prev = null;
    this.next = null;
  }
}

class DoublyLinkedList {
  constructor() {
    this.head = null;
    this.tail = null;
    this.length = 0;
  }
  
  add(value) {
    const newNode = new DoublyListNode(value);
    
    if (!this.head) {
      this.head = newNode;
      this.tail = newNode;
    } else {
      this.tail.next = newNode;
      newNode.prev = this.tail;
      this.tail = newNode;
    }
    
    this.length++;
    return this;
  }
  
  addFirst(value) {
    const newNode = new DoublyListNode(value);
    
    if (!this.head) {
      this.head = newNode;
      this.tail = newNode;
    } else {
      newNode.next = this.head;
      this.head.prev = newNode;
      this.head = newNode;
    }
    
    this.length++;
    return this;
  }
  
  addAt(index, value) {
    if (index < 0 || index > this.length) {
      throw new Error('Index out of bounds');
    }
    
    if (index === 0) return this.addFirst(value);
    if (index === this.length) return this.add(value);
    
    const newNode = new DoublyListNode(value);
    let current = this.head;
    
    for (let i = 0; i < index; i++) {
      current = current.next;
    }
    
    newNode.prev = current.prev;
    newNode.next = current;
    current.prev.next = newNode;
    current.prev = newNode;
    
    this.length++;
    return this;
  }
  
  remove(value) {
    if (!this.head) return null;
    
    let current = this.head;
    
    while (current) {
      if (current.value === value) {
        if (current.prev) {
          current.prev.next = current.next;
        } else {
          this.head = current.next;
        }
        
        if (current.next) {
          current.next.prev = current.prev;
        } else {
          this.tail = current.prev;
        }
        
        this.length--;
        return current.value;
      }
      current = current.next;
    }
    
    return null;
  }
  
  removeFirst() {
    if (!this.head) return null;
    
    const value = this.head.value;
    this.head = this.head.next;
    
    if (this.head) {
      this.head.prev = null;
    } else {
      this.tail = null;
    }
    
    this.length--;
    return value;
  }
  
  removeLast() {
    if (!this.tail) return null;
    
    const value = this.tail.value;
    this.tail = this.tail.prev;
    
    if (this.tail) {
      this.tail.next = null;
    } else {
      this.head = null;
    }
    
    this.length--;
    return value;
  }
  
  find(value) {
    let current = this.head;
    let index = 0;
    
    while (current) {
      if (current.value === value) {
        return { value: current.value, index };
      }
      current = current.next;
      index++;
    }
    
    return null;
  }
  
  get(index) {
    if (index < 0 || index >= this.length) return null;
    
    let current;
    
    // Optimize by starting from head or tail
    if (index < this.length / 2) {
      current = this.head;
      for (let i = 0; i < index; i++) {
        current = current.next;
      }
    } else {
      current = this.tail;
      for (let i = this.length - 1; i > index; i--) {
        current = current.prev;
      }
    }
    
    return current.value;
  }
  
  contains(value) {
    return this.find(value) !== null;
  }
  
  size() {
    return this.length;
  }
  
  isEmpty() {
    return this.length === 0;
  }
  
  clear() {
    this.head = null;
    this.tail = null;
    this.length = 0;
  }
  
  toArray() {
    const result = [];
    let current = this.head;
    
    while (current) {
      result.push(current.value);
      current = current.next;
    }
    
    return result;
  }
  
  toArrayReverse() {
    const result = [];
    let current = this.tail;
    
    while (current) {
      result.push(current.value);
      current = current.prev;
    }
    
    return result;
  }
  
  print() {
    console.log(this.toArray().join(' <-> '));
  }
  
  reverse() {
    let current = this.head;
    let temp = null;
    
    // Swap next and prev for all nodes
    while (current) {
      temp = current.prev;
      current.prev = current.next;
      current.next = temp;
      current = current.prev;
    }
    
    // Swap head and tail
    temp = this.head;
    this.head = this.tail;
    this.tail = temp;
    
    return this;
  }
}

// Usage
console.log('\n=== Doubly Linked List ===');
const dlist = new DoublyLinkedList();
dlist.add('A').add('B').add('C');
console.log('List:', dlist.toArray()); // ['A', 'B', 'C']
console.log('Reverse:', dlist.toArrayReverse()); // ['C', 'B', 'A']

dlist.addFirst('Z');
console.log('After addFirst:', dlist.toArray()); // ['Z', 'A', 'B', 'C']

dlist.remove('A');
console.log('After remove:', dlist.toArray()); // ['Z', 'B', 'C']
```

### **Bonus: Common Linked List Operations**
```javascript
// Find middle element
LinkedList.prototype.findMiddle = function() {
  let slow = this.head;
  let fast = this.head;
  
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
  }
  
  return slow ? slow.value : null;
};

// Detect if list has cycle
LinkedList.prototype.hasCycle = function() {
  let slow = this.head;
  let fast = this.head;
  
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    
    if (slow === fast) {
      return true;
    }
  }
  
  return false;
};

// Get nth node from end
LinkedList.prototype.getNthFromEnd = function(n) {
  let first = this.head;
  let second = this.head;
  
  // Move first pointer n steps ahead
  for (let i = 0; i < n; i++) {
    if (!first) return null;
    first = first.next;
  }
  
  // Move both pointers until first reaches end
  while (first) {
    first = first.next;
    second = second.next;
  }
  
  return second ? second.value : null;
};

// Remove duplicates
LinkedList.prototype.removeDuplicates = function() {
  if (!this.head) return this;
  
  const seen = new Set();
  let current = this.head;
  let prev = null;
  
  while (current) {
    if (seen.has(current.value)) {
      prev.next = current.next;
      if (!prev.next) {
        this.tail = prev;
      }
      this.length--;
    } else {
      seen.add(current.value);
      prev = current;
    }
    current = current.next;
  }
  
  return this;
};

// Merge two sorted lists
function mergeSortedLists(list1, list2) {
  const dummy = new ListNode(0);
  let current = dummy;
  let p1 = list1.head;
  let p2 = list2.head;
  
  while (p1 && p2) {
    if (p1.value < p2.value) {
      current.next = p1;
      p1 = p1.next;
    } else {
      current.next = p2;
      p2 = p2.next;
    }
    current = current.next;
  }
  
  current.next = p1 || p2;
  
  const merged = new LinkedList();
  merged.head = dummy.next;
  
  // Calculate length and find tail
  let temp = merged.head;
  while (temp) {
    merged.length++;
    if (!temp.next) {
      merged.tail = temp;
    }
    temp = temp.next;
  }
  
  return merged;
}

// Test additional methods
console.log('\n=== Additional Operations ===');
const testList = new LinkedList();
testList.add(1).add(2).add(3).add(4).add(5);
console.log('Middle:', testList.findMiddle()); // 3
console.log('2nd from end:', testList.getNthFromEnd(2)); // 4

const dupList = new LinkedList();
dupList.add(1).add(2).add(2).add(3).add(1);
console.log('With duplicates:', dupList.toArray()); // [1, 2, 2, 3, 1]
dupList.removeDuplicates();
console.log('Without duplicates:', dupList.toArray()); // [1, 2, 3]
```

**Interview Tips:**
- Linked List: nodes connected via pointers, dynamic size
- Singly: each node has next pointer, one-way traversal
- Doubly: each node has prev and next, bidirectional traversal
- Operations: add O(1) at head, O(n) at tail without tail pointer
- Search/Find: O(n) time complexity (must traverse)
- Space: O(n) for n elements, each node has extra pointer overhead
- Advantages: dynamic size, O(1) insertion/deletion at known position
- Disadvantages: O(n) access time, no random access, extra memory for pointers
- Common problems: cycle detection, reverse, merge sorted lists, find middle
- Two pointer technique: slow/fast pointers for middle, cycle detection
- Use cases: undo functionality, browser history, music playlist

</details>

64. Check if a linked list has a cycle

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Floyd's Cycle Detection (Tortoise and Hare)**
```javascript
/**
 * Detect cycle using two pointers (optimal solution)
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Algorithm: Use slow and fast pointers
 * - Slow moves 1 step at a time
 * - Fast moves 2 steps at a time
 * - If they meet, there's a cycle
 */
class ListNode {
  constructor(value) {
    this.value = value;
    this.next = null;
  }
}

function hasCycle(head) {
  if (!head || !head.next) {
    return false;
  }
  
  let slow = head;
  let fast = head;
  
  while (fast && fast.next) {
    slow = slow.next;        // Move 1 step
    fast = fast.next.next;   // Move 2 steps
    
    if (slow === fast) {
      return true; // Cycle detected
    }
  }
  
  return false; // No cycle
}

// Test examples
console.log('=== Floyd\'s Cycle Detection ===');

// Create list with cycle: 1 -> 2 -> 3 -> 4 -> 2 (cycle back)
const node1 = new ListNode(1);
const node2 = new ListNode(2);
const node3 = new ListNode(3);
const node4 = new ListNode(4);

node1.next = node2;
node2.next = node3;
node3.next = node4;
node4.next = node2; // Creates cycle

console.log('Has cycle:', hasCycle(node1)); // true

// Create list without cycle: 1 -> 2 -> 3 -> 4
const a1 = new ListNode(1);
const a2 = new ListNode(2);
const a3 = new ListNode(3);
const a4 = new ListNode(4);

a1.next = a2;
a2.next = a3;
a3.next = a4;

console.log('Has cycle:', hasCycle(a1)); // false
console.log('Empty list:', hasCycle(null)); // false
```

### **Approach 2: Hash Set (Simple but uses extra space)**
```javascript
/**
 * Detect cycle using hash set to track visited nodes
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function hasCycleHashSet(head) {
  if (!head) {
    return false;
  }
  
  const visited = new Set();
  let current = head;
  
  while (current) {
    if (visited.has(current)) {
      return true; // Already visited, cycle exists
    }
    
    visited.add(current);
    current = current.next;
  }
  
  return false;
}

// Test
console.log('\n=== Hash Set Method ===');
console.log('Has cycle (hash):', hasCycleHashSet(node1)); // true
console.log('Has cycle (hash):', hasCycleHashSet(a1)); // false
```

### **Approach 3: Find Cycle Start Position**
```javascript
/**
 * Not only detect cycle but find where it starts
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Algorithm:
 * 1. Use Floyd's to detect cycle
 * 2. Reset slow to head, keep fast at meeting point
 * 3. Move both 1 step at a time until they meet
 * 4. Meeting point is cycle start
 */
function detectCycleStart(head) {
  if (!head || !head.next) {
    return null;
  }
  
  let slow = head;
  let fast = head;
  let hasCycle = false;
  
  // Detect if cycle exists
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    
    if (slow === fast) {
      hasCycle = true;
      break;
    }
  }
  
  if (!hasCycle) {
    return null;
  }
  
  // Find cycle start
  slow = head;
  
  while (slow !== fast) {
    slow = slow.next;
    fast = fast.next;
  }
  
  return slow; // Cycle start node
}

// Test
console.log('\n=== Find Cycle Start ===');
const cycleStart = detectCycleStart(node1);
console.log('Cycle starts at value:', cycleStart ? cycleStart.value : 'No cycle'); // 2
console.log('No cycle start:', detectCycleStart(a1) ? 'Found' : 'No cycle'); // No cycle
```

### **Approach 4: Get Cycle Length**
```javascript
/**
 * Find the length of the cycle if it exists
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
function getCycleLength(head) {
  if (!head || !head.next) {
    return 0;
  }
  
  let slow = head;
  let fast = head;
  
  // Detect cycle
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    
    if (slow === fast) {
      // Count nodes in cycle
      let length = 1;
      let current = slow.next;
      
      while (current !== slow) {
        length++;
        current = current.next;
      }
      
      return length;
    }
  }
  
  return 0; // No cycle
}

// Test
console.log('\n=== Cycle Length ===');
console.log('Cycle length:', getCycleLength(node1)); // 3 (nodes 2, 3, 4)
console.log('Cycle length:', getCycleLength(a1)); // 0
```

### **Bonus: Complete Cycle Analysis**
```javascript
/**
 * Complete cycle analysis - detect, find start, get length
 */
function analyzeCycle(head) {
  if (!head || !head.next) {
    return {
      hasCycle: false,
      cycleStart: null,
      cycleLength: 0,
      nodesBeforeCycle: 0
    };
  }
  
  let slow = head;
  let fast = head;
  
  // Phase 1: Detect cycle
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    
    if (slow === fast) {
      break; // Cycle detected
    }
  }
  
  // No cycle
  if (!fast || !fast.next) {
    return {
      hasCycle: false,
      cycleStart: null,
      cycleLength: 0,
      nodesBeforeCycle: 0
    };
  }
  
  // Phase 2: Find cycle start and count nodes before cycle
  slow = head;
  let nodesBeforeCycle = 0;
  
  while (slow !== fast) {
    slow = slow.next;
    fast = fast.next;
    nodesBeforeCycle++;
  }
  
  const cycleStart = slow;
  
  // Phase 3: Calculate cycle length
  let cycleLength = 1;
  let current = slow.next;
  
  while (current !== slow) {
    cycleLength++;
    current = current.next;
  }
  
  return {
    hasCycle: true,
    cycleStart: cycleStart,
    cycleStartValue: cycleStart.value,
    cycleLength: cycleLength,
    nodesBeforeCycle: nodesBeforeCycle
  };
}

// Test
console.log('\n=== Complete Analysis ===');
const analysis1 = analyzeCycle(node1);
console.log('Analysis with cycle:', {
  hasCycle: analysis1.hasCycle,
  cycleStartValue: analysis1.cycleStartValue,
  cycleLength: analysis1.cycleLength,
  nodesBeforeCycle: analysis1.nodesBeforeCycle
});
// { hasCycle: true, cycleStartValue: 2, cycleLength: 3, nodesBeforeCycle: 1 }

const analysis2 = analyzeCycle(a1);
console.log('Analysis without cycle:', analysis2);
// { hasCycle: false, cycleStart: null, cycleLength: 0, nodesBeforeCycle: 0 }
```

### **Practical Implementation with LinkedList Class**
```javascript
/**
 * Add cycle detection to LinkedList class
 */
class CycleDetectableList {
  constructor() {
    this.head = null;
    this.length = 0;
  }
  
  add(value) {
    const newNode = new ListNode(value);
    
    if (!this.head) {
      this.head = newNode;
    } else {
      let current = this.head;
      while (current.next) {
        current = current.next;
      }
      current.next = newNode;
    }
    
    this.length++;
    return this;
  }
  
  // Create cycle for testing (connect last to nth node)
  createCycle(position) {
    if (position < 0 || position >= this.length) {
      return;
    }
    
    let cycleNode = this.head;
    for (let i = 0; i < position; i++) {
      cycleNode = cycleNode.next;
    }
    
    let last = this.head;
    while (last.next) {
      last = last.next;
    }
    
    last.next = cycleNode;
  }
  
  hasCycle() {
    return hasCycle(this.head);
  }
  
  detectCycleStart() {
    return detectCycleStart(this.head);
  }
  
  getCycleLength() {
    return getCycleLength(this.head);
  }
  
  analyzeCycle() {
    return analyzeCycle(this.head);
  }
  
  // Remove cycle if exists
  removeCycle() {
    const start = this.detectCycleStart();
    if (!start) return false;
    
    let current = start;
    while (current.next !== start) {
      current = current.next;
    }
    
    current.next = null;
    return true;
  }
}

// Test
console.log('\n=== LinkedList with Cycle Detection ===');
const list = new CycleDetectableList();
list.add(1).add(2).add(3).add(4).add(5);

console.log('Before cycle:', list.hasCycle()); // false

list.createCycle(2); // Connect to node with value 3
console.log('After creating cycle:', list.hasCycle()); // true

const info = list.analyzeCycle();
console.log('Cycle info:', info);

console.log('Removing cycle:', list.removeCycle()); // true
console.log('After removal:', list.hasCycle()); // false
```

### **Edge Cases and Testing**
```javascript
// Test edge cases
console.log('\n=== Edge Cases ===');

// Single node with self cycle
const selfLoop = new ListNode(1);
selfLoop.next = selfLoop;
console.log('Self loop:', hasCycle(selfLoop)); // true

// Two nodes with cycle
const n1 = new ListNode(1);
const n2 = new ListNode(2);
n1.next = n2;
n2.next = n1;
console.log('Two node cycle:', hasCycle(n1)); // true

// Single node, no cycle
const single = new ListNode(1);
console.log('Single node:', hasCycle(single)); // false

// Empty list
console.log('Empty list:', hasCycle(null)); // false
```

**Interview Tips:**
- Floyd's Cycle Detection (Tortoise and Hare) is optimal: O(n) time, O(1) space
- Key insight: if there's a cycle, fast pointer will eventually meet slow pointer
- Hash set approach is simpler but uses O(n) space
- To find cycle start: reset slow to head after detection, move both 1 step until they meet
- Cycle length: once detected, count steps in one complete loop
- Mathematical proof: if cycle exists, fast catches slow within cycle length iterations
- Common variations: return boolean, return cycle start, return cycle length
- Edge cases: empty list, single node, self-loop, no cycle
- Applications: memory leak detection, infinite loop detection
- Follow-up questions: remove cycle, find cycle length, nodes before cycle

</details>

65. Reverse a linked list

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Iterative (Optimal)**
```javascript
/**
 * Reverse linked list iteratively
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Algorithm: Use three pointers (prev, current, next)
 * Reverse links one by one
 */
class ListNode {
  constructor(value) {
    this.value = value;
    this.next = null;
  }
}

function reverseList(head) {
  let prev = null;
  let current = head;
  
  while (current) {
    const next = current.next;  // Save next node
    current.next = prev;         // Reverse the link
    prev = current;              // Move prev forward
    current = next;              // Move current forward
  }
  
  return prev; // New head
}

// Helper to create list from array
function createList(arr) {
  if (arr.length === 0) return null;
  
  const head = new ListNode(arr[0]);
  let current = head;
  
  for (let i = 1; i < arr.length; i++) {
    current.next = new ListNode(arr[i]);
    current = current.next;
  }
  
  return head;
}

// Helper to convert list to array
function toArray(head) {
  const result = [];
  let current = head;
  
  while (current) {
    result.push(current.value);
    current = current.next;
  }
  
  return result;
}

// Test
console.log('=== Iterative Reverse ===');
let list1 = createList([1, 2, 3, 4, 5]);
console.log('Original:', toArray(list1)); // [1, 2, 3, 4, 5]

list1 = reverseList(list1);
console.log('Reversed:', toArray(list1)); // [5, 4, 3, 2, 1]

// Edge cases
console.log('Empty:', toArray(reverseList(null))); // []
console.log('Single:', toArray(reverseList(createList([1])))); // [1]
```

### **Approach 2: Recursive**
```javascript
/**
 * Reverse linked list recursively
 * Time Complexity: O(n)
 * Space Complexity: O(n) due to recursion stack
 */
function reverseListRecursive(head) {
  // Base cases
  if (!head || !head.next) {
    return head;
  }
  
  // Recursive case
  const newHead = reverseListRecursive(head.next);
  
  // Reverse the link
  head.next.next = head;
  head.next = null;
  
  return newHead;
}

// Test
console.log('\n=== Recursive Reverse ===');
let list2 = createList([1, 2, 3, 4, 5]);
console.log('Original:', toArray(list2)); // [1, 2, 3, 4, 5]

list2 = reverseListRecursive(list2);
console.log('Reversed:', toArray(list2)); // [5, 4, 3, 2, 1]
```

### **Approach 3: Reverse Between Two Positions**
```javascript
/**
 * Reverse linked list between positions left and right
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Example: 1->2->3->4->5, left=2, right=4
 * Result: 1->4->3->2->5
 */
function reverseBetween(head, left, right) {
  if (!head || left === right) {
    return head;
  }
  
  const dummy = new ListNode(0);
  dummy.next = head;
  let prev = dummy;
  
  // Move to node before left position
  for (let i = 1; i < left; i++) {
    prev = prev.next;
  }
  
  // Reverse the sublist
  let current = prev.next;
  
  for (let i = 0; i < right - left; i++) {
    const next = current.next;
    current.next = next.next;
    next.next = prev.next;
    prev.next = next;
  }
  
  return dummy.next;
}

// Test
console.log('\n=== Reverse Between ===');
let list3 = createList([1, 2, 3, 4, 5]);
console.log('Original:', toArray(list3)); // [1, 2, 3, 4, 5]

list3 = reverseBetween(list3, 2, 4);
console.log('Reversed 2-4:', toArray(list3)); // [1, 4, 3, 2, 5]
```

### **Approach 4: Reverse in K Groups**
```javascript
/**
 * Reverse nodes in k-group
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Example: 1->2->3->4->5, k=2
 * Result: 2->1->4->3->5
 * 
 * Example: 1->2->3->4->5, k=3
 * Result: 3->2->1->4->5
 */
function reverseKGroup(head, k) {
  if (!head || k === 1) {
    return head;
  }
  
  // Check if we have k nodes
  let count = 0;
  let current = head;
  
  while (current && count < k) {
    current = current.next;
    count++;
  }
  
  if (count < k) {
    return head; // Not enough nodes to reverse
  }
  
  // Reverse first k nodes
  let prev = null;
  current = head;
  
  for (let i = 0; i < k; i++) {
    const next = current.next;
    current.next = prev;
    prev = current;
    current = next;
  }
  
  // Recursively reverse remaining groups
  head.next = reverseKGroup(current, k);
  
  return prev; // New head of reversed group
}

// Test
console.log('\n=== Reverse K Groups ===');
let list4 = createList([1, 2, 3, 4, 5, 6, 7, 8]);
console.log('Original:', toArray(list4)); // [1, 2, 3, 4, 5, 6, 7, 8]

list4 = reverseKGroup(list4, 3);
console.log('Reversed k=3:', toArray(list4)); // [3, 2, 1, 6, 5, 4, 7, 8]

let list5 = createList([1, 2, 3, 4, 5]);
list5 = reverseKGroup(list5, 2);
console.log('Reversed k=2:', toArray(list5)); // [2, 1, 4, 3, 5]
```

### **Bonus: Reverse with LinkedList Class**
```javascript
/**
 * LinkedList class with reverse methods
 */
class LinkedList {
  constructor() {
    this.head = null;
    this.tail = null;
    this.length = 0;
  }
  
  add(value) {
    const newNode = new ListNode(value);
    
    if (!this.head) {
      this.head = newNode;
      this.tail = newNode;
    } else {
      this.tail.next = newNode;
      this.tail = newNode;
    }
    
    this.length++;
    return this;
  }
  
  // Reverse entire list (iterative)
  reverse() {
    if (!this.head || !this.head.next) {
      return this;
    }
    
    let prev = null;
    let current = this.head;
    this.tail = this.head; // Old head becomes tail
    
    while (current) {
      const next = current.next;
      current.next = prev;
      prev = current;
      current = next;
    }
    
    this.head = prev; // New head
    return this;
  }
  
  // Reverse using recursion
  reverseRecursive() {
    if (!this.head || !this.head.next) {
      return this;
    }
    
    this.tail = this.head;
    this.head = this._reverseHelper(this.head);
    return this;
  }
  
  _reverseHelper(node) {
    if (!node || !node.next) {
      return node;
    }
    
    const newHead = this._reverseHelper(node.next);
    node.next.next = node;
    node.next = null;
    
    return newHead;
  }
  
  // Reverse between positions
  reverseBetween(left, right) {
    if (!this.head || left === right) {
      return this;
    }
    
    this.head = reverseBetween(this.head, left, right);
    
    // Update tail if needed
    this.tail = this.head;
    while (this.tail.next) {
      this.tail = this.tail.next;
    }
    
    return this;
  }
  
  // Reverse in k groups
  reverseKGroup(k) {
    this.head = reverseKGroup(this.head, k);
    
    // Update tail
    this.tail = this.head;
    while (this.tail && this.tail.next) {
      this.tail = this.tail.next;
    }
    
    return this;
  }
  
  toArray() {
    return toArray(this.head);
  }
  
  print() {
    console.log(this.toArray().join(' -> '));
  }
}

// Test
console.log('\n=== LinkedList Class ===');
const list = new LinkedList();
list.add(1).add(2).add(3).add(4).add(5);
console.log('Original:', list.toArray()); // [1, 2, 3, 4, 5]

list.reverse();
console.log('After reverse:', list.toArray()); // [5, 4, 3, 2, 1]

list.reverse(); // Reverse back
list.reverseBetween(2, 4);
console.log('Reverse 2-4:', list.toArray()); // [1, 4, 3, 2, 5]
```

### **Alternative: Stack-Based Reverse**
```javascript
/**
 * Reverse using stack (not optimal but educational)
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function reverseListWithStack(head) {
  if (!head) return null;
  
  const stack = [];
  let current = head;
  
  // Push all nodes to stack
  while (current) {
    stack.push(current.value);
    current = current.next;
  }
  
  // Rebuild list with reversed values
  const newHead = new ListNode(stack.pop());
  current = newHead;
  
  while (stack.length > 0) {
    current.next = new ListNode(stack.pop());
    current = current.next;
  }
  
  return newHead;
}

// Test
console.log('\n=== Stack-Based Reverse ===');
let list6 = createList([1, 2, 3, 4, 5]);
list6 = reverseListWithStack(list6);
console.log('Stack reversed:', toArray(list6)); // [5, 4, 3, 2, 1]
```

### **Visualizing the Reverse Process**
```javascript
/**
 * Step-by-step visualization of iterative reverse
 */
function reverseListVisualize(head) {
  console.log('\n=== Reverse Visualization ===');
  console.log('Initial:', toArray(head));
  
  let prev = null;
  let current = head;
  let step = 1;
  
  while (current) {
    console.log(`\nStep ${step}:`);
    console.log('  Current:', current.value);
    console.log('  Prev:', prev ? prev.value : 'null');
    
    const next = current.next;
    console.log('  Next:', next ? next.value : 'null');
    
    current.next = prev;
    console.log('  Action: Point', current.value, 'to', prev ? prev.value : 'null');
    
    prev = current;
    current = next;
    step++;
  }
  
  console.log('\nFinal reversed list:', toArray(prev));
  return prev;
}

// Demo
let demoList = createList([1, 2, 3]);
reverseListVisualize(demoList);
```

**Interview Tips:**
- Iterative approach is optimal: O(n) time, O(1) space
- Use three pointers: prev, current, next
- Key step: reverse the link (current.next = prev) before moving forward
- Recursive approach is elegant but uses O(n) stack space
- Common variations: reverse between positions, reverse in k groups
- Edge cases: empty list, single node, two nodes
- Don't forget to update head and tail pointers if using LinkedList class
- Practice drawing: visualize pointer movements on paper
- Common mistakes: losing reference to next node, not updating tail
- Follow-ups: reverse in place, reverse alternate nodes, reverse by value range

</details>

---

## **HARD LEVEL (66-95)**

### **Advanced Algorithms**

66. Implement binary search

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Iterative Binary Search**
```javascript
/**
 * Binary search implementation (iterative)
 * Time Complexity: O(log n)
 * Space Complexity: O(1)
 * 
 * Requirements: Array must be sorted
 * Returns index of target or -1 if not found
 */
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  
  while (left <= right) {
    // Calculate middle index (avoid overflow)
    const mid = Math.floor(left + (right - left) / 2);
    
    if (arr[mid] === target) {
      return mid; // Found target
    } else if (arr[mid] < target) {
      left = mid + 1; // Search right half
    } else {
      right = mid - 1; // Search left half
    }
  }
  
  return -1; // Target not found
}

// Test examples
console.log('=== Iterative Binary Search ===');
const arr = [1, 3, 5, 7, 9, 11, 13, 15, 17, 19];

console.log('Find 7:', binarySearch(arr, 7)); // 3
console.log('Find 13:', binarySearch(arr, 13)); // 6
console.log('Find 20:', binarySearch(arr, 20)); // -1
console.log('Find 1:', binarySearch(arr, 1)); // 0
console.log('Find 19:', binarySearch(arr, 19)); // 9

// Edge cases
console.log('Empty array:', binarySearch([], 5)); // -1
console.log('Single element (found):', binarySearch([5], 5)); // 0
console.log('Single element (not found):', binarySearch([5], 3)); // -1
```

### **Approach 2: Recursive Binary Search**
```javascript
/**
 * Binary search implementation (recursive)
 * Time Complexity: O(log n)
 * Space Complexity: O(log n) due to recursion stack
 */
function binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
  // Base case: target not found
  if (left > right) {
    return -1;
  }
  
  const mid = Math.floor(left + (right - left) / 2);
  
  // Base case: found target
  if (arr[mid] === target) {
    return mid;
  }
  
  // Recursive cases
  if (arr[mid] < target) {
    return binarySearchRecursive(arr, target, mid + 1, right);
  } else {
    return binarySearchRecursive(arr, target, left, mid - 1);
  }
}

// Test
console.log('\n=== Recursive Binary Search ===');
console.log('Find 9:', binarySearchRecursive(arr, 9)); // 4
console.log('Find 15:', binarySearchRecursive(arr, 15)); // 7
console.log('Find 100:', binarySearchRecursive(arr, 100)); // -1
```

### **Approach 3: Binary Search Variations**
```javascript
/**
 * Find first occurrence of target
 * Useful when array has duplicates
 */
function binarySearchFirst(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  let result = -1;
  
  while (left <= right) {
    const mid = Math.floor(left + (right - left) / 2);
    
    if (arr[mid] === target) {
      result = mid;
      right = mid - 1; // Continue searching left for first occurrence
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return result;
}

/**
 * Find last occurrence of target
 */
function binarySearchLast(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  let result = -1;
  
  while (left <= right) {
    const mid = Math.floor(left + (right - left) / 2);
    
    if (arr[mid] === target) {
      result = mid;
      left = mid + 1; // Continue searching right for last occurrence
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return result;
}

/**
 * Find number of occurrences of target
 */
function countOccurrences(arr, target) {
  const first = binarySearchFirst(arr, target);
  if (first === -1) return 0;
  
  const last = binarySearchLast(arr, target);
  return last - first + 1;
}

// Test with duplicates
console.log('\n=== Binary Search Variations ===');
const arrWithDuplicates = [1, 2, 2, 2, 3, 4, 4, 5, 5, 5, 5, 6];

console.log('First occurrence of 5:', binarySearchFirst(arrWithDuplicates, 5)); // 7
console.log('Last occurrence of 5:', binarySearchLast(arrWithDuplicates, 5)); // 10
console.log('Count of 5:', countOccurrences(arrWithDuplicates, 5)); // 4
console.log('Count of 2:', countOccurrences(arrWithDuplicates, 2)); // 3
console.log('Count of 7:', countOccurrences(arrWithDuplicates, 7)); // 0
```

### **Approach 4: Lower and Upper Bound**
```javascript
/**
 * Find lower bound (first element >= target)
 * Returns insertion position if target not found
 */
function lowerBound(arr, target) {
  let left = 0;
  let right = arr.length;
  
  while (left < right) {
    const mid = Math.floor(left + (right - left) / 2);
    
    if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid;
    }
  }
  
  return left;
}

/**
 * Find upper bound (first element > target)
 */
function upperBound(arr, target) {
  let left = 0;
  let right = arr.length;
  
  while (left < right) {
    const mid = Math.floor(left + (right - left) / 2);
    
    if (arr[mid] <= target) {
      left = mid + 1;
    } else {
      right = mid;
    }
  }
  
  return left;
}

// Test
console.log('\n=== Lower and Upper Bound ===');
const arr2 = [1, 2, 4, 4, 5, 7, 9];

console.log('Lower bound of 4:', lowerBound(arr2, 4)); // 2 (first 4)
console.log('Upper bound of 4:', upperBound(arr2, 4)); // 4 (after last 4)
console.log('Lower bound of 3:', lowerBound(arr2, 3)); // 2 (position where 3 would be)
console.log('Upper bound of 3:', upperBound(arr2, 3)); // 2 (same as lower bound)
console.log('Lower bound of 10:', lowerBound(arr2, 10)); // 7 (end of array)
```

### **Bonus: Binary Search on Answer**
```javascript
/**
 * Find square root using binary search
 * Example of "binary search on answer" pattern
 */
function sqrt(x) {
  if (x < 2) return x;
  
  let left = 1;
  let right = Math.floor(x / 2);
  let result = 0;
  
  while (left <= right) {
    const mid = Math.floor(left + (right - left) / 2);
    const square = mid * mid;
    
    if (square === x) {
      return mid;
    } else if (square < x) {
      result = mid; // Store potential answer
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return result;
}

/**
 * Search in rotated sorted array
 */
function searchRotated(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  
  while (left <= right) {
    const mid = Math.floor(left + (right - left) / 2);
    
    if (arr[mid] === target) {
      return mid;
    }
    
    // Determine which half is sorted
    if (arr[left] <= arr[mid]) {
      // Left half is sorted
      if (target >= arr[left] && target < arr[mid]) {
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    } else {
      // Right half is sorted
      if (target > arr[mid] && target <= arr[right]) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
  }
  
  return -1;
}

/**
 * Find peak element (element greater than neighbors)
 */
function findPeakElement(arr) {
  let left = 0;
  let right = arr.length - 1;
  
  while (left < right) {
    const mid = Math.floor(left + (right - left) / 2);
    
    if (arr[mid] < arr[mid + 1]) {
      left = mid + 1; // Peak is on right
    } else {
      right = mid; // Peak is on left or at mid
    }
  }
  
  return left;
}

// Test advanced patterns
console.log('\n=== Advanced Binary Search ===');
console.log('Square root of 16:', sqrt(16)); // 4
console.log('Square root of 8:', sqrt(8)); // 2

const rotated = [4, 5, 6, 7, 0, 1, 2];
console.log('Search 0 in rotated:', searchRotated(rotated, 0)); // 4
console.log('Search 3 in rotated:', searchRotated(rotated, 3)); // -1

const peaks = [1, 2, 3, 1];
console.log('Find peak:', findPeakElement(peaks)); // 2 (element 3)
```

### **Bonus: Binary Search in 2D Matrix**
```javascript
/**
 * Search in row-wise and column-wise sorted matrix
 */
function searchMatrix(matrix, target) {
  if (!matrix.length || !matrix[0].length) return false;
  
  const rows = matrix.length;
  const cols = matrix[0].length;
  
  // Treat 2D matrix as 1D sorted array
  let left = 0;
  let right = rows * cols - 1;
  
  while (left <= right) {
    const mid = Math.floor(left + (right - left) / 2);
    
    // Convert 1D index to 2D
    const row = Math.floor(mid / cols);
    const col = mid % cols;
    const value = matrix[row][col];
    
    if (value === target) {
      return true;
    } else if (value < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  
  return false;
}

// Test
console.log('\n=== Binary Search in 2D Matrix ===');
const matrix = [
  [1,  3,  5,  7],
  [10, 11, 16, 20],
  [23, 30, 34, 50]
];

console.log('Search 3:', searchMatrix(matrix, 3)); // true
console.log('Search 13:', searchMatrix(matrix, 13)); // false
```

### **Performance Comparison**
```javascript
/**
 * Compare binary search vs linear search
 */
function linearSearch(arr, target) {
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === target) {
      return i;
    }
  }
  return -1;
}

function benchmark(size) {
  const arr = Array.from({ length: size }, (_, i) => i);
  const target = size - 1; // Worst case
  
  console.log(`\nArray size: ${size.toLocaleString()}`);
  
  // Binary search
  let start = performance.now();
  binarySearch(arr, target);
  let time = performance.now() - start;
  console.log(`Binary Search: ${time.toFixed(4)}ms`);
  
  // Linear search
  start = performance.now();
  linearSearch(arr, target);
  time = performance.now() - start;
  console.log(`Linear Search: ${time.toFixed(4)}ms`);
}

console.log('\n=== Performance Comparison ===');
benchmark(1000);
benchmark(100000);
benchmark(1000000);
```

**Interview Tips:**
- Binary search requires sorted array, O(log n) time complexity
- Key concept: eliminate half of search space each iteration
- Calculate mid as `left + (right - left) / 2` to avoid integer overflow
- Iterative is better than recursive (no stack space)
- Common variations: find first/last occurrence, count occurrences
- Lower bound: first element >= target, Upper bound: first element > target
- Advanced: rotated array, peak element, 2D matrix, sqrt using binary search
- Watch out for infinite loops: ensure left/right pointers move
- Edge cases: empty array, single element, target at boundaries
- Template: while (left <= right) for exact match, while (left < right) for bounds
- Applications: searching, finding insertion point, optimization problems
- Follow-ups: search in rotated array, find peak, search in 2D matrix

</details>

67. Implement quicksort algorithm

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Classic Quicksort (Lomuto Partition)**
```javascript
/**
 * Quicksort implementation using Lomuto partition scheme
 * Time Complexity: 
 *   - Best/Average: O(n log n)
 *   - Worst: O(n²) when array is already sorted
 * Space Complexity: O(log n) for recursion stack
 */
function quickSort(arr, left = 0, right = arr.length - 1) {
  if (left < right) {
    // Partition and get pivot index
    const pivotIndex = partition(arr, left, right);
    
    // Recursively sort left and right partitions
    quickSort(arr, left, pivotIndex - 1);
    quickSort(arr, pivotIndex + 1, right);
  }
  
  return arr;
}

function partition(arr, left, right) {
  // Choose rightmost element as pivot
  const pivot = arr[right];
  let i = left - 1; // Index of smaller element
  
  for (let j = left; j < right; j++) {
    if (arr[j] <= pivot) {
      i++;
      // Swap arr[i] and arr[j]
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
  }
  
  // Swap pivot to its correct position
  [arr[i + 1], arr[right]] = [arr[right], arr[i + 1]];
  
  return i + 1; // Return pivot index
}

// Test
console.log('=== Classic Quicksort (Lomuto) ===');
const arr1 = [10, 7, 8, 9, 1, 5];
console.log('Original:', [...arr1]);
quickSort(arr1);
console.log('Sorted:', arr1); // [1, 5, 7, 8, 9, 10]

const arr2 = [64, 34, 25, 12, 22, 11, 90];
console.log('\nOriginal:', [...arr2]);
quickSort(arr2);
console.log('Sorted:', arr2); // [11, 12, 22, 25, 34, 64, 90]
```

### **Approach 2: Hoare Partition Scheme**
```javascript
/**
 * Quicksort using Hoare partition (more efficient)
 * Performs fewer swaps than Lomuto partition
 */
function quickSortHoare(arr, left = 0, right = arr.length - 1) {
  if (left < right) {
    const pivotIndex = partitionHoare(arr, left, right);
    
    quickSortHoare(arr, left, pivotIndex);
    quickSortHoare(arr, pivotIndex + 1, right);
  }
  
  return arr;
}

function partitionHoare(arr, left, right) {
  // Choose middle element as pivot
  const pivot = arr[Math.floor((left + right) / 2)];
  let i = left - 1;
  let j = right + 1;
  
  while (true) {
    // Find element >= pivot from left
    do {
      i++;
    } while (arr[i] < pivot);
    
    // Find element <= pivot from right
    do {
      j--;
    } while (arr[j] > pivot);
    
    if (i >= j) {
      return j;
    }
    
    // Swap elements
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
}

// Test
console.log('\n=== Quicksort (Hoare) ===');
const arr3 = [10, 7, 8, 9, 1, 5];
console.log('Original:', [...arr3]);
quickSortHoare(arr3);
console.log('Sorted:', arr3); // [1, 5, 7, 8, 9, 10]
```

### **Approach 3: Randomized Quicksort**
```javascript
/**
 * Randomized quicksort to avoid worst case
 * Randomly select pivot to ensure O(n log n) average case
 */
function quickSortRandomized(arr, left = 0, right = arr.length - 1) {
  if (left < right) {
    const pivotIndex = partitionRandomized(arr, left, right);
    
    quickSortRandomized(arr, left, pivotIndex - 1);
    quickSortRandomized(arr, pivotIndex + 1, right);
  }
  
  return arr;
}

function partitionRandomized(arr, left, right) {
  // Randomly select pivot
  const randomIndex = left + Math.floor(Math.random() * (right - left + 1));
  
  // Swap random pivot with rightmost element
  [arr[randomIndex], arr[right]] = [arr[right], arr[randomIndex]];
  
  // Use standard Lomuto partition
  return partition(arr, left, right);
}

// Test
console.log('\n=== Randomized Quicksort ===');
const arr4 = [10, 7, 8, 9, 1, 5];
console.log('Original:', [...arr4]);
quickSortRandomized(arr4);
console.log('Sorted:', arr4); // [1, 5, 7, 8, 9, 10]
```

### **Approach 4: Three-Way Quicksort (Handle Duplicates)**
```javascript
/**
 * Three-way quicksort for arrays with many duplicates
 * Partitions into: < pivot, = pivot, > pivot
 */
function quickSort3Way(arr, left = 0, right = arr.length - 1) {
  if (left >= right) return arr;
  
  // Partition into three parts
  const [lt, gt] = partition3Way(arr, left, right);
  
  // Recursively sort left and right
  quickSort3Way(arr, left, lt - 1);
  quickSort3Way(arr, gt + 1, right);
  
  return arr;
}

function partition3Way(arr, left, right) {
  const pivot = arr[left];
  let lt = left;      // arr[left..lt-1] < pivot
  let i = left + 1;   // arr[lt..i-1] = pivot
  let gt = right;     // arr[gt+1..right] > pivot
  
  while (i <= gt) {
    if (arr[i] < pivot) {
      [arr[lt], arr[i]] = [arr[i], arr[lt]];
      lt++;
      i++;
    } else if (arr[i] > pivot) {
      [arr[i], arr[gt]] = [arr[gt], arr[i]];
      gt--;
    } else {
      i++;
    }
  }
  
  return [lt, gt];
}

// Test
console.log('\n=== Three-Way Quicksort ===');
const arr5 = [4, 2, 7, 2, 9, 2, 4, 2, 1];
console.log('Original:', [...arr5]);
quickSort3Way(arr5);
console.log('Sorted:', arr5); // [1, 2, 2, 2, 2, 4, 4, 7, 9]
```

### **Approach 5: Iterative Quicksort**
```javascript
/**
 * Iterative quicksort using explicit stack
 * Avoids recursion stack overflow for large arrays
 */
function quickSortIterative(arr) {
  const stack = [];
  stack.push([0, arr.length - 1]);
  
  while (stack.length > 0) {
    const [left, right] = stack.pop();
    
    if (left < right) {
      const pivotIndex = partition(arr, left, right);
      
      // Push subarrays to stack
      stack.push([left, pivotIndex - 1]);
      stack.push([pivotIndex + 1, right]);
    }
  }
  
  return arr;
}

// Test
console.log('\n=== Iterative Quicksort ===');
const arr6 = [10, 7, 8, 9, 1, 5];
console.log('Original:', [...arr6]);
quickSortIterative(arr6);
console.log('Sorted:', arr6); // [1, 5, 7, 8, 9, 10]
```

### **Bonus: Hybrid Quicksort (with Insertion Sort)**
```javascript
/**
 * Hybrid approach: use insertion sort for small subarrays
 * More efficient for small partitions
 */
function quickSortHybrid(arr, left = 0, right = arr.length - 1) {
  const THRESHOLD = 10;
  
  if (right - left + 1 <= THRESHOLD) {
    // Use insertion sort for small subarrays
    insertionSort(arr, left, right);
    return arr;
  }
  
  if (left < right) {
    const pivotIndex = partition(arr, left, right);
    quickSortHybrid(arr, left, pivotIndex - 1);
    quickSortHybrid(arr, pivotIndex + 1, right);
  }
  
  return arr;
}

function insertionSort(arr, left, right) {
  for (let i = left + 1; i <= right; i++) {
    const key = arr[i];
    let j = i - 1;
    
    while (j >= left && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    
    arr[j + 1] = key;
  }
}

// Test
console.log('\n=== Hybrid Quicksort ===');
const arr7 = [10, 7, 8, 9, 1, 5, 3, 2, 4, 6];
console.log('Original:', [...arr7]);
quickSortHybrid(arr7);
console.log('Sorted:', arr7); // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

### **Bonus: Quick Select (Find Kth Smallest)**
```javascript
/**
 * Quick select algorithm - find kth smallest element
 * Average time: O(n), Worst: O(n²)
 */
function quickSelect(arr, k) {
  return quickSelectHelper([...arr], 0, arr.length - 1, k - 1);
}

function quickSelectHelper(arr, left, right, k) {
  if (left === right) {
    return arr[left];
  }
  
  const pivotIndex = partition(arr, left, right);
  
  if (k === pivotIndex) {
    return arr[k];
  } else if (k < pivotIndex) {
    return quickSelectHelper(arr, left, pivotIndex - 1, k);
  } else {
    return quickSelectHelper(arr, pivotIndex + 1, right, k);
  }
}

// Test
console.log('\n=== Quick Select ===');
const arr8 = [10, 7, 8, 9, 1, 5];
console.log('Array:', arr8);
console.log('3rd smallest:', quickSelect(arr8, 3)); // 7
console.log('1st smallest:', quickSelect(arr8, 1)); // 1
console.log('6th smallest:', quickSelect(arr8, 6)); // 10
```

### **Visualization and Testing**
```javascript
/**
 * Quicksort with step-by-step visualization
 */
function quickSortVisualize(arr, left = 0, right = arr.length - 1, depth = 0) {
  const indent = '  '.repeat(depth);
  console.log(`${indent}Sorting [${arr.slice(left, right + 1).join(', ')}]`);
  
  if (left < right) {
    const pivotIndex = partition(arr, left, right);
    console.log(`${indent}Pivot: ${arr[pivotIndex]} at index ${pivotIndex}`);
    console.log(`${indent}After partition: [${arr.slice(left, right + 1).join(', ')}]`);
    
    quickSortVisualize(arr, left, pivotIndex - 1, depth + 1);
    quickSortVisualize(arr, pivotIndex + 1, right, depth + 1);
  }
  
  return arr;
}

// Demo
console.log('\n=== Quicksort Visualization ===');
const demo = [5, 2, 8, 1, 9];
console.log('Original:', [...demo]);
quickSortVisualize(demo);
console.log('Final:', demo);

// Edge cases
console.log('\n=== Edge Cases ===');
console.log('Empty:', quickSort([])); // []
console.log('Single:', quickSort([1])); // [1]
console.log('Two:', quickSort([2, 1])); // [1, 2]
console.log('Already sorted:', quickSort([1, 2, 3, 4, 5])); // [1, 2, 3, 4, 5]
console.log('Reverse sorted:', quickSort([5, 4, 3, 2, 1])); // [1, 2, 3, 4, 5]
console.log('All same:', quickSort([3, 3, 3, 3])); // [3, 3, 3, 3]
```

**Interview Tips:**
- Quicksort is divide-and-conquer algorithm, sorts in-place
- Average case: O(n log n), Worst case: O(n²) when already sorted
- Lomuto partition: simpler but more swaps, Hoare: fewer swaps
- Randomized pivot selection avoids worst case
- Three-way partition handles duplicates efficiently
- Not stable (doesn't preserve relative order of equal elements)
- Practical: often fastest for average case, used in many standard libraries
- Space: O(log n) for recursion, O(1) auxiliary space
- Improvements: randomization, median-of-three pivot, hybrid with insertion sort
- Quick select: find kth smallest in O(n) average time
- Common follow-ups: handle duplicates, make stable, avoid worst case
- Interview gotchas: handle edge cases (empty, single, all same)

</details>

68. Implement mergesort algorithm

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Classic Mergesort (Top-Down)**
```javascript
/**
 * Mergesort implementation using divide and conquer
 * Time Complexity: O(n log n) in all cases
 * Space Complexity: O(n) for temporary arrays
 * 
 * Algorithm:
 * 1. Divide array into two halves
 * 2. Recursively sort each half
 * 3. Merge the sorted halves
 */
function mergeSort(arr) {
  // Base case: arrays with 0 or 1 element are already sorted
  if (arr.length <= 1) {
    return arr;
  }
  
  // Divide
  const mid = Math.floor(arr.length / 2);
  const left = arr.slice(0, mid);
  const right = arr.slice(mid);
  
  // Conquer and merge
  return merge(mergeSort(left), mergeSort(right));
}

function merge(left, right) {
  const result = [];
  let i = 0, j = 0;
  
  // Merge elements in sorted order
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
    }
  }
  
  // Add remaining elements from left
  while (i < left.length) {
    result.push(left[i]);
    i++;
  }
  
  // Add remaining elements from right
  while (j < right.length) {
    result.push(right[j]);
    j++;
  }
  
  return result;
}

// Test
console.log('=== Classic Mergesort ===');
const arr1 = [38, 27, 43, 3, 9, 82, 10];
console.log('Original:', arr1);
console.log('Sorted:', mergeSort(arr1)); // [3, 9, 10, 27, 38, 43, 82]

const arr2 = [64, 34, 25, 12, 22, 11, 90];
console.log('\nOriginal:', arr2);
console.log('Sorted:', mergeSort(arr2)); // [11, 12, 22, 25, 34, 64, 90]
```

### **Approach 2: In-Place Mergesort**
```javascript
/**
 * Mergesort that modifies the original array
 * Avoids creating new arrays in recursive calls
 */
function mergeSortInPlace(arr, left = 0, right = arr.length - 1) {
  if (left < right) {
    const mid = Math.floor((left + right) / 2);
    
    // Sort left and right halves
    mergeSortInPlace(arr, left, mid);
    mergeSortInPlace(arr, mid + 1, right);
    
    // Merge sorted halves
    mergeInPlace(arr, left, mid, right);
  }
  
  return arr;
}

function mergeInPlace(arr, left, mid, right) {
  // Create temporary arrays
  const leftArr = arr.slice(left, mid + 1);
  const rightArr = arr.slice(mid + 1, right + 1);
  
  let i = 0, j = 0, k = left;
  
  // Merge back into original array
  while (i < leftArr.length && j < rightArr.length) {
    if (leftArr[i] <= rightArr[j]) {
      arr[k] = leftArr[i];
      i++;
    } else {
      arr[k] = rightArr[j];
      j++;
    }
    k++;
  }
  
  // Copy remaining elements
  while (i < leftArr.length) {
    arr[k] = leftArr[i];
    i++;
    k++;
  }
  
  while (j < rightArr.length) {
    arr[k] = rightArr[j];
    j++;
    k++;
  }
}

// Test
console.log('\n=== In-Place Mergesort ===');
const arr3 = [38, 27, 43, 3, 9, 82, 10];
console.log('Original:', [...arr3]);
mergeSortInPlace(arr3);
console.log('Sorted:', arr3); // [3, 9, 10, 27, 38, 43, 82]
```

### **Approach 3: Bottom-Up Mergesort (Iterative)**
```javascript
/**
 * Iterative mergesort (bottom-up approach)
 * Avoids recursion overhead
 * Time: O(n log n), Space: O(n)
 */
function mergeSortIterative(arr) {
  const n = arr.length;
  if (n <= 1) return arr;
  
  // Start with subarrays of size 1, then 2, 4, 8, ...
  for (let size = 1; size < n; size *= 2) {
    // Merge subarrays of current size
    for (let left = 0; left < n; left += 2 * size) {
      const mid = Math.min(left + size - 1, n - 1);
      const right = Math.min(left + 2 * size - 1, n - 1);
      
      if (mid < right) {
        mergeInPlace(arr, left, mid, right);
      }
    }
  }
  
  return arr;
}

// Test
console.log('\n=== Iterative Mergesort ===');
const arr4 = [38, 27, 43, 3, 9, 82, 10];
console.log('Original:', [...arr4]);
mergeSortIterative(arr4);
console.log('Sorted:', arr4); // [3, 9, 10, 27, 38, 43, 82]
```

### **Approach 4: Mergesort with Custom Comparator**
```javascript
/**
 * Mergesort with custom comparison function
 * Allows sorting by different criteria
 */
function mergeSortCustom(arr, compareFn = (a, b) => a - b) {
  if (arr.length <= 1) return arr;
  
  const mid = Math.floor(arr.length / 2);
  const left = mergeSortCustom(arr.slice(0, mid), compareFn);
  const right = mergeSortCustom(arr.slice(mid), compareFn);
  
  return mergeCustom(left, right, compareFn);
}

function mergeCustom(left, right, compareFn) {
  const result = [];
  let i = 0, j = 0;
  
  while (i < left.length && j < right.length) {
    if (compareFn(left[i], right[j]) <= 0) {
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
    }
  }
  
  return result.concat(left.slice(i)).concat(right.slice(j));
}

// Test with different comparators
console.log('\n=== Mergesort with Custom Comparator ===');

// Sort in descending order
const arr5 = [5, 2, 8, 1, 9];
console.log('Descending:', mergeSortCustom(arr5, (a, b) => b - a)); // [9, 8, 5, 2, 1]

// Sort objects by property
const people = [
  { name: 'John', age: 30 },
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 35 }
];
console.log('Sort by age:', mergeSortCustom(people, (a, b) => a.age - b.age));
// [{ name: 'Alice', age: 25 }, { name: 'John', age: 30 }, { name: 'Bob', age: 35 }]

// Sort strings by length
const words = ['banana', 'pie', 'apple', 'strawberry'];
console.log('Sort by length:', mergeSortCustom(words, (a, b) => a.length - b.length));
// ['pie', 'apple', 'banana', 'strawberry']
```

### **Bonus: Merge K Sorted Arrays**
```javascript
/**
 * Merge k sorted arrays using mergesort approach
 * Time: O(N log k) where N is total elements
 */
function mergeKSortedArrays(arrays) {
  if (arrays.length === 0) return [];
  if (arrays.length === 1) return arrays[0];
  
  // Divide and conquer
  const mid = Math.floor(arrays.length / 2);
  const left = mergeKSortedArrays(arrays.slice(0, mid));
  const right = mergeKSortedArrays(arrays.slice(mid));
  
  return merge(left, right);
}

// Test
console.log('\n=== Merge K Sorted Arrays ===');
const sortedArrays = [
  [1, 4, 7],
  [2, 5, 8],
  [3, 6, 9]
];
console.log('Merged:', mergeKSortedArrays(sortedArrays)); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### **Bonus: Count Inversions Using Mergesort**
```javascript
/**
 * Count inversions (pairs where i < j but arr[i] > arr[j])
 * Uses modified mergesort
 * Time: O(n log n)
 */
function countInversions(arr) {
  if (arr.length <= 1) return { sorted: arr, count: 0 };
  
  const mid = Math.floor(arr.length / 2);
  const left = countInversions(arr.slice(0, mid));
  const right = countInversions(arr.slice(mid));
  
  const merged = mergeAndCount(left.sorted, right.sorted);
  
  return {
    sorted: merged.sorted,
    count: left.count + right.count + merged.count
  };
}

function mergeAndCount(left, right) {
  const result = [];
  let count = 0;
  let i = 0, j = 0;
  
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      result.push(left[i]);
      i++;
    } else {
      result.push(right[j]);
      j++;
      // All remaining elements in left are inversions
      count += left.length - i;
    }
  }
  
  return {
    sorted: result.concat(left.slice(i)).concat(right.slice(j)),
    count
  };
}

// Test
console.log('\n=== Count Inversions ===');
const arr6 = [8, 4, 2, 1];
const result = countInversions(arr6);
console.log('Array:', arr6);
console.log('Inversions:', result.count); // 6
console.log('Sorted:', result.sorted); // [1, 2, 4, 8]
```

### **Bonus: Natural Mergesort**
```javascript
/**
 * Natural mergesort - takes advantage of existing sorted runs
 * More efficient on partially sorted data
 */
function naturalMergeSort(arr) {
  if (arr.length <= 1) return arr;
  
  let runs = findRuns(arr);
  
  while (runs.length > 1) {
    const newRuns = [];
    
    for (let i = 0; i < runs.length; i += 2) {
      if (i + 1 < runs.length) {
        newRuns.push(merge(runs[i], runs[i + 1]));
      } else {
        newRuns.push(runs[i]);
      }
    }
    
    runs = newRuns;
  }
  
  return runs[0];
}

function findRuns(arr) {
  const runs = [];
  let current = [arr[0]];
  
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] >= arr[i - 1]) {
      current.push(arr[i]);
    } else {
      runs.push(current);
      current = [arr[i]];
    }
  }
  
  runs.push(current);
  return runs;
}

// Test
console.log('\n=== Natural Mergesort ===');
const arr7 = [1, 3, 5, 2, 4, 6, 7, 8, 9];
console.log('Original:', arr7);
console.log('Sorted:', naturalMergeSort(arr7)); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### **Visualization and Testing**
```javascript
/**
 * Mergesort with step-by-step visualization
 */
function mergeSortVisualize(arr, depth = 0) {
  const indent = '  '.repeat(depth);
  console.log(`${indent}Dividing: [${arr.join(', ')}]`);
  
  if (arr.length <= 1) {
    console.log(`${indent}Base case: [${arr.join(', ')}]`);
    return arr;
  }
  
  const mid = Math.floor(arr.length / 2);
  const left = arr.slice(0, mid);
  const right = arr.slice(mid);
  
  const sortedLeft = mergeSortVisualize(left, depth + 1);
  const sortedRight = mergeSortVisualize(right, depth + 1);
  
  const merged = merge(sortedLeft, sortedRight);
  console.log(`${indent}Merging: [${sortedLeft.join(', ')}] + [${sortedRight.join(', ')}] = [${merged.join(', ')}]`);
  
  return merged;
}

// Demo
console.log('\n=== Mergesort Visualization ===');
const demo = [5, 2, 8, 1];
console.log('Original:', demo);
const sorted = mergeSortVisualize(demo);
console.log('Final:', sorted);

// Edge cases
console.log('\n=== Edge Cases ===');
console.log('Empty:', mergeSort([])); // []
console.log('Single:', mergeSort([1])); // [1]
console.log('Two:', mergeSort([2, 1])); // [1, 2]
console.log('Already sorted:', mergeSort([1, 2, 3, 4, 5])); // [1, 2, 3, 4, 5]
console.log('Reverse sorted:', mergeSort([5, 4, 3, 2, 1])); // [1, 2, 3, 4, 5]
console.log('Duplicates:', mergeSort([3, 1, 3, 2, 1])); // [1, 1, 2, 3, 3]
```

### **Performance Comparison**
```javascript
/**
 * Compare mergesort with other sorting algorithms
 */
function benchmark(size) {
  const arr = Array.from({ length: size }, () => Math.floor(Math.random() * size));
  
  console.log(`\nArray size: ${size.toLocaleString()}`);
  
  // Mergesort
  let start = performance.now();
  mergeSort([...arr]);
  let time = performance.now() - start;
  console.log(`Mergesort: ${time.toFixed(4)}ms`);
  
  // Built-in sort
  start = performance.now();
  [...arr].sort((a, b) => a - b);
  time = performance.now() - start;
  console.log(`Built-in sort: ${time.toFixed(4)}ms`);
}

console.log('\n=== Performance Comparison ===');
benchmark(1000);
benchmark(10000);
```

**Interview Tips:**
- Mergesort: divide-and-conquer, stable sorting algorithm
- Time: O(n log n) in all cases (best, average, worst)
- Space: O(n) for temporary arrays, not in-place
- Stable: preserves relative order of equal elements
- Predictable performance: always O(n log n), unlike quicksort
- Divide: split array in half, Conquer: recursively sort, Combine: merge sorted halves
- Top-down (recursive) vs bottom-up (iterative)
- Applications: external sorting (large datasets), count inversions, merge k sorted arrays
- Advantages: stable, predictable, works well on linked lists
- Disadvantages: requires O(n) extra space, slower for small arrays
- Optimizations: natural mergesort (uses existing runs), hybrid with insertion sort
- Follow-ups: merge k sorted arrays, count inversions, sort linked list
- Edge cases: empty, single element, already sorted, duplicates

</details>

69. Find the kth largest element in an array

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Sorting (Simple but not optimal)**
```javascript
/**
 * Find kth largest by sorting
 * Time Complexity: O(n log n)
 * Space Complexity: O(1) or O(n) depending on sort implementation
 */
function findKthLargestSort(nums, k) {
  // Sort in descending order
  nums.sort((a, b) => b - a);
  
  // Return kth element (k-1 index)
  return nums[k - 1];
}

// Test
console.log('=== Sorting Approach ===');
const arr1 = [3, 2, 1, 5, 6, 4];
console.log('Array:', arr1);
console.log('2nd largest:', findKthLargestSort([...arr1], 2)); // 5
console.log('4th largest:', findKthLargestSort([...arr1], 4)); // 3
```

### **Approach 2: Min Heap (Optimal for small k)**
```javascript
/**
 * Find kth largest using min heap
 * Time Complexity: O(n log k)
 * Space Complexity: O(k)
 * 
 * Maintain heap of k largest elements
 */
class MinHeap {
  constructor() {
    this.heap = [];
  }
  
  size() {
    return this.heap.length;
  }
  
  peek() {
    return this.heap[0];
  }
  
  push(val) {
    this.heap.push(val);
    this.bubbleUp(this.heap.length - 1);
  }
  
  pop() {
    if (this.size() === 0) return null;
    if (this.size() === 1) return this.heap.pop();
    
    const min = this.heap[0];
    this.heap[0] = this.heap.pop();
    this.bubbleDown(0);
    return min;
  }
  
  bubbleUp(index) {
    while (index > 0) {
      const parentIndex = Math.floor((index - 1) / 2);
      
      if (this.heap[index] >= this.heap[parentIndex]) break;
      
      [this.heap[index], this.heap[parentIndex]] = 
        [this.heap[parentIndex], this.heap[index]];
      
      index = parentIndex;
    }
  }
  
  bubbleDown(index) {
    while (true) {
      let smallest = index;
      const leftChild = 2 * index + 1;
      const rightChild = 2 * index + 2;
      
      if (leftChild < this.size() && this.heap[leftChild] < this.heap[smallest]) {
        smallest = leftChild;
      }
      
      if (rightChild < this.size() && this.heap[rightChild] < this.heap[smallest]) {
        smallest = rightChild;
      }
      
      if (smallest === index) break;
      
      [this.heap[index], this.heap[smallest]] = 
        [this.heap[smallest], this.heap[index]];
      
      index = smallest;
    }
  }
}

function findKthLargestHeap(nums, k) {
  const minHeap = new MinHeap();
  
  // Maintain heap of k largest elements
  for (let num of nums) {
    minHeap.push(num);
    
    if (minHeap.size() > k) {
      minHeap.pop(); // Remove smallest
    }
  }
  
  // Root is kth largest
  return minHeap.peek();
}

// Test
console.log('\n=== Min Heap Approach ===');
const arr2 = [3, 2, 1, 5, 6, 4];
console.log('Array:', arr2);
console.log('2nd largest:', findKthLargestHeap(arr2, 2)); // 5
console.log('4th largest:', findKthLargestHeap(arr2, 4)); // 3
```

### **Approach 3: Quickselect (Optimal - Average O(n))**
```javascript
/**
 * Quickselect algorithm (partition-based)
 * Time Complexity: O(n) average, O(n²) worst case
 * Space Complexity: O(1)
 * 
 * Similar to quicksort but only recurse on one partition
 */
function findKthLargestQuickselect(nums, k) {
  // Convert to finding (n-k)th smallest (0-indexed)
  return quickselect(nums, 0, nums.length - 1, nums.length - k);
}

function quickselect(nums, left, right, k) {
  if (left === right) {
    return nums[left];
  }
  
  // Partition and get pivot index
  const pivotIndex = partition(nums, left, right);
  
  if (k === pivotIndex) {
    return nums[k];
  } else if (k < pivotIndex) {
    return quickselect(nums, left, pivotIndex - 1, k);
  } else {
    return quickselect(nums, pivotIndex + 1, right, k);
  }
}

function partition(nums, left, right) {
  // Randomize pivot to avoid worst case
  const randomIndex = left + Math.floor(Math.random() * (right - left + 1));
  [nums[randomIndex], nums[right]] = [nums[right], nums[randomIndex]];
  
  const pivot = nums[right];
  let i = left - 1;
  
  for (let j = left; j < right; j++) {
    if (nums[j] <= pivot) {
      i++;
      [nums[i], nums[j]] = [nums[j], nums[i]];
    }
  }
  
  [nums[i + 1], nums[right]] = [nums[right], nums[i + 1]];
  return i + 1;
}

// Test
console.log('\n=== Quickselect Approach ===');
const arr3 = [3, 2, 1, 5, 6, 4];
console.log('Array:', arr3);
console.log('2nd largest:', findKthLargestQuickselect([...arr3], 2)); // 5
console.log('4th largest:', findKthLargestQuickselect([...arr3], 4)); // 3

const arr4 = [3, 2, 3, 1, 2, 4, 5, 5, 6];
console.log('\nArray:', arr4);
console.log('4th largest:', findKthLargestQuickselect([...arr4], 4)); // 4
```

### **Approach 4: Using Median of Medians**
```javascript
/**
 * Quickselect with median of medians pivot selection
 * Guarantees O(n) worst case time
 */
function findKthLargestMedianOfMedians(nums, k) {
  return quickselectMedianOfMedians(nums, 0, nums.length - 1, nums.length - k);
}

function quickselectMedianOfMedians(nums, left, right, k) {
  if (left === right) {
    return nums[left];
  }
  
  // Use median of medians for pivot
  const pivotIndex = medianOfMedians(nums, left, right);
  const newPivotIndex = partitionAroundPivot(nums, left, right, pivotIndex);
  
  if (k === newPivotIndex) {
    return nums[k];
  } else if (k < newPivotIndex) {
    return quickselectMedianOfMedians(nums, left, newPivotIndex - 1, k);
  } else {
    return quickselectMedianOfMedians(nums, newPivotIndex + 1, right, k);
  }
}

function medianOfMedians(nums, left, right) {
  const n = right - left + 1;
  
  if (n <= 5) {
    // Sort small group and return median
    const group = nums.slice(left, right + 1).sort((a, b) => a - b);
    return left + Math.floor(group.length / 2);
  }
  
  // Find medians of groups of 5
  const medians = [];
  for (let i = left; i <= right; i += 5) {
    const groupEnd = Math.min(i + 4, right);
    const group = nums.slice(i, groupEnd + 1).sort((a, b) => a - b);
    medians.push(group[Math.floor(group.length / 2)]);
  }
  
  // Recursively find median of medians
  return medianOfMedians(medians, 0, medians.length - 1);
}

function partitionAroundPivot(nums, left, right, pivotIndex) {
  const pivotValue = nums[pivotIndex];
  
  // Move pivot to end
  [nums[pivotIndex], nums[right]] = [nums[right], nums[pivotIndex]];
  
  let i = left - 1;
  for (let j = left; j < right; j++) {
    if (nums[j] <= pivotValue) {
      i++;
      [nums[i], nums[j]] = [nums[j], nums[i]];
    }
  }
  
  [nums[i + 1], nums[right]] = [nums[right], nums[i + 1]];
  return i + 1;
}

// Test
console.log('\n=== Median of Medians Approach ===');
const arr5 = [3, 2, 1, 5, 6, 4];
console.log('Array:', arr5);
console.log('2nd largest:', findKthLargestMedianOfMedians([...arr5], 2)); // 5
```

### **Bonus: Find Kth Largest in Stream**
```javascript
/**
 * Find kth largest in a stream of numbers
 * Uses min heap for efficient updates
 */
class KthLargest {
  constructor(k, nums) {
    this.k = k;
    this.minHeap = new MinHeap();
    
    for (let num of nums) {
      this.add(num);
    }
  }
  
  add(val) {
    this.minHeap.push(val);
    
    if (this.minHeap.size() > this.k) {
      this.minHeap.pop();
    }
    
    return this.minHeap.peek();
  }
  
  getKthLargest() {
    return this.minHeap.peek();
  }
}

// Test
console.log('\n=== Kth Largest in Stream ===');
const kthLargest = new KthLargest(3, [4, 5, 8, 2]);
console.log('Initial 3rd largest:', kthLargest.getKthLargest()); // 4
console.log('Add 3:', kthLargest.add(3)); // 4
console.log('Add 5:', kthLargest.add(5)); // 5
console.log('Add 10:', kthLargest.add(10)); // 5
console.log('Add 9:', kthLargest.add(9)); // 8
console.log('Add 4:', kthLargest.add(4)); // 8
```

### **Bonus: Find Top K Elements**
```javascript
/**
 * Find top k largest elements (not just kth)
 * Returns array of k largest elements
 */
function findTopK(nums, k) {
  const minHeap = new MinHeap();
  
  for (let num of nums) {
    minHeap.push(num);
    
    if (minHeap.size() > k) {
      minHeap.pop();
    }
  }
  
  // Extract all elements from heap
  const result = [];
  while (minHeap.size() > 0) {
    result.push(minHeap.pop());
  }
  
  return result.reverse(); // Return in descending order
}

// Test
console.log('\n=== Top K Elements ===');
const arr6 = [3, 2, 1, 5, 6, 4];
console.log('Array:', arr6);
console.log('Top 3:', findTopK(arr6, 3)); // [6, 5, 4]
console.log('Top 2:', findTopK(arr6, 2)); // [6, 5]
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Single element
console.log('Single element:', findKthLargestQuickselect([5], 1)); // 5

// All same elements
console.log('All same:', findKthLargestQuickselect([3, 3, 3, 3], 2)); // 3

// k = 1 (largest)
console.log('Largest:', findKthLargestQuickselect([3, 2, 1, 5, 6, 4], 1)); // 6

// k = n (smallest)
console.log('Smallest:', findKthLargestQuickselect([3, 2, 1, 5, 6, 4], 6)); // 1

// Negative numbers
console.log('Negative:', findKthLargestQuickselect([-1, -2, -3, -4], 2)); // -2

// Large k
const large = Array.from({ length: 1000 }, (_, i) => i);
console.log('Large array 500th:', findKthLargestQuickselect([...large], 500)); // 500
```

### **Performance Comparison**
```javascript
/**
 * Compare different approaches
 */
function benchmarkKthLargest(size, k) {
  const arr = Array.from({ length: size }, () => Math.floor(Math.random() * size));
  
  console.log(`\nArray size: ${size.toLocaleString()}, k: ${k}`);
  
  // Sorting
  let start = performance.now();
  findKthLargestSort([...arr], k);
  let time = performance.now() - start;
  console.log(`Sorting: ${time.toFixed(4)}ms`);
  
  // Min Heap
  start = performance.now();
  findKthLargestHeap([...arr], k);
  time = performance.now() - start;
  console.log(`Min Heap: ${time.toFixed(4)}ms`);
  
  // Quickselect
  start = performance.now();
  findKthLargestQuickselect([...arr], k);
  time = performance.now() - start;
  console.log(`Quickselect: ${time.toFixed(4)}ms`);
}

console.log('\n=== Performance Comparison ===');
benchmarkKthLargest(10000, 100);
benchmarkKthLargest(10000, 5000);
```

**Interview Tips:**
- Three main approaches: sort O(n log n), heap O(n log k), quickselect O(n) average
- Quickselect is optimal for average case: O(n) time, O(1) space
- Min heap approach best when k is small or processing stream
- Sorting simplest but not optimal: O(n log n) time
- Quickselect similar to quicksort but only recurse on one partition
- Convert "kth largest" to "(n-k)th smallest" for implementation
- Median of medians guarantees O(n) worst case but complex
- For streaming data, maintain heap of size k
- Common variations: kth smallest, top k elements, kth largest in stream
- Edge cases: k = 1 (max), k = n (min), duplicates, negative numbers
- Optimization: randomize pivot to avoid worst case O(n²)
- Follow-ups: find median, top k frequent elements, kth closest to x

</details>

70. Implement a function to find all combinations of a string

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Recursive Backtracking (All Subsets)**
```javascript
/**
 * Find all combinations (subsets) of a string
 * Time Complexity: O(2^n) - 2^n subsets
 * Space Complexity: O(n) recursion depth
 * 
 * Generates all possible subsets including empty string
 */
function findCombinations(str) {
  const result = [];
  
  function backtrack(index, current) {
    // Add current combination
    result.push(current);
    
    // Try adding each remaining character
    for (let i = index; i < str.length; i++) {
      backtrack(i + 1, current + str[i]);
    }
  }
  
  backtrack(0, '');
  return result;
}

// Test
console.log('=== All Combinations (Subsets) ===');
console.log('abc:', findCombinations('abc'));
// ['', 'a', 'ab', 'abc', 'ac', 'b', 'bc', 'c']

console.log('\nab:', findCombinations('ab'));
// ['', 'a', 'ab', 'b']

console.log('\n123:', findCombinations('123'));
// ['', '1', '12', '123', '13', '2', '23', '3']
```

### **Approach 2: Iterative with Bit Manipulation**
```javascript
/**
 * Generate all combinations using bit manipulation
 * Each bit represents whether to include character at that position
 * Time: O(n * 2^n), Space: O(1) excluding output
 */
function findCombinationsBits(str) {
  const result = [];
  const n = str.length;
  const totalCombinations = Math.pow(2, n);
  
  // Iterate through all possible bit patterns
  for (let i = 0; i < totalCombinations; i++) {
    let combination = '';
    
    // Check each bit
    for (let j = 0; j < n; j++) {
      // If jth bit is set, include str[j]
      if ((i & (1 << j)) !== 0) {
        combination += str[j];
      }
    }
    
    result.push(combination);
  }
  
  return result;
}

// Test
console.log('\n=== Combinations with Bit Manipulation ===');
console.log('abc:', findCombinationsBits('abc'));
// ['', 'a', 'b', 'ab', 'c', 'ac', 'bc', 'abc']

console.log('\n12:', findCombinationsBits('12'));
// ['', '1', '2', '12']
```

### **Approach 3: Fixed-Length Combinations**
```javascript
/**
 * Find all combinations of specific length k
 * Time: O(C(n,k) * k) where C(n,k) is binomial coefficient
 * Space: O(k) recursion depth
 */
function findCombinationsOfLength(str, k) {
  const result = [];
  
  function backtrack(index, current) {
    // Found combination of length k
    if (current.length === k) {
      result.push(current);
      return;
    }
    
    // Try remaining characters
    for (let i = index; i < str.length; i++) {
      backtrack(i + 1, current + str[i]);
    }
  }
  
  backtrack(0, '');
  return result;
}

// Test
console.log('\n=== Combinations of Specific Length ===');
console.log('abc, length 2:', findCombinationsOfLength('abc', 2));
// ['ab', 'ac', 'bc']

console.log('\nabcd, length 2:', findCombinationsOfLength('abcd', 2));
// ['ab', 'ac', 'ad', 'bc', 'bd', 'cd']

console.log('\nabcd, length 3:', findCombinationsOfLength('abcd', 3));
// ['abc', 'abd', 'acd', 'bcd']

console.log('\n12345, length 3:', findCombinationsOfLength('12345', 3));
// ['123', '124', '125', '134', '135', '145', '234', '235', '245', '345']
```

### **Approach 4: Combinations with Duplicates Handling**
```javascript
/**
 * Find unique combinations when string has duplicate characters
 * Time: O(2^n), Space: O(n)
 */
function findUniqueCombinations(str) {
  const result = new Set();
  const chars = str.split('').sort(); // Sort to group duplicates
  
  function backtrack(index, current) {
    result.add(current);
    
    for (let i = index; i < chars.length; i++) {
      // Skip duplicates at same level
      if (i > index && chars[i] === chars[i - 1]) continue;
      
      backtrack(i + 1, current + chars[i]);
    }
  }
  
  backtrack(0, '');
  return Array.from(result);
}

// Test
console.log('\n=== Unique Combinations (with duplicates) ===');
console.log('aab:', findUniqueCombinations('aab'));
// ['', 'a', 'aa', 'aab', 'ab', 'b']

console.log('\n112:', findUniqueCombinations('112'));
// ['', '1', '11', '112', '12', '2']
```

### **Approach 5: Iterative Building (Bottom-Up)**
```javascript
/**
 * Build combinations iteratively
 * Start with empty set, add each character to existing combinations
 * Time: O(2^n), Space: O(2^n)
 */
function findCombinationsIterative(str) {
  let result = [''];
  
  for (let char of str) {
    const newCombinations = [];
    
    // For each existing combination, create a new one by adding current char
    for (let combination of result) {
      newCombinations.push(combination + char);
    }
    
    result = result.concat(newCombinations);
  }
  
  return result;
}

// Test
console.log('\n=== Iterative Building ===');
console.log('abc:', findCombinationsIterative('abc'));
// ['', 'a', 'b', 'ab', 'c', 'ac', 'bc', 'abc']

console.log('\nxy:', findCombinationsIterative('xy'));
// ['', 'x', 'y', 'xy']
```

### **Bonus: Combinations with Repetition**
```javascript
/**
 * Find combinations with repetition allowed
 * Each character can be used multiple times
 * Generate combinations of length k
 */
function findCombinationsWithRepetition(str, k) {
  const result = [];
  const chars = [...new Set(str.split(''))]; // Unique characters
  
  function backtrack(current, start) {
    if (current.length === k) {
      result.push(current);
      return;
    }
    
    // Can reuse same character (start from current position, not i+1)
    for (let i = start; i < chars.length; i++) {
      backtrack(current + chars[i], i);
    }
  }
  
  backtrack('', 0);
  return result;
}

// Test
console.log('\n=== Combinations with Repetition ===');
console.log('abc, length 2:', findCombinationsWithRepetition('abc', 2));
// ['aa', 'ab', 'ac', 'bb', 'bc', 'cc']

console.log('\n12, length 3:', findCombinationsWithRepetition('12', 3));
// ['111', '112', '122', '222']
```

### **Bonus: Grouped by Length**
```javascript
/**
 * Group combinations by their length
 * Returns object with lengths as keys
 */
function findCombinationsGrouped(str) {
  const grouped = {};
  
  function backtrack(index, current) {
    const len = current.length;
    if (!grouped[len]) grouped[len] = [];
    grouped[len].push(current);
    
    for (let i = index; i < str.length; i++) {
      backtrack(i + 1, current + str[i]);
    }
  }
  
  backtrack(0, '');
  return grouped;
}

// Test
console.log('\n=== Grouped by Length ===');
const grouped = findCombinationsGrouped('abc');
console.log('Length 0:', grouped[0]); // ['']
console.log('Length 1:', grouped[1]); // ['a', 'b', 'c']
console.log('Length 2:', grouped[2]); // ['ab', 'ac', 'bc']
console.log('Length 3:', grouped[3]); // ['abc']
```

### **Bonus: Count Only (Optimization)**
```javascript
/**
 * Count combinations without generating them
 * For n characters: 2^n total combinations
 * For length k: C(n, k) = n! / (k! * (n-k)!)
 */
function countCombinations(n) {
  return Math.pow(2, n);
}

function countCombinationsOfLength(n, k) {
  if (k > n || k < 0) return 0;
  if (k === 0 || k === n) return 1;
  
  // Calculate C(n, k) = n! / (k! * (n-k)!)
  let result = 1;
  for (let i = 0; i < k; i++) {
    result = result * (n - i) / (i + 1);
  }
  return Math.floor(result);
}

// Test
console.log('\n=== Count Combinations ===');
console.log('String length 3, total combinations:', countCombinations(3)); // 8
console.log('String length 5, total combinations:', countCombinations(5)); // 32
console.log('C(5, 2):', countCombinationsOfLength(5, 2)); // 10
console.log('C(6, 3):', countCombinationsOfLength(6, 3)); // 20
```

### **Visualization and Edge Cases**
```javascript
/**
 * Visualize the combination generation process
 */
function findCombinationsVisualize(str) {
  const result = [];
  
  function backtrack(index, current, depth) {
    const indent = '  '.repeat(depth);
    console.log(`${indent}At index ${index}, current: "${current}"`);
    
    result.push(current);
    
    for (let i = index; i < str.length; i++) {
      console.log(`${indent}Adding "${str[i]}"`);
      backtrack(i + 1, current + str[i], depth + 1);
    }
  }
  
  console.log('\n=== Combination Generation Process ===');
  backtrack(0, '', 0);
  return result;
}

// Demo
console.log('\nGenerating combinations of "ab":');
findCombinationsVisualize('ab');

// Edge cases
console.log('\n=== Edge Cases ===');
console.log('Empty string:', findCombinations('')); // ['']
console.log('Single char:', findCombinations('a')); // ['', 'a']
console.log('Length 0:', findCombinationsOfLength('abc', 0)); // ['']
console.log('Length > string:', findCombinationsOfLength('ab', 3)); // []
console.log('All same chars:', findUniqueCombinations('aaa')); 
// ['', 'a', 'aa', 'aaa']
```

### **Performance Testing**
```javascript
/**
 * Test performance with different string lengths
 */
function benchmarkCombinations(length) {
  const str = 'a'.repeat(length);
  
  console.log(`\nString length: ${length}`);
  console.log(`Expected combinations: ${Math.pow(2, length)}`);
  
  const start = performance.now();
  const combinations = findCombinations(str);
  const time = performance.now() - start;
  
  console.log(`Generated: ${combinations.length} combinations`);
  console.log(`Time: ${time.toFixed(4)}ms`);
}

console.log('\n=== Performance Testing ===');
benchmarkCombinations(5);   // 32 combinations
benchmarkCombinations(10);  // 1024 combinations
benchmarkCombinations(15);  // 32768 combinations
```

### **Real-World Applications**
```javascript
/**
 * Generate password combinations for brute force analysis
 */
function generatePasswordCombinations(charset, length) {
  const result = [];
  
  function backtrack(current) {
    if (current.length === length) {
      result.push(current);
      return;
    }
    
    for (let char of charset) {
      backtrack(current + char);
    }
  }
  
  backtrack('');
  return result;
}

console.log('\n=== Password Combinations ===');
const digits = '012';
console.log('2-digit pins from 012:', generatePasswordCombinations(digits, 2));
// ['00', '01', '02', '10', '11', '12', '20', '21', '22']

/**
 * Find all possible substrings (different from subsets)
 */
function findSubstrings(str) {
  const result = new Set();
  
  for (let i = 0; i < str.length; i++) {
    for (let j = i + 1; j <= str.length; j++) {
      result.add(str.substring(i, j));
    }
  }
  
  return Array.from(result).sort();
}

console.log('\n=== All Substrings (vs Combinations) ===');
console.log('Substrings of "abc":', findSubstrings('abc'));
// ['a', 'ab', 'abc', 'b', 'bc', 'c']
console.log('Combinations of "abc":', findCombinations('abc').filter(s => s));
// ['a', 'ab', 'abc', 'ac', 'b', 'bc', 'c']
```

**Interview Tips:**
- Combinations = subsets = power set (all possible selections)
- Total combinations of n elements: 2^n (including empty set)
- Combinations of length k from n: C(n,k) = n!/(k!(n-k)!)
- Two main approaches: backtracking O(2^n) or bit manipulation O(n*2^n)
- Backtracking more flexible: can filter, set constraints, handle duplicates
- Bit manipulation efficient for small strings, each bit = include/exclude
- Order doesn't matter in combinations: "ab" same as "ba" (vs permutations)
- Handle duplicates: sort string first, skip duplicates at same recursion level
- Fixed-length: add length constraint to backtracking
- Space complexity: O(n) for recursion, O(2^n) for storing results
- Optimizations: count without generating, early termination, pruning
- Related problems: permutations (order matters), substrings (contiguous), subsequences
- Applications: feature selection, set operations, combinations generator
- Edge cases: empty string, single character, all duplicates, k > n
- Clarify: include empty set? duplicates allowed? fixed length? order matters?
- Follow-ups: k-length combinations, with repetition, unique only, count only

</details>

71. Solve the "Two Sum" problem

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Brute Force (Two Nested Loops)**
```javascript
/**
 * Find two numbers that sum to target using brute force
 * Time Complexity: O(n²)
 * Space Complexity: O(1)
 */
function twoSumBruteForce(nums, target) {
  const n = nums.length;
  
  // Check every pair
  for (let i = 0; i < n; i++) {
    for (let j = i + 1; j < n; j++) {
      if (nums[i] + nums[j] === target) {
        return [i, j];
      }
    }
  }
  
  return null; // No solution found
}

// Test
console.log('=== Brute Force Approach ===');
console.log('[2,7,11,15], target 9:', twoSumBruteForce([2, 7, 11, 15], 9)); // [0, 1]
console.log('[3,2,4], target 6:', twoSumBruteForce([3, 2, 4], 6)); // [1, 2]
console.log('[3,3], target 6:', twoSumBruteForce([3, 3], 6)); // [0, 1]
```

### **Approach 2: Hash Map (Optimal - One Pass)**
```javascript
/**
 * Use hash map to find complement in O(n) time
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * For each number, check if (target - number) exists in map
 */
function twoSum(nums, target) {
  const map = new Map();
  
  for (let i = 0; i < nums.length; i++) {
    const complement = target - nums[i];
    
    // Check if complement exists
    if (map.has(complement)) {
      return [map.get(complement), i];
    }
    
    // Store current number and index
    map.set(nums[i], i);
  }
  
  return null;
}

// Test
console.log('\n=== Hash Map Approach (Optimal) ===');
console.log('[2,7,11,15], target 9:', twoSum([2, 7, 11, 15], 9)); // [0, 1]
console.log('[3,2,4], target 6:', twoSum([3, 2, 4], 6)); // [1, 2]
console.log('[3,3], target 6:', twoSum([3, 3], 6)); // [0, 1]
console.log('[1,2,3,4], target 7:', twoSum([1, 2, 3, 4], 7)); // [2, 3]
```

### **Approach 3: Two Pointers (Sorted Array)**
```javascript
/**
 * Two pointers approach for sorted array
 * Time Complexity: O(n log n) for sorting + O(n) = O(n log n)
 * Space Complexity: O(n) to store indices
 * 
 * Only use if array is already sorted or sorting is acceptable
 */
function twoSumTwoPointers(nums, target) {
  // Create array of [value, originalIndex] pairs
  const indexed = nums.map((num, i) => [num, i]);
  
  // Sort by value
  indexed.sort((a, b) => a[0] - b[0]);
  
  let left = 0;
  let right = indexed.length - 1;
  
  while (left < right) {
    const sum = indexed[left][0] + indexed[right][0];
    
    if (sum === target) {
      return [indexed[left][1], indexed[right][1]].sort((a, b) => a - b);
    } else if (sum < target) {
      left++;
    } else {
      right--;
    }
  }
  
  return null;
}

// Test
console.log('\n=== Two Pointers Approach ===');
console.log('[2,7,11,15], target 9:', twoSumTwoPointers([2, 7, 11, 15], 9)); // [0, 1]
console.log('[3,2,4], target 6:', twoSumTwoPointers([3, 2, 4], 6)); // [1, 2]
console.log('[1,5,3,7], target 8:', twoSumTwoPointers([1, 5, 3, 7], 8)); // [0, 3]
```

### **Approach 4: Return All Pairs**
```javascript
/**
 * Find all pairs that sum to target (not just indices)
 * Returns array of pairs
 * Time: O(n), Space: O(n)
 */
function twoSumAllPairs(nums, target) {
  const seen = new Set();
  const pairs = [];
  const used = new Set();
  
  for (let num of nums) {
    const complement = target - num;
    
    if (seen.has(complement)) {
      // Create sorted pair to avoid duplicates
      const pair = [Math.min(num, complement), Math.max(num, complement)];
      const pairKey = pair.join(',');
      
      if (!used.has(pairKey)) {
        pairs.push(pair);
        used.add(pairKey);
      }
    }
    
    seen.add(num);
  }
  
  return pairs;
}

// Test
console.log('\n=== All Pairs ===');
console.log('[1,2,3,4,3], target 6:', twoSumAllPairs([1, 2, 3, 4, 3], 6));
// [[2, 4], [3, 3]]

console.log('[1,1,1,1], target 2:', twoSumAllPairs([1, 1, 1, 1], 2));
// [[1, 1]]

console.log('[2,7,11,15,4,5], target 9:', twoSumAllPairs([2, 7, 11, 15, 4, 5], 9));
// [[2, 7], [4, 5]]
```

### **Bonus: Two Sum with Multiple Solutions**
```javascript
/**
 * Find all index pairs that sum to target
 * Returns all possible index combinations
 */
function twoSumAllIndices(nums, target) {
  const result = [];
  
  for (let i = 0; i < nums.length; i++) {
    for (let j = i + 1; j < nums.length; j++) {
      if (nums[i] + nums[j] === target) {
        result.push([i, j]);
      }
    }
  }
  
  return result;
}

// Test
console.log('\n=== All Index Pairs ===');
console.log('[1,2,3,2,4], target 4:', twoSumAllIndices([1, 2, 3, 2, 4], 4));
// [[0, 2], [1, 3]]

console.log('[3,3,3], target 6:', twoSumAllIndices([3, 3, 3], 6));
// [[0, 1], [0, 2], [1, 2]]
```

### **Bonus: Two Sum - Count Pairs**
```javascript
/**
 * Count how many pairs sum to target
 * Time: O(n), Space: O(n)
 */
function twoSumCount(nums, target) {
  const map = new Map();
  let count = 0;
  
  for (let num of nums) {
    const complement = target - num;
    
    if (map.has(complement)) {
      count += map.get(complement);
    }
    
    map.set(num, (map.get(num) || 0) + 1);
  }
  
  return count;
}

// Test
console.log('\n=== Count Pairs ===');
console.log('[1,2,3,2,4], target 4:', twoSumCount([1, 2, 3, 2, 4], 4)); // 2
console.log('[1,1,1,1], target 2:', twoSumCount([1, 1, 1, 1], 2)); // 6
console.log('[3,3,3], target 6:', twoSumCount([3, 3, 3], 6)); // 3
```

### **Bonus: Two Sum - Sorted Array (No Extra Space)**
```javascript
/**
 * Two sum for pre-sorted array
 * Time: O(n), Space: O(1)
 */
function twoSumSorted(nums, target) {
  let left = 0;
  let right = nums.length - 1;
  
  while (left < right) {
    const sum = nums[left] + nums[right];
    
    if (sum === target) {
      return [left, right];
    } else if (sum < target) {
      left++;
    } else {
      right--;
    }
  }
  
  return null;
}

// Test
console.log('\n=== Sorted Array (No Extra Space) ===');
console.log('[1,2,3,4], target 5:', twoSumSorted([1, 2, 3, 4], 5)); // [0, 3]
console.log('[2,7,11,15], target 9:', twoSumSorted([2, 7, 11, 15], 9)); // [0, 1]
```

### **Bonus: Two Sum - Data Structure Design**
```javascript
/**
 * Design a data structure that supports add and find
 * Optimize for either add or find operation
 */
class TwoSum {
  constructor() {
    this.nums = [];
    this.map = new Map();
  }
  
  // Add number to data structure
  add(num) {
    this.nums.push(num);
    this.map.set(num, (this.map.get(num) || 0) + 1);
  }
  
  // Find if there exists any pair that sums to target
  find(target) {
    for (let num of this.map.keys()) {
      const complement = target - num;
      
      if (complement === num) {
        // Need at least 2 occurrences
        if (this.map.get(num) >= 2) return true;
      } else {
        if (this.map.has(complement)) return true;
      }
    }
    
    return false;
  }
}

// Test
console.log('\n=== TwoSum Data Structure ===');
const twoSumDS = new TwoSum();
twoSumDS.add(1);
twoSumDS.add(3);
twoSumDS.add(5);
console.log('Find 4:', twoSumDS.find(4)); // true (1 + 3)
console.log('Find 7:', twoSumDS.find(7)); // false
twoSumDS.add(2);
console.log('Find 7:', twoSumDS.find(7)); // true (2 + 5)
console.log('Find 6:', twoSumDS.find(6)); // true (1 + 5)
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// No solution
console.log('No solution:', twoSum([1, 2, 3], 7)); // null

// Negative numbers
console.log('Negative:', twoSum([-1, -2, -3, -4], -6)); // [1, 3] or [2, 3]

// Zero
console.log('With zero:', twoSum([0, 4, 3, 0], 0)); // [0, 3]

// Large numbers
console.log('Large:', twoSum([1000000000, 1000000001], 2000000001)); // [0, 1]

// Minimum array
console.log('Two elements:', twoSum([1, 2], 3)); // [0, 1]

// Same number twice
console.log('Same twice:', twoSum([3, 3], 6)); // [0, 1]

// Multiple valid pairs (returns first found)
console.log('Multiple pairs:', twoSum([1, 2, 3, 4, 5], 5)); // [0, 3] or [1, 2]
```

### **Performance Comparison**
```javascript
/**
 * Compare different approaches
 */
function benchmarkTwoSum(size) {
  const nums = Array.from({ length: size }, () => Math.floor(Math.random() * size));
  const target = nums[0] + nums[1]; // Ensure solution exists
  
  console.log(`\nArray size: ${size.toLocaleString()}`);
  
  // Brute Force
  let start = performance.now();
  twoSumBruteForce(nums, target);
  let time = performance.now() - start;
  console.log(`Brute Force: ${time.toFixed(4)}ms`);
  
  // Hash Map
  start = performance.now();
  twoSum(nums, target);
  time = performance.now() - start;
  console.log(`Hash Map: ${time.toFixed(4)}ms`);
  
  // Two Pointers
  start = performance.now();
  twoSumTwoPointers(nums, target);
  time = performance.now() - start;
  console.log(`Two Pointers: ${time.toFixed(4)}ms`);
}

console.log('\n=== Performance Comparison ===');
benchmarkTwoSum(1000);
benchmarkTwoSum(5000);
```

**Interview Tips:**
- Classic problem, know it well - very common in interviews
- Hash map approach is optimal: O(n) time, O(n) space
- Key insight: for each number, check if (target - number) exists
- One-pass hash map: store as you iterate, check complement first
- Two pointers only works if array is sorted: O(n) time, O(1) space
- Brute force O(n²) acceptable only for very small arrays
- Handle edge cases: no solution, duplicates, negative numbers, zero
- Clarify: return indices or values? first pair or all pairs? sorted array?
- Similar problems: 3Sum, 4Sum, Two Sum II (sorted), Two Sum III (data structure)
- Variations: count pairs, find all pairs, closest to target, less than target
- Can't use same element twice (i ≠ j)
- If sorted array given, use two pointers (O(1) space)
- For unsorted, hash map is best (O(n) time)
- Follow-ups: what if multiple solutions? what if need all pairs? what about 3sum?
- Real applications: finding complements, pair matching, subset sum variant

</details>

72. Solve the "Three Sum" problem

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Sort + Two Pointers (Optimal)**
```javascript
/**
 * Find all unique triplets that sum to zero
 * Time Complexity: O(n²)
 * Space Complexity: O(1) excluding output
 * 
 * Sort array, fix one element, use two pointers for remaining two
 */
function threeSum(nums) {
  const result = [];
  const n = nums.length;
  
  // Sort array for two pointers approach
  nums.sort((a, b) => a - b);
  
  for (let i = 0; i < n - 2; i++) {
    // Skip duplicates for first element
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    
    // Two pointers for remaining elements
    let left = i + 1;
    let right = n - 1;
    
    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];
      
      if (sum === 0) {
        result.push([nums[i], nums[left], nums[right]]);
        
        // Skip duplicates for second element
        while (left < right && nums[left] === nums[left + 1]) left++;
        // Skip duplicates for third element
        while (left < right && nums[right] === nums[right - 1]) right--;
        
        left++;
        right--;
      } else if (sum < 0) {
        left++;
      } else {
        right--;
      }
    }
  }
  
  return result;
}

// Test
console.log('=== Three Sum (Optimal) ===');
console.log('[-1,0,1,2,-1,-4]:', threeSum([-1, 0, 1, 2, -1, -4]));
// [[-1, -1, 2], [-1, 0, 1]]

console.log('\n[0,0,0]:', threeSum([0, 0, 0]));
// [[0, 0, 0]]

console.log('\n[0,1,1]:', threeSum([0, 1, 1]));
// []

console.log('\n[-2,0,1,1,2]:', threeSum([-2, 0, 1, 1, 2]));
// [[-2, 0, 2], [-2, 1, 1]]
```

### **Approach 2: Hash Set (Alternative)**
```javascript
/**
 * Use hash set for each pair to find third element
 * Time: O(n²), Space: O(n)
 */
function threeSumHashSet(nums) {
  const result = [];
  const n = nums.length;
  
  // Sort to handle duplicates
  nums.sort((a, b) => a - b);
  
  for (let i = 0; i < n - 2; i++) {
    // Skip duplicate first elements
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    
    const seen = new Set();
    const target = -nums[i];
    
    for (let j = i + 1; j < n; j++) {
      const complement = target - nums[j];
      
      if (seen.has(complement)) {
        result.push([nums[i], complement, nums[j]]);
        
        // Skip duplicates for second element
        while (j + 1 < n && nums[j] === nums[j + 1]) j++;
      }
      
      seen.add(nums[j]);
    }
  }
  
  return result;
}

// Test
console.log('\n=== Three Sum (Hash Set) ===');
console.log('[-1,0,1,2,-1,-4]:', threeSumHashSet([-1, 0, 1, 2, -1, -4]));
// [[-1, -1, 2], [-1, 0, 1]]

console.log('\n[-2,0,0,2,2]:', threeSumHashSet([-2, 0, 0, 2, 2]));
// [[-2, 0, 2]]
```

### **Approach 3: Three Sum with Custom Target**
```javascript
/**
 * Find triplets that sum to any target (not just 0)
 * Time: O(n²), Space: O(1)
 */
function threeSumTarget(nums, target) {
  const result = [];
  const n = nums.length;
  
  nums.sort((a, b) => a - b);
  
  for (let i = 0; i < n - 2; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    
    let left = i + 1;
    let right = n - 1;
    
    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];
      
      if (sum === target) {
        result.push([nums[i], nums[left], nums[right]]);
        
        while (left < right && nums[left] === nums[left + 1]) left++;
        while (left < right && nums[right] === nums[right - 1]) right--;
        
        left++;
        right--;
      } else if (sum < target) {
        left++;
      } else {
        right--;
      }
    }
  }
  
  return result;
}

// Test
console.log('\n=== Three Sum with Target ===');
console.log('[1,2,3,4,5], target=9:', threeSumTarget([1, 2, 3, 4, 5], 9));
// [[1, 3, 5], [2, 3, 4]]

console.log('\n[-1,0,1,2,-1,-4], target=0:', threeSumTarget([-1, 0, 1, 2, -1, -4], 0));
// [[-1, -1, 2], [-1, 0, 1]]

console.log('\n[1,2,3,4], target=10:', threeSumTarget([1, 2, 3, 4], 10));
// []
```

### **Approach 4: Three Sum Closest**
```javascript
/**
 * Find triplet with sum closest to target
 * Time: O(n²), Space: O(1)
 */
function threeSumClosest(nums, target) {
  nums.sort((a, b) => a - b);
  let closestSum = nums[0] + nums[1] + nums[2];
  
  for (let i = 0; i < nums.length - 2; i++) {
    let left = i + 1;
    let right = nums.length - 1;
    
    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];
      
      // Update closest sum if current is closer
      if (Math.abs(target - sum) < Math.abs(target - closestSum)) {
        closestSum = sum;
      }
      
      if (sum === target) {
        return sum; // Exact match
      } else if (sum < target) {
        left++;
      } else {
        right--;
      }
    }
  }
  
  return closestSum;
}

// Test
console.log('\n=== Three Sum Closest ===');
console.log('[-1,2,1,-4], target=1:', threeSumClosest([-1, 2, 1, -4], 1)); // 2
console.log('[0,0,0], target=1:', threeSumClosest([0, 0, 0], 1)); // 0
console.log('[1,1,1,0], target=-100:', threeSumClosest([1, 1, 1, 0], -100)); // 2
```

### **Approach 5: Three Sum Smaller**
```javascript
/**
 * Count triplets with sum less than target
 * Time: O(n²), Space: O(1)
 */
function threeSumSmaller(nums, target) {
  nums.sort((a, b) => a - b);
  let count = 0;
  
  for (let i = 0; i < nums.length - 2; i++) {
    let left = i + 1;
    let right = nums.length - 1;
    
    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];
      
      if (sum < target) {
        // All elements between left and right work with current left
        count += right - left;
        left++;
      } else {
        right--;
      }
    }
  }
  
  return count;
}

// Test
console.log('\n=== Three Sum Smaller ===');
console.log('[-2,0,1,3], target=2:', threeSumSmaller([-2, 0, 1, 3], 2)); // 2
console.log('[0,0,0], target=0:', threeSumSmaller([0, 0, 0], 0)); // 0
console.log('[1,1,1,1], target=5:', threeSumSmaller([1, 1, 1, 1], 5)); // 4
```

### **Bonus: Three Sum with Multiplicity**
```javascript
/**
 * Count triplets (i,j,k) where i < j < k and nums[i]+nums[j]+nums[k] = target
 * Allows using same value multiple times if at different indices
 * Time: O(n²), Space: O(n)
 */
function threeSumMultiplicity(nums, target) {
  const MOD = 1e9 + 7;
  const count = new Map();
  let result = 0;
  
  // Count frequency of each number
  for (let num of nums) {
    count.set(num, (count.get(num) || 0) + 1);
  }
  
  const uniqueNums = [...count.keys()].sort((a, b) => a - b);
  
  for (let i = 0; i < uniqueNums.length; i++) {
    for (let j = i; j < uniqueNums.length; j++) {
      const k = target - uniqueNums[i] - uniqueNums[j];
      
      if (!count.has(k)) continue;
      
      if (i === j && j === k) {
        // All three same: C(n,3) = n*(n-1)*(n-2)/6
        const n = count.get(uniqueNums[i]);
        result += n * (n - 1) * (n - 2) / 6;
      } else if (i === j && j < k && uniqueNums[k] > uniqueNums[j]) {
        // Two same: C(n,2) * m = n*(n-1)/2 * m
        const n = count.get(uniqueNums[i]);
        const m = count.get(k);
        result += n * (n - 1) / 2 * m;
      } else if (j < k && uniqueNums[k] > uniqueNums[j]) {
        // All different: n * m * p
        result += count.get(uniqueNums[i]) * count.get(uniqueNums[j]) * count.get(k);
      }
    }
  }
  
  return result % MOD;
}

// Test
console.log('\n=== Three Sum Multiplicity ===');
console.log('[1,1,2,2,3,3,4,4,5,5], target=8:', threeSumMultiplicity([1,1,2,2,3,3,4,4,5,5], 8));
console.log('[1,1,2,2,2,2], target=5:', threeSumMultiplicity([1,1,2,2,2,2], 5));
```

### **Bonus: All Unique Triplets (No Duplicates)**
```javascript
/**
 * Find all unique triplets without duplicates
 * Uses Set to ensure uniqueness
 */
function threeSumUnique(nums) {
  const result = new Set();
  const n = nums.length;
  
  for (let i = 0; i < n - 2; i++) {
    const seen = new Set();
    
    for (let j = i + 1; j < n; j++) {
      const complement = -(nums[i] + nums[j]);
      
      if (seen.has(complement)) {
        // Create sorted triplet to avoid duplicates
        const triplet = [nums[i], nums[j], complement].sort((a, b) => a - b);
        result.add(triplet.join(','));
      }
      
      seen.add(nums[j]);
    }
  }
  
  return Array.from(result).map(s => s.split(',').map(Number));
}

// Test
console.log('\n=== Unique Triplets ===');
console.log('[-1,0,1,2,-1,-4]:', threeSumUnique([-1, 0, 1, 2, -1, -4]));
// [[-1, -1, 2], [-1, 0, 1]]
```

### **Visualization**
```javascript
/**
 * Visualize the three sum process
 */
function threeSumVisualize(nums) {
  console.log('\n=== Three Sum Visualization ===');
  console.log('Input:', nums);
  
  const result = [];
  nums.sort((a, b) => a - b);
  console.log('Sorted:', nums);
  
  for (let i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) {
      console.log(`Skipping duplicate i=${i}: ${nums[i]}`);
      continue;
    }
    
    console.log(`\nFixed nums[${i}] = ${nums[i]}, target = ${-nums[i]}`);
    
    let left = i + 1;
    let right = nums.length - 1;
    
    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];
      console.log(`  Checking: ${nums[i]} + ${nums[left]} + ${nums[right]} = ${sum}`);
      
      if (sum === 0) {
        console.log(`    ✓ Found triplet!`);
        result.push([nums[i], nums[left], nums[right]]);
        
        while (left < right && nums[left] === nums[left + 1]) {
          console.log(`    Skipping duplicate left: ${nums[left]}`);
          left++;
        }
        while (left < right && nums[right] === nums[right - 1]) {
          console.log(`    Skipping duplicate right: ${nums[right]}`);
          right--;
        }
        
        left++;
        right--;
      } else if (sum < 0) {
        console.log(`    Sum too small, move left pointer`);
        left++;
      } else {
        console.log(`    Sum too large, move right pointer`);
        right--;
      }
    }
  }
  
  console.log('\nResult:', result);
  return result;
}

// Demo
threeSumVisualize([-1, 0, 1, 2, -1, -4]);
```

### **Edge Cases**
```javascript
console.log('\n=== Edge Cases ===');

// Empty array
console.log('Empty:', threeSum([])); // []

// Less than 3 elements
console.log('Two elements:', threeSum([1, 2])); // []

// All zeros
console.log('All zeros:', threeSum([0, 0, 0, 0])); // [[0, 0, 0]]

// All positive (no solution)
console.log('All positive:', threeSum([1, 2, 3, 4])); // []

// All negative (no solution)
console.log('All negative:', threeSum([-1, -2, -3, -4])); // []

// Exactly 3 elements
console.log('Exactly 3:', threeSum([1, -1, 0])); // [[1, -1, 0]]

// Multiple solutions
console.log('Multiple:', threeSum([-4, -2, -1, 0, 1, 2, 3, 4]));
// [[-4, 0, 4], [-4, 1, 3], [-2, -1, 3], [-2, 0, 2], [-1, 0, 1]]

// Large numbers
console.log('Large:', threeSum([1000000, -1000000, 0])); // [[-1000000, 0, 1000000]]
```

### **Performance Comparison**
```javascript
/**
 * Compare different approaches
 */
function benchmarkThreeSum(size) {
  const nums = Array.from({ length: size }, () => Math.floor(Math.random() * size) - size / 2);
  
  console.log(`\nArray size: ${size.toLocaleString()}`);
  
  // Two Pointers
  let start = performance.now();
  threeSum([...nums]);
  let time = performance.now() - start;
  console.log(`Two Pointers: ${time.toFixed(4)}ms`);
  
  // Hash Set
  start = performance.now();
  threeSumHashSet([...nums]);
  time = performance.now() - start;
  console.log(`Hash Set: ${time.toFixed(4)}ms`);
}

console.log('\n=== Performance Comparison ===');
benchmarkThreeSum(100);
benchmarkThreeSum(500);
benchmarkThreeSum(1000);
```

**Interview Tips:**
- Extension of Two Sum to three numbers
- Sort + two pointers is optimal: O(n²) time, O(1) space
- Key: fix one element, use two sum on remaining
- Critical: handle duplicates by skipping same consecutive values
- Sort array first: enables two pointers and duplicate handling
- For i, j, k: skip if nums[i] === nums[i-1], nums[j] === nums[j+1], nums[k] === nums[k-1]
- Why sort? Allows two pointers and easy duplicate detection
- O(n²) is best possible for this problem
- Variations: target sum, closest sum, smaller than target, count triplets
- Can't improve time complexity beyond O(n²)
- Common mistake: forgetting to skip duplicates
- Edge cases: empty, < 3 elements, all zeros, no solution, all same
- Follow-ups: 4Sum O(n³), kSum, three sum closest, three sum smaller
- Clarify: unique triplets? indices or values? sorted input?
- Optimization: break early if nums[i] > 0 and sorted (all positive)
- Applications: subset sum, combination problems, finding patterns

</details>

73. Find the longest common prefix among an array of strings

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Horizontal Scanning**
```javascript
/**
 * Compare strings one by one, reducing prefix each time
 * Time Complexity: O(S) where S is sum of all characters
 * Space Complexity: O(1)
 */
function longestCommonPrefix(strs) {
  if (strs.length === 0) return '';
  
  let prefix = strs[0];
  
  for (let i = 1; i < strs.length; i++) {
    // Reduce prefix until it matches start of current string
    while (strs[i].indexOf(prefix) !== 0) {
      prefix = prefix.substring(0, prefix.length - 1);
      if (prefix === '') return '';
    }
  }
  
  return prefix;
}

// Test
console.log('=== Horizontal Scanning ===');
console.log('["flower","flow","flight"]:', longestCommonPrefix(['flower', 'flow', 'flight'])); // "fl"
console.log('["dog","racecar","car"]:', longestCommonPrefix(['dog', 'racecar', 'car'])); // ""
console.log('["interspecies","interstellar","interstate"]:', 
  longestCommonPrefix(['interspecies', 'interstellar', 'interstate'])); // "inters"
```

### **Approach 2: Vertical Scanning**
```javascript
/**
 * Compare characters at same position across all strings
 * Time: O(S), Space: O(1)
 */
function longestCommonPrefixVertical(strs) {
  if (strs.length === 0) return '';
  
  // Check each character position
  for (let i = 0; i < strs[0].length; i++) {
    const char = strs[0][i];
    
    // Check if this character matches in all strings
    for (let j = 1; j < strs.length; j++) {
      if (i >= strs[j].length || strs[j][i] !== char) {
        return strs[0].substring(0, i);
      }
    }
  }
  
  return strs[0];
}

// Test
console.log('\n=== Vertical Scanning ===');
console.log('["flower","flow","flight"]:', longestCommonPrefixVertical(['flower', 'flow', 'flight'])); // "fl"
console.log('["ab","a"]:', longestCommonPrefixVertical(['ab', 'a'])); // "a"
console.log('["hello","hello","hello"]:', longestCommonPrefixVertical(['hello', 'hello', 'hello'])); // "hello"
```

### **Approach 3: Divide and Conquer**
```javascript
/**
 * Divide array in half, find prefix recursively, then merge
 * Time: O(S), Space: O(m log n) for recursion
 */
function longestCommonPrefixDivide(strs) {
  if (strs.length === 0) return '';
  return divideConquer(strs, 0, strs.length - 1);
}

function divideConquer(strs, left, right) {
  if (left === right) {
    return strs[left];
  }
  
  const mid = Math.floor((left + right) / 2);
  const lcpLeft = divideConquer(strs, left, mid);
  const lcpRight = divideConquer(strs, mid + 1, right);
  
  return commonPrefix(lcpLeft, lcpRight);
}

function commonPrefix(str1, str2) {
  const minLen = Math.min(str1.length, str2.length);
  
  for (let i = 0; i < minLen; i++) {
    if (str1[i] !== str2[i]) {
      return str1.substring(0, i);
    }
  }
  
  return str1.substring(0, minLen);
}

// Test
console.log('\n=== Divide and Conquer ===');
console.log('["flower","flow","flight"]:', longestCommonPrefixDivide(['flower', 'flow', 'flight'])); // "fl"
console.log('["a","a","b"]:', longestCommonPrefixDivide(['a', 'a', 'b'])); // ""
```

### **Approach 4: Binary Search**
```javascript
/**
 * Binary search on prefix length
 * Time: O(S log m) where m is length of shortest string
 * Space: O(1)
 */
function longestCommonPrefixBinary(strs) {
  if (strs.length === 0) return '';
  
  // Find minimum length
  let minLen = Math.min(...strs.map(s => s.length));
  
  let low = 0;
  let high = minLen;
  
  while (low <= high) {
    const mid = Math.floor((low + high) / 2);
    
    if (isCommonPrefix(strs, mid)) {
      low = mid + 1;
    } else {
      high = mid - 1;
    }
  }
  
  return strs[0].substring(0, high);
}

function isCommonPrefix(strs, len) {
  const prefix = strs[0].substring(0, len);
  
  for (let i = 1; i < strs.length; i++) {
    if (!strs[i].startsWith(prefix)) {
      return false;
    }
  }
  
  return true;
}

// Test
console.log('\n=== Binary Search ===');
console.log('["flower","flow","flight"]:', longestCommonPrefixBinary(['flower', 'flow', 'flight'])); // "fl"
console.log('["abc","abcd","abcde"]:', longestCommonPrefixBinary(['abc', 'abcd', 'abcde'])); // "abc"
```

### **Approach 5: Trie (Efficient for Multiple Queries)**
```javascript
/**
 * Use Trie data structure
 * Useful when need to find LCP multiple times
 * Time: O(S) to build + O(m) per query
 * Space: O(S)
 */
class TrieNode {
  constructor() {
    this.children = new Map();
    this.isEnd = false;
  }
}

class Trie {
  constructor() {
    this.root = new TrieNode();
  }
  
  insert(word) {
    let node = this.root;
    
    for (let char of word) {
      if (!node.children.has(char)) {
        node.children.set(char, new TrieNode());
      }
      node = node.children.get(char);
    }
    
    node.isEnd = true;
  }
  
  longestCommonPrefix() {
    let prefix = '';
    let node = this.root;
    
    while (node.children.size === 1 && !node.isEnd) {
      const char = [...node.children.keys()][0];
      prefix += char;
      node = node.children.get(char);
    }
    
    return prefix;
  }
}

function longestCommonPrefixTrie(strs) {
  if (strs.length === 0) return '';
  if (strs.length === 1) return strs[0];
  
  const trie = new Trie();
  
  // Insert all strings
  for (let str of strs) {
    if (str === '') return ''; // Empty string means no common prefix
    trie.insert(str);
  }
  
  return trie.longestCommonPrefix();
}

// Test
console.log('\n=== Trie Approach ===');
console.log('["flower","flow","flight"]:', longestCommonPrefixTrie(['flower', 'flow', 'flight'])); // "fl"
console.log('["prefix","pretest","prepare"]:', longestCommonPrefixTrie(['prefix', 'pretest', 'prepare'])); // "pre"
```

### **Bonus: Longest Common Prefix of Two Strings**
```javascript
/**
 * Find LCP of just two strings (helper function)
 */
function lcpTwo(str1, str2) {
  const minLen = Math.min(str1.length, str2.length);
  let i = 0;
  
  while (i < minLen && str1[i] === str2[i]) {
    i++;
  }
  
  return str1.substring(0, i);
}

// Test
console.log('\n=== LCP of Two Strings ===');
console.log('flower, flow:', lcpTwo('flower', 'flow')); // "flow"
console.log('abc, xyz:', lcpTwo('abc', 'xyz')); // ""
console.log('hello, hello:', lcpTwo('hello', 'hello')); // "hello"
```

### **Bonus: Longest Common Suffix**
```javascript
/**
 * Find longest common suffix instead of prefix
 */
function longestCommonSuffix(strs) {
  if (strs.length === 0) return '';
  
  let suffix = strs[0];
  
  for (let i = 1; i < strs.length; i++) {
    while (!strs[i].endsWith(suffix)) {
      suffix = suffix.substring(1);
      if (suffix === '') return '';
    }
  }
  
  return suffix;
}

// Test
console.log('\n=== Longest Common Suffix ===');
console.log('["testing","walking","running"]:', longestCommonSuffix(['testing', 'walking', 'running'])); // "ing"
console.log('["abc","bbc","cbc"]:', longestCommonSuffix(['abc', 'bbc', 'cbc'])); // "bc"
```

### **Bonus: All Common Prefixes**
```javascript
/**
 * Find all common prefixes at each position
 */
function allCommonPrefixes(strs) {
  if (strs.length === 0) return [];
  
  const prefixes = [];
  
  for (let i = 0; i < strs[0].length; i++) {
    const char = strs[0][i];
    let isCommon = true;
    
    for (let j = 1; j < strs.length; j++) {
      if (i >= strs[j].length || strs[j][i] !== char) {
        isCommon = false;
        break;
      }
    }
    
    if (isCommon) {
      prefixes.push(strs[0].substring(0, i + 1));
    } else {
      break;
    }
  }
  
  return prefixes;
}

// Test
console.log('\n=== All Common Prefixes ===');
console.log('["flower","flow","flight"]:', allCommonPrefixes(['flower', 'flow', 'flight'])); // ["f", "fl"]
console.log('["abc","abcd","ab"]:', allCommonPrefixes(['abc', 'abcd', 'ab'])); // ["a", "ab"]
```

### **Edge Cases**
```javascript
console.log('\n=== Edge Cases ===');

// Empty array
console.log('Empty array:', longestCommonPrefix([])); // ""

// Single string
console.log('Single string:', longestCommonPrefix(['hello'])); // "hello"

// Empty string in array
console.log('Contains empty:', longestCommonPrefix(['hello', '', 'world'])); // ""

// No common prefix
console.log('No common:', longestCommonPrefix(['abc', 'def', 'ghi'])); // ""

// All same strings
console.log('All same:', longestCommonPrefix(['test', 'test', 'test'])); // "test"

// One string is prefix of another
console.log('One is prefix:', longestCommonPrefix(['flow', 'flower', 'flowing'])); // "flow"

// Very short strings
console.log('Single chars:', longestCommonPrefix(['a', 'a', 'b'])); // ""

// Case sensitive
console.log('Case sensitive:', longestCommonPrefix(['Hello', 'hello'])); // ""
```

### **Performance Comparison**
```javascript
/**
 * Compare different approaches
 */
function benchmarkLCP(size, stringLength) {
  const strs = Array.from({ length: size }, () => {
    const prefix = 'common';
    const suffix = Array.from({ length: stringLength - prefix.length }, 
      () => String.fromCharCode(97 + Math.floor(Math.random() * 26))).join('');
    return prefix + suffix;
  });
  
  console.log(`\nArray size: ${size}, String length: ${stringLength}`);
  
  // Horizontal
  let start = performance.now();
  longestCommonPrefix(strs);
  let time = performance.now() - start;
  console.log(`Horizontal: ${time.toFixed(4)}ms`);
  
  // Vertical
  start = performance.now();
  longestCommonPrefixVertical(strs);
  time = performance.now() - start;
  console.log(`Vertical: ${time.toFixed(4)}ms`);
  
  // Binary Search
  start = performance.now();
  longestCommonPrefixBinary(strs);
  time = performance.now() - start;
  console.log(`Binary Search: ${time.toFixed(4)}ms`);
  
  // Trie
  start = performance.now();
  longestCommonPrefixTrie(strs);
  time = performance.now() - start;
  console.log(`Trie: ${time.toFixed(4)}ms`);
}

console.log('\n=== Performance Comparison ===');
benchmarkLCP(100, 50);
benchmarkLCP(1000, 100);
```

**Interview Tips:**
- Multiple valid approaches: horizontal, vertical, divide & conquer, binary search, trie
- Horizontal scanning simplest: compare strings one by one
- Vertical scanning more efficient: stops at first mismatch
- All approaches O(S) where S = sum of all characters
- Key insight: LCP can't be longer than shortest string
- Edge case: empty array, single string, empty string in array
- Optimization: find shortest string first, limit search to its length
- Trie useful when finding LCP multiple times (preprocessing)
- Binary search on prefix length: O(S log m) where m = min length
- Related problems: longest common suffix, substring problems
- Clarify: case sensitive? special characters? Unicode?
- Common mistakes: not handling empty strings, not checking bounds
- Can optimize by finding shortest string first
- Applications: autocomplete, finding common path, DNA sequence analysis
- Follow-ups: longest common substring, edit distance, pattern matching

</details>

74. Implement a function to validate a binary search tree

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Recursive with Range (Optimal)**
```javascript
/**
 * Validate BST using recursive range checking
 * Time Complexity: O(n) - visit each node once
 * Space Complexity: O(h) - recursion stack, h = height
 * 
 * BST property: left.val < node.val < right.val
 * AND all left subtree < node.val < all right subtree
 */
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

function isValidBST(root) {
  return validate(root, -Infinity, Infinity);
}

function validate(node, min, max) {
  // Empty tree is valid BST
  if (node === null) return true;
  
  // Current node must be within range
  if (node.val <= min || node.val >= max) {
    return false;
  }
  
  // Recursively validate left and right subtrees
  // Left subtree: all values must be < node.val
  // Right subtree: all values must be > node.val
  return validate(node.left, min, node.val) && 
         validate(node.right, node.val, max);
}

// Test
console.log('=== Recursive with Range (Optimal) ===');

// Valid BST:     2
//               / \
//              1   3
const tree1 = new TreeNode(2, new TreeNode(1), new TreeNode(3));
console.log('Valid BST [2,1,3]:', isValidBST(tree1)); // true

// Invalid BST:   5
//               / \
//              1   4
//                 / \
//                3   6
const tree2 = new TreeNode(5, 
  new TreeNode(1),
  new TreeNode(4, new TreeNode(3), new TreeNode(6))
);
console.log('Invalid BST [5,1,4,null,null,3,6]:', isValidBST(tree2)); // false

// Valid BST:     5
//               / \
//              3   7
//             /   / \
//            2   6   8
const tree3 = new TreeNode(5,
  new TreeNode(3, new TreeNode(2)),
  new TreeNode(7, new TreeNode(6), new TreeNode(8))
);
console.log('Valid BST [5,3,7,2,null,6,8]:', isValidBST(tree3)); // true
```

### **Approach 2: Inorder Traversal (Property-Based)**
```javascript
/**
 * Use inorder traversal property
 * Inorder of BST is sorted in ascending order
 * Time: O(n), Space: O(n)
 */
function isValidBSTInorder(root) {
  const values = [];
  
  function inorder(node) {
    if (node === null) return;
    
    inorder(node.left);
    values.push(node.val);
    inorder(node.right);
  }
  
  inorder(root);
  
  // Check if array is strictly increasing
  for (let i = 1; i < values.length; i++) {
    if (values[i] <= values[i - 1]) {
      return false;
    }
  }
  
  return true;
}

// Test
console.log('\n=== Inorder Traversal ===');
const tree4 = new TreeNode(2, new TreeNode(1), new TreeNode(3));
console.log('Valid BST [2,1,3]:', isValidBSTInorder(tree4)); // true

const tree5 = new TreeNode(5,
  new TreeNode(1),
  new TreeNode(4, new TreeNode(3), new TreeNode(6))
);
console.log('Invalid BST [5,1,4,3,6]:', isValidBSTInorder(tree5)); // false
```

### **Approach 3: Inorder with Early Termination**
```javascript
/**
 * Inorder traversal with early exit
 * Don't need to store all values
 * Time: O(n), Space: O(h)
 */
function isValidBSTInorderOptimized(root) {
  let prev = -Infinity;
  
  function inorder(node) {
    if (node === null) return true;
    
    // Check left subtree
    if (!inorder(node.left)) return false;
    
    // Check current node
    if (node.val <= prev) return false;
    prev = node.val;
    
    // Check right subtree
    return inorder(node.right);
  }
  
  return inorder(root);
}

// Test
console.log('\n=== Inorder with Early Termination ===');
const tree6 = new TreeNode(2, new TreeNode(2));
console.log('Duplicate values [2,2]:', isValidBSTInorderOptimized(tree6)); // false

const tree7 = new TreeNode(10,
  new TreeNode(5, new TreeNode(3), new TreeNode(7)),
  new TreeNode(15, new TreeNode(12), new TreeNode(20))
);
console.log('Valid BST [10,5,15,3,7,12,20]:', isValidBSTInorderOptimized(tree7)); // true
```

### **Approach 4: Iterative with Stack**
```javascript
/**
 * Iterative approach using explicit stack
 * Avoids recursion overhead
 * Time: O(n), Space: O(h)
 */
function isValidBSTIterative(root) {
  const stack = [];
  let current = root;
  let prev = -Infinity;
  
  while (current !== null || stack.length > 0) {
    // Go to leftmost node
    while (current !== null) {
      stack.push(current);
      current = current.left;
    }
    
    // Process current node
    current = stack.pop();
    
    // Check if values are in ascending order
    if (current.val <= prev) {
      return false;
    }
    
    prev = current.val;
    current = current.right;
  }
  
  return true;
}

// Test
console.log('\n=== Iterative with Stack ===');
const tree8 = new TreeNode(1, null, new TreeNode(1));
console.log('Invalid [1,null,1]:', isValidBSTIterative(tree8)); // false

const tree9 = new TreeNode(3,
  new TreeNode(1, null, new TreeNode(2)),
  new TreeNode(5, new TreeNode(4), new TreeNode(6))
);
console.log('Valid BST [3,1,5,null,2,4,6]:', isValidBSTIterative(tree9)); // true
```

### **Approach 5: BFS with Range Validation**
```javascript
/**
 * Level-order traversal with min/max ranges
 * Time: O(n), Space: O(n)
 */
function isValidBSTBFS(root) {
  if (root === null) return true;
  
  const queue = [[root, -Infinity, Infinity]];
  
  while (queue.length > 0) {
    const [node, min, max] = queue.shift();
    
    // Check current node
    if (node.val <= min || node.val >= max) {
      return false;
    }
    
    // Add children with updated ranges
    if (node.left !== null) {
      queue.push([node.left, min, node.val]);
    }
    
    if (node.right !== null) {
      queue.push([node.right, node.val, max]);
    }
  }
  
  return true;
}

// Test
console.log('\n=== BFS with Range Validation ===');
const tree10 = new TreeNode(8,
  new TreeNode(4, new TreeNode(2), new TreeNode(6)),
  new TreeNode(12, new TreeNode(10), new TreeNode(14))
);
console.log('Valid BST [8,4,12,2,6,10,14]:', isValidBSTBFS(tree10)); // true
```

### **Bonus: Validate BST with Duplicates Allowed**
```javascript
/**
 * Some BST definitions allow duplicates
 * Either left.val <= node.val < right.val
 * Or left.val < node.val <= right.val
 */
function isValidBSTWithDuplicates(root, allowLeft = true) {
  return validateWithDuplicates(root, -Infinity, Infinity, allowLeft);
}

function validateWithDuplicates(node, min, max, allowLeft) {
  if (node === null) return true;
  
  if (allowLeft) {
    // Allow duplicates on left: left.val <= node.val < right.val
    if (node.val < min || node.val >= max) {
      return false;
    }
  } else {
    // Allow duplicates on right: left.val < node.val <= right.val
    if (node.val <= min || node.val > max) {
      return false;
    }
  }
  
  return validateWithDuplicates(node.left, min, node.val, allowLeft) &&
         validateWithDuplicates(node.right, node.val, max, allowLeft);
}

// Test
console.log('\n=== BST with Duplicates ===');
const tree11 = new TreeNode(2, new TreeNode(2), new TreeNode(3));
console.log('With duplicates on left [2,2,3]:', isValidBSTWithDuplicates(tree11, true)); // true
console.log('With duplicates on right [2,2,3]:', isValidBSTWithDuplicates(tree11, false)); // false
```

### **Bonus: Count Valid BST Nodes**
```javascript
/**
 * Count how many nodes satisfy BST property
 */
function countValidBSTNodes(root) {
  let count = 0;
  
  function validate(node, min, max) {
    if (node === null) return;
    
    if (node.val > min && node.val < max) {
      count++;
    }
    
    validate(node.left, min, node.val);
    validate(node.right, node.val, max);
  }
  
  validate(root, -Infinity, Infinity);
  return count;
}

// Test
console.log('\n=== Count Valid BST Nodes ===');
const tree12 = new TreeNode(5,
  new TreeNode(1),
  new TreeNode(4, new TreeNode(3), new TreeNode(6))
);
console.log('Valid nodes count:', countValidBSTNodes(tree12)); // 4 (5,1,4,6 valid, 3 invalid)
```

### **Bonus: Find Invalid Node**
```javascript
/**
 * Find the first node that violates BST property
 */
function findInvalidNode(root) {
  let result = null;
  
  function validate(node, min, max) {
    if (node === null || result !== null) return true;
    
    if (node.val <= min || node.val >= max) {
      result = node;
      return false;
    }
    
    return validate(node.left, min, node.val) &&
           validate(node.right, node.val, max);
  }
  
  validate(root, -Infinity, Infinity);
  return result;
}

// Test
console.log('\n=== Find Invalid Node ===');
const tree13 = new TreeNode(5,
  new TreeNode(1),
  new TreeNode(4, new TreeNode(3), new TreeNode(6))
);
const invalid = findInvalidNode(tree13);
console.log('Invalid node value:', invalid ? invalid.val : null); // 4
```

### **Visualization and Testing**
```javascript
/**
 * Visualize BST validation process
 */
function isValidBSTVisualize(root) {
  console.log('\n=== BST Validation Visualization ===');
  
  function validate(node, min, max, depth = 0) {
    const indent = '  '.repeat(depth);
    
    if (node === null) {
      console.log(`${indent}null - Valid (base case)`);
      return true;
    }
    
    console.log(`${indent}Node ${node.val}, range: (${min}, ${max})`);
    
    if (node.val <= min || node.val >= max) {
      console.log(`${indent}✗ Invalid! ${node.val} not in range (${min}, ${max})`);
      return false;
    }
    
    console.log(`${indent}✓ ${node.val} is in range`);
    
    console.log(`${indent}Checking left subtree (must be < ${node.val}):`);
    const leftValid = validate(node.left, min, node.val, depth + 1);
    
    console.log(`${indent}Checking right subtree (must be > ${node.val}):`);
    const rightValid = validate(node.right, node.val, max, depth + 1);
    
    return leftValid && rightValid;
  }
  
  return validate(root, -Infinity, Infinity);
}

// Demo
const demoTree = new TreeNode(5,
  new TreeNode(3, new TreeNode(2), new TreeNode(4)),
  new TreeNode(7, new TreeNode(6), new TreeNode(8))
);
console.log('Result:', isValidBSTVisualize(demoTree));
```

### **Edge Cases**
```javascript
console.log('\n=== Edge Cases ===');

// Empty tree
console.log('Empty tree:', isValidBST(null)); // true

// Single node
console.log('Single node:', isValidBST(new TreeNode(1))); // true

// Two nodes - left child
console.log('Root with left child:', isValidBST(new TreeNode(2, new TreeNode(1)))); // true

// Two nodes - right child
console.log('Root with right child:', isValidBST(new TreeNode(1, null, new TreeNode(2)))); // true

// Invalid - left child greater
console.log('Left > root:', isValidBST(new TreeNode(1, new TreeNode(2)))); // false

// Invalid - right child smaller
console.log('Right < root:', isValidBST(new TreeNode(2, null, new TreeNode(1)))); // false

// All same values
console.log('All same:', isValidBST(new TreeNode(1, new TreeNode(1), new TreeNode(1)))); // false

// Negative numbers
const negTree = new TreeNode(0, new TreeNode(-10), new TreeNode(10));
console.log('With negatives:', isValidBST(negTree)); // true

// Large tree
const largeTree = new TreeNode(50,
  new TreeNode(25, new TreeNode(12), new TreeNode(37)),
  new TreeNode(75, new TreeNode(62), new TreeNode(87))
);
console.log('Large valid BST:', isValidBST(largeTree)); // true

// Subtle invalid case: left child's right > root
const subtleInvalid = new TreeNode(5,
  new TreeNode(4, null, new TreeNode(6)),
  new TreeNode(7)
);
console.log('Subtle invalid:', isValidBST(subtleInvalid)); // false (6 > 5)
```

### **Performance Testing**
```javascript
/**
 * Build perfect BST for testing
 */
function buildPerfectBST(values) {
  if (values.length === 0) return null;
  
  const mid = Math.floor(values.length / 2);
  const node = new TreeNode(values[mid]);
  
  node.left = buildPerfectBST(values.slice(0, mid));
  node.right = buildPerfectBST(values.slice(mid + 1));
  
  return node;
}

function benchmarkBSTValidation(size) {
  const values = Array.from({ length: size }, (_, i) => i);
  const tree = buildPerfectBST(values);
  
  console.log(`\nTree size: ${size} nodes`);
  
  // Recursive with range
  let start = performance.now();
  isValidBST(tree);
  let time = performance.now() - start;
  console.log(`Recursive range: ${time.toFixed(4)}ms`);
  
  // Inorder
  start = performance.now();
  isValidBSTInorder(tree);
  time = performance.now() - start;
  console.log(`Inorder: ${time.toFixed(4)}ms`);
  
  // Iterative
  start = performance.now();
  isValidBSTIterative(tree);
  time = performance.now() - start;
  console.log(`Iterative: ${time.toFixed(4)}ms`);
}

console.log('\n=== Performance Comparison ===');
benchmarkBSTValidation(1000);
benchmarkBSTValidation(10000);
```

**Interview Tips:**
- BST property: left subtree < node < right subtree (for ALL nodes, not just immediate children)
- Common mistake: only checking immediate children (fails for subtree violations)
- Recursive with range is most elegant: O(n) time, O(h) space
- Key insight: pass valid range (min, max) down recursively
- Inorder traversal should be strictly increasing
- Can't have duplicates in standard BST definition (clarify in interview!)
- Handle edge cases: null tree (valid), single node (valid)
- Subtle case: left subtree's right child > root (still invalid)
- Integer overflow: use -Infinity/Infinity instead of MIN_INT/MAX_INT
- Space complexity: O(h) for recursion, O(n) worst case (skewed tree)
- Iterative avoids recursion overhead
- Related problems: BST construction, BST operations, convert to sorted array
- Clarify: duplicates allowed? null tree valid? definition of BST
- Applications: validate data structures, database indexes, search trees
- Follow-ups: recover BST, find kth smallest in BST, serialize/deserialize

</details>

75. Find the lowest common ancestor in a binary tree

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Recursive Path Search**
```javascript
/**
 * Find LCA by searching for both nodes
 * Time Complexity: O(n)
 * Space Complexity: O(h) for recursion
 * 
 * LCA is the lowest node that has both p and q in its subtrees
 */
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

function lowestCommonAncestor(root, p, q) {
  // Base case: reached null or found one of the nodes
  if (root === null || root === p || root === q) {
    return root;
  }
  
  // Search in left and right subtrees
  const left = lowestCommonAncestor(root.left, p, q);
  const right = lowestCommonAncestor(root.right, p, q);
  
  // If both left and right found nodes, current node is LCA
  if (left !== null && right !== null) {
    return root;
  }
  
  // Return whichever side found a node
  return left !== null ? left : right;
}

// Test
console.log('=== Recursive Path Search ===');

//        3
//       / \
//      5   1
//     / \ / \
//    6  2 0  8
//      / \
//     7   4
const node7 = new TreeNode(7);
const node4 = new TreeNode(4);
const node2 = new TreeNode(2, node7, node4);
const node6 = new TreeNode(6);
const node5 = new TreeNode(5, node6, node2);
const node0 = new TreeNode(0);
const node8 = new TreeNode(8);
const node1 = new TreeNode(1, node0, node8);
const root = new TreeNode(3, node5, node1);

console.log('LCA(5, 1):', lowestCommonAncestor(root, node5, node1).val); // 3
console.log('LCA(5, 4):', lowestCommonAncestor(root, node5, node4).val); // 5
console.log('LCA(6, 2):', lowestCommonAncestor(root, node6, node2).val); // 5
console.log('LCA(7, 4):', lowestCommonAncestor(root, node7, node4).val); // 2
```

### **Approach 2: Store Parent Pointers**
```javascript
/**
 * Store parent pointers and trace path
 * Time: O(n), Space: O(n)
 */
function lowestCommonAncestorWithParent(root, p, q) {
  // Build parent map
  const parent = new Map();
  parent.set(root, null);
  
  const queue = [root];
  
  while (queue.length > 0) {
    const node = queue.shift();
    
    if (node.left) {
      parent.set(node.left, node);
      queue.push(node.left);
    }
    
    if (node.right) {
      parent.set(node.right, node);
      queue.push(node.right);
    }
  }
  
  // Get ancestors of p
  const ancestors = new Set();
  let current = p;
  
  while (current !== null) {
    ancestors.add(current);
    current = parent.get(current);
  }
  
  // Trace q's path until we find common ancestor
  current = q;
  while (current !== null) {
    if (ancestors.has(current)) {
      return current;
    }
    current = parent.get(current);
  }
  
  return null;
}

// Test
console.log('\n=== With Parent Pointers ===');
console.log('LCA(5, 1):', lowestCommonAncestorWithParent(root, node5, node1).val); // 3
console.log('LCA(5, 4):', lowestCommonAncestorWithParent(root, node5, node4).val); // 5
```

### **Approach 3: Store Paths**
```javascript
/**
 * Find paths from root to both nodes, then find divergence point
 * Time: O(n), Space: O(n)
 */
function lowestCommonAncestorPaths(root, p, q) {
  const pathP = [];
  const pathQ = [];
  
  findPath(root, p, pathP);
  findPath(root, q, pathQ);
  
  let lca = null;
  let i = 0;
  
  while (i < pathP.length && i < pathQ.length && pathP[i] === pathQ[i]) {
    lca = pathP[i];
    i++;
  }
  
  return lca;
}

function findPath(root, target, path) {
  if (root === null) return false;
  
  path.push(root);
  
  if (root === target) return true;
  
  if (findPath(root.left, target, path) || findPath(root.right, target, path)) {
    return true;
  }
  
  path.pop();
  return false;
}

// Test
console.log('\n=== Store Paths ===');
console.log('LCA(5, 1):', lowestCommonAncestorPaths(root, node5, node1).val); // 3
console.log('LCA(6, 2):', lowestCommonAncestorPaths(root, node6, node2).val); // 5
```

### **Approach 4: LCA in BST (Optimized for BST)**
```javascript
/**
 * If tree is BST, can optimize using values
 * Time: O(h), Space: O(1) iterative or O(h) recursive
 */
function lowestCommonAncestorBST(root, p, q) {
  const small = Math.min(p.val, q.val);
  const large = Math.max(p.val, q.val);
  
  let current = root;
  
  while (current !== null) {
    if (current.val > large) {
      // Both in left subtree
      current = current.left;
    } else if (current.val < small) {
      // Both in right subtree
      current = current.right;
    } else {
      // Split point found
      return current;
    }
  }
  
  return null;
}

// Test
console.log('\n=== LCA in BST ===');

//      6
//     / \
//    2   8
//   / \ / \
//  0  4 7  9
//    / \
//   3   5
const bst0 = new TreeNode(0);
const bst3 = new TreeNode(3);
const bst5 = new TreeNode(5);
const bst4 = new TreeNode(4, bst3, bst5);
const bst2 = new TreeNode(2, bst0, bst4);
const bst7 = new TreeNode(7);
const bst9 = new TreeNode(9);
const bst8 = new TreeNode(8, bst7, bst9);
const bstRoot = new TreeNode(6, bst2, bst8);

console.log('LCA(2, 8) in BST:', lowestCommonAncestorBST(bstRoot, bst2, bst8).val); // 6
console.log('LCA(2, 4) in BST:', lowestCommonAncestorBST(bstRoot, bst2, bst4).val); // 2
console.log('LCA(3, 5) in BST:', lowestCommonAncestorBST(bstRoot, bst3, bst5).val); // 4
```

### **Approach 5: Iterative with Stack**
```javascript
/**
 * Iterative approach avoiding recursion
 * Time: O(n), Space: O(n)
 */
function lowestCommonAncestorIterative(root, p, q) {
  const parent = new Map();
  const stack = [root];
  parent.set(root, null);
  
  // Build parent map until both p and q are found
  while (!parent.has(p) || !parent.has(q)) {
    const node = stack.pop();
    
    if (node.left) {
      parent.set(node.left, node);
      stack.push(node.left);
    }
    
    if (node.right) {
      parent.set(node.right, node);
      stack.push(node.right);
    }
  }
  
  // Collect all ancestors of p
  const ancestors = new Set();
  while (p !== null) {
    ancestors.add(p);
    p = parent.get(p);
  }
  
  // First ancestor of q that's also ancestor of p is LCA
  while (!ancestors.has(q)) {
    q = parent.get(q);
  }
  
  return q;
}

// Test
console.log('\n=== Iterative Approach ===');
console.log('LCA(5, 1):', lowestCommonAncestorIterative(root, node5, node1).val); // 3
console.log('LCA(7, 4):', lowestCommonAncestorIterative(root, node7, node4).val); // 2
```

### **Bonus: Find Distance Between Two Nodes**
```javascript
/**
 * Find distance between two nodes using LCA
 * Distance = dist(root, p) + dist(root, q) - 2 * dist(root, lca)
 */
function findDistance(root, p, q) {
  const lca = lowestCommonAncestor(root, p, q);
  
  const distP = findLevel(lca, p, 0);
  const distQ = findLevel(lca, q, 0);
  
  return distP + distQ;
}

function findLevel(root, target, level) {
  if (root === null) return -1;
  if (root === target) return level;
  
  const left = findLevel(root.left, target, level + 1);
  if (left !== -1) return left;
  
  return findLevel(root.right, target, level + 1);
}

// Test
console.log('\n=== Distance Between Nodes ===');
console.log('Distance(5, 1):', findDistance(root, node5, node1)); // 2
console.log('Distance(7, 4):', findDistance(root, node7, node4)); // 2
console.log('Distance(6, 8):', findDistance(root, node6, node8)); // 4
```

### **Bonus: All Ancestors of a Node**
```javascript
/**
 * Find all ancestors of a given node
 */
function findAllAncestors(root, target) {
  const ancestors = [];
  
  function findAncestorsHelper(node) {
    if (node === null) return false;
    
    if (node === target) return true;
    
    if (findAncestorsHelper(node.left) || findAncestorsHelper(node.right)) {
      ancestors.push(node.val);
      return true;
    }
    
    return false;
  }
  
  findAncestorsHelper(root);
  return ancestors;
}

// Test
console.log('\n=== All Ancestors ===');
console.log('Ancestors of 7:', findAllAncestors(root, node7)); // [2, 5, 3]
console.log('Ancestors of 4:', findAllAncestors(root, node4)); // [2, 5, 3]
console.log('Ancestors of 0:', findAllAncestors(root, node0)); // [1, 3]
```

### **Bonus: LCA of Multiple Nodes**
```javascript
/**
 * Find LCA of more than 2 nodes
 */
function lowestCommonAncestorMultiple(root, nodes) {
  const nodeSet = new Set(nodes);
  
  function helper(node) {
    if (node === null) return null;
    
    if (nodeSet.has(node)) return node;
    
    const left = helper(node.left);
    const right = helper(node.right);
    
    if (left !== null && right !== null) return node;
    
    return left !== null ? left : right;
  }
  
  return helper(root);
}

// Test
console.log('\n=== LCA of Multiple Nodes ===');
console.log('LCA([6, 7, 4]):', lowestCommonAncestorMultiple(root, [node6, node7, node4]).val); // 5
console.log('LCA([0, 8, 4]):', lowestCommonAncestorMultiple(root, [node0, node8, node4]).val); // 3
```

### **Visualization**
```javascript
/**
 * Visualize LCA finding process
 */
function lowestCommonAncestorVisualize(root, p, q) {
  console.log('\n=== LCA Visualization ===');
  console.log(`Finding LCA of ${p.val} and ${q.val}`);
  
  function helper(node, depth = 0) {
    const indent = '  '.repeat(depth);
    
    if (node === null) {
      console.log(`${indent}null - return null`);
      return null;
    }
    
    console.log(`${indent}At node ${node.val}`);
    
    if (node === p || node === q) {
      console.log(`${indent}Found target node ${node.val}!`);
      return node;
    }
    
    console.log(`${indent}Searching left subtree:`);
    const left = helper(node.left, depth + 1);
    
    console.log(`${indent}Searching right subtree:`);
    const right = helper(node.right, depth + 1);
    
    if (left !== null && right !== null) {
      console.log(`${indent}Both subtrees found nodes - ${node.val} is LCA!`);
      return node;
    }
    
    const result = left !== null ? left : right;
    console.log(`${indent}Returning ${result ? result.val : 'null'} from node ${node.val}`);
    return result;
  }
  
  return helper(root);
}

// Demo
const result = lowestCommonAncestorVisualize(root, node7, node4);
console.log(`\nFinal result: ${result.val}`);
```

### **Edge Cases**
```javascript
console.log('\n=== Edge Cases ===');

// One node is ancestor of another
console.log('One is ancestor (5, 6):', lowestCommonAncestor(root, node5, node6).val); // 5

// Nodes are same
const single = new TreeNode(1);
console.log('Same node:', lowestCommonAncestor(single, single, single).val); // 1

// Root is LCA
console.log('Root is LCA:', lowestCommonAncestor(root, node5, node1).val); // 3

// Two leaf nodes
console.log('Two leaves (6, 0):', lowestCommonAncestor(root, node6, node0).val); // 3

// Siblings
console.log('Siblings (5, 1):', lowestCommonAncestor(root, node5, node1).val); // 3

// Small tree
const small = new TreeNode(1, new TreeNode(2));
const leaf = small.left;
console.log('Small tree:', lowestCommonAncestor(small, small, leaf).val); // 1
```

### **Performance Testing**
```javascript
/**
 * Build balanced tree for testing
 */
function buildBalancedTree(depth) {
  if (depth === 0) return null;
  
  const node = new TreeNode(depth);
  node.left = buildBalancedTree(depth - 1);
  node.right = buildBalancedTree(depth - 1);
  
  return node;
}

function benchmarkLCA(depth) {
  const tree = buildBalancedTree(depth);
  const nodes = [];
  
  function collectNodes(node) {
    if (node === null) return;
    nodes.push(node);
    collectNodes(node.left);
    collectNodes(node.right);
  }
  
  collectNodes(tree);
  
  const p = nodes[Math.floor(nodes.length / 3)];
  const q = nodes[Math.floor(nodes.length * 2 / 3)];
  
  console.log(`\nTree depth: ${depth}, nodes: ${nodes.length}`);
  
  // Recursive
  let start = performance.now();
  lowestCommonAncestor(tree, p, q);
  let time = performance.now() - start;
  console.log(`Recursive: ${time.toFixed(4)}ms`);
  
  // Parent pointers
  start = performance.now();
  lowestCommonAncestorWithParent(tree, p, q);
  time = performance.now() - start;
  console.log(`Parent pointers: ${time.toFixed(4)}ms`);
  
  // Paths
  start = performance.now();
  lowestCommonAncestorPaths(tree, p, q);
  time = performance.now() - start;
  console.log(`Paths: ${time.toFixed(4)}ms`);
}

console.log('\n=== Performance Comparison ===');
benchmarkLCA(10);
benchmarkLCA(15);
```

**Interview Tips:**
- LCA = lowest (deepest) node that has both p and q as descendants
- Key insight: if left and right subtrees both contain target nodes, current node is LCA
- Recursive approach most elegant: O(n) time, O(h) space
- Node can be descendant of itself (if p is ancestor of q, then p is LCA)
- Base case: null or found one of target nodes
- For BST: can optimize to O(h) by using BST property
- Parent pointers approach: trace paths up and find first common ancestor
- Path storage: find paths to both nodes, then find last common node
- Handle edge cases: one is ancestor of other, nodes are same, root is LCA
- Clarify: guaranteed both nodes exist? BST or general tree? parent pointers available?
- Related problems: distance between nodes, all ancestors, LCA of multiple nodes
- Applications: file system paths, organizational hierarchy, version control
- Follow-ups: LCA in BST, LCA of multiple nodes, find distance, serialize tree
- Common mistake: assuming LCA can't be one of the input nodes
- Optimization for BST: O(h) time using value comparison

</details>

### **Asynchronous Programming**

76. Implement Promise.all() from scratch

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Promise.all Implementation**
```javascript
/**
 * Promise.all - waits for all promises to resolve or first rejection
 * Time Complexity: O(n) where n is number of promises
 * Space Complexity: O(n) for results array
 * 
 * Returns: Promise that resolves with array of results
 * Rejects: If any promise rejects
 */
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    // Handle empty array
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let completedCount = 0;
    
    promises.forEach((promise, index) => {
      // Wrap non-promise values with Promise.resolve
      Promise.resolve(promise)
        .then(result => {
          results[index] = result;
          completedCount++;
          
          // All promises resolved
          if (completedCount === promises.length) {
            resolve(results);
          }
        })
        .catch(error => {
          // First rejection rejects entire Promise.all
          reject(error);
        });
    });
  });
}

// Test
console.log('=== Basic Promise.all ===');

const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
const p3 = Promise.resolve(3);

promiseAll([p1, p2, p3])
  .then(results => console.log('All resolved:', results)) // [1, 2, 3]
  .catch(error => console.log('Error:', error));

// Test with delay
const delay = (ms, value) => new Promise(resolve => setTimeout(() => resolve(value), ms));

promiseAll([delay(100, 'a'), delay(50, 'b'), delay(150, 'c')])
  .then(results => console.log('With delays:', results)) // ['a', 'b', 'c']
  .catch(error => console.log('Error:', error));

// Test with rejection
const p4 = Promise.resolve(1);
const p5 = Promise.reject('Error!');
const p6 = Promise.resolve(3);

promiseAll([p4, p5, p6])
  .then(results => console.log('Results:', results))
  .catch(error => console.log('Caught rejection:', error)); // 'Error!'
```

### **Approach 2: Handle Non-Promise Values**
```javascript
/**
 * More robust version handling non-promise values
 */
function promiseAllRobust(iterable) {
  return new Promise((resolve, reject) => {
    // Convert iterable to array
    const promises = Array.from(iterable);
    
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = new Array(promises.length);
    let completedCount = 0;
    let rejected = false;
    
    promises.forEach((item, index) => {
      // Handle both promises and non-promise values
      Promise.resolve(item)
        .then(result => {
          if (rejected) return; // Don't process if already rejected
          
          results[index] = result;
          completedCount++;
          
          if (completedCount === promises.length) {
            resolve(results);
          }
        })
        .catch(error => {
          if (!rejected) {
            rejected = true;
            reject(error);
          }
        });
    });
  });
}

// Test
console.log('\n=== Robust Promise.all ===');

// Mix of promises and values
promiseAllRobust([1, Promise.resolve(2), 3, delay(50, 4)])
  .then(results => console.log('Mixed values:', results)) // [1, 2, 3, 4]
  .catch(error => console.log('Error:', error));

// Test with Set
const promiseSet = new Set([
  Promise.resolve('a'),
  Promise.resolve('b'),
  Promise.resolve('c')
]);

promiseAllRobust(promiseSet)
  .then(results => console.log('From Set:', results)) // ['a', 'b', 'c']
  .catch(error => console.log('Error:', error));
```

### **Approach 3: With Index Preservation**
```javascript
/**
 * Ensure results maintain order regardless of completion order
 */
function promiseAllOrdered(promises) {
  return new Promise((resolve, reject) => {
    if (!Array.isArray(promises) || promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let settled = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          // Store at correct index
          results[index] = value;
          settled++;
          
          if (settled === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}

// Test - results should be in order regardless of completion time
console.log('\n=== Order Preservation ===');

promiseAllOrdered([
  delay(200, 'slow'),
  delay(50, 'fast'),
  delay(100, 'medium')
])
  .then(results => console.log('Ordered:', results)) // ['slow', 'fast', 'medium']
  .catch(error => console.log('Error:', error));
```

### **Approach 4: With Detailed Error Information**
```javascript
/**
 * Enhanced version that provides error context
 */
function promiseAllWithContext(promises) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let completedCount = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(result => {
          results[index] = result;
          completedCount++;
          
          if (completedCount === promises.length) {
            resolve(results);
          }
        })
        .catch(error => {
          // Provide context about which promise failed
          reject({
            error,
            index,
            message: `Promise at index ${index} failed`
          });
        });
    });
  });
}

// Test
console.log('\n=== With Error Context ===');

promiseAllWithContext([
  Promise.resolve(1),
  Promise.reject(new Error('Failed!')),
  Promise.resolve(3)
])
  .then(results => console.log('Results:', results))
  .catch(error => console.log('Error with context:', error));
```

### **Approach 5: Async/Await Implementation**
```javascript
/**
 * Modern implementation using async/await
 */
async function promiseAllAsync(promises) {
  if (promises.length === 0) {
    return [];
  }
  
  const results = [];
  
  // Create array of promise wrappers that capture index
  const wrappedPromises = promises.map((promise, index) => 
    Promise.resolve(promise).then(result => {
      results[index] = result;
      return result;
    })
  );
  
  // Wait for all to complete
  await Promise.all(wrappedPromises);
  
  return results;
}

// Test
console.log('\n=== Async/Await Version ===');

(async () => {
  try {
    const results = await promiseAllAsync([
      delay(100, 'x'),
      delay(50, 'y'),
      delay(75, 'z')
    ]);
    console.log('Async results:', results); // ['x', 'y', 'z']
  } catch (error) {
    console.log('Error:', error);
  }
})();
```

### **Bonus: Promise.all with Progress Tracking**
```javascript
/**
 * Promise.all with progress callbacks
 */
function promiseAllWithProgress(promises, onProgress) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let completedCount = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(result => {
          results[index] = result;
          completedCount++;
          
          // Report progress
          if (onProgress) {
            onProgress({
              completed: completedCount,
              total: promises.length,
              percentage: (completedCount / promises.length) * 100,
              latest: result
            });
          }
          
          if (completedCount === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}

// Test
console.log('\n=== With Progress Tracking ===');

promiseAllWithProgress(
  [delay(100, 'A'), delay(200, 'B'), delay(150, 'C')],
  progress => console.log(`Progress: ${progress.percentage.toFixed(0)}% (${progress.completed}/${progress.total})`)
)
  .then(results => console.log('Final:', results))
  .catch(error => console.log('Error:', error));
```

### **Bonus: Promise.all with Timeout**
```javascript
/**
 * Promise.all with overall timeout
 */
function promiseAllWithTimeout(promises, timeoutMs) {
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error(`Timeout after ${timeoutMs}ms`)), timeoutMs);
  });
  
  return Promise.race([
    promiseAll(promises),
    timeoutPromise
  ]);
}

// Test
console.log('\n=== With Timeout ===');

promiseAllWithTimeout([delay(100, 1), delay(200, 2), delay(300, 3)], 250)
  .then(results => console.log('Completed:', results))
  .catch(error => console.log('Timeout error:', error.message));

promiseAllWithTimeout([delay(50, 1), delay(100, 2)], 500)
  .then(results => console.log('Completed in time:', results))
  .catch(error => console.log('Error:', error.message));
```

### **Comparison with Native Promise.all**
```javascript
/**
 * Compare behavior with native implementation
 */
console.log('\n=== Comparison with Native ===');

const testPromises = [
  Promise.resolve(1),
  Promise.resolve(2),
  Promise.resolve(3)
];

// Custom implementation
promiseAll(testPromises)
  .then(results => console.log('Custom:', results));

// Native implementation
Promise.all(testPromises)
  .then(results => console.log('Native:', results));

// Both reject on first error
const mixedPromises = [
  Promise.resolve(1),
  Promise.reject('Error!'),
  delay(1000, 3) // This won't complete
];

promiseAll(mixedPromises)
  .then(results => console.log('Custom results:', results))
  .catch(error => console.log('Custom error:', error));

Promise.all(mixedPromises)
  .then(results => console.log('Native results:', results))
  .catch(error => console.log('Native error:', error));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty array
promiseAll([])
  .then(results => console.log('Empty array:', results)) // []
  .catch(error => console.log('Error:', error));

// Single promise
promiseAll([Promise.resolve('single')])
  .then(results => console.log('Single promise:', results)) // ['single']
  .catch(error => console.log('Error:', error));

// All rejected
promiseAll([Promise.reject('a'), Promise.reject('b')])
  .then(results => console.log('Results:', results))
  .catch(error => console.log('First rejection:', error)); // 'a'

// Mixed resolved/rejected - order matters
promiseAll([
  delay(100, 'slow resolve'),
  Promise.reject('fast reject')
])
  .then(results => console.log('Results:', results))
  .catch(error => console.log('Rejected:', error)); // 'fast reject'

// Non-promise values
promiseAll([1, 2, 3])
  .then(results => console.log('Non-promises:', results)) // [1, 2, 3]
  .catch(error => console.log('Error:', error));

// Nested promises
promiseAll([Promise.resolve(Promise.resolve(1))])
  .then(results => console.log('Nested:', results)) // [1]
  .catch(error => console.log('Error:', error));
```

### **Performance Testing**
```javascript
/**
 * Test performance with many promises
 */
async function benchmarkPromiseAll(count) {
  const promises = Array.from({ length: count }, (_, i) => 
    delay(Math.random() * 10, i)
  );
  
  console.log(`\nBenchmarking ${count} promises:`);
  
  // Custom implementation
  let start = performance.now();
  await promiseAll(promises);
  let time = performance.now() - start;
  console.log(`Custom: ${time.toFixed(2)}ms`);
  
  // Native implementation
  const promises2 = Array.from({ length: count }, (_, i) => 
    delay(Math.random() * 10, i)
  );
  
  start = performance.now();
  await Promise.all(promises2);
  time = performance.now() - start;
  console.log(`Native: ${time.toFixed(2)}ms`);
}

console.log('\n=== Performance Testing ===');
setTimeout(() => {
  benchmarkPromiseAll(100);
  setTimeout(() => benchmarkPromiseAll(1000), 200);
}, 500);
```

**Interview Tips:**
- Promise.all waits for ALL promises to resolve, or first rejection
- Returns promise that resolves with array of results in same order
- Fails fast: rejects immediately on first rejection
- Results preserve input order, not completion order
- Must handle non-promise values (wrap with Promise.resolve)
- Empty array resolves immediately with []
- Key insight: track completion count, resolve when count === length
- Common mistake: not preserving result order
- Use forEach with index to maintain order
- Difference from Promise.race: waits for all, not first
- Related: Promise.allSettled (waits for all, doesn't reject)
- Related: Promise.race (resolves/rejects with first settled)
- Related: Promise.any (resolves with first fulfilled)
- Applications: parallel API calls, batch operations, concurrent tasks
- Edge cases: empty array, single promise, all rejected, mixed values
- Clarify: should handle non-promise values? timeout? progress tracking?
- Follow-ups: implement Promise.race, Promise.allSettled, add concurrency limit

</details>

77. Implement Promise.race() from scratch

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Promise.race Implementation**
```javascript
/**
 * Promise.race - returns first settled promise (resolved or rejected)
 * Time Complexity: O(n) to set up handlers
 * Space Complexity: O(1)
 * 
 * Returns: Promise that settles with first settled promise's result
 */
function promiseRace(promises) {
  return new Promise((resolve, reject) => {
    // Empty array never settles (matches native behavior)
    if (promises.length === 0) {
      return; // Promise stays pending forever
    }
    
    promises.forEach(promise => {
      Promise.resolve(promise)
        .then(resolve)  // First to resolve wins
        .catch(reject); // First to reject wins
    });
  });
}

// Test
console.log('=== Basic Promise.race ===');

const delay = (ms, value) => new Promise(resolve => 
  setTimeout(() => resolve(value), ms)
);

promiseRace([
  delay(100, 'slow'),
  delay(50, 'fast'),
  delay(200, 'slowest')
])
  .then(result => console.log('Winner:', result)) // 'fast'
  .catch(error => console.log('Error:', error));

// Test with rejection
promiseRace([
  delay(100, 'slow resolve'),
  Promise.reject('fast reject')
])
  .then(result => console.log('Result:', result))
  .catch(error => console.log('Rejected:', error)); // 'fast reject'
```

### **Approach 2: Handle Non-Promise Values**
```javascript
/**
 * More robust version handling non-promise values
 */
function promiseRaceRobust(iterable) {
  return new Promise((resolve, reject) => {
    const promises = Array.from(iterable);
    
    if (promises.length === 0) {
      return; // Stays pending
    }
    
    let settled = false;
    
    promises.forEach(item => {
      Promise.resolve(item)
        .then(value => {
          if (!settled) {
            settled = true;
            resolve(value);
          }
        })
        .catch(error => {
          if (!settled) {
            settled = true;
            reject(error);
          }
        });
    });
  });
}

// Test
console.log('\n=== Robust Promise.race ===');

// Immediate value wins
promiseRaceRobust([
  'immediate',
  delay(100, 'delayed')
])
  .then(result => console.log('Winner:', result)) // 'immediate'
  .catch(error => console.log('Error:', error));

// Mix of promises and values
promiseRaceRobust([
  delay(50, 'a'),
  Promise.resolve('b'),
  delay(25, 'c')
])
  .then(result => console.log('First:', result)) // 'b' (resolves immediately)
  .catch(error => console.log('Error:', error));
```

### **Approach 3: With Winner Information**
```javascript
/**
 * Enhanced version that provides context about winner
 */
function promiseRaceWithInfo(promises) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      return;
    }
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          resolve({
            value,
            index,
            status: 'fulfilled'
          });
        })
        .catch(error => {
          reject({
            error,
            index,
            status: 'rejected'
          });
        });
    });
  });
}

// Test
console.log('\n=== With Winner Info ===');

promiseRaceWithInfo([
  delay(100, 'slow'),
  delay(30, 'fast'),
  delay(200, 'slowest')
])
  .then(result => console.log('Winner:', result))
  .catch(error => console.log('Error:', error));
```

### **Approach 4: Async/Await Implementation**
```javascript
/**
 * Using async/await syntax (wrapper around native)
 */
async function promiseRaceAsync(promises) {
  if (promises.length === 0) {
    return new Promise(() => {}); // Never resolves
  }
  
  return Promise.race(promises.map(p => Promise.resolve(p)));
}

// Test
console.log('\n=== Async/Await Version ===');

(async () => {
  try {
    const result = await promiseRaceAsync([
      delay(100, 'a'),
      delay(50, 'b'),
      delay(75, 'c')
    ]);
    console.log('Async winner:', result); // 'b'
  } catch (error) {
    console.log('Error:', error);
  }
})();
```

### **Approach 5: With Cancellation**
```javascript
/**
 * Race with ability to cancel remaining promises
 */
function promiseRaceWithCancel(promises) {
  const controllers = promises.map(() => ({ cancelled: false }));
  
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      return;
    }
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          // Mark others as cancelled
          controllers.forEach((ctrl, i) => {
            if (i !== index) ctrl.cancelled = true;
          });
          resolve({ value, index, controllers });
        })
        .catch(error => {
          controllers.forEach((ctrl, i) => {
            if (i !== index) ctrl.cancelled = true;
          });
          reject({ error, index, controllers });
        });
    });
  });
}

// Test
console.log('\n=== With Cancellation Support ===');

promiseRaceWithCancel([
  delay(100, 'slow'),
  delay(30, 'fast')
])
  .then(result => {
    console.log('Winner:', result.value);
    console.log('Controllers:', result.controllers.map(c => c.cancelled));
  })
  .catch(error => console.log('Error:', error));
```

### **Bonus: Promise.race with Timeout**
```javascript
/**
 * Race with automatic timeout
 */
function promiseRaceWithTimeout(promises, timeoutMs) {
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error(`Timeout after ${timeoutMs}ms`)), timeoutMs);
  });
  
  return promiseRace([...promises, timeoutPromise]);
}

// Test
console.log('\n=== Race with Timeout ===');

promiseRaceWithTimeout([delay(200, 'slow')], 100)
  .then(result => console.log('Result:', result))
  .catch(error => console.log('Timed out:', error.message));

promiseRaceWithTimeout([delay(50, 'fast')], 100)
  .then(result => console.log('Completed:', result))
  .catch(error => console.log('Error:', error.message));
```

### **Bonus: First Successful (Promise.any polyfill)**
```javascript
/**
 * Similar to race but only resolves on success
 * Rejects only if all promises reject
 */
function promiseAny(promises) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      reject(new Error('All promises were rejected'));
      return;
    }
    
    const errors = [];
    let rejectedCount = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(resolve) // First success wins
        .catch(error => {
          errors[index] = error;
          rejectedCount++;
          
          if (rejectedCount === promises.length) {
            reject(new Error('All promises rejected'));
          }
        });
    });
  });
}

// Test
console.log('\n=== Promise.any (First Success) ===');

promiseAny([
  Promise.reject('error 1'),
  delay(50, 'success'),
  Promise.reject('error 2')
])
  .then(result => console.log('First success:', result)) // 'success'
  .catch(error => console.log('All failed:', error));

promiseAny([
  Promise.reject('error 1'),
  Promise.reject('error 2')
])
  .then(result => console.log('Result:', result))
  .catch(error => console.log('All rejected:', error.message));
```

### **Bonus: Race with All Results**
```javascript
/**
 * Race but track all completions
 */
function promiseRaceWithAll(promises) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      resolve({ winner: null, all: [] });
      return;
    }
    
    let winner = null;
    const results = [];
    let completedCount = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          if (winner === null) {
            winner = { value, index, status: 'fulfilled' };
            resolve(winner);
          }
          
          results[index] = { value, status: 'fulfilled' };
          completedCount++;
        })
        .catch(error => {
          if (winner === null) {
            winner = { error, index, status: 'rejected' };
            reject(winner);
          }
          
          results[index] = { error, status: 'rejected' };
          completedCount++;
        });
    });
  });
}

// Test
console.log('\n=== Race with All Results ===');

promiseRaceWithAll([
  delay(100, 'a'),
  delay(50, 'b'),
  delay(75, 'c')
])
  .then(result => console.log('Winner:', result))
  .catch(error => console.log('Error:', error));
```

### **Comparison with Native Promise.race**
```javascript
/**
 * Compare behavior with native implementation
 */
console.log('\n=== Comparison with Native ===');

const testPromises = [
  delay(100, 'slow'),
  delay(50, 'fast'),
  delay(150, 'slowest')
];

// Custom implementation
promiseRace(testPromises)
  .then(result => console.log('Custom winner:', result));

// Native implementation
Promise.race(testPromises)
  .then(result => console.log('Native winner:', result));

// Both handle rejection
const mixedPromises = [
  delay(100, 'slow'),
  Promise.reject('fast error')
];

promiseRace(mixedPromises)
  .then(result => console.log('Custom result:', result))
  .catch(error => console.log('Custom error:', error));

Promise.race(mixedPromises)
  .then(result => console.log('Native result:', result))
  .catch(error => console.log('Native error:', error));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty array - stays pending
const emptyRace = promiseRace([]);
setTimeout(() => console.log('Empty race still pending'), 100);

// Single promise
promiseRace([Promise.resolve('only')])
  .then(result => console.log('Single:', result)) // 'only'
  .catch(error => console.log('Error:', error));

// Immediate vs delayed
promiseRace([
  Promise.resolve('immediate'),
  delay(1000, 'delayed')
])
  .then(result => console.log('Immediate wins:', result)) // 'immediate'
  .catch(error => console.log('Error:', error));

// All rejected - first rejection wins
promiseRace([
  delay(50, Promise.reject('slow error')),
  Promise.reject('fast error')
])
  .then(result => console.log('Result:', result))
  .catch(error => console.log('First error:', error)); // 'fast error'

// Non-promise values
promiseRace([1, 2, 3])
  .then(result => console.log('Non-promise:', result)) // 1
  .catch(error => console.log('Error:', error));

// Mixed types
promiseRace([
  delay(100, 'promise'),
  'immediate value',
  delay(50, 'another promise')
])
  .then(result => console.log('Mixed:', result)) // 'immediate value'
  .catch(error => console.log('Error:', error));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications of Promise.race
 */

// 1. Request with timeout
function fetchWithTimeout(url, timeout) {
  const fetchPromise = fetch(url);
  const timeoutPromise = new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Request timeout')), timeout)
  );
  
  return promiseRace([fetchPromise, timeoutPromise]);
}

// 2. First available resource
function getFirstAvailableServer(servers) {
  const requests = servers.map(server => 
    fetch(`${server}/health`).then(() => server)
  );
  
  return promiseRace(requests);
}

// 3. Race between cache and network
function getCachedOrFresh(key, fetchFn) {
  const cachePromise = caches.match(key);
  const networkPromise = fetchFn().then(data => {
    // Update cache in background
    return data;
  });
  
  return promiseRace([cachePromise, networkPromise]);
}

console.log('\n=== Use Cases ===');
console.log('fetchWithTimeout - race between fetch and timeout');
console.log('getFirstAvailableServer - first healthy server wins');
console.log('getCachedOrFresh - cache vs network, fastest wins');
```

### **Performance Testing**
```javascript
/**
 * Test performance with many promises
 */
async function benchmarkPromiseRace(count) {
  const promises = Array.from({ length: count }, (_, i) => 
    delay(Math.random() * 100, i)
  );
  
  console.log(`\nBenchmarking race with ${count} promises:`);
  
  // Custom implementation
  let start = performance.now();
  await promiseRace(promises);
  let time = performance.now() - start;
  console.log(`Custom: ${time.toFixed(2)}ms`);
  
  // Native implementation
  const promises2 = Array.from({ length: count }, (_, i) => 
    delay(Math.random() * 100, i)
  );
  
  start = performance.now();
  await Promise.race(promises2);
  time = performance.now() - start;
  console.log(`Native: ${time.toFixed(2)}ms`);
}

console.log('\n=== Performance Testing ===');
setTimeout(() => {
  benchmarkPromiseRace(100);
  setTimeout(() => benchmarkPromiseRace(1000), 200);
}, 600);
```

**Interview Tips:**
- Promise.race settles with FIRST promise to settle (resolve or reject)
- Unlike Promise.all, doesn't wait for all promises
- Empty array stays pending forever (never resolves or rejects)
- First to finish wins, whether success or failure
- Other promises continue executing (can't be cancelled in JS)
- Must wrap non-promise values with Promise.resolve
- Key insight: attach handlers to all, first to call resolve/reject wins
- Promise.race vs Promise.any: race takes first settled, any takes first fulfilled
- Promise.race vs Promise.all: race takes first, all waits for all
- Common use cases: timeouts, first available resource, cache vs network
- Applications: request timeouts, fastest server, redundant requests
- Edge cases: empty array, single promise, immediate values, all rejected
- Can't cancel other promises after winner determined (limitation of JS)
- Useful for implementing timeouts by racing with delay promise
- Related: Promise.any (first success), Promise.allSettled (all settled)
- Clarify: what happens to other promises? need winner info? timeout?
- Follow-ups: implement Promise.any, add timeout, track all results

</details>

78. Implement Promise.allSettled() from scratch

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Promise.allSettled Implementation**
```javascript
/**
 * Promise.allSettled - waits for all promises to settle (resolve or reject)
 * Time Complexity: O(n) where n is number of promises
 * Space Complexity: O(n) for results array
 * 
 * Returns: Promise that resolves with array of result objects
 * Never rejects - always waits for all to settle
 */
function promiseAllSettled(promises) {
  return new Promise((resolve) => {
    // Handle empty array
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let settledCount = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = {
            status: 'fulfilled',
            value: value
          };
          settledCount++;
          
          if (settledCount === promises.length) {
            resolve(results);
          }
        })
        .catch(reason => {
          results[index] = {
            status: 'rejected',
            reason: reason
          };
          settledCount++;
          
          if (settledCount === promises.length) {
            resolve(results);
          }
        });
    });
  });
}

// Test
console.log('=== Basic Promise.allSettled ===');

const delay = (ms, value) => new Promise(resolve => 
  setTimeout(() => resolve(value), ms)
);

promiseAllSettled([
  Promise.resolve(1),
  Promise.reject('Error!'),
  delay(100, 3)
])
  .then(results => {
    console.log('All settled:', results);
    // [
    //   { status: 'fulfilled', value: 1 },
    //   { status: 'rejected', reason: 'Error!' },
    //   { status: 'fulfilled', value: 3 }
    // ]
  });

// Test with all rejected
promiseAllSettled([
  Promise.reject('error 1'),
  Promise.reject('error 2'),
  Promise.reject('error 3')
])
  .then(results => {
    console.log('\nAll rejected:', results);
    // All have status: 'rejected'
  });
```

### **Approach 2: Handle Non-Promise Values**
```javascript
/**
 * More robust version handling non-promise values
 */
function promiseAllSettledRobust(iterable) {
  return new Promise((resolve) => {
    const promises = Array.from(iterable);
    
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = new Array(promises.length);
    let settledCount = 0;
    
    promises.forEach((item, index) => {
      Promise.resolve(item)
        .then(value => {
          results[index] = { status: 'fulfilled', value };
        })
        .catch(reason => {
          results[index] = { status: 'rejected', reason };
        })
        .finally(() => {
          settledCount++;
          if (settledCount === promises.length) {
            resolve(results);
          }
        });
    });
  });
}

// Test
console.log('\n=== Robust Promise.allSettled ===');

// Mix of promises and values
promiseAllSettledRobust([
  1,
  Promise.resolve(2),
  Promise.reject('error'),
  delay(50, 4)
])
  .then(results => console.log('Mixed values:', results));
```

### **Approach 3: Without Promise.finally**
```javascript
/**
 * Implementation without using .finally() for older environments
 */
function promiseAllSettledCompat(promises) {
  return new Promise((resolve) => {
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let settledCount = 0;
    
    const checkComplete = () => {
      settledCount++;
      if (settledCount === promises.length) {
        resolve(results);
      }
    };
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(
          value => {
            results[index] = { status: 'fulfilled', value };
            checkComplete();
          },
          reason => {
            results[index] = { status: 'rejected', reason };
            checkComplete();
          }
        );
    });
  });
}

// Test
console.log('\n=== Compatible Version ===');

promiseAllSettledCompat([
  Promise.resolve('success'),
  Promise.reject('failure'),
  delay(75, 'delayed')
])
  .then(results => console.log('Compat results:', results));
```

### **Approach 4: Async/Await Implementation**
```javascript
/**
 * Modern implementation using async/await
 */
async function promiseAllSettledAsync(promises) {
  if (promises.length === 0) {
    return [];
  }
  
  const settledPromises = promises.map(async (promise, index) => {
    try {
      const value = await Promise.resolve(promise);
      return { status: 'fulfilled', value };
    } catch (reason) {
      return { status: 'rejected', reason };
    }
  });
  
  return Promise.all(settledPromises);
}

// Test
console.log('\n=== Async/Await Version ===');

(async () => {
  const results = await promiseAllSettledAsync([
    Promise.resolve('a'),
    Promise.reject('b'),
    delay(50, 'c')
  ]);
  console.log('Async results:', results);
})();
```

### **Approach 5: With Statistics**
```javascript
/**
 * Enhanced version that provides statistics
 */
function promiseAllSettledWithStats(promises) {
  return new Promise((resolve) => {
    if (promises.length === 0) {
      resolve({ results: [], stats: { fulfilled: 0, rejected: 0 } });
      return;
    }
    
    const results = [];
    let settledCount = 0;
    let fulfilledCount = 0;
    let rejectedCount = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = { status: 'fulfilled', value };
          fulfilledCount++;
          settledCount++;
          
          if (settledCount === promises.length) {
            resolve({
              results,
              stats: {
                fulfilled: fulfilledCount,
                rejected: rejectedCount,
                total: promises.length
              }
            });
          }
        })
        .catch(reason => {
          results[index] = { status: 'rejected', reason };
          rejectedCount++;
          settledCount++;
          
          if (settledCount === promises.length) {
            resolve({
              results,
              stats: {
                fulfilled: fulfilledCount,
                rejected: rejectedCount,
                total: promises.length
              }
            });
          }
        });
    });
  });
}

// Test
console.log('\n=== With Statistics ===');

promiseAllSettledWithStats([
  Promise.resolve(1),
  Promise.reject('error'),
  Promise.resolve(3),
  Promise.reject('another error')
])
  .then(result => {
    console.log('Results:', result.results);
    console.log('Stats:', result.stats); // { fulfilled: 2, rejected: 2, total: 4 }
  });
```

### **Bonus: With Progress Tracking**
```javascript
/**
 * Promise.allSettled with progress callbacks
 */
function promiseAllSettledWithProgress(promises, onProgress) {
  return new Promise((resolve) => {
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let settledCount = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = { status: 'fulfilled', value };
        })
        .catch(reason => {
          results[index] = { status: 'rejected', reason };
        })
        .finally(() => {
          settledCount++;
          
          if (onProgress) {
            onProgress({
              settled: settledCount,
              total: promises.length,
              percentage: (settledCount / promises.length) * 100,
              latest: results[index]
            });
          }
          
          if (settledCount === promises.length) {
            resolve(results);
          }
        });
    });
  });
}

// Test
console.log('\n=== With Progress ===');

promiseAllSettledWithProgress(
  [
    delay(100, 'A'),
    Promise.reject('Error B'),
    delay(200, 'C')
  ],
  progress => console.log(`Progress: ${progress.percentage.toFixed(0)}% (${progress.settled}/${progress.total})`)
)
  .then(results => console.log('Final:', results));
```

### **Bonus: Filter Results by Status**
```javascript
/**
 * Helper functions to filter settled results
 */
function getFulfilled(results) {
  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}

function getRejected(results) {
  return results
    .filter(r => r.status === 'rejected')
    .map(r => r.reason);
}

function getByStatus(results, status) {
  return results.filter(r => r.status === status);
}

// Test
console.log('\n=== Filter Results ===');

promiseAllSettled([
  Promise.resolve(1),
  Promise.reject('error 1'),
  Promise.resolve(3),
  Promise.reject('error 2'),
  Promise.resolve(5)
])
  .then(results => {
    console.log('All results:', results);
    console.log('Fulfilled values:', getFulfilled(results)); // [1, 3, 5]
    console.log('Rejected reasons:', getRejected(results)); // ['error 1', 'error 2']
    console.log('Success count:', getByStatus(results, 'fulfilled').length); // 3
  });
```

### **Bonus: With Timeout**
```javascript
/**
 * allSettled with per-promise timeout
 */
function promiseAllSettledWithTimeout(promises, timeoutMs) {
  const wrappedPromises = promises.map((promise, index) => {
    const timeoutPromise = new Promise((_, reject) => {
      setTimeout(() => reject(new Error(`Promise ${index} timeout`)), timeoutMs);
    });
    
    return Promise.race([Promise.resolve(promise), timeoutPromise]);
  });
  
  return promiseAllSettled(wrappedPromises);
}

// Test
console.log('\n=== With Timeout ===');

promiseAllSettledWithTimeout([
  delay(50, 'fast'),
  delay(200, 'slow'),
  delay(100, 'medium')
], 150)
  .then(results => {
    console.log('With timeout results:', results);
    // fast and medium succeed, slow times out
  });
```

### **Comparison with Promise.all**
```javascript
/**
 * Demonstrate difference between allSettled and all
 */
console.log('\n=== allSettled vs all ===');

const mixedPromises = [
  Promise.resolve(1),
  Promise.reject('Error!'),
  Promise.resolve(3)
];

// Promise.all - rejects on first error
Promise.all(mixedPromises)
  .then(results => console.log('Promise.all results:', results))
  .catch(error => console.log('Promise.all error:', error)); // 'Error!'

// Promise.allSettled - waits for all
promiseAllSettled(mixedPromises)
  .then(results => {
    console.log('Promise.allSettled results:');
    results.forEach((r, i) => console.log(`  [${i}]:`, r));
  });
```

### **Comparison with Native Implementation**
```javascript
/**
 * Compare with native Promise.allSettled
 */
console.log('\n=== Comparison with Native ===');

const testPromises = [
  Promise.resolve('success'),
  Promise.reject('failure'),
  delay(50, 'delayed')
];

// Custom implementation
promiseAllSettled(testPromises)
  .then(results => console.log('Custom:', results));

// Native implementation (if available)
if (Promise.allSettled) {
  Promise.allSettled(testPromises)
    .then(results => console.log('Native:', results));
}
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty array
promiseAllSettled([])
  .then(results => console.log('Empty array:', results)); // []

// Single promise - resolved
promiseAllSettled([Promise.resolve('single')])
  .then(results => console.log('Single resolved:', results));
// [{ status: 'fulfilled', value: 'single' }]

// Single promise - rejected
promiseAllSettled([Promise.reject('error')])
  .then(results => console.log('Single rejected:', results));
// [{ status: 'rejected', reason: 'error' }]

// All resolved
promiseAllSettled([
  Promise.resolve(1),
  Promise.resolve(2),
  Promise.resolve(3)
])
  .then(results => console.log('All resolved:', results));

// All rejected
promiseAllSettled([
  Promise.reject('a'),
  Promise.reject('b'),
  Promise.reject('c')
])
  .then(results => console.log('All rejected:', results));

// Non-promise values
promiseAllSettled([1, 2, 3])
  .then(results => console.log('Non-promises:', results));
// All have status: 'fulfilled'

// Mixed timing
promiseAllSettled([
  delay(100, 'slow'),
  Promise.resolve('fast'),
  delay(50, 'medium')
])
  .then(results => console.log('Mixed timing:', results));

// Nested promises
promiseAllSettled([
  Promise.resolve(Promise.resolve(1))
])
  .then(results => console.log('Nested:', results));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Batch API calls with partial failures
async function fetchMultipleEndpoints(urls) {
  const requests = urls.map(url => fetch(url).then(r => r.json()));
  const results = await promiseAllSettled(requests);
  
  const successful = getFulfilled(results);
  const failed = getRejected(results);
  
  return { successful, failed, results };
}

// 2. Process all regardless of failures
async function processAllItems(items, processFunc) {
  const promises = items.map(item => processFunc(item));
  const results = await promiseAllSettled(promises);
  
  return {
    processed: results.filter(r => r.status === 'fulfilled').length,
    failed: results.filter(r => r.status === 'rejected').length,
    details: results
  };
}

// 3. Cleanup operations that might fail
async function cleanupResources(resources) {
  const cleanupPromises = resources.map(r => r.cleanup());
  const results = await promiseAllSettled(cleanupPromises);
  
  const failures = results.filter(r => r.status === 'rejected');
  if (failures.length > 0) {
    console.warn(`${failures.length} cleanups failed`);
  }
}

console.log('\n=== Use Cases ===');
console.log('fetchMultipleEndpoints - continue despite some failures');
console.log('processAllItems - get success/failure counts');
console.log('cleanupResources - attempt all cleanups');
```

### **Performance Testing**
```javascript
/**
 * Test performance with many promises
 */
async function benchmarkAllSettled(count) {
  // Mix of resolved and rejected
  const promises = Array.from({ length: count }, (_, i) => 
    i % 3 === 0 
      ? Promise.reject(`error ${i}`)
      : delay(Math.random() * 10, i)
  );
  
  console.log(`\nBenchmarking ${count} promises (some rejected):`);
  
  // Custom implementation
  let start = performance.now();
  await promiseAllSettled(promises);
  let time = performance.now() - start;
  console.log(`Custom: ${time.toFixed(2)}ms`);
  
  // Native implementation
  if (Promise.allSettled) {
    const promises2 = Array.from({ length: count }, (_, i) => 
      i % 3 === 0 
        ? Promise.reject(`error ${i}`)
        : delay(Math.random() * 10, i)
    );
    
    start = performance.now();
    await Promise.allSettled(promises2);
    time = performance.now() - start;
    console.log(`Native: ${time.toFixed(2)}ms`);
  }
}

console.log('\n=== Performance Testing ===');
setTimeout(() => {
  benchmarkAllSettled(100);
  setTimeout(() => benchmarkAllSettled(500), 200);
}, 800);
```

**Interview Tips:**
- Promise.allSettled waits for ALL promises to settle (never rejects)
- Returns array of result objects: { status, value/reason }
- Status is either 'fulfilled' or 'rejected'
- Different from Promise.all: doesn't reject on first failure
- Use when you need results from all promises regardless of failures
- Results preserve input order, not completion order
- Empty array resolves immediately with []
- Always resolves (never rejects), even if all promises reject
- Each result has status field to identify fulfilled vs rejected
- Fulfilled: { status: 'fulfilled', value }
- Rejected: { status: 'rejected', reason }
- Added in ES2020, may need polyfill for older environments
- Key insight: track settled count, resolve when all settled
- Common use cases: batch operations, partial failures acceptable, cleanup
- Difference from Promise.all: all rejects on first error, allSettled waits
- Can filter results by status to separate successes from failures
- Applications: batch API calls, parallel processing, resource cleanup
- Edge cases: empty array, all resolved, all rejected, non-promises
- Follow-ups: implement Promise.any, Promise.race, add progress tracking
- Clarify: need statistics? progress updates? timeout per promise?

</details>

79. Create a function to retry failed promises n times

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Retry with Recursion**
```javascript
/**
 * Retry a promise-returning function n times
 * Time Complexity: O(n) attempts
 * Space Complexity: O(n) call stack
 */
async function retry(fn, maxRetries) {
  let lastError;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      const result = await fn();
      return result;
    } catch (error) {
      lastError = error;
      
      if (attempt < maxRetries) {
        console.log(`Attempt ${attempt + 1} failed, retrying...`);
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries + 1} attempts: ${lastError.message}`);
}

// Test
console.log('=== Basic Retry ===');

let callCount = 0;
const unreliableFunction = () => {
  callCount++;
  console.log(`Call ${callCount}`);
  
  if (callCount < 3) {
    return Promise.reject(new Error('Failed!'));
  }
  
  return Promise.resolve('Success!');
};

retry(unreliableFunction, 5)
  .then(result => console.log('Result:', result)) // 'Success!' after 3 attempts
  .catch(error => console.log('Error:', error.message));
```

### **Approach 2: Retry with Delay**
```javascript
/**
 * Retry with delay between attempts
 */
function retryWithDelay(fn, maxRetries, delay) {
  return new Promise(async (resolve, reject) => {
    let lastError;
    
    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        const result = await fn();
        resolve(result);
        return;
      } catch (error) {
        lastError = error;
        
        if (attempt < maxRetries) {
          console.log(`Attempt ${attempt + 1} failed, waiting ${delay}ms...`);
          await new Promise(r => setTimeout(r, delay));
        }
      }
    }
    
    reject(new Error(`Failed after ${maxRetries + 1} attempts: ${lastError.message}`));
  });
}

// Test
console.log('\n=== Retry with Delay ===');

let attempts = 0;
const delayedFunction = () => {
  attempts++;
  console.log(`Attempt ${attempts} at ${new Date().toLocaleTimeString()}`);
  
  if (attempts < 3) {
    return Promise.reject(new Error('Not yet'));
  }
  
  return Promise.resolve('Done!');
};

retryWithDelay(delayedFunction, 5, 500)
  .then(result => console.log('Result:', result))
  .catch(error => console.log('Error:', error.message));
```

### **Approach 3: Exponential Backoff**
```javascript
/**
 * Retry with exponential backoff
 * Delay increases exponentially: delay, delay*2, delay*4, etc.
 */
async function retryWithBackoff(fn, maxRetries, initialDelay = 1000) {
  let lastError;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      
      if (attempt < maxRetries) {
        const delay = initialDelay * Math.pow(2, attempt);
        console.log(`Attempt ${attempt + 1} failed, waiting ${delay}ms (exponential backoff)...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries + 1} attempts: ${lastError.message}`);
}

// Test
console.log('\n=== Exponential Backoff ===');

let backoffAttempts = 0;
const backoffFunction = () => {
  backoffAttempts++;
  console.log(`Backoff attempt ${backoffAttempts}`);
  
  if (backoffAttempts < 4) {
    return Promise.reject(new Error('Failed'));
  }
  
  return Promise.resolve('Finally succeeded!');
};

retryWithBackoff(backoffFunction, 5, 100)
  .then(result => console.log('Result:', result))
  .catch(error => console.log('Error:', error.message));
```

### **Approach 4: Retry with Custom Condition**
```javascript
/**
 * Retry only if condition is met (e.g., certain error types)
 */
async function retryIf(fn, maxRetries, shouldRetry) {
  let lastError;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      
      if (attempt < maxRetries && shouldRetry(error, attempt)) {
        console.log(`Attempt ${attempt + 1} failed with retryable error, retrying...`);
      } else if (attempt < maxRetries) {
        console.log(`Non-retryable error, aborting...`);
        throw error;
      }
    }
  }
  
  throw lastError;
}

// Test
console.log('\n=== Retry with Condition ===');

let conditionalAttempts = 0;
const conditionalFunction = () => {
  conditionalAttempts++;
  
  if (conditionalAttempts === 1) {
    return Promise.reject({ code: 'NETWORK_ERROR', retryable: true });
  } else if (conditionalAttempts === 2) {
    return Promise.reject({ code: 'AUTH_ERROR', retryable: false });
  }
  
  return Promise.resolve('Success');
};

retryIf(
  conditionalFunction,
  5,
  (error) => error.retryable === true
)
  .then(result => console.log('Result:', result))
  .catch(error => console.log('Caught non-retryable error:', error.code));
```

### **Approach 5: Retry with Callbacks**
```javascript
/**
 * Retry with callbacks for monitoring
 */
async function retryWithCallbacks(fn, maxRetries, options = {}) {
  const {
    delay = 0,
    onRetry = () => {},
    onError = () => {},
    onSuccess = () => {}
  } = options;
  
  let lastError;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      const result = await fn();
      onSuccess(result, attempt);
      return result;
    } catch (error) {
      lastError = error;
      onError(error, attempt);
      
      if (attempt < maxRetries) {
        onRetry(attempt + 1, maxRetries);
        
        if (delay > 0) {
          await new Promise(resolve => setTimeout(resolve, delay));
        }
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries + 1} attempts: ${lastError.message}`);
}

// Test
console.log('\n=== Retry with Callbacks ===');

let callbackAttempts = 0;
const callbackFunction = () => {
  callbackAttempts++;
  
  if (callbackAttempts < 3) {
    return Promise.reject(new Error('Temporary failure'));
  }
  
  return Promise.resolve('Success!');
};

retryWithCallbacks(callbackFunction, 5, {
  delay: 100,
  onRetry: (attempt, max) => console.log(`Retrying... (${attempt}/${max})`),
  onError: (error, attempt) => console.log(`Error on attempt ${attempt + 1}: ${error.message}`),
  onSuccess: (result, attempt) => console.log(`Succeeded on attempt ${attempt + 1}`)
})
  .then(result => console.log('Final result:', result))
  .catch(error => console.log('Final error:', error.message));
```

### **Bonus: Retry Multiple Functions**
```javascript
/**
 * Retry multiple promise functions with shared retry logic
 */
async function retryAll(functions, maxRetries) {
  const results = await Promise.all(
    functions.map(fn => retry(fn, maxRetries))
  );
  return results;
}

// Test
console.log('\n=== Retry Multiple Functions ===');

const func1 = () => Promise.resolve('Function 1 success');

let func2Attempts = 0;
const func2 = () => {
  func2Attempts++;
  return func2Attempts < 2 
    ? Promise.reject(new Error('Func 2 fail'))
    : Promise.resolve('Function 2 success');
};

retryAll([func1, func2], 3)
  .then(results => console.log('All results:', results))
  .catch(error => console.log('Error:', error));
```

### **Bonus: Retry with Timeout**
```javascript
/**
 * Retry with timeout for each attempt
 */
async function retryWithTimeout(fn, maxRetries, timeout) {
  let lastError;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      const timeoutPromise = new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Timeout')), timeout)
      );
      
      const result = await Promise.race([fn(), timeoutPromise]);
      return result;
    } catch (error) {
      lastError = error;
      
      if (attempt < maxRetries) {
        console.log(`Attempt ${attempt + 1} failed (${error.message}), retrying...`);
      }
    }
  }
  
  throw new Error(`Failed after ${maxRetries + 1} attempts: ${lastError.message}`);
}

// Test
console.log('\n=== Retry with Timeout ===');

let timeoutAttempts = 0;
const slowFunction = () => {
  timeoutAttempts++;
  const delay = timeoutAttempts < 3 ? 2000 : 100; // First 2 attempts timeout
  
  return new Promise(resolve => 
    setTimeout(() => resolve(`Success on attempt ${timeoutAttempts}`), delay)
  );
};

retryWithTimeout(slowFunction, 3, 500)
  .then(result => console.log('Result:', result))
  .catch(error => console.log('Error:', error.message));
```

### **Bonus: Retry Class**
```javascript
/**
 * Reusable retry class with configuration
 */
class Retrier {
  constructor(options = {}) {
    this.maxRetries = options.maxRetries || 3;
    this.delay = options.delay || 1000;
    this.backoff = options.backoff || false;
    this.onRetry = options.onRetry || (() => {});
  }
  
  async execute(fn) {
    let lastError;
    
    for (let attempt = 0; attempt <= this.maxRetries; attempt++) {
      try {
        return await fn();
      } catch (error) {
        lastError = error;
        
        if (attempt < this.maxRetries) {
          this.onRetry(attempt + 1, error);
          
          const delay = this.backoff 
            ? this.delay * Math.pow(2, attempt)
            : this.delay;
          
          await new Promise(resolve => setTimeout(resolve, delay));
        }
      }
    }
    
    throw new Error(`Failed after ${this.maxRetries + 1} attempts: ${lastError.message}`);
  }
}

// Test
console.log('\n=== Retry Class ===');

const retrier = new Retrier({
  maxRetries: 3,
  delay: 200,
  backoff: true,
  onRetry: (attempt, error) => console.log(`Retry attempt ${attempt}: ${error.message}`)
});

let classAttempts = 0;
const testFunction = () => {
  classAttempts++;
  
  if (classAttempts < 3) {
    return Promise.reject(new Error('Not ready'));
  }
  
  return Promise.resolve('Ready!');
};

retrier.execute(testFunction)
  .then(result => console.log('Class result:', result))
  .catch(error => console.log('Class error:', error.message));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Immediate success (no retries needed)
retry(() => Promise.resolve('immediate'), 3)
  .then(result => console.log('Immediate success:', result))
  .catch(error => console.log('Error:', error));

// Always fails
let alwaysFails = 0;
retry(() => {
  alwaysFails++;
  return Promise.reject(new Error(`Failure ${alwaysFails}`));
}, 2)
  .then(result => console.log('Result:', result))
  .catch(error => console.log('Always fails:', error.message));

// Zero retries
retry(() => Promise.reject(new Error('No retries')), 0)
  .then(result => console.log('Result:', result))
  .catch(error => console.log('Zero retries:', error.message));

// Synchronous error
retry(() => {
  throw new Error('Sync error');
}, 2)
  .then(result => console.log('Result:', result))
  .catch(error => console.log('Sync error caught:', error.message));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Retry HTTP request
async function fetchWithRetry(url, options, maxRetries = 3) {
  return retry(
    () => fetch(url, options).then(r => {
      if (!r.ok) throw new Error(`HTTP ${r.status}`);
      return r.json();
    }),
    maxRetries
  );
}

// 2. Retry database connection
async function connectWithRetry(connectionString, maxRetries = 5) {
  return retryWithBackoff(
    () => database.connect(connectionString),
    maxRetries,
    2000 // Start with 2 second delay
  );
}

// 3. Retry only on network errors
async function apiCallWithRetry(endpoint) {
  return retryIf(
    () => api.call(endpoint),
    3,
    (error) => error.code === 'NETWORK_ERROR' || error.code === 'TIMEOUT'
  );
}

console.log('\n=== Use Cases ===');
console.log('fetchWithRetry - retry failed HTTP requests');
console.log('connectWithRetry - retry database connections with backoff');
console.log('apiCallWithRetry - retry only network errors, not auth errors');
```

**Interview Tips:**
- Retry pattern: attempt operation, if fails, try again up to max attempts
- Common strategies: fixed delay, exponential backoff, jittered backoff
- Exponential backoff: delay doubles each retry (1s, 2s, 4s, 8s...)
- Use for transient failures: network issues, rate limits, temporary unavailability
- Don't retry: authentication errors, validation errors, 404s
- Key decisions: max retries, delay strategy, which errors to retry
- For loop approach cleaner than recursion (avoids stack overflow)
- Track attempt number for logging and decision making
- Add callbacks/hooks for monitoring and debugging
- Consider timeout per attempt to prevent hanging
- Jitter: add randomness to prevent thundering herd
- Circuit breaker pattern: stop retrying after threshold
- Idempotency important: operation should be safe to retry
- Applications: API calls, database operations, file uploads, message queues
- Edge cases: immediate success, always fails, zero retries, sync errors
- Clarify: fixed delay or backoff? which errors to retry? timeout needed?
- Follow-ups: implement circuit breaker, add jitter, retry with rate limiting
- Common mistakes: infinite retries, no delay, retrying non-idempotent operations

</details>

80. Implement a promise-based delay function

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Delay Function**
```javascript
/**
 * Simple promise-based delay function
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * Returns a promise that resolves after specified milliseconds
 */
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Test
console.log('=== Basic Delay ===');

(async () => {
  console.log('Start:', new Date().toLocaleTimeString());
  await delay(1000);
  console.log('After 1s:', new Date().toLocaleTimeString());
  await delay(500);
  console.log('After 0.5s more:', new Date().toLocaleTimeString());
})();
```

### **Approach 2: Delay with Value**
```javascript
/**
 * Delay that resolves with a specific value
 */
function delayWithValue(ms, value) {
  return new Promise(resolve => setTimeout(() => resolve(value), ms));
}

// Test
console.log('\n=== Delay with Value ===');

(async () => {
  const result1 = await delayWithValue(500, 'Hello');
  console.log('Result 1:', result1); // 'Hello'
  
  const result2 = await delayWithValue(300, { data: 'World' });
  console.log('Result 2:', result2); // { data: 'World' }
})();
```

### **Approach 3: Cancellable Delay**
```javascript
/**
 * Delay with cancellation support
 */
function cancellableDelay(ms) {
  let timeoutId;
  let rejectFn;
  
  const promise = new Promise((resolve, reject) => {
    rejectFn = reject;
    timeoutId = setTimeout(resolve, ms);
  });
  
  promise.cancel = () => {
    clearTimeout(timeoutId);
    rejectFn(new Error('Delay cancelled'));
  };
  
  return promise;
}

// Test
console.log('\n=== Cancellable Delay ===');

(async () => {
  const delayPromise = cancellableDelay(2000);
  
  // Cancel after 500ms
  setTimeout(() => {
    console.log('Cancelling delay...');
    delayPromise.cancel();
  }, 500);
  
  try {
    await delayPromise;
    console.log('Delay completed');
  } catch (error) {
    console.log('Caught:', error.message); // 'Delay cancelled'
  }
})();
```

### **Approach 4: Delay with Callback**
```javascript
/**
 * Delay with optional callback on completion
 */
function delayWithCallback(ms, callback) {
  return new Promise(resolve => {
    setTimeout(() => {
      const result = callback ? callback() : undefined;
      resolve(result);
    }, ms);
  });
}

// Test
console.log('\n=== Delay with Callback ===');

(async () => {
  const result = await delayWithCallback(400, () => {
    console.log('Callback executed!');
    return 'Callback result';
  });
  
  console.log('Returned:', result); // 'Callback result'
})();
```

### **Approach 5: Delay with Progress**
```javascript
/**
 * Delay with progress callbacks
 */
function delayWithProgress(ms, onProgress, interval = 100) {
  return new Promise(resolve => {
    const startTime = Date.now();
    
    const progressInterval = setInterval(() => {
      const elapsed = Date.now() - startTime;
      const progress = Math.min(elapsed / ms, 1);
      
      if (onProgress) {
        onProgress({
          elapsed,
          total: ms,
          progress: progress * 100,
          remaining: ms - elapsed
        });
      }
      
      if (elapsed >= ms) {
        clearInterval(progressInterval);
      }
    }, interval);
    
    setTimeout(() => {
      clearInterval(progressInterval);
      resolve();
    }, ms);
  });
}

// Test
console.log('\n=== Delay with Progress ===');

(async () => {
  console.log('Starting delay with progress...');
  
  await delayWithProgress(1000, (info) => {
    console.log(`Progress: ${info.progress.toFixed(0)}% (${info.elapsed}ms / ${info.total}ms)`);
  }, 250);
  
  console.log('Delay complete!');
})();
```

### **Bonus: Minimum Delay Wrapper**
```javascript
/**
 * Ensure a function takes at least a minimum time
 * Useful for showing loading indicators
 */
async function withMinDelay(fn, minMs) {
  const [result] = await Promise.all([
    fn(),
    delay(minMs)
  ]);
  return result;
}

// Test
console.log('\n=== Minimum Delay Wrapper ===');

(async () => {
  const fastFunction = () => Promise.resolve('Fast result');
  
  console.log('Starting...');
  const start = Date.now();
  
  const result = await withMinDelay(fastFunction, 1000);
  
  const elapsed = Date.now() - start;
  console.log('Result:', result);
  console.log('Took at least:', elapsed, 'ms'); // ~1000ms
})();
```

### **Bonus: Delay Between Operations**
```javascript
/**
 * Add delay between async operations
 */
async function delayBetween(operations, delayMs) {
  const results = [];
  
  for (let i = 0; i < operations.length; i++) {
    const result = await operations[i]();
    results.push(result);
    
    // Don't delay after last operation
    if (i < operations.length - 1) {
      await delay(delayMs);
    }
  }
  
  return results;
}

// Test
console.log('\n=== Delay Between Operations ===');

(async () => {
  const operations = [
    () => Promise.resolve('Op 1'),
    () => Promise.resolve('Op 2'),
    () => Promise.resolve('Op 3')
  ];
  
  console.log('Starting operations with delays...');
  const start = Date.now();
  
  const results = await delayBetween(operations, 300);
  
  const elapsed = Date.now() - start;
  console.log('Results:', results);
  console.log('Total time:', elapsed, 'ms'); // ~600ms (2 delays)
})();
```

### **Bonus: Timeout Function**
```javascript
/**
 * Create timeout wrapper using delay
 */
function timeout(promise, ms) {
  const timeoutPromise = delay(ms).then(() => {
    throw new Error(`Operation timed out after ${ms}ms`);
  });
  
  return Promise.race([promise, timeoutPromise]);
}

// Test
console.log('\n=== Timeout Function ===');

(async () => {
  // Fast operation - succeeds
  try {
    const fast = delayWithValue(200, 'Quick!');
    const result = await timeout(fast, 500);
    console.log('Fast operation:', result);
  } catch (error) {
    console.log('Error:', error.message);
  }
  
  // Slow operation - times out
  try {
    const slow = delayWithValue(2000, 'Slow...');
    const result = await timeout(slow, 500);
    console.log('Slow operation:', result);
  } catch (error) {
    console.log('Timeout error:', error.message);
  }
})();
```

### **Bonus: Throttled Delay**
```javascript
/**
 * Throttle function calls with delay
 */
function throttleWithDelay(fn, delayMs) {
  let lastCall = 0;
  
  return async function(...args) {
    const now = Date.now();
    const timeSinceLastCall = now - lastCall;
    
    if (timeSinceLastCall < delayMs) {
      await delay(delayMs - timeSinceLastCall);
    }
    
    lastCall = Date.now();
    return fn(...args);
  };
}

// Test
console.log('\n=== Throttled with Delay ===');

(async () => {
  let callCount = 0;
  const apiCall = (msg) => {
    callCount++;
    console.log(`Call ${callCount}: ${msg} at ${new Date().toLocaleTimeString()}`);
    return Promise.resolve(`Result ${callCount}`);
  };
  
  const throttled = throttleWithDelay(apiCall, 500);
  
  // These calls will be spaced at least 500ms apart
  await throttled('First');
  await throttled('Second');
  await throttled('Third');
})();
```

### **Bonus: Sleep Function (Alias)**
```javascript
/**
 * Sleep function - common alias for delay
 */
const sleep = delay;

// Can also be written as:
function sleep2(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Test
console.log('\n=== Sleep Function ===');

(async () => {
  console.log('Going to sleep...');
  await sleep(800);
  console.log('Woke up!');
})();
```

### **Bonus: Delay Class**
```javascript
/**
 * Delay class with utilities
 */
class Delay {
  constructor(ms) {
    this.ms = ms;
    this.timeoutId = null;
    this.promise = null;
  }
  
  start() {
    this.promise = new Promise((resolve, reject) => {
      this.resolve = resolve;
      this.reject = reject;
      this.timeoutId = setTimeout(() => resolve(), this.ms);
    });
    return this.promise;
  }
  
  cancel() {
    if (this.timeoutId) {
      clearTimeout(this.timeoutId);
      this.reject(new Error('Delay cancelled'));
    }
  }
  
  static wait(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
  
  static async sequence(delays) {
    for (const ms of delays) {
      await Delay.wait(ms);
    }
  }
}

// Test
console.log('\n=== Delay Class ===');

(async () => {
  // Instance usage
  const d = new Delay(500);
  d.start().then(() => console.log('Instance delay done'));
  
  // Static usage
  await Delay.wait(600);
  console.log('Static delay done');
  
  // Sequence
  console.log('Starting sequence...');
  await Delay.sequence([200, 300, 400]);
  console.log('Sequence done');
})();
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications of delay function
 */

// 1. Rate limiting API calls
async function rateLimitedFetch(urls, delayBetween = 1000) {
  const results = [];
  
  for (const url of urls) {
    const response = await fetch(url);
    results.push(await response.json());
    
    if (urls.indexOf(url) < urls.length - 1) {
      await delay(delayBetween);
    }
  }
  
  return results;
}

// 2. Retry with exponential backoff
async function retryWithDelay(fn, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      
      const delayMs = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1} after ${delayMs}ms`);
      await delay(delayMs);
    }
  }
}

// 3. Simulate loading state
async function fetchWithMinLoading(fetchFn, minLoadingTime = 1000) {
  const startTime = Date.now();
  const data = await fetchFn();
  const elapsed = Date.now() - startTime;
  
  if (elapsed < minLoadingTime) {
    await delay(minLoadingTime - elapsed);
  }
  
  return data;
}

// 4. Debounce with delay
function debounceAsync(fn, wait) {
  let timeoutId;
  
  return function(...args) {
    return new Promise((resolve) => {
      clearTimeout(timeoutId);
      
      timeoutId = setTimeout(async () => {
        const result = await fn(...args);
        resolve(result);
      }, wait);
    });
  };
}

console.log('\n=== Use Cases ===');
console.log('rateLimitedFetch - delay between API calls');
console.log('retryWithDelay - exponential backoff retry');
console.log('fetchWithMinLoading - minimum loading indicator time');
console.log('debounceAsync - debounce async operations');
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Zero delay
(async () => {
  console.log('Zero delay start');
  await delay(0);
  console.log('Zero delay end (next tick)');
})();

// Very long delay (shouldn't cause issues)
delay(100000).then(() => console.log('Very long delay'));

// Negative delay (treated as 0)
(async () => {
  await delay(-100);
  console.log('Negative delay completed immediately');
})();

// Chained delays
(async () => {
  await delay(100)
    .then(() => delay(100))
    .then(() => delay(100));
  console.log('Chained delays completed');
})();

// Parallel delays
(async () => {
  const start = Date.now();
  
  await Promise.all([
    delay(300),
    delay(200),
    delay(400)
  ]);
  
  const elapsed = Date.now() - start;
  console.log(`Parallel delays: ~${elapsed}ms (should be ~400ms, not 900ms)`);
})();
```

### **Performance Considerations**
```javascript
/**
 * Compare different delay patterns
 */
console.log('\n=== Performance Patterns ===');

// Sequential delays - total time = sum of delays
async function sequential() {
  const start = Date.now();
  
  await delay(100);
  await delay(100);
  await delay(100);
  
  console.log('Sequential:', Date.now() - start, 'ms'); // ~300ms
}

// Parallel delays - total time = max delay
async function parallel() {
  const start = Date.now();
  
  await Promise.all([
    delay(100),
    delay(100),
    delay(100)
  ]);
  
  console.log('Parallel:', Date.now() - start, 'ms'); // ~100ms
}

setTimeout(() => {
  sequential().then(() => parallel());
}, 1500);
```

### **Comparison with setTimeout**
```javascript
/**
 * Demonstrate advantages of promise-based delay
 */
console.log('\n=== Promise Delay vs setTimeout ===');

// setTimeout - callback based
setTimeout(() => {
  console.log('setTimeout callback');
}, 500);

// Promise delay - async/await
(async () => {
  await delay(500);
  console.log('Promise delay with await');
})();

// Chaining with setTimeout is messy
setTimeout(() => {
  console.log('Step 1');
  setTimeout(() => {
    console.log('Step 2');
    setTimeout(() => {
      console.log('Step 3');
    }, 100);
  }, 100);
}, 100);

// Promise delay is cleaner
(async () => {
  await delay(600);
  console.log('Clean Step 1');
  await delay(100);
  console.log('Clean Step 2');
  await delay(100);
  console.log('Clean Step 3');
})();
```

**Interview Tips:**
- Promise-based delay wraps setTimeout in a Promise
- Enables use with async/await for cleaner async code
- Basic: `new Promise(resolve => setTimeout(resolve, ms))`
- Can resolve with value: `setTimeout(() => resolve(value), ms)`
- Common aliases: sleep, wait, pause
- Use cases: rate limiting, delays between operations, retries, animations
- Advantages over setTimeout: composable, can use with async/await, chainable
- Can add cancellation by tracking timeoutId and calling clearTimeout
- Zero/negative delay still defers to next event loop tick
- Parallel delays: Promise.all waits for slowest, not sum of all
- Sequential delays: await each one, total = sum of delays
- Useful for: debouncing, throttling, polling, retries, loading states
- Related patterns: timeout wrapper, minimum delay, throttle, debounce
- Applications: API rate limiting, UI animations, simulating network delay
- Edge cases: zero delay, negative delay, very long delay, cancellation
- Clarify: need cancellation? resolve with value? progress tracking?
- Follow-ups: implement timeout, debounce, throttle, retry with backoff

</details>

81. Create a function to execute promises sequentially

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Sequential Execution with Async/Await**
```javascript
/**
 * Execute promise-returning functions sequentially
 * Time Complexity: O(n) where n is number of functions
 * Space Complexity: O(n) for results array
 */
async function executeSequentially(promiseFunctions) {
  const results = [];
  
  for (const fn of promiseFunctions) {
    const result = await fn();
    results.push(result);
  }
  
  return results;
}

// Test
console.log('=== Basic Sequential Execution ===');

const delay = (ms, value) => new Promise(resolve => 
  setTimeout(() => resolve(value), ms)
);

(async () => {
  const functions = [
    () => delay(300, 'First'),
    () => delay(100, 'Second'),
    () => delay(200, 'Third')
  ];
  
  console.log('Starting sequential execution...');
  const start = Date.now();
  
  const results = await executeSequentially(functions);
  
  console.log('Results:', results); // ['First', 'Second', 'Third']
  console.log('Time:', Date.now() - start, 'ms'); // ~600ms (300+100+200)
})();
```

### **Approach 2: Using Reduce**
```javascript
/**
 * Sequential execution using Array.reduce
 */
async function executeSequentiallyReduce(promiseFunctions) {
  return promiseFunctions.reduce(async (previousPromise, currentFn) => {
    const results = await previousPromise;
    const result = await currentFn();
    return [...results, result];
  }, Promise.resolve([]));
}

// Test
console.log('\n=== Using Reduce ===');

(async () => {
  const functions = [
    () => delay(200, 'A'),
    () => delay(150, 'B'),
    () => delay(100, 'C')
  ];
  
  const results = await executeSequentiallyReduce(functions);
  console.log('Reduce results:', results); // ['A', 'B', 'C']
})();
```

### **Approach 3: Sequential with Error Handling**
```javascript
/**
 * Sequential execution with individual error handling
 */
async function executeSequentiallyWithErrors(promiseFunctions, options = {}) {
  const { stopOnError = true, defaultValue = null } = options;
  const results = [];
  
  for (let i = 0; i < promiseFunctions.length; i++) {
    try {
      const result = await promiseFunctions[i]();
      results.push({ status: 'fulfilled', value: result, index: i });
    } catch (error) {
      results.push({ status: 'rejected', reason: error, index: i });
      
      if (stopOnError) {
        throw new Error(`Stopped at index ${i}: ${error.message}`);
      }
    }
  }
  
  return results;
}

// Test
console.log('\n=== With Error Handling ===');

(async () => {
  const functions = [
    () => Promise.resolve('Success 1'),
    () => Promise.reject(new Error('Failed!')),
    () => Promise.resolve('Success 3')
  ];
  
  // Continue on error
  const results = await executeSequentiallyWithErrors(functions, { stopOnError: false });
  console.log('Results:', results);
  
  // Stop on error
  try {
    await executeSequentiallyWithErrors(functions, { stopOnError: true });
  } catch (error) {
    console.log('Stopped:', error.message);
  }
})();
```

### **Approach 4: Sequential with Index and Context**
```javascript
/**
 * Sequential execution with access to previous results
 */
async function executeSequentiallyWithContext(promiseFunctions) {
  const results = [];
  
  for (let i = 0; i < promiseFunctions.length; i++) {
    const result = await promiseFunctions[i](results, i);
    results.push(result);
  }
  
  return results;
}

// Test
console.log('\n=== With Context ===');

(async () => {
  const functions = [
    () => delay(100, 1),
    (prev) => delay(100, prev[0] + 1), // Access previous results
    (prev) => delay(100, prev[0] + prev[1]) // Sum of previous
  ];
  
  const results = await executeSequentiallyWithContext(functions);
  console.log('Context results:', results); // [1, 2, 3]
})();
```

### **Approach 5: Sequential Pipeline**
```javascript
/**
 * Pipeline where output of one becomes input of next
 */
async function pipeline(initialValue, ...functions) {
  let result = initialValue;
  
  for (const fn of functions) {
    result = await fn(result);
  }
  
  return result;
}

// Test
console.log('\n=== Pipeline ===');

(async () => {
  const result = await pipeline(
    5,
    async (x) => { await delay(100); return x * 2; },
    async (x) => { await delay(100); return x + 3; },
    async (x) => { await delay(100); return x - 1; }
  );
  
  console.log('Pipeline result:', result); // ((5 * 2) + 3) - 1 = 12
})();
```

### **Bonus: Sequential with Batching**
```javascript
/**
 * Execute in sequential batches
 */
async function executeInBatches(promiseFunctions, batchSize) {
  const results = [];
  
  for (let i = 0; i < promiseFunctions.length; i += batchSize) {
    const batch = promiseFunctions.slice(i, i + batchSize);
    
    // Execute batch in parallel
    const batchResults = await Promise.all(batch.map(fn => fn()));
    results.push(...batchResults);
    
    console.log(`Batch ${Math.floor(i / batchSize) + 1} complete`);
  }
  
  return results;
}

// Test
console.log('\n=== Sequential Batches ===');

(async () => {
  const functions = Array.from({ length: 10 }, (_, i) => 
    () => delay(100, `Item ${i + 1}`)
  );
  
  const start = Date.now();
  const results = await executeInBatches(functions, 3);
  
  console.log('Batch results:', results.length, 'items');
  console.log('Time:', Date.now() - start, 'ms'); // ~400ms (4 batches)
})();
```

### **Bonus: Sequential with Progress**
```javascript
/**
 * Sequential execution with progress tracking
 */
async function executeSequentiallyWithProgress(promiseFunctions, onProgress) {
  const results = [];
  const total = promiseFunctions.length;
  
  for (let i = 0; i < total; i++) {
    const result = await promiseFunctions[i]();
    results.push(result);
    
    if (onProgress) {
      onProgress({
        completed: i + 1,
        total,
        percentage: ((i + 1) / total) * 100,
        current: result
      });
    }
  }
  
  return results;
}

// Test
console.log('\n=== With Progress ===');

(async () => {
  const functions = [
    () => delay(200, 'Task 1'),
    () => delay(300, 'Task 2'),
    () => delay(100, 'Task 3')
  ];
  
  const results = await executeSequentiallyWithProgress(functions, (progress) => {
    console.log(`Progress: ${progress.percentage.toFixed(0)}% - ${progress.current}`);
  });
  
  console.log('All done:', results);
})();
```

### **Bonus: Sequential with Retry**
```javascript
/**
 * Sequential execution with retry on failure
 */
async function executeSequentiallyWithRetry(promiseFunctions, maxRetries = 3) {
  const results = [];
  
  for (let i = 0; i < promiseFunctions.length; i++) {
    let lastError;
    
    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        const result = await promiseFunctions[i]();
        results.push(result);
        break; // Success, move to next function
      } catch (error) {
        lastError = error;
        
        if (attempt < maxRetries) {
          console.log(`Function ${i} failed, retry ${attempt + 1}/${maxRetries}`);
          await delay(100 * Math.pow(2, attempt)); // Exponential backoff
        }
      }
    }
    
    if (results.length !== i + 1) {
      throw new Error(`Function ${i} failed after ${maxRetries + 1} attempts: ${lastError.message}`);
    }
  }
  
  return results;
}

// Test
console.log('\n=== With Retry ===');

(async () => {
  let attempts = 0;
  const functions = [
    () => Promise.resolve('Success 1'),
    () => {
      attempts++;
      return attempts < 3 
        ? Promise.reject(new Error('Temporary failure'))
        : Promise.resolve('Success 2');
    },
    () => Promise.resolve('Success 3')
  ];
  
  const results = await executeSequentiallyWithRetry(functions, 3);
  console.log('Retry results:', results);
})();
```

### **Bonus: Comparison - Sequential vs Parallel**
```javascript
/**
 * Compare execution times
 */
async function compareExecution() {
  const functions = [
    () => delay(300, 'A'),
    () => delay(200, 'B'),
    () => delay(100, 'C')
  ];
  
  console.log('\n=== Sequential vs Parallel ===');
  
  // Sequential
  const seqStart = Date.now();
  await executeSequentially(functions);
  console.log('Sequential time:', Date.now() - seqStart, 'ms'); // ~600ms
  
  // Parallel
  const parStart = Date.now();
  await Promise.all(functions.map(fn => fn()));
  console.log('Parallel time:', Date.now() - parStart, 'ms'); // ~300ms
}

setTimeout(() => compareExecution(), 1500);
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty array
(async () => {
  const results = await executeSequentially([]);
  console.log('Empty array:', results); // []
})();

// Single function
(async () => {
  const results = await executeSequentially([
    () => Promise.resolve('Single')
  ]);
  console.log('Single function:', results); // ['Single']
})();

// Functions returning non-promises
(async () => {
  const results = await executeSequentially([
    () => 'Sync 1',
    () => Promise.resolve('Async'),
    () => 'Sync 2'
  ]);
  console.log('Mixed sync/async:', results); // ['Sync 1', 'Async', 'Sync 2']
})();

// One function rejects
(async () => {
  try {
    await executeSequentially([
      () => Promise.resolve('OK'),
      () => Promise.reject(new Error('Failed')),
      () => Promise.resolve('Never reached')
    ]);
  } catch (error) {
    console.log('Rejection stops execution:', error.message);
  }
})();
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Sequential database migrations
async function runMigrations(migrations) {
  console.log('\n=== Database Migrations ===');
  
  for (let i = 0; i < migrations.length; i++) {
    console.log(`Running migration ${i + 1}/${migrations.length}...`);
    await migrations[i]();
  }
  
  console.log('All migrations complete');
}

// 2. Sequential file processing
async function processFiles(files) {
  const results = [];
  
  for (const file of files) {
    const content = await readFile(file);
    const processed = await processContent(content);
    await writeFile(file, processed);
    results.push({ file, status: 'processed' });
  }
  
  return results;
}

// 3. Step-by-step wizard/form
async function processWizardSteps(steps) {
  const answers = [];
  
  for (const step of steps) {
    const answer = await step.prompt();
    answers.push(answer);
    
    // Each step might depend on previous answers
    if (step.validate && !step.validate(answers)) {
      throw new Error('Validation failed');
    }
  }
  
  return answers;
}

// 4. Sequential API calls with dependencies
async function fetchUserData(userId) {
  const user = await fetchUser(userId);
  const posts = await fetchUserPosts(user.id);
  const comments = await fetchPostComments(posts.map(p => p.id));
  
  return { user, posts, comments };
}

console.log('\nUse Cases:');
console.log('runMigrations - database migrations must run in order');
console.log('processFiles - avoid file conflicts');
console.log('processWizardSteps - each step depends on previous');
console.log('fetchUserData - each request needs data from previous');
```

### **Performance Considerations**
```javascript
/**
 * When to use sequential vs parallel
 */
console.log('\n=== Sequential vs Parallel Decision ===');

// Use Sequential When:
// 1. Operations have dependencies
// 2. Need to avoid race conditions
// 3. Rate limiting (don't overwhelm server)
// 4. Order matters
// 5. Limited resources (memory, connections)

// Use Parallel When:
// 1. No dependencies between operations
// 2. Want maximum speed
// 3. Operations are independent
// 4. Order doesn't matter

// Example: Rate-limited API calls
async function rateLimitedFetch(urls) {
  const results = [];
  
  for (const url of urls) {
    const result = await fetch(url);
    results.push(await result.json());
    
    // Delay between requests to respect rate limits
    await delay(1000);
  }
  
  return results;
}

console.log('Sequential good for: dependencies, rate limiting, resource constraints');
console.log('Parallel good for: independent operations, maximum speed');
```

**Interview Tips:**
- Sequential execution: wait for each promise before starting next
- Use for: dependent operations, rate limiting, ordered execution
- Basic: use for loop with await (not forEach!)
- forEach won't work with await - it doesn't wait for async functions
- Use for...of loop for sequential execution with await
- Array.reduce can work but for loop is clearer
- Sequential time = sum of all operation times
- Parallel time = max of all operation times
- Sequential vs Parallel: know when to use each
- Dependencies require sequential execution
- Pipeline pattern: output of one becomes input of next
- Can combine: sequential batches where each batch runs in parallel
- Error handling: decide if one failure should stop all
- Progress tracking useful for long-running sequential tasks
- Applications: migrations, file processing, dependent API calls, wizards
- Edge cases: empty array, single item, mixed sync/async, early rejection
- Common mistake: using forEach with await (doesn't work!)
- Clarify: should errors stop execution? need progress? retries?
- Follow-ups: implement batching, add retry logic, rate limiting

</details>

82. Implement parallel promise execution with concurrency limit

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Concurrency Limiter**
```javascript
/**
 * Execute promises with limited concurrency
 * Time Complexity: O(n) where n is number of tasks
 * Space Complexity: O(concurrency) for active promises
 * 
 * Limits number of promises running simultaneously
 */
async function promiseConcurrency(tasks, concurrency) {
  const results = [];
  const executing = [];
  
  for (const [index, task] of tasks.entries()) {
    const promise = Promise.resolve().then(() => task()).then(result => {
      results[index] = result;
    });
    
    executing.push(promise);
    
    if (executing.length >= concurrency) {
      await Promise.race(executing);
      // Remove completed promises
      executing.splice(executing.findIndex(p => p === promise), 1);
    }
  }
  
  await Promise.all(executing);
  return results;
}

// Test
console.log('=== Basic Concurrency Limiter ===');

const delay = (ms, value) => new Promise(resolve => 
  setTimeout(() => {
    console.log(`Completed: ${value}`);
    resolve(value);
  }, ms)
);

(async () => {
  const tasks = [
    () => delay(1000, 'Task 1'),
    () => delay(500, 'Task 2'),
    () => delay(800, 'Task 3'),
    () => delay(300, 'Task 4'),
    () => delay(600, 'Task 5')
  ];
  
  console.log('Starting with concurrency 2...');
  const start = Date.now();
  
  const results = await promiseConcurrency(tasks, 2);
  
  console.log('Results:', results);
  console.log('Time:', Date.now() - start, 'ms');
  // With concurrency 2: ~2300ms (not all at once)
})();
```

### **Approach 2: Using Promise Pool**
```javascript
/**
 * Better implementation using promise pool
 */
async function promisePool(tasks, concurrency) {
  const results = [];
  let index = 0;
  
  async function worker() {
    while (index < tasks.length) {
      const currentIndex = index++;
      const task = tasks[currentIndex];
      
      try {
        results[currentIndex] = await task();
      } catch (error) {
        results[currentIndex] = { error };
      }
    }
  }
  
  // Create worker pool
  const workers = Array(Math.min(concurrency, tasks.length))
    .fill(null)
    .map(() => worker());
  
  await Promise.all(workers);
  return results;
}

// Test
console.log('\n=== Promise Pool ===');

(async () => {
  const tasks = Array.from({ length: 10 }, (_, i) => 
    () => delay(Math.random() * 1000, `Task ${i + 1}`)
  );
  
  console.log('Starting pool with concurrency 3...');
  const start = Date.now();
  
  const results = await promisePool(tasks, 3);
  
  console.log('Pool results:', results.length, 'tasks');
  console.log('Time:', Date.now() - start, 'ms');
})();
```

### **Approach 3: With Error Handling and Retry**
```javascript
/**
 * Concurrency with comprehensive error handling
 */
async function promiseConcurrencyWithRetry(tasks, concurrency, options = {}) {
  const { maxRetries = 0, onProgress = null } = options;
  const results = [];
  let index = 0;
  let completed = 0;
  
  async function worker() {
    while (index < tasks.length) {
      const currentIndex = index++;
      const task = tasks[currentIndex];
      let lastError;
      
      // Retry logic
      for (let attempt = 0; attempt <= maxRetries; attempt++) {
        try {
          results[currentIndex] = {
            status: 'fulfilled',
            value: await task()
          };
          break;
        } catch (error) {
          lastError = error;
          
          if (attempt < maxRetries) {
            console.log(`Task ${currentIndex} failed, retry ${attempt + 1}/${maxRetries}`);
            await new Promise(r => setTimeout(r, 100 * Math.pow(2, attempt)));
          }
        }
      }
      
      if (!results[currentIndex]) {
        results[currentIndex] = {
          status: 'rejected',
          reason: lastError
        };
      }
      
      completed++;
      
      if (onProgress) {
        onProgress({
          completed,
          total: tasks.length,
          percentage: (completed / tasks.length) * 100
        });
      }
    }
  }
  
  const workers = Array(Math.min(concurrency, tasks.length))
    .fill(null)
    .map(() => worker());
  
  await Promise.all(workers);
  return results;
}

// Test
console.log('\n=== With Retry ===');

(async () => {
  let taskFailCount = 0;
  
  const tasks = [
    () => Promise.resolve('Success 1'),
    () => {
      taskFailCount++;
      return taskFailCount < 2 
        ? Promise.reject(new Error('Temporary failure'))
        : Promise.resolve('Success 2');
    },
    () => Promise.resolve('Success 3'),
    () => Promise.reject(new Error('Permanent failure'))
  ];
  
  const results = await promiseConcurrencyWithRetry(tasks, 2, {
    maxRetries: 2,
    onProgress: (p) => console.log(`Progress: ${p.percentage.toFixed(0)}%`)
  });
  
  console.log('Retry results:', results);
})();
```

### **Approach 4: P-Limit Style Implementation**
```javascript
/**
 * Popular p-limit library style
 */
function pLimit(concurrency) {
  const queue = [];
  let activeCount = 0;
  
  const next = () => {
    activeCount--;
    
    if (queue.length > 0) {
      queue.shift()();
    }
  };
  
  const run = async (fn, resolve, reject) => {
    activeCount++;
    
    try {
      const result = await fn();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      next();
    }
  };
  
  const enqueue = (fn) => {
    return new Promise((resolve, reject) => {
      const execute = () => run(fn, resolve, reject);
      
      if (activeCount < concurrency) {
        execute();
      } else {
        queue.push(execute);
      }
    });
  };
  
  return enqueue;
}

// Test
console.log('\n=== P-Limit Style ===');

(async () => {
  const limit = pLimit(2);
  
  const tasks = [
    () => delay(500, 'A'),
    () => delay(300, 'B'),
    () => delay(700, 'C'),
    () => delay(200, 'D'),
    () => delay(400, 'E')
  ];
  
  console.log('P-Limit with concurrency 2...');
  const start = Date.now();
  
  const results = await Promise.all(tasks.map(task => limit(task)));
  
  console.log('P-Limit results:', results);
  console.log('Time:', Date.now() - start, 'ms');
})();
```

### **Approach 5: Class-Based Implementation**
```javascript
/**
 * Reusable class for concurrent execution
 */
class ConcurrencyLimiter {
  constructor(concurrency) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }
  
  async run(task) {
    while (this.running >= this.concurrency) {
      await new Promise(resolve => this.queue.push(resolve));
    }
    
    this.running++;
    
    try {
      return await task();
    } finally {
      this.running--;
      
      const resolve = this.queue.shift();
      if (resolve) resolve();
    }
  }
  
  async runAll(tasks) {
    return Promise.all(tasks.map(task => this.run(task)));
  }
  
  getStatus() {
    return {
      running: this.running,
      queued: this.queue.length,
      concurrency: this.concurrency
    };
  }
}

// Test
console.log('\n=== Class-Based Limiter ===');

(async () => {
  const limiter = new ConcurrencyLimiter(3);
  
  const tasks = Array.from({ length: 8 }, (_, i) => 
    () => delay(500, `Task ${i + 1}`)
  );
  
  console.log('Class-based with concurrency 3...');
  
  // Monitor status
  const statusInterval = setInterval(() => {
    const status = limiter.getStatus();
    console.log(`Running: ${status.running}, Queued: ${status.queued}`);
  }, 200);
  
  const results = await limiter.runAll(tasks);
  
  clearInterval(statusInterval);
  console.log('Class results:', results.length, 'tasks');
})();
```

### **Bonus: With Rate Limiting**
```javascript
/**
 * Combine concurrency with rate limiting
 */
async function promiseWithRateLimit(tasks, concurrency, rateLimit) {
  const { requestsPerInterval, interval } = rateLimit;
  const results = [];
  let index = 0;
  let requestsInInterval = 0;
  let intervalStart = Date.now();
  
  async function worker() {
    while (index < tasks.length) {
      const currentIndex = index++;
      
      // Check rate limit
      if (requestsInInterval >= requestsPerInterval) {
        const elapsed = Date.now() - intervalStart;
        if (elapsed < interval) {
          await new Promise(r => setTimeout(r, interval - elapsed));
        }
        requestsInInterval = 0;
        intervalStart = Date.now();
      }
      
      requestsInInterval++;
      results[currentIndex] = await tasks[currentIndex]();
    }
  }
  
  const workers = Array(concurrency).fill(null).map(() => worker());
  await Promise.all(workers);
  return results;
}

// Test
console.log('\n=== With Rate Limiting ===');

(async () => {
  const tasks = Array.from({ length: 10 }, (_, i) => 
    () => delay(100, `Task ${i + 1}`)
  );
  
  console.log('Rate limit: 3 requests per second, concurrency: 5');
  const start = Date.now();
  
  const results = await promiseWithRateLimit(tasks, 5, {
    requestsPerInterval: 3,
    interval: 1000
  });
  
  console.log('Rate limited results:', results.length);
  console.log('Time:', Date.now() - start, 'ms'); // ~4000ms (10 tasks / 3 per sec)
})();
```

### **Bonus: Batch Processing**
```javascript
/**
 * Process tasks in fixed-size batches
 */
async function batchProcess(tasks, batchSize) {
  const results = [];
  
  for (let i = 0; i < tasks.length; i += batchSize) {
    const batch = tasks.slice(i, i + batchSize);
    console.log(`Processing batch ${Math.floor(i / batchSize) + 1}...`);
    
    const batchResults = await Promise.all(batch.map(task => task()));
    results.push(...batchResults);
  }
  
  return results;
}

// Test
console.log('\n=== Batch Processing ===');

(async () => {
  const tasks = Array.from({ length: 10 }, (_, i) => 
    () => delay(200, `Batch item ${i + 1}`)
  );
  
  const results = await batchProcess(tasks, 3);
  console.log('Batch results:', results.length, 'items');
})();
```

### **Bonus: Priority Queue**
```javascript
/**
 * Concurrency with task priorities
 */
class PriorityConcurrencyLimiter {
  constructor(concurrency) {
    this.concurrency = concurrency;
    this.running = 0;
    this.queue = [];
  }
  
  async run(task, priority = 0) {
    return new Promise((resolve, reject) => {
      this.queue.push({ task, priority, resolve, reject });
      this.queue.sort((a, b) => b.priority - a.priority);
      this.processQueue();
    });
  }
  
  async processQueue() {
    while (this.running < this.concurrency && this.queue.length > 0) {
      const { task, resolve, reject } = this.queue.shift();
      this.running++;
      
      task()
        .then(resolve)
        .catch(reject)
        .finally(() => {
          this.running--;
          this.processQueue();
        });
    }
  }
}

// Test
console.log('\n=== Priority Queue ===');

(async () => {
  const limiter = new PriorityConcurrencyLimiter(2);
  
  const results = await Promise.all([
    limiter.run(() => delay(300, 'Low priority'), 1),
    limiter.run(() => delay(300, 'High priority'), 10),
    limiter.run(() => delay(300, 'Medium priority'), 5),
    limiter.run(() => delay(300, 'Highest priority'), 20)
  ]);
  
  console.log('Priority results:', results);
})();
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty tasks
(async () => {
  const results = await promisePool([], 5);
  console.log('Empty tasks:', results); // []
})();

// Concurrency higher than task count
(async () => {
  const tasks = [
    () => delay(100, 'A'),
    () => delay(100, 'B')
  ];
  
  const results = await promisePool(tasks, 10);
  console.log('High concurrency:', results); // All run in parallel
})();

// Concurrency of 1 (sequential)
(async () => {
  const tasks = [
    () => delay(100, '1'),
    () => delay(100, '2'),
    () => delay(100, '3')
  ];
  
  const start = Date.now();
  const results = await promisePool(tasks, 1);
  console.log('Concurrency 1 (sequential):', results);
  console.log('Time:', Date.now() - start, 'ms'); // ~300ms
})();

// Mix of fast and slow tasks
(async () => {
  const tasks = [
    () => delay(1000, 'Slow'),
    () => delay(10, 'Fast 1'),
    () => delay(10, 'Fast 2'),
    () => delay(10, 'Fast 3')
  ];
  
  const start = Date.now();
  const results = await promisePool(tasks, 2);
  console.log('Mixed speed results:', results);
  console.log('Time:', Date.now() - start, 'ms');
})();
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Batch API requests
async function batchFetchUsers(userIds, concurrency = 5) {
  const limit = pLimit(concurrency);
  
  const tasks = userIds.map(id => 
    () => fetch(`/api/users/${id}`).then(r => r.json())
  );
  
  return Promise.all(tasks.map(task => limit(task)));
}

// 2. File processing
async function processFilesInParallel(files, maxConcurrent = 3) {
  const limiter = new ConcurrencyLimiter(maxConcurrent);
  
  const tasks = files.map(file => async () => {
    const content = await readFile(file);
    const processed = await processContent(content);
    await writeFile(file, processed);
    return { file, status: 'processed' };
  });
  
  return limiter.runAll(tasks);
}

// 3. Web scraping
async function scrapeUrls(urls, concurrency = 5) {
  return promisePool(
    urls.map(url => () => scrapeUrl(url)),
    concurrency
  );
}

// 4. Image resizing
async function resizeImages(images, concurrency = 4) {
  const limit = pLimit(concurrency);
  
  return Promise.all(
    images.map(image => 
      limit(async () => {
        const resized = await resizeImage(image);
        await saveImage(resized);
        return resized;
      })
    )
  );
}

console.log('\nUse Cases:');
console.log('batchFetchUsers - avoid overwhelming API');
console.log('processFilesInParallel - limit CPU/memory usage');
console.log('scrapeUrls - respectful web scraping');
console.log('resizeImages - prevent memory overflow');
```

### **Performance Comparison**
```javascript
/**
 * Compare different concurrency levels
 */
async function benchmarkConcurrency() {
  console.log('\n=== Concurrency Benchmarks ===');
  
  const createTasks = () => Array.from({ length: 20 }, (_, i) => 
    () => delay(100, `Task ${i + 1}`)
  );
  
  // No limit (all parallel)
  let start = Date.now();
  await Promise.all(createTasks().map(t => t()));
  console.log('No limit (20 parallel):', Date.now() - start, 'ms'); // ~100ms
  
  // Concurrency 5
  start = Date.now();
  await promisePool(createTasks(), 5);
  console.log('Concurrency 5:', Date.now() - start, 'ms'); // ~400ms
  
  // Concurrency 2
  start = Date.now();
  await promisePool(createTasks(), 2);
  console.log('Concurrency 2:', Date.now() - start, 'ms'); // ~1000ms
  
  // Sequential (concurrency 1)
  start = Date.now();
  await promisePool(createTasks(), 1);
  console.log('Sequential (1):', Date.now() - start, 'ms'); // ~2000ms
}

setTimeout(() => benchmarkConcurrency(), 3000);
```

**Interview Tips:**
- Concurrency limit controls max simultaneous promises
- Use worker pool pattern: workers pull from queue
- Key insight: start new task when one completes
- Different from batching: dynamic vs fixed grouping
- Track active promises, start new when one finishes
- Common approaches: worker pool, semaphore, queue
- Worker pool best: create N workers that process queue
- Index variable shared across workers ensures order preservation
- Benefits: prevent overwhelming resources, rate limiting, memory control
- Applications: API calls, file I/O, CPU-intensive tasks, web scraping
- Concurrency vs Parallelism: concurrency limits tasks, parallelism is hardware
- Total time ≈ (total tasks / concurrency) * average task time
- Error handling: decide if one failure stops all or continues
- Can combine with: retries, rate limiting, priorities, progress tracking
- Popular libraries: p-limit, p-queue, bottleneck
- Edge cases: empty array, concurrency > tasks, concurrency = 1, all fail
- Clarify: error handling? retries? rate limiting? priorities?
- Follow-ups: add rate limiting, implement priority queue, add backpressure

</details>

83. Create a function to cache async function results (memoization)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Async Memoization**
```javascript
/**
 * Simple async memoization
 * Time Complexity: O(1) for cached results, O(n) for new calls
 * Space Complexity: O(n) where n is number of unique inputs
 */
function memoizeAsync(fn) {
  const cache = new Map();
  
  return async function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Cache hit:', key);
      return cache.get(key);
    }
    
    console.log('Cache miss, computing:', key);
    const result = await fn(...args);
    cache.set(key, result);
    
    return result;
  };
}

// Test
console.log('=== Basic Async Memoization ===');

const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

const expensiveAsync = async (x, y) => {
  await delay(1000);
  return x + y;
};

(async () => {
  const memoized = memoizeAsync(expensiveAsync);
  
  console.log('First call (5, 3):');
  console.log('Result:', await memoized(5, 3)); // Takes 1s
  
  console.log('\nSecond call (5, 3):');
  console.log('Result:', await memoized(5, 3)); // Instant (cached)
  
  console.log('\nThird call (2, 4):');
  console.log('Result:', await memoized(2, 4)); // Takes 1s (new args)
})();
```

### **Approach 2: Handle Concurrent Calls**
```javascript
/**
 * Prevent duplicate calls for same arguments
 * If multiple calls made while first is pending, reuse same promise
 */
function memoizeAsyncWithPending(fn) {
  const cache = new Map();
  const pending = new Map();
  
  return async function(...args) {
    const key = JSON.stringify(args);
    
    // Return cached result
    if (cache.has(key)) {
      console.log('Cache hit:', key);
      return cache.get(key);
    }
    
    // Return pending promise
    if (pending.has(key)) {
      console.log('Returning pending promise:', key);
      return pending.get(key);
    }
    
    console.log('New call:', key);
    
    // Create new promise
    const promise = fn(...args).then(result => {
      cache.set(key, result);
      pending.delete(key);
      return result;
    }).catch(error => {
      pending.delete(key);
      throw error;
    });
    
    pending.set(key, promise);
    return promise;
  };
}

// Test
console.log('\n=== Handle Concurrent Calls ===');

let callCount = 0;
const slowFunction = async (x) => {
  callCount++;
  console.log(`Actual call #${callCount} for x=${x}`);
  await delay(1000);
  return x * 2;
};

(async () => {
  const memoized = memoizeAsyncWithPending(slowFunction);
  
  // Make 3 concurrent calls with same argument
  const [r1, r2, r3] = await Promise.all([
    memoized(5),
    memoized(5),
    memoized(5)
  ]);
  
  console.log('Results:', r1, r2, r3); // Only 1 actual call!
})();
```

### **Approach 3: With TTL (Time To Live)**
```javascript
/**
 * Memoization with cache expiration
 */
function memoizeAsyncWithTTL(fn, ttl = 60000) {
  const cache = new Map();
  
  return async function(...args) {
    const key = JSON.stringify(args);
    const now = Date.now();
    
    if (cache.has(key)) {
      const { value, timestamp } = cache.get(key);
      
      if (now - timestamp < ttl) {
        console.log('Cache hit (fresh):', key);
        return value;
      } else {
        console.log('Cache expired:', key);
        cache.delete(key);
      }
    }
    
    console.log('Cache miss:', key);
    const result = await fn(...args);
    
    cache.set(key, {
      value: result,
      timestamp: now
    });
    
    return result;
  };
}

// Test
console.log('\n=== With TTL ===');

const apiCall = async (id) => {
  await delay(500);
  return { id, data: `Data for ${id}`, timestamp: Date.now() };
};

(async () => {
  const memoized = memoizeAsyncWithTTL(apiCall, 2000); // 2 second TTL
  
  console.log('Call 1:');
  const r1 = await memoized(123);
  console.log('Result:', r1);
  
  await delay(500);
  
  console.log('\nCall 2 (within TTL):');
  const r2 = await memoized(123);
  console.log('Result:', r2);
  
  await delay(2000);
  
  console.log('\nCall 3 (after TTL):');
  const r3 = await memoized(123);
  console.log('Result:', r3);
})();
```

### **Approach 4: With Max Cache Size (LRU)**
```javascript
/**
 * LRU cache for async functions
 */
function memoizeAsyncLRU(fn, maxSize = 100) {
  const cache = new Map();
  
  return async function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('LRU cache hit:', key);
      // Move to end (most recently used)
      const value = cache.get(key);
      cache.delete(key);
      cache.set(key, value);
      return value;
    }
    
    console.log('LRU cache miss:', key);
    const result = await fn(...args);
    
    // Evict oldest if at capacity
    if (cache.size >= maxSize) {
      const oldestKey = cache.keys().next().value;
      console.log('Evicting oldest:', oldestKey);
      cache.delete(oldestKey);
    }
    
    cache.set(key, result);
    return result;
  };
}

// Test
console.log('\n=== LRU Cache ===');

(async () => {
  const memoized = memoizeAsyncLRU(async (x) => {
    await delay(100);
    return x * 2;
  }, 3); // Max 3 items
  
  await memoized(1);
  await memoized(2);
  await memoized(3);
  await memoized(4); // Evicts 1
  
  console.log('\nAccessing 1 again (should be cache miss):');
  await memoized(1);
})();
```

### **Approach 5: With Custom Key Generator**
```javascript
/**
 * Memoization with custom key generation
 */
function memoizeAsyncCustomKey(fn, keyGenerator) {
  const cache = new Map();
  
  return async function(...args) {
    const key = keyGenerator ? keyGenerator(...args) : JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Custom key hit:', key);
      return cache.get(key);
    }
    
    console.log('Custom key miss:', key);
    const result = await fn(...args);
    cache.set(key, result);
    
    return result;
  };
}

// Test
console.log('\n=== Custom Key Generator ===');

const fetchUser = async (user) => {
  await delay(500);
  return { ...user, fetchedAt: Date.now() };
};

(async () => {
  // Use user.id as key instead of whole object
  const memoized = memoizeAsyncCustomKey(
    fetchUser,
    (user) => user.id
  );
  
  const user1 = { id: 123, name: 'Alice' };
  const user2 = { id: 123, name: 'Alice (different instance)' };
  
  await memoized(user1);
  await memoized(user2); // Cache hit! Same id
})();
```

### **Bonus: Memoization Class with Features**
```javascript
/**
 * Full-featured memoization class
 */
class AsyncMemoizer {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.cache = new Map();
    this.pending = new Map();
    this.options = {
      ttl: options.ttl || Infinity,
      maxSize: options.maxSize || Infinity,
      keyGenerator: options.keyGenerator || JSON.stringify,
      onCacheHit: options.onCacheHit || (() => {}),
      onCacheMiss: options.onCacheMiss || (() => {})
    };
  }
  
  async execute(...args) {
    const key = this.options.keyGenerator(args);
    
    // Check cache
    if (this.cache.has(key)) {
      const { value, timestamp } = this.cache.get(key);
      
      if (Date.now() - timestamp < this.options.ttl) {
        this.options.onCacheHit(key);
        return value;
      } else {
        this.cache.delete(key);
      }
    }
    
    // Check pending
    if (this.pending.has(key)) {
      return this.pending.get(key);
    }
    
    this.options.onCacheMiss(key);
    
    // Execute
    const promise = this.fn(...args).then(result => {
      this.cache.set(key, {
        value: result,
        timestamp: Date.now()
      });
      this.pending.delete(key);
      
      // LRU eviction
      if (this.cache.size > this.options.maxSize) {
        const oldestKey = this.cache.keys().next().value;
        this.cache.delete(oldestKey);
      }
      
      return result;
    }).catch(error => {
      this.pending.delete(key);
      throw error;
    });
    
    this.pending.set(key, promise);
    return promise;
  }
  
  clear() {
    this.cache.clear();
    this.pending.clear();
  }
  
  delete(...args) {
    const key = this.options.keyGenerator(args);
    this.cache.delete(key);
  }
  
  getStats() {
    return {
      cacheSize: this.cache.size,
      pendingSize: this.pending.size
    };
  }
}

// Test
console.log('\n=== Memoization Class ===');

(async () => {
  const memoizer = new AsyncMemoizer(
    async (x) => {
      await delay(500);
      return x * 2;
    },
    {
      ttl: 3000,
      maxSize: 5,
      onCacheHit: (key) => console.log('Hit:', key),
      onCacheMiss: (key) => console.log('Miss:', key)
    }
  );
  
  await memoizer.execute(5);
  await memoizer.execute(5);
  
  console.log('Stats:', memoizer.getStats());
  
  memoizer.clear();
  console.log('After clear:', memoizer.getStats());
})();
```

### **Bonus: Selective Caching**
```javascript
/**
 * Cache only successful results
 */
function memoizeAsyncSelective(fn, shouldCache) {
  const cache = new Map();
  
  return async function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = await fn(...args);
    
    if (!shouldCache || shouldCache(result, args)) {
      cache.set(key, result);
    }
    
    return result;
  };
}

// Test
console.log('\n=== Selective Caching ===');

(async () => {
  const api = async (id) => {
    await delay(200);
    return id > 0 ? { id, data: 'success' } : { id, error: 'invalid' };
  };
  
  // Only cache successful results
  const memoized = memoizeAsyncSelective(
    api,
    (result) => !result.error
  );
  
  await memoized(5); // Cached
  await memoized(-1); // Not cached
  await memoized(5); // Cache hit
  await memoized(-1); // Called again
})();
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. API response caching
const cachedFetch = memoizeAsyncWithTTL(
  async (url) => {
    const response = await fetch(url);
    return response.json();
  },
  300000 // 5 minutes
);

// 2. Database query caching
const getUserById = memoizeAsyncWithPending(
  async (userId) => {
    return database.query('SELECT * FROM users WHERE id = ?', [userId]);
  }
);

// 3. Expensive computation caching
const complexCalculation = memoizeAsync(
  async (input) => {
    // Heavy computation
    await delay(5000);
    return performComplexMath(input);
  }
);

// 4. File system operations
const readFileCache = memoizeAsyncWithTTL(
  async (filePath) => {
    return fs.promises.readFile(filePath, 'utf8');
  },
  60000 // 1 minute
);

console.log('\nUse Cases:');
console.log('cachedFetch - reduce API calls');
console.log('getUserById - avoid duplicate DB queries');
console.log('complexCalculation - cache expensive operations');
console.log('readFileCache - reduce file I/O');
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// No arguments
(async () => {
  const noArgs = memoizeAsync(async () => {
    await delay(100);
    return Math.random();
  });
  
  const r1 = await noArgs();
  const r2 = await noArgs();
  console.log('No args - same result?', r1 === r2); // true
})();

// Rejected promises
(async () => {
  let attempts = 0;
  const failing = memoizeAsync(async () => {
    attempts++;
    await delay(100);
    throw new Error(`Attempt ${attempts}`);
  });
  
  try {
    await failing();
  } catch (e) {
    console.log('First error:', e.message);
  }
  
  // Should NOT cache errors
  try {
    await failing();
  } catch (e) {
    console.log('Second error:', e.message);
  }
})();

// Object arguments
(async () => {
  const withObject = memoizeAsync(async (obj) => {
    await delay(100);
    return obj.value * 2;
  });
  
  await withObject({ value: 5 });
  await withObject({ value: 5 }); // Cache hit (JSON.stringify makes same key)
})();

// Array arguments
(async () => {
  const withArray = memoizeAsync(async (arr) => {
    await delay(100);
    return arr.reduce((a, b) => a + b, 0);
  });
  
  await withArray([1, 2, 3]);
  await withArray([1, 2, 3]); // Cache hit
})();
```

### **Performance Comparison**
```javascript
/**
 * Measure cache effectiveness
 */
async function benchmarkMemoization() {
  console.log('\n=== Memoization Benchmarks ===');
  
  const expensive = async (x) => {
    await delay(100);
    return x * 2;
  };
  
  // Without memoization
  let start = Date.now();
  await Promise.all([
    expensive(5),
    expensive(5),
    expensive(5),
    expensive(5),
    expensive(5)
  ]);
  console.log('Without cache (5 calls):', Date.now() - start, 'ms'); // ~100ms
  
  // With memoization
  const memoized = memoizeAsyncWithPending(expensive);
  start = Date.now();
  await Promise.all([
    memoized(5),
    memoized(5),
    memoized(5),
    memoized(5),
    memoized(5)
  ]);
  console.log('With cache (5 calls, same arg):', Date.now() - start, 'ms'); // ~100ms (1 call)
}

setTimeout(() => benchmarkMemoization(), 4000);
```

**Interview Tips:**
- Memoization: cache function results to avoid recalculation
- For async: must handle promises, concurrent calls, errors
- Basic: Map with JSON.stringify(args) as key
- Handle pending: prevent duplicate concurrent calls
- TTL: expire cache after time period
- LRU: limit cache size, evict least recently used
- Custom key: for complex objects, use specific properties
- Don't cache errors (usually) - allow retry
- Key generation critical: must be deterministic and unique
- Benefits: reduce API calls, database queries, expensive computations
- Trade-offs: memory usage vs speed, stale data risk
- Popular libraries: memoizee, p-memoize, mem
- Applications: API responses, DB queries, file reads, calculations
- Edge cases: no args, objects, arrays, errors, concurrent calls
- Clarify: TTL needed? max cache size? error handling? key generation?
- Follow-ups: implement cache warming, add statistics, distributed cache

</details>

84. Implement a polling function that checks a condition repeatedly

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Polling with Interval**
```javascript
/**
 * Simple polling that checks condition at intervals
 * Time Complexity: O(1) per check
 * Space Complexity: O(1)
 * 
 * Polls a condition function until it returns true or timeout
 */
async function poll(conditionFn, interval = 1000, timeout = 30000) {
  const startTime = Date.now();
  
  return new Promise((resolve, reject) => {
    const checkCondition = async () => {
      try {
        const result = await conditionFn();
        
        if (result) {
          console.log('Condition met!');
          resolve(result);
        } else if (Date.now() - startTime >= timeout) {
          reject(new Error('Polling timeout'));
        } else {
          console.log('Condition not met, retrying...');
          setTimeout(checkCondition, interval);
        }
      } catch (error) {
        reject(error);
      }
    };
    
    checkCondition();
  });
}

// Test
console.log('=== Basic Polling ===');

(async () => {
  let attempts = 0;
  
  const condition = async () => {
    attempts++;
    console.log(`Attempt ${attempts}`);
    // Succeed after 3 attempts
    return attempts >= 3;
  };
  
  try {
    await poll(condition, 500, 5000);
    console.log('Success after', attempts, 'attempts');
  } catch (error) {
    console.error('Failed:', error.message);
  }
})();
```

### **Approach 2: With Exponential Backoff**
```javascript
/**
 * Polling with increasing intervals (exponential backoff)
 */
async function pollWithBackoff(conditionFn, options = {}) {
  const {
    initialInterval = 1000,
    maxInterval = 30000,
    multiplier = 2,
    timeout = 60000
  } = options;
  
  const startTime = Date.now();
  let currentInterval = initialInterval;
  
  return new Promise((resolve, reject) => {
    const checkCondition = async () => {
      try {
        const result = await conditionFn();
        
        if (result) {
          console.log('Condition met!');
          resolve(result);
          return;
        }
        
        if (Date.now() - startTime >= timeout) {
          reject(new Error('Polling timeout'));
          return;
        }
        
        console.log(`Retrying in ${currentInterval}ms...`);
        setTimeout(checkCondition, currentInterval);
        
        // Increase interval (exponential backoff)
        currentInterval = Math.min(currentInterval * multiplier, maxInterval);
      } catch (error) {
        reject(error);
      }
    };
    
    checkCondition();
  });
}

// Test
console.log('\n=== With Exponential Backoff ===');

(async () => {
  let attempts = 0;
  
  const condition = async () => {
    attempts++;
    console.log(`Attempt ${attempts} at ${new Date().toLocaleTimeString()}`);
    return attempts >= 4;
  };
  
  await pollWithBackoff(condition, {
    initialInterval: 500,
    multiplier: 2,
    maxInterval: 5000,
    timeout: 30000
  });
  
  console.log('Success with backoff');
})();
```

### **Approach 3: With Validation and Callbacks**
```javascript
/**
 * Advanced polling with lifecycle callbacks
 */
async function pollAdvanced(conditionFn, options = {}) {
  const {
    interval = 1000,
    timeout = 30000,
    validate = null,
    onAttempt = null,
    onSuccess = null,
    onError = null,
    onTimeout = null
  } = options;
  
  const startTime = Date.now();
  let attemptCount = 0;
  
  return new Promise((resolve, reject) => {
    const checkCondition = async () => {
      attemptCount++;
      
      try {
        if (onAttempt) {
          onAttempt(attemptCount, Date.now() - startTime);
        }
        
        const result = await conditionFn();
        
        // Validate result if validator provided
        const isValid = validate ? validate(result) : result;
        
        if (isValid) {
          if (onSuccess) {
            onSuccess(result, attemptCount);
          }
          resolve(result);
          return;
        }
        
        if (Date.now() - startTime >= timeout) {
          const error = new Error(`Polling timeout after ${attemptCount} attempts`);
          if (onTimeout) {
            onTimeout(attemptCount);
          }
          reject(error);
          return;
        }
        
        setTimeout(checkCondition, interval);
      } catch (error) {
        if (onError) {
          onError(error, attemptCount);
        }
        reject(error);
      }
    };
    
    checkCondition();
  });
}

// Test
console.log('\n=== Advanced Polling with Callbacks ===');

(async () => {
  let value = 0;
  
  const condition = async () => {
    value += 10;
    return value;
  };
  
  try {
    const result = await pollAdvanced(condition, {
      interval: 300,
      timeout: 5000,
      validate: (val) => val >= 50,
      onAttempt: (count, elapsed) => {
        console.log(`Attempt ${count} (${elapsed}ms elapsed)`);
      },
      onSuccess: (result, count) => {
        console.log(`Success! Value: ${result} after ${count} attempts`);
      }
    });
    
    console.log('Final result:', result);
  } catch (error) {
    console.error('Failed:', error.message);
  }
})();
```

### **Approach 4: Polling Class with Control**
```javascript
/**
 * Reusable polling class with start/stop control
 */
class Poller {
  constructor(conditionFn, options = {}) {
    this.conditionFn = conditionFn;
    this.interval = options.interval || 1000;
    this.timeout = options.timeout || 30000;
    this.onAttempt = options.onAttempt;
    this.onSuccess = options.onSuccess;
    this.onError = options.onError;
    
    this.isPolling = false;
    this.timeoutId = null;
    this.startTime = null;
    this.attemptCount = 0;
  }
  
  start() {
    if (this.isPolling) {
      throw new Error('Polling already in progress');
    }
    
    this.isPolling = true;
    this.startTime = Date.now();
    this.attemptCount = 0;
    
    return new Promise((resolve, reject) => {
      this.resolve = resolve;
      this.reject = reject;
      this.check();
    });
  }
  
  async check() {
    if (!this.isPolling) return;
    
    this.attemptCount++;
    
    try {
      if (this.onAttempt) {
        this.onAttempt(this.attemptCount);
      }
      
      const result = await this.conditionFn();
      
      if (result) {
        this.stop();
        if (this.onSuccess) {
          this.onSuccess(result, this.attemptCount);
        }
        this.resolve(result);
        return;
      }
      
      const elapsed = Date.now() - this.startTime;
      
      if (elapsed >= this.timeout) {
        this.stop();
        this.reject(new Error(`Timeout after ${this.attemptCount} attempts`));
        return;
      }
      
      this.timeoutId = setTimeout(() => this.check(), this.interval);
    } catch (error) {
      this.stop();
      if (this.onError) {
        this.onError(error, this.attemptCount);
      }
      this.reject(error);
    }
  }
  
  stop() {
    this.isPolling = false;
    if (this.timeoutId) {
      clearTimeout(this.timeoutId);
      this.timeoutId = null;
    }
  }
  
  getStatus() {
    return {
      isPolling: this.isPolling,
      attemptCount: this.attemptCount,
      elapsed: this.startTime ? Date.now() - this.startTime : 0
    };
  }
}

// Test
console.log('\n=== Polling Class ===');

(async () => {
  let count = 0;
  
  const poller = new Poller(
    async () => {
      count++;
      return count >= 5;
    },
    {
      interval: 300,
      timeout: 10000,
      onAttempt: (attempt) => {
        const status = poller.getStatus();
        console.log(`Attempt ${attempt}, elapsed: ${status.elapsed}ms`);
      }
    }
  );
  
  try {
    await poller.start();
    console.log('Poller succeeded!');
  } catch (error) {
    console.error('Poller failed:', error.message);
  }
})();
```

### **Approach 5: With Immediate First Check**
```javascript
/**
 * Poll with immediate first check (no initial delay)
 */
async function pollImmediate(conditionFn, interval = 1000, timeout = 30000) {
  const startTime = Date.now();
  
  const check = async () => {
    try {
      const result = await conditionFn();
      
      if (result) {
        return result;
      }
      
      if (Date.now() - startTime >= timeout) {
        throw new Error('Polling timeout');
      }
      
      // Wait before next check
      await new Promise(resolve => setTimeout(resolve, interval));
      
      // Recursive check
      return check();
    } catch (error) {
      throw error;
    }
  };
  
  return check();
}

// Test
console.log('\n=== Immediate First Check ===');

(async () => {
  let attempts = 0;
  
  const condition = async () => {
    attempts++;
    console.log(`Immediate check attempt ${attempts}`);
    return attempts >= 3;
  };
  
  const start = Date.now();
  await pollImmediate(condition, 400, 5000);
  console.log('Completed in:', Date.now() - start, 'ms');
})();
```

### **Bonus: Poll Until Stable**
```javascript
/**
 * Poll until value stabilizes (doesn't change for N checks)
 */
async function pollUntilStable(getFn, options = {}) {
  const {
    interval = 1000,
    timeout = 30000,
    stableChecks = 3
  } = options;
  
  const startTime = Date.now();
  let previousValue = Symbol('initial');
  let stableCount = 0;
  
  while (Date.now() - startTime < timeout) {
    const currentValue = await getFn();
    
    if (currentValue === previousValue) {
      stableCount++;
      console.log(`Stable check ${stableCount}/${stableChecks}`);
      
      if (stableCount >= stableChecks) {
        console.log('Value stabilized!');
        return currentValue;
      }
    } else {
      console.log('Value changed:', previousValue, '->', currentValue);
      stableCount = 0;
      previousValue = currentValue;
    }
    
    await new Promise(resolve => setTimeout(resolve, interval));
  }
  
  throw new Error('Timeout waiting for stable value');
}

// Test
console.log('\n=== Poll Until Stable ===');

(async () => {
  let value = 0;
  let attempts = 0;
  
  const getValue = async () => {
    attempts++;
    if (attempts <= 3) {
      value = Math.random();
    }
    // After attempt 3, value stays the same
    return value;
  };
  
  const result = await pollUntilStable(getValue, {
    interval: 300,
    stableChecks: 3
  });
  
  console.log('Stable value:', result);
})();
```

### **Bonus: Conditional Polling (Rate Aware)**
```javascript
/**
 * Poll with rate limiting and conditional checks
 */
async function pollConditional(conditionFn, options = {}) {
  const {
    interval = 1000,
    timeout = 30000,
    maxAttempts = Infinity,
    shouldContinue = () => true
  } = options;
  
  const startTime = Date.now();
  let attempts = 0;
  
  while (Date.now() - startTime < timeout && attempts < maxAttempts) {
    attempts++;
    
    // Check if should continue polling
    if (!shouldContinue(attempts)) {
      throw new Error('Polling stopped by shouldContinue');
    }
    
    const result = await conditionFn();
    
    if (result) {
      return { result, attempts, elapsed: Date.now() - startTime };
    }
    
    await new Promise(resolve => setTimeout(resolve, interval));
  }
  
  throw new Error(`Failed after ${attempts} attempts`);
}

// Test
console.log('\n=== Conditional Polling ===');

(async () => {
  let attempts = 0;
  
  try {
    const { result, attempts: totalAttempts } = await pollConditional(
      async () => {
        attempts++;
        return attempts >= 10; // Won't reach this
      },
      {
        interval: 200,
        maxAttempts: 5, // Stop after 5 attempts
        shouldContinue: (attempt) => {
          console.log(`Attempt ${attempt}`);
          return true;
        }
      }
    );
    console.log('Result:', result);
  } catch (error) {
    console.log('Stopped:', error.message);
  }
})();
```

### **Real-World Use Cases**
```javascript
/**
 * Practical polling scenarios
 */

// 1. Wait for API resource to be ready
async function waitForResource(resourceId) {
  return poll(
    async () => {
      const response = await fetch(`/api/resources/${resourceId}`);
      const data = await response.json();
      return data.status === 'ready' ? data : false;
    },
    2000, // Check every 2 seconds
    60000 // 1 minute timeout
  );
}

// 2. Wait for file upload to complete
async function waitForUpload(uploadId) {
  return pollAdvanced(
    async () => {
      const status = await checkUploadStatus(uploadId);
      return status;
    },
    {
      interval: 1000,
      timeout: 300000, // 5 minutes
      validate: (status) => status.state === 'completed',
      onAttempt: (count) => {
        console.log(`Checking upload status (attempt ${count})...`);
      }
    }
  );
}

// 3. Wait for DOM element to appear
async function waitForElement(selector, maxWait = 10000) {
  return poll(
    () => document.querySelector(selector),
    100, // Check every 100ms
    maxWait
  );
}

// 4. Wait for service health check
async function waitForServiceHealth(serviceUrl) {
  return pollWithBackoff(
    async () => {
      try {
        const response = await fetch(`${serviceUrl}/health`);
        return response.ok;
      } catch {
        return false;
      }
    },
    {
      initialInterval: 1000,
      multiplier: 2,
      maxInterval: 30000,
      timeout: 300000 // 5 minutes
    }
  );
}

// 5. Wait for async job completion
async function waitForJobCompletion(jobId) {
  const poller = new Poller(
    async () => {
      const job = await fetchJob(jobId);
      return job.status === 'completed' ? job : false;
    },
    {
      interval: 5000,
      timeout: 600000, // 10 minutes
      onAttempt: (attempt) => console.log(`Checking job ${jobId} (${attempt})...`)
    }
  );
  
  return poller.start();
}

console.log('\nReal-world use cases:');
console.log('1. waitForResource - poll API until ready');
console.log('2. waitForUpload - track file upload progress');
console.log('3. waitForElement - wait for DOM element');
console.log('4. waitForServiceHealth - service readiness check');
console.log('5. waitForJobCompletion - async job tracking');
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Immediate success
(async () => {
  console.log('Test: Immediate success');
  let attempts = 0;
  
  await poll(
    async () => {
      attempts++;
      return true; // Success immediately
    },
    1000,
    5000
  );
  
  console.log('Attempts:', attempts); // 1
})();

// Immediate timeout
(async () => {
  console.log('\nTest: Zero timeout');
  
  try {
    await poll(
      async () => false,
      1000,
      0 // Immediate timeout
    );
  } catch (error) {
    console.log('Caught timeout:', error.message);
  }
})();

// Condition throws error
(async () => {
  console.log('\nTest: Condition throws error');
  
  try {
    await poll(
      async () => {
        throw new Error('Condition error');
      },
      500,
      2000
    );
  } catch (error) {
    console.log('Caught error:', error.message);
  }
})();

// Synchronous condition
(async () => {
  console.log('\nTest: Synchronous condition');
  let count = 0;
  
  await poll(
    () => {
      count++;
      return count >= 3;
    },
    200,
    5000
  );
  
  console.log('Sync condition worked, count:', count);
})();
```

### **Performance Considerations**
```javascript
/**
 * Optimize polling performance
 */

// Avoid polling - use events when possible
class EventBasedWaiter {
  constructor() {
    this.listeners = new Map();
  }
  
  wait(eventName, timeout = 30000) {
    return new Promise((resolve, reject) => {
      const timeoutId = setTimeout(() => {
        this.listeners.delete(eventName);
        reject(new Error('Timeout'));
      }, timeout);
      
      this.listeners.set(eventName, (data) => {
        clearTimeout(timeoutId);
        resolve(data);
      });
    });
  }
  
  emit(eventName, data) {
    const listener = this.listeners.get(eventName);
    if (listener) {
      listener(data);
      this.listeners.delete(eventName);
    }
  }
}

// Compare polling vs events
console.log('\n=== Performance: Polling vs Events ===');

(async () => {
  // Polling approach
  let pollingValue = null;
  setTimeout(() => { pollingValue = 'ready'; }, 2000);
  
  const pollStart = Date.now();
  await poll(() => pollingValue === 'ready', 100, 5000);
  console.log('Polling took:', Date.now() - pollStart, 'ms');
  
  // Event approach
  const waiter = new EventBasedWaiter();
  setTimeout(() => waiter.emit('ready', 'ready'), 2000);
  
  const eventStart = Date.now();
  await waiter.wait('ready', 5000);
  console.log('Event took:', Date.now() - eventStart, 'ms');
  
  console.log('\nEvents are instant, polling has check interval overhead');
})();
```

**Interview Tips:**
- Polling: repeatedly check condition until success or timeout
- Key params: condition function, interval, timeout
- Basic pattern: check → if false, wait interval → repeat
- Always include timeout to prevent infinite loops
- Exponential backoff reduces server load for long waits
- First check can be immediate (no initial delay) or after first interval
- Return meaningful data: result, attempt count, elapsed time
- Handle errors in condition function - should stop or retry?
- Class-based allows start/stop control, useful for cleanup
- Consider rate limiting for API polling
- Validate results: condition true doesn't always mean success
- Applications: API status checks, job completion, DOM elements, service health
- Better alternatives: WebSockets, Server-Sent Events, webhooks when available
- Trade-offs: simple but inefficient, adds latency and load
- Edge cases: immediate success, immediate timeout, errors, sync conditions
- Clarify: interval strategy? backoff? max attempts? immediate check? callbacks?
- Follow-ups: implement backoff, add jitter, combine with events, distributed polling

</details>

85. Build a simple task queue with priority

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic FIFO Queue**
```javascript
/**
 * Simple First-In-First-Out task queue
 * Time Complexity: O(1) for enqueue, O(1) for dequeue
 * Space Complexity: O(n) where n is queue size
 */
class TaskQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.queue = [];
    this.running = 0;
  }
  
  async add(task) {
    return new Promise((resolve, reject) => {
      this.queue.push({ task, resolve, reject });
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { task, resolve, reject } = this.queue.shift();
    
    try {
      const result = await task();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process(); // Process next task
    }
  }
  
  size() {
    return this.queue.length;
  }
  
  pending() {
    return this.running;
  }
}

// Test
console.log('=== Basic FIFO Queue ===');

(async () => {
  const queue = new TaskQueue(2); // Concurrency of 2
  
  const delay = (ms, value) => new Promise(resolve => 
    setTimeout(() => {
      console.log(`Completed: ${value}`);
      resolve(value);
    }, ms)
  );
  
  console.log('Adding 5 tasks with concurrency 2...');
  
  const results = await Promise.all([
    queue.add(() => delay(1000, 'Task 1')),
    queue.add(() => delay(500, 'Task 2')),
    queue.add(() => delay(800, 'Task 3')),
    queue.add(() => delay(300, 'Task 4')),
    queue.add(() => delay(600, 'Task 5'))
  ]);
  
  console.log('All tasks completed:', results);
})();
```

### **Approach 2: Priority Queue**
```javascript
/**
 * Task queue with priority levels
 * Higher priority tasks execute first
 */
class PriorityTaskQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.queue = [];
    this.running = 0;
  }
  
  async add(task, priority = 0) {
    return new Promise((resolve, reject) => {
      const item = { task, priority, resolve, reject };
      
      // Insert in priority order (higher priority first)
      let inserted = false;
      for (let i = 0; i < this.queue.length; i++) {
        if (priority > this.queue[i].priority) {
          this.queue.splice(i, 0, item);
          inserted = true;
          break;
        }
      }
      
      if (!inserted) {
        this.queue.push(item);
      }
      
      console.log(`Added task with priority ${priority}`);
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { task, priority, resolve, reject } = this.queue.shift();
    
    console.log(`Executing task with priority ${priority}`);
    
    try {
      const result = await task();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process();
    }
  }
  
  size() {
    return this.queue.length;
  }
  
  clear() {
    this.queue = [];
  }
}

// Test
console.log('\n=== Priority Queue ===');

(async () => {
  const queue = new PriorityTaskQueue(1); // Process one at a time
  
  const delay = (ms, value) => new Promise(resolve => 
    setTimeout(() => {
      console.log(`Result: ${value}`);
      resolve(value);
    }, ms)
  );
  
  // Add tasks with different priorities
  queue.add(() => delay(100, 'Low priority (1)'), 1);
  queue.add(() => delay(100, 'High priority (10)'), 10);
  queue.add(() => delay(100, 'Medium priority (5)'), 5);
  queue.add(() => delay(100, 'Highest priority (20)'), 20);
  
  await new Promise(resolve => setTimeout(resolve, 1000));
  console.log('Execution order should be: 20, 10, 5, 1');
})();
```

### **Approach 3: With Named Priorities**
```javascript
/**
 * Queue with named priority levels
 */
class NamedPriorityQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.priorities = {
      LOW: 1,
      NORMAL: 5,
      HIGH: 10,
      URGENT: 20
    };
    this.queue = [];
    this.running = 0;
  }
  
  async add(task, priority = 'NORMAL') {
    const numericPriority = this.priorities[priority] || this.priorities.NORMAL;
    
    return new Promise((resolve, reject) => {
      const item = { task, priority: numericPriority, name: priority, resolve, reject };
      
      // Binary search insertion for better performance
      let left = 0;
      let right = this.queue.length;
      
      while (left < right) {
        const mid = Math.floor((left + right) / 2);
        if (this.queue[mid].priority < numericPriority) {
          right = mid;
        } else {
          left = mid + 1;
        }
      }
      
      this.queue.splice(left, 0, item);
      console.log(`Added ${priority} task (${numericPriority})`);
      
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const { task, name, resolve, reject } = this.queue.shift();
    
    console.log(`Executing ${name} task`);
    
    try {
      const result = await task();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process();
    }
  }
  
  getStats() {
    return {
      queued: this.queue.length,
      running: this.running,
      queueByPriority: this.queue.reduce((acc, item) => {
        acc[item.name] = (acc[item.name] || 0) + 1;
        return acc;
      }, {})
    };
  }
}

// Test
console.log('\n=== Named Priority Queue ===');

(async () => {
  const queue = new NamedPriorityQueue(2);
  
  const task = (name) => async () => {
    await new Promise(r => setTimeout(r, 200));
    return name;
  };
  
  queue.add(task('Task A'), 'LOW');
  queue.add(task('Task B'), 'URGENT');
  queue.add(task('Task C'), 'HIGH');
  queue.add(task('Task D'), 'NORMAL');
  queue.add(task('Task E'), 'URGENT');
  
  console.log('Stats:', queue.getStats());
  
  await new Promise(r => setTimeout(r, 1500));
})();
```

### **Approach 4: Advanced Queue with Features**
```javascript
/**
 * Full-featured task queue
 */
class AdvancedTaskQueue {
  constructor(options = {}) {
    this.concurrency = options.concurrency || 1;
    this.queue = [];
    this.running = 0;
    this.completed = 0;
    this.failed = 0;
    this.listeners = {
      start: [],
      success: [],
      error: [],
      complete: []
    };
  }
  
  on(event, callback) {
    if (this.listeners[event]) {
      this.listeners[event].push(callback);
    }
  }
  
  emit(event, data) {
    if (this.listeners[event]) {
      this.listeners[event].forEach(cb => cb(data));
    }
  }
  
  async add(task, options = {}) {
    const {
      priority = 0,
      retry = 0,
      timeout = null,
      name = 'unnamed'
    } = options;
    
    return new Promise((resolve, reject) => {
      const item = {
        task,
        priority,
        retry,
        timeout,
        name,
        resolve,
        reject,
        attempts: 0
      };
      
      // Insert by priority
      let inserted = false;
      for (let i = 0; i < this.queue.length; i++) {
        if (priority > this.queue[i].priority) {
          this.queue.splice(i, 0, item);
          inserted = true;
          break;
        }
      }
      
      if (!inserted) {
        this.queue.push(item);
      }
      
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const item = this.queue.shift();
    
    this.emit('start', { name: item.name, priority: item.priority });
    
    try {
      item.attempts++;
      
      let result;
      if (item.timeout) {
        result = await this.withTimeout(item.task(), item.timeout);
      } else {
        result = await item.task();
      }
      
      this.completed++;
      this.emit('success', { name: item.name, result });
      item.resolve(result);
    } catch (error) {
      // Retry logic
      if (item.attempts <= item.retry) {
        console.log(`Retrying ${item.name} (${item.attempts}/${item.retry})`);
        this.queue.unshift(item); // Add back to front
      } else {
        this.failed++;
        this.emit('error', { name: item.name, error });
        item.reject(error);
      }
    } finally {
      this.running--;
      this.emit('complete', this.getStats());
      this.process();
    }
  }
  
  withTimeout(promise, ms) {
    return Promise.race([
      promise,
      new Promise((_, reject) => 
        setTimeout(() => reject(new Error('Task timeout')), ms)
      )
    ]);
  }
  
  getStats() {
    return {
      queued: this.queue.length,
      running: this.running,
      completed: this.completed,
      failed: this.failed
    };
  }
  
  pause() {
    this.paused = true;
  }
  
  resume() {
    this.paused = false;
    this.process();
  }
  
  clear() {
    this.queue = [];
  }
}

// Test
console.log('\n=== Advanced Queue ===');

(async () => {
  const queue = new AdvancedTaskQueue({ concurrency: 2 });
  
  queue.on('start', (data) => console.log(`Started: ${data.name}`));
  queue.on('success', (data) => console.log(`Success: ${data.name}`));
  queue.on('error', (data) => console.log(`Error: ${data.name}`));
  
  const task = (name, shouldFail = false) => async () => {
    await new Promise(r => setTimeout(r, 300));
    if (shouldFail) throw new Error(`${name} failed`);
    return `${name} result`;
  };
  
  await Promise.allSettled([
    queue.add(task('Task 1'), { priority: 5, name: 'Task 1' }),
    queue.add(task('Task 2', true), { priority: 10, retry: 2, name: 'Task 2' }),
    queue.add(task('Task 3'), { priority: 1, name: 'Task 3' })
  ]);
  
  console.log('Final stats:', queue.getStats());
})();
```

### **Approach 5: Using Heap for Optimal Priority**
```javascript
/**
 * Priority queue using min-heap for O(log n) insertions
 */
class HeapPriorityQueue {
  constructor(concurrency = 1) {
    this.concurrency = concurrency;
    this.heap = [];
    this.running = 0;
  }
  
  async add(task, priority = 0) {
    return new Promise((resolve, reject) => {
      const item = { task, priority, resolve, reject };
      this.heap.push(item);
      this.bubbleUp(this.heap.length - 1);
      this.process();
    });
  }
  
  bubbleUp(index) {
    while (index > 0) {
      const parentIndex = Math.floor((index - 1) / 2);
      
      // Max heap (higher priority = higher value)
      if (this.heap[index].priority <= this.heap[parentIndex].priority) {
        break;
      }
      
      [this.heap[index], this.heap[parentIndex]] = 
        [this.heap[parentIndex], this.heap[index]];
      
      index = parentIndex;
    }
  }
  
  extractMax() {
    if (this.heap.length === 0) return null;
    if (this.heap.length === 1) return this.heap.pop();
    
    const max = this.heap[0];
    this.heap[0] = this.heap.pop();
    this.bubbleDown(0);
    
    return max;
  }
  
  bubbleDown(index) {
    while (true) {
      const leftChild = 2 * index + 1;
      const rightChild = 2 * index + 2;
      let largest = index;
      
      if (leftChild < this.heap.length && 
          this.heap[leftChild].priority > this.heap[largest].priority) {
        largest = leftChild;
      }
      
      if (rightChild < this.heap.length && 
          this.heap[rightChild].priority > this.heap[largest].priority) {
        largest = rightChild;
      }
      
      if (largest === index) break;
      
      [this.heap[index], this.heap[largest]] = 
        [this.heap[largest], this.heap[index]];
      
      index = largest;
    }
  }
  
  async process() {
    if (this.running >= this.concurrency || this.heap.length === 0) {
      return;
    }
    
    this.running++;
    const { task, priority, resolve, reject } = this.extractMax();
    
    console.log(`Processing priority ${priority}`);
    
    try {
      const result = await task();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process();
    }
  }
}

// Test
console.log('\n=== Heap Priority Queue ===');

(async () => {
  const queue = new HeapPriorityQueue(1);
  
  const task = (value) => async () => {
    await new Promise(r => setTimeout(r, 100));
    console.log(`Executed: ${value}`);
    return value;
  };
  
  // Add in random order
  queue.add(task('P3'), 3);
  queue.add(task('P8'), 8);
  queue.add(task('P1'), 1);
  queue.add(task('P5'), 5);
  queue.add(task('P10'), 10);
  
  await new Promise(r => setTimeout(r, 1000));
  console.log('Should execute in order: 10, 8, 5, 3, 1');
})();
```

### **Bonus: Rate-Limited Queue**
```javascript
/**
 * Task queue with rate limiting
 */
class RateLimitedQueue {
  constructor(options = {}) {
    this.concurrency = options.concurrency || 1;
    this.rateLimit = options.rateLimit || null; // { requests, per (ms) }
    this.queue = [];
    this.running = 0;
    this.requestTimestamps = [];
  }
  
  async add(task, priority = 0) {
    return new Promise((resolve, reject) => {
      this.queue.push({ task, priority, resolve, reject });
      this.queue.sort((a, b) => b.priority - a.priority);
      this.process();
    });
  }
  
  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }
    
    // Check rate limit
    if (this.rateLimit) {
      const now = Date.now();
      const windowStart = now - this.rateLimit.per;
      
      // Remove old timestamps
      this.requestTimestamps = this.requestTimestamps.filter(t => t > windowStart);
      
      if (this.requestTimestamps.length >= this.rateLimit.requests) {
        const oldestRequest = this.requestTimestamps[0];
        const waitTime = this.rateLimit.per - (now - oldestRequest);
        console.log(`Rate limit reached, waiting ${waitTime}ms...`);
        setTimeout(() => this.process(), waitTime);
        return;
      }
      
      this.requestTimestamps.push(now);
    }
    
    this.running++;
    const { task, resolve, reject } = this.queue.shift();
    
    try {
      const result = await task();
      resolve(result);
    } catch (error) {
      reject(error);
    } finally {
      this.running--;
      this.process();
    }
  }
}

// Test
console.log('\n=== Rate-Limited Queue ===');

(async () => {
  const queue = new RateLimitedQueue({
    concurrency: 5,
    rateLimit: { requests: 3, per: 1000 } // 3 requests per second
  });
  
  const task = (id) => async () => {
    console.log(`Task ${id} at ${new Date().toLocaleTimeString()}`);
    await new Promise(r => setTimeout(r, 100));
    return id;
  };
  
  // Add 10 tasks
  for (let i = 1; i <= 10; i++) {
    queue.add(task(i));
  }
  
  await new Promise(r => setTimeout(r, 5000));
})();
```

### **Real-World Use Cases**
```javascript
/**
 * Practical queue applications
 */

// 1. Background job processing
const jobQueue = new PriorityTaskQueue(3);

async function processJob(job) {
  return jobQueue.add(
    async () => {
      console.log(`Processing job ${job.id}`);
      await performWork(job);
      return { jobId: job.id, status: 'completed' };
    },
    job.priority
  );
}

// 2. API request batching
const apiQueue = new RateLimitedQueue({
  concurrency: 5,
  rateLimit: { requests: 100, per: 60000 } // 100 requests per minute
});

async function makeApiCall(endpoint, data) {
  return apiQueue.add(async () => {
    return fetch(endpoint, { method: 'POST', body: JSON.stringify(data) });
  });
}

// 3. Image processing pipeline
const imageQueue = new AdvancedTaskQueue({ concurrency: 4 });

async function processImage(image, priority) {
  return imageQueue.add(
    async () => {
      const resized = await resize(image);
      const optimized = await optimize(resized);
      await save(optimized);
      return optimized;
    },
    { priority, retry: 2, timeout: 30000, name: image.filename }
  );
}

// 4. Email sending queue
const emailQueue = new PriorityTaskQueue(2);

async function sendEmail(email, priority = 'NORMAL') {
  const priorities = { LOW: 1, NORMAL: 5, HIGH: 10, URGENT: 20 };
  
  return emailQueue.add(
    async () => {
      await emailService.send(email);
      return { sent: true, messageId: email.id };
    },
    priorities[priority]
  );
}

console.log('\nReal-world use cases:');
console.log('1. processJob - background job processing');
console.log('2. makeApiCall - rate-limited API requests');
console.log('3. processImage - concurrent image processing');
console.log('4. sendEmail - priority email queue');
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty queue
const emptyQueue = new TaskQueue(1);
console.log('Empty queue size:', emptyQueue.size()); // 0

// Single task
(async () => {
  const singleQueue = new TaskQueue(1);
  const result = await singleQueue.add(async () => 'single');
  console.log('Single task result:', result);
})();

// Task that throws error
(async () => {
  const errorQueue = new TaskQueue(1);
  try {
    await errorQueue.add(async () => {
      throw new Error('Task failed');
    });
  } catch (error) {
    console.log('Caught task error:', error.message);
  }
})();

// Same priority tasks (FIFO within priority)
(async () => {
  const fifoQueue = new PriorityTaskQueue(1);
  
  const results = [];
  fifoQueue.add(async () => { results.push('A'); }, 5);
  fifoQueue.add(async () => { results.push('B'); }, 5);
  fifoQueue.add(async () => { results.push('C'); }, 5);
  
  await new Promise(r => setTimeout(r, 500));
  console.log('FIFO within priority:', results); // ['A', 'B', 'C']
})();
```

### **Performance Comparison**
```javascript
/**
 * Compare different queue implementations
 */
async function benchmarkQueues() {
  console.log('\n=== Queue Benchmarks ===');
  
  const taskCount = 100;
  const task = () => async () => {
    await new Promise(r => setTimeout(r, 10));
    return 'done';
  };
  
  // Basic queue
  let start = Date.now();
  const basicQueue = new TaskQueue(10);
  await Promise.all(Array.from({ length: taskCount }, () => 
    basicQueue.add(task())
  ));
  console.log('Basic queue (100 tasks, concurrency 10):', Date.now() - start, 'ms');
  
  // Priority queue (with sorting overhead)
  start = Date.now();
  const priorityQueue = new PriorityTaskQueue(10);
  await Promise.all(Array.from({ length: taskCount }, (_, i) => 
    priorityQueue.add(task(), Math.random() * 10)
  ));
  console.log('Priority queue (100 tasks, random priorities):', Date.now() - start, 'ms');
  
  // Heap queue (optimal for many priority changes)
  start = Date.now();
  const heapQueue = new HeapPriorityQueue(10);
  await Promise.all(Array.from({ length: taskCount }, (_, i) => 
    heapQueue.add(task(), Math.random() * 10)
  ));
  console.log('Heap queue (100 tasks, random priorities):', Date.now() - start, 'ms');
}

setTimeout(() => benchmarkQueues(), 6000);
```

**Interview Tips:**
- Task queue: manages async task execution with concurrency control
- Basic queue: FIFO (first in, first out), process tasks in order
- Priority queue: higher priority tasks execute first
- Key operations: add (enqueue), process (dequeue), size, stats
- Concurrency: limit simultaneous task execution
- Priority implementation: array with sorting (simple) or heap (optimal O(log n))
- Heap best for frequent priority insertions (O(log n) vs O(n))
- Handle task errors: catch and continue processing queue
- Features: retry logic, timeouts, events, pause/resume, stats
- Rate limiting: prevent overwhelming external services
- Applications: job processing, API calls, image processing, email sending
- Trade-offs: memory for queued tasks, complexity vs features
- Named priorities easier to use: LOW, NORMAL, HIGH, URGENT
- Events useful for monitoring: start, success, error, complete
- Edge cases: empty queue, errors, same priorities, zero concurrency
- Clarify: priority levels? retry logic? rate limits? events? max queue size?
- Follow-ups: add persistence, distributed queue, dead letter queue, scheduling

</details>

### **Advanced Functions**

86. Implement function currying

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Currying**
```javascript
/**
 * Simple curry implementation
 * Time Complexity: O(1) per call
 * Space Complexity: O(n) where n is number of arguments
 * 
 * Transforms f(a, b, c) into f(a)(b)(c)
 */
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...nextArgs) {
        return curried.apply(this, args.concat(nextArgs));
      };
    }
  };
}

// Test
console.log('=== Basic Currying ===');

function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6
console.log(curriedAdd(1, 2, 3)); // 6

function multiply(a, b, c, d) {
  return a * b * c * d;
}

const curriedMultiply = curry(multiply);
console.log(curriedMultiply(2)(3)(4)(5)); // 120
console.log(curriedMultiply(2, 3)(4)(5)); // 120
console.log(curriedMultiply(2)(3, 4, 5)); // 120
```

### **Approach 2: Infinite Currying**
```javascript
/**
 * Curry that accepts unlimited arguments
 * Keeps accepting until explicitly converted
 */
function infiniteCurry(fn) {
  const accumulated = [];
  
  function curried(...args) {
    accumulated.push(...args);
    return curried;
  }
  
  curried.valueOf = function() {
    return fn(...accumulated);
  };
  
  curried.toString = function() {
    return fn(...accumulated).toString();
  };
  
  return curried;
}

// Test
console.log('\n=== Infinite Currying ===');

const sum = (...args) => args.reduce((a, b) => a + b, 0);
const infiniteSum = infiniteCurry(sum);

console.log(+infiniteSum(1)(2)(3)(4)(5)); // 15
console.log(+infiniteSum(10)(20)(30)); // 60

// Another approach: sum with terminator
function infiniteSum2(a) {
  let total = a;
  
  function add(b) {
    if (b === undefined) {
      return total;
    }
    total += b;
    return add;
  }
  
  return add;
}

console.log(infiniteSum2(1)(2)(3)(4)()); // 10
console.log(infiniteSum2(5)(10)()); // 15
```

### **Approach 3: Placeholder Support**
```javascript
/**
 * Currying with placeholder support
 * Allows skipping arguments using placeholder
 */
function curryWithPlaceholder(fn, placeholder = curry.placeholder) {
  return function curried(...args) {
    // Check if we have enough non-placeholder arguments
    const nonPlaceholderCount = args.filter(arg => arg !== placeholder).length;
    
    if (nonPlaceholderCount >= fn.length && !args.includes(placeholder)) {
      return fn.apply(this, args);
    }
    
    return function(...nextArgs) {
      // Merge args, replacing placeholders with new args
      const mergedArgs = args.map(arg => 
        arg === placeholder && nextArgs.length ? nextArgs.shift() : arg
      );
      
      // Add remaining nextArgs
      return curried.apply(this, [...mergedArgs, ...nextArgs]);
    };
  };
}

curryWithPlaceholder.placeholder = Symbol('placeholder');
const _ = curryWithPlaceholder.placeholder;

// Test
console.log('\n=== Placeholder Support ===');

function subtract(a, b, c) {
  return a - b - c;
}

const curriedSubtract = curryWithPlaceholder(subtract);

console.log(curriedSubtract(10)(5)(2)); // 3
console.log(curriedSubtract(_, 5)(10)(2)); // 3 (10 - 5 - 2)
console.log(curriedSubtract(_, _, 2)(10)(5)); // 3 (10 - 5 - 2)
console.log(curriedSubtract(10, _, 2)(5)); // 3

function greet(greeting, name, punctuation) {
  return `${greeting}, ${name}${punctuation}`;
}

const curriedGreet = curryWithPlaceholder(greet);
console.log(curriedGreet('Hello')(_, '!')('World')); // "Hello, World!"
console.log(curriedGreet(_, 'Alice')('Hi')('.')); // "Hi, Alice."
```

### **Approach 4: Partial Application**
```javascript
/**
 * Partial application (similar to currying but more flexible)
 */
function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

// Advanced partial with placeholder
function partialWithPlaceholder(fn, ...presetArgs) {
  const _ = partialWithPlaceholder.placeholder;
  
  return function(...laterArgs) {
    const args = presetArgs.map(arg => 
      arg === _ && laterArgs.length ? laterArgs.shift() : arg
    );
    
    return fn(...args, ...laterArgs);
  };
}

partialWithPlaceholder.placeholder = Symbol('_');
const __ = partialWithPlaceholder.placeholder;

// Test
console.log('\n=== Partial Application ===');

function divide(a, b, c) {
  return a / b / c;
}

const divideBy2 = partial(divide, __, 2);
console.log(divideBy2(100, 5)); // 100 / 2 / 5 = 10

const divideBy2And5 = partial(divide, __, 2, 5);
console.log(divideBy2And5(100)); // 100 / 2 / 5 = 10

function log(level, timestamp, message) {
  return `[${level}] ${timestamp}: ${message}`;
}

const logError = partial(log, 'ERROR');
console.log(logError('2024-01-01', 'System failure'));

const logWithTime = partial(log, __, '2024-01-01');
console.log(logWithTime('INFO', 'System started'));
```

### **Approach 5: Auto-Currying with Variable Arguments**
```javascript
/**
 * Smart curry that handles variable argument functions
 */
function autoCurry(fn, arity = fn.length) {
  return function curried(...args) {
    if (args.length >= arity) {
      return fn.apply(this, args);
    }
    
    return function(...nextArgs) {
      return curried.apply(this, [...args, ...nextArgs]);
    };
  };
}

// For functions with rest parameters
function flexibleCurry(fn, minArgs) {
  return function curried(...args) {
    if (args.length >= minArgs) {
      return fn(...args);
    }
    
    return (...nextArgs) => curried(...args, ...nextArgs);
  };
}

// Test
console.log('\n=== Auto-Currying ===');

// Fixed arity
function sum3(a, b, c) {
  return a + b + c;
}

const curriedSum = autoCurry(sum3);
console.log(curriedSum(1)(2)(3)); // 6

// Variable arity
function sumAll(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

const curriedSumAll = flexibleCurry(sumAll, 3);
console.log(curriedSumAll(1)(2)(3)); // 6
console.log(curriedSumAll(1, 2)(3, 4, 5)); // 15

// With explicit arity
function multiply(...args) {
  return args.reduce((a, b) => a * b, 1);
}

const multiply4 = autoCurry(multiply, 4);
console.log(multiply4(2)(3)(4)(5)); // 120
```

### **Bonus: Curry Right (Right-to-Left)**
```javascript
/**
 * Curry from right to left
 */
function curryRight(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    
    return function(...nextArgs) {
      return curried.apply(this, [...nextArgs, ...args]);
    };
  };
}

// Test
console.log('\n=== Curry Right ===');

function describe(adj1, adj2, noun) {
  return `${adj1} ${adj2} ${noun}`;
}

const curriedDescribe = curryRight(describe);

// Arguments fill from right
console.log(curriedDescribe('cat')('lazy')('old')); // "old lazy cat"
console.log(curriedDescribe('dog', 'brown')('happy')); // "happy brown dog"

// Compare with regular curry
const normalCurried = curry(describe);
console.log(normalCurried('old')('lazy')('cat')); // "old lazy cat"
```

### **Bonus: Named Parameters Curry**
```javascript
/**
 * Curry with named parameters
 */
function curriedFunction(config) {
  const required = Object.keys(config);
  const values = {};
  
  function curried(params = {}) {
    Object.assign(values, params);
    
    const hasAll = required.every(key => key in values);
    
    if (hasAll) {
      return config.fn(values);
    }
    
    return curried;
  }
  
  return curried;
}

// Test
console.log('\n=== Named Parameters ===');

const createUser = curriedFunction({
  fn: ({ name, email, age }) => ({ name, email, age }),
  name: true,
  email: true,
  age: true
});

const withName = createUser({ name: 'Alice' });
const withEmail = withName({ email: 'alice@example.com' });
const user = withEmail({ age: 30 });

console.log('User:', user);

// Can provide multiple at once
const user2 = createUser({ name: 'Bob', age: 25 })({ email: 'bob@example.com' });
console.log('User 2:', user2);
```

### **Real-World Use Cases**
```javascript
/**
 * Practical curry applications
 */

// 1. Reusable validators
function validate(predicate, message, value) {
  return predicate(value) ? { valid: true, value } : { valid: false, message };
}

const curriedValidate = curry(validate);

const isRequired = curriedValidate(
  value => value != null && value !== '',
  'This field is required'
);

const isEmail = curriedValidate(
  value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
  'Invalid email format'
);

const minLength = (min) => curriedValidate(
  value => value.length >= min,
  `Minimum length is ${min}`
);

console.log('\n=== Validators ===');
console.log(isRequired('hello')); // { valid: true, value: 'hello' }
console.log(isRequired('')); // { valid: false, message: ... }
console.log(isEmail('test@example.com')); // { valid: true, ... }
console.log(minLength(5)('hi')); // { valid: false, ... }

// 2. API request builder
function makeRequest(method, baseUrl, endpoint, params) {
  const url = `${baseUrl}${endpoint}?${new URLSearchParams(params)}`;
  return { method, url };
}

const curriedRequest = curry(makeRequest);
const getRequest = curriedRequest('GET');
const apiGet = getRequest('https://api.example.com');

console.log('\n=== API Builder ===');
console.log(apiGet('/users')({ page: 1, limit: 10 }));
console.log(apiGet('/posts')({ category: 'tech' }));

// 3. Logger with levels
function log(level, timestamp, component, message) {
  return `[${level}] ${timestamp} [${component}]: ${message}`;
}

const curriedLog = curry(log);
const errorLog = curriedLog('ERROR');
const errorLogNow = errorLog(new Date().toISOString());
const authErrorLog = errorLogNow('Auth');

console.log('\n=== Logger ===');
console.log(authErrorLog('Login failed'));
console.log(authErrorLog('Invalid token'));

// 4. Data transformation pipeline
function map(fn, array) {
  return array.map(fn);
}

function filter(predicate, array) {
  return array.filter(predicate);
}

const curriedMap = curry(map);
const curriedFilter = curry(filter);

const double = curriedMap(x => x * 2);
const onlyEven = curriedFilter(x => x % 2 === 0);

console.log('\n=== Data Pipeline ===');
const numbers = [1, 2, 3, 4, 5];
console.log('Original:', numbers);
console.log('Doubled:', double(numbers));
console.log('Even only:', onlyEven(numbers));
console.log('Even then doubled:', double(onlyEven(numbers)));

// 5. HTML builder
function createElement(tag, attrs, children) {
  const attrStr = Object.entries(attrs)
    .map(([k, v]) => `${k}="${v}"`)
    .join(' ');
  return `<${tag} ${attrStr}>${children}</${tag}>`;
}

const curriedElement = curry(createElement);
const div = curriedElement('div');
const button = curriedElement('button');

const primaryButton = button({ class: 'btn-primary' });
const dangerButton = button({ class: 'btn-danger' });

console.log('\n=== HTML Builder ===');
console.log(primaryButton('Click Me'));
console.log(dangerButton('Delete'));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Zero arguments
function noArgs() {
  return 'no arguments';
}

const curriedNoArgs = curry(noArgs);
console.log('No args:', curriedNoArgs()); // "no arguments"

// Single argument
function single(a) {
  return a * 2;
}

const curriedSingle = curry(single);
console.log('Single arg:', curriedSingle(5)); // 10

// With 'this' context
const obj = {
  multiplier: 10,
  multiply: function(a, b) {
    return this.multiplier * a * b;
  }
};

const curriedMethod = curry(obj.multiply);
const boundMethod = curriedMethod.bind(obj);
console.log('With context:', boundMethod(2)(3)); // 60

// Extra arguments ignored
function twoArgs(a, b) {
  return a + b;
}

const curriedTwo = curry(twoArgs);
console.log('Extra args:', curriedTwo(1, 2, 3, 4)); // 3 (extra ignored)

// Curried function as argument
function apply(fn, value) {
  return fn(value);
}

const curriedApply = curry(apply);
const applyDouble = curriedApply(x => x * 2);
console.log('Higher-order:', applyDouble(5)); // 10
```

### **Performance Comparison**
```javascript
/**
 * Measure curry overhead
 */
console.log('\n=== Performance ===');

function regularAdd(a, b, c) {
  return a + b + c;
}

const curriedAddPerf = curry(regularAdd);

// Regular function
console.time('Regular (1000 calls)');
for (let i = 0; i < 1000; i++) {
  regularAdd(1, 2, 3);
}
console.timeEnd('Regular (1000 calls)');

// Curried function (all at once)
console.time('Curried all args (1000 calls)');
for (let i = 0; i < 1000; i++) {
  curriedAddPerf(1, 2, 3);
}
console.timeEnd('Curried all args (1000 calls)');

// Curried function (one by one)
console.time('Curried step by step (1000 calls)');
for (let i = 0; i < 1000; i++) {
  curriedAddPerf(1)(2)(3);
}
console.timeEnd('Curried step by step (1000 calls)');

console.log('\nNote: Currying adds overhead but provides flexibility');
```

### **Advanced: Curry with Memoization**
```javascript
/**
 * Combine curry with memoization
 */
function curriedMemoize(fn) {
  const cache = new Map();
  
  return function curried(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log('Cache hit:', key);
      return cache.get(key);
    }
    
    if (args.length >= fn.length) {
      const result = fn.apply(this, args);
      cache.set(key, result);
      return result;
    }
    
    return function(...nextArgs) {
      return curried.apply(this, [...args, ...nextArgs]);
    };
  };
}

// Test
console.log('\n=== Curry with Memoization ===');

function expensiveAdd(a, b, c) {
  console.log('Computing...');
  return a + b + c;
}

const memoizedCurry = curriedMemoize(expensiveAdd);

console.log(memoizedCurry(1)(2)(3)); // Computing... 6
console.log(memoizedCurry(1, 2)(3)); // Cache hit: [1,2,3]
console.log(memoizedCurry(1, 2, 3)); // Cache hit: [1,2,3]
```

**Interview Tips:**
- Currying: transform f(a, b, c) into f(a)(b)(c)
- Enables partial application: fix some arguments, get function for rest
- Check if enough arguments: args.length >= fn.length
- If not enough: return function that accepts more
- If enough: call original function with all arguments
- Recursive approach: keep accepting until satisfied
- Use Function.length to get expected parameter count
- Spread operator (...) to collect and merge arguments
- Preserve 'this' context with apply/call
- Benefits: reusability, composition, point-free style
- Difference from partial: curry always returns unary, partial can take multiple
- Placeholder support: skip arguments using special symbol
- Auto-currying: can provide multiple args at once f(a, b)(c) or f(a)(b)(c)
- Applications: validators, loggers, API builders, data transformations
- Trade-offs: flexibility vs performance overhead
- Popular libraries: lodash.curry, ramda.curry
- Edge cases: no args, single arg, extra args, 'this' binding, rest parameters
- Clarify: fixed arity? placeholder support? right-to-left? memoization?
- Follow-ups: implement uncurry, curry with default values, async curry

</details>

87. Implement function composition (compose and pipe)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Compose (Right-to-Left)**
```javascript
/**
 * Compose functions from right to left
 * compose(f, g, h)(x) = f(g(h(x)))
 * Time Complexity: O(n) where n is number of functions
 * Space Complexity: O(1)
 */
function compose(...fns) {
  return function(value) {
    return fns.reduceRight((acc, fn) => fn(acc), value);
  };
}

// Test
console.log('=== Basic Compose (Right-to-Left) ===');

const add5 = x => x + 5;
const multiply3 = x => x * 3;
const subtract2 = x => x - 2;

const composed = compose(subtract2, multiply3, add5);

console.log('Input: 10');
console.log('Steps: 10 -> add5 -> 15 -> multiply3 -> 45 -> subtract2 -> 43');
console.log('Result:', composed(10)); // 43

// Another example
const toUpper = str => str.toUpperCase();
const exclaim = str => str + '!';
const greet = name => `Hello, ${name}`;

const shout = compose(exclaim, toUpper, greet);
console.log(shout('alice')); // "HELLO, ALICE!"
```

### **Approach 2: Pipe (Left-to-Right)**
```javascript
/**
 * Pipe functions from left to right
 * pipe(f, g, h)(x) = h(g(f(x)))
 */
function pipe(...fns) {
  return function(value) {
    return fns.reduce((acc, fn) => fn(acc), value);
  };
}

// Test
console.log('\n=== Pipe (Left-to-Right) ===');

const piped = pipe(add5, multiply3, subtract2);

console.log('Input: 10');
console.log('Steps: 10 -> add5 -> 15 -> multiply3 -> 45 -> subtract2 -> 43');
console.log('Result:', piped(10)); // 43

// More readable flow
const processName = pipe(
  name => name.trim(),
  name => name.toLowerCase(),
  name => name.charAt(0).toUpperCase() + name.slice(1)
);

console.log(processName('  ALICE  ')); // "Alice"
console.log(processName('  bOb  ')); // "Bob"
```

### **Approach 3: Async Compose and Pipe**
```javascript
/**
 * Compose and pipe for async functions
 */
function composeAsync(...fns) {
  return async function(value) {
    return fns.reduceRight(
      async (acc, fn) => fn(await acc),
      value
    );
  };
}

function pipeAsync(...fns) {
  return async function(value) {
    return fns.reduce(
      async (acc, fn) => fn(await acc),
      value
    );
  };
}

// Test
console.log('\n=== Async Composition ===');

const delay = (ms) => new Promise(resolve => setTimeout(resolve, ms));

const asyncAdd5 = async (x) => {
  await delay(100);
  console.log('Adding 5:', x, '->', x + 5);
  return x + 5;
};

const asyncMultiply3 = async (x) => {
  await delay(100);
  console.log('Multiplying by 3:', x, '->', x * 3);
  return x * 3;
};

const asyncSubtract2 = async (x) => {
  await delay(100);
  console.log('Subtracting 2:', x, '->', x - 2);
  return x - 2;
};

(async () => {
  const asyncPiped = pipeAsync(asyncAdd5, asyncMultiply3, asyncSubtract2);
  const result = await asyncPiped(10);
  console.log('Async result:', result); // 43
})();
```

### **Approach 4: Compose with Multiple Arguments**
```javascript
/**
 * Compose that handles multiple arguments for first function
 */
function composeMulti(...fns) {
  return function(...args) {
    const [first, ...rest] = fns.reverse();
    const firstResult = first(...args);
    
    return rest.reduce((acc, fn) => fn(acc), firstResult);
  };
}

// Test
console.log('\n=== Compose with Multiple Args ===');

const sum = (a, b, c) => a + b + c;
const double = x => x * 2;
const addTen = x => x + 10;

const calculate = composeMulti(addTen, double, sum);

console.log('calculate(2, 3, 5)');
console.log('Steps: sum(2,3,5) -> 10 -> double -> 20 -> addTen -> 30');
console.log('Result:', calculate(2, 3, 5)); // 30

// Format and log
const formatName = (first, last) => `${first} ${last}`;
const uppercase = str => str.toUpperCase();
const wrap = str => `[${str}]`;

const formatAndWrap = composeMulti(wrap, uppercase, formatName);
console.log(formatAndWrap('john', 'doe')); // "[JOHN DOE]"
```

### **Approach 5: Point-Free Style Utilities**
```javascript
/**
 * Helper functions for point-free composition
 */
function map(fn) {
  return array => array.map(fn);
}

function filter(predicate) {
  return array => array.filter(predicate);
}

function reduce(fn, initial) {
  return array => array.reduce(fn, initial);
}

function prop(key) {
  return obj => obj[key];
}

function trace(label) {
  return value => {
    console.log(`${label}:`, value);
    return value;
  };
}

// Test
console.log('\n=== Point-Free Style ===');

const users = [
  { name: 'Alice', age: 30, active: true },
  { name: 'Bob', age: 25, active: false },
  { name: 'Charlie', age: 35, active: true },
  { name: 'David', age: 28, active: true }
];

const getActiveUserNames = pipe(
  trace('Input'),
  filter(user => user.active),
  trace('After filter'),
  map(prop('name')),
  trace('After map'),
  map(name => name.toUpperCase())
);

console.log('Result:', getActiveUserNames(users));

// Math pipeline
const numbers = [1, 2, 3, 4, 5];

const sumOfDoubledEvens = pipe(
  filter(x => x % 2 === 0),
  map(x => x * 2),
  reduce((a, b) => a + b, 0)
);

console.log('Sum of doubled evens:', sumOfDoubledEvens(numbers)); // 12
```

### **Bonus: Compose with Error Handling**
```javascript
/**
 * Composition with try-catch for each function
 */
function composeSafe(...fns) {
  return function(value) {
    return fns.reduceRight((acc, fn, index) => {
      try {
        return fn(acc);
      } catch (error) {
        console.error(`Error in function ${fns.length - index}:`, error.message);
        throw error;
      }
    }, value);
  };
}

// Test
console.log('\n=== Safe Composition ===');

const divide10 = x => 10 / x;
const addOne = x => x + 1;
const maybeError = x => {
  if (x > 5) throw new Error('Value too large');
  return x;
};

const safeCalc = composeSafe(maybeError, divide10, addOne);

try {
  console.log(safeCalc(1)); // Works: (1+1) = 2, 10/2 = 5, 5 ok
} catch (e) {
  console.log('Caught:', e.message);
}

try {
  console.log(safeCalc(0)); // Error: 10/1 = 10, too large
} catch (e) {
  console.log('Caught:', e.message);
}
```

### **Bonus: Transducers**
```javascript
/**
 * Advanced: Transducers for efficient composition
 */
function transduce(xform, reducer, initial, collection) {
  const transformedReducer = xform(reducer);
  return collection.reduce(transformedReducer, initial);
}

const mapTransducer = (fn) => (reducer) => (acc, value) => 
  reducer(acc, fn(value));

const filterTransducer = (predicate) => (reducer) => (acc, value) =>
  predicate(value) ? reducer(acc, value) : acc;

// Test
console.log('\n=== Transducers ===');

const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Compose transducers
const xform = compose(
  mapTransducer(x => x * 2),
  filterTransducer(x => x > 10)
);

const result = transduce(
  xform,
  (acc, value) => [...acc, value],
  [],
  data
);

console.log('Transducer result:', result); // [12, 14, 16, 18, 20]

// Without transducers (less efficient - multiple iterations)
const traditional = data
  .map(x => x * 2)
  .filter(x => x > 10);

console.log('Traditional result:', traditional);
console.log('Transducers iterate once, traditional iterates twice');
```

### **Real-World Use Cases**
```javascript
/**
 * Practical composition examples
 */

// 1. Data validation pipeline
const isString = val => typeof val === 'string';
const isNotEmpty = val => val.trim().length > 0;
const hasValidLength = val => val.length >= 3 && val.length <= 20;
const hasNoSpecialChars = val => /^[a-zA-Z0-9]+$/.test(val);

const validate = (checks) => (value) => {
  for (const check of checks) {
    if (!check(value)) return false;
  }
  return true;
};

const isValidUsername = validate([
  isString,
  isNotEmpty,
  hasValidLength,
  hasNoSpecialChars
]);

console.log('\n=== Validation Pipeline ===');
console.log('alice123:', isValidUsername('alice123')); // true
console.log('ab:', isValidUsername('ab')); // false (too short)
console.log('alice@123:', isValidUsername('alice@123')); // false (special char)

// 2. Data transformation pipeline
const parseJSON = str => JSON.parse(str);
const extractUsers = data => data.users;
const sortByAge = users => [...users].sort((a, b) => a.age - b.age);
const formatUsers = users => users.map(u => `${u.name} (${u.age})`);

const processUserData = pipe(
  parseJSON,
  extractUsers,
  sortByAge,
  formatUsers
);

const jsonData = '{"users":[{"name":"Alice","age":30},{"name":"Bob","age":25}]}';
console.log('\n=== Data Pipeline ===');
console.log(processUserData(jsonData));

// 3. Middleware pattern
const logRequest = (req) => {
  console.log('Request:', req.method, req.url);
  return req;
};

const authenticate = (req) => {
  req.user = req.headers.auth ? { id: 1, name: 'User' } : null;
  return req;
};

const authorize = (req) => {
  if (!req.user) throw new Error('Unauthorized');
  return req;
};

const handleRequest = (req) => {
  return { status: 200, data: 'Success' };
};

const middleware = pipe(
  logRequest,
  authenticate,
  authorize,
  handleRequest
);

console.log('\n=== Middleware ===');
try {
  const response = middleware({
    method: 'GET',
    url: '/api/users',
    headers: { auth: 'token123' }
  });
  console.log('Response:', response);
} catch (e) {
  console.log('Error:', e.message);
}

// 4. String manipulation
const removeSpaces = str => str.replace(/\s+/g, '');
const toLowerCase = str => str.toLowerCase();
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1);
const addPrefix = prefix => str => `${prefix}${str}`;

const slugify = pipe(
  toLowerCase,
  str => str.replace(/\s+/g, '-'),
  str => str.replace(/[^a-z0-9-]/g, '')
);

console.log('\n=== String Processing ===');
console.log(slugify('Hello World! 123')); // "hello-world-123"

const formatTitle = pipe(
  toLowerCase,
  str => str.split(' '),
  words => words.map(capitalize),
  words => words.join(' ')
);

console.log(formatTitle('hello world from javascript')); // "Hello World From Javascript"

// 5. Promise chain
const fetchUser = (id) => Promise.resolve({ id, name: 'User' + id });
const fetchPosts = (user) => Promise.resolve({ ...user, posts: ['Post1', 'Post2'] });
const fetchComments = (data) => Promise.resolve({ ...data, comments: ['Comment1'] });

const getUserWithData = pipeAsync(
  fetchUser,
  fetchPosts,
  fetchComments
);

console.log('\n=== Promise Chain ===');
getUserWithData(123).then(result => {
  console.log('User data:', result);
});
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty composition
const emptyCompose = compose();
console.log('Empty compose:', emptyCompose(5)); // 5

const emptyPipe = pipe();
console.log('Empty pipe:', emptyPipe(5)); // 5

// Single function
const singleCompose = compose(x => x * 2);
console.log('Single compose:', singleCompose(5)); // 10

// Identity function
const identity = x => x;
const withIdentity = compose(identity, x => x * 2, identity);
console.log('With identity:', withIdentity(5)); // 10

// Composition of compositions
const composed1 = compose(x => x + 1, x => x * 2);
const composed2 = compose(x => x - 3, x => x / 2);
const superCompose = compose(composed2, composed1);
console.log('Composed compositions:', superCompose(10)); // ((10*2)+1)/2-3 = 7.5

// Type changes through pipeline
const toNumber = pipe(
  str => str.trim(),
  str => parseInt(str, 10),
  num => num * 2,
  num => num.toString()
);

console.log('Type changes:', toNumber('  42  ')); // "84"
```

### **Performance Considerations**
```javascript
/**
 * Optimize composition
 */
console.log('\n=== Performance ===');

// Manual vs composed
function manual(x) {
  return subtract2(multiply3(add5(x)));
}

const composed2 = compose(subtract2, multiply3, add5);

console.time('Manual (10000 calls)');
for (let i = 0; i < 10000; i++) {
  manual(10);
}
console.timeEnd('Manual (10000 calls)');

console.time('Composed (10000 calls)');
for (let i = 0; i < 10000; i++) {
  composed2(10);
}
console.timeEnd('Composed (10000 calls)');

console.log('Composition adds minimal overhead for readability benefits');
```

**Interview Tips:**
- Composition: combine functions to create new functions
- Compose: right-to-left execution (like math: f(g(x)))
- Pipe: left-to-right execution (more intuitive for data flow)
- Implementation: use reduce/reduceRight with initial value
- Each function takes output of previous as input
- Unary functions (one argument) compose best
- For multiple args: only first function needs to accept them
- Benefits: reusability, testability, readability, declarative style
- Point-free style: define functions without mentioning arguments
- Async composition: use async/await with reduce
- Error handling: wrap each function in try-catch or use Either monad
- Transducers: efficient composition avoiding multiple iterations
- Applications: validation, data transformation, middleware, string processing
- Compose vs pipe: same behavior, different order (preference)
- Works well with: map, filter, reduce (array methods)
- Debugging: add trace function to log intermediate values
- Edge cases: empty composition (identity), single function, type changes
- Clarify: compose or pipe? async support? error handling? multiple args?
- Follow-ups: implement flow (alias for pipe), add tap for side effects, partial composition

</details>

88. Implement a deep equality check function

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Deep Equality**
```javascript
/**
 * Deep equality check for objects and arrays
 * Time Complexity: O(n) where n is total number of properties
 * Space Complexity: O(d) where d is depth (recursion stack)
 */
function deepEqual(a, b) {
  // Same reference or both primitives equal
  if (a === b) return true;
  
  // If either is null or not an object
  if (a == null || b == null || typeof a !== 'object' || typeof b !== 'object') {
    return false;
  }
  
  // Get keys
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  
  // Different number of keys
  if (keysA.length !== keysB.length) return false;
  
  // Check each key
  for (const key of keysA) {
    if (!keysB.includes(key)) return false;
    if (!deepEqual(a[key], b[key])) return false;
  }
  
  return true;
}

// Test
console.log('=== Basic Deep Equality ===');

console.log(deepEqual(1, 1)); // true
console.log(deepEqual('hello', 'hello')); // true
console.log(deepEqual(null, null)); // true

console.log(deepEqual({ a: 1 }, { a: 1 })); // true
console.log(deepEqual({ a: 1, b: 2 }, { b: 2, a: 1 })); // true

console.log(deepEqual([1, 2, 3], [1, 2, 3])); // true
console.log(deepEqual([1, [2, 3]], [1, [2, 3]])); // true

console.log(deepEqual({ a: 1 }, { a: 2 })); // false
console.log(deepEqual([1, 2], [1, 2, 3])); // false

const nested = { a: { b: { c: 1 } } };
console.log(deepEqual(nested, { a: { b: { c: 1 } } })); // true
```

### **Approach 2: Handle Special Cases**
```javascript
/**
 * Deep equality with special type handling
 */
function deepEqualAdvanced(a, b) {
  // Strict equality
  if (a === b) return true;
  
  // Handle NaN
  if (Number.isNaN(a) && Number.isNaN(b)) return true;
  
  // Handle Date objects
  if (a instanceof Date && b instanceof Date) {
    return a.getTime() === b.getTime();
  }
  
  // Handle RegExp
  if (a instanceof RegExp && b instanceof RegExp) {
    return a.source === b.source && a.flags === b.flags;
  }
  
  // Handle Arrays
  if (Array.isArray(a) && Array.isArray(b)) {
    if (a.length !== b.length) return false;
    
    for (let i = 0; i < a.length; i++) {
      if (!deepEqualAdvanced(a[i], b[i])) return false;
    }
    
    return true;
  }
  
  // Handle null or primitives
  if (a == null || b == null || typeof a !== 'object' || typeof b !== 'object') {
    return false;
  }
  
  // Handle objects
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  
  if (keysA.length !== keysB.length) return false;
  
  for (const key of keysA) {
    if (!keysB.includes(key)) return false;
    if (!deepEqualAdvanced(a[key], b[key])) return false;
  }
  
  return true;
}

// Test
console.log('\n=== Advanced Deep Equality ===');

// NaN
console.log('NaN vs NaN:', deepEqualAdvanced(NaN, NaN)); // true

// Dates
const date1 = new Date('2024-01-01');
const date2 = new Date('2024-01-01');
const date3 = new Date('2024-01-02');
console.log('Same dates:', deepEqualAdvanced(date1, date2)); // true
console.log('Different dates:', deepEqualAdvanced(date1, date3)); // false

// RegExp
const regex1 = /test/gi;
const regex2 = /test/gi;
const regex3 = /test/i;
console.log('Same regex:', deepEqualAdvanced(regex1, regex2)); // true
console.log('Different flags:', deepEqualAdvanced(regex1, regex3)); // false

// Mixed
const obj1 = {
  date: new Date('2024-01-01'),
  regex: /test/i,
  array: [1, 2, 3]
};
const obj2 = {
  date: new Date('2024-01-01'),
  regex: /test/i,
  array: [1, 2, 3]
};
console.log('Complex object:', deepEqualAdvanced(obj1, obj2)); // true
```

### **Approach 3: With Circular Reference Detection**
```javascript
/**
 * Deep equality that handles circular references
 */
function deepEqualCircular(a, b, seenA = new WeakMap(), seenB = new WeakMap()) {
  // Strict equality
  if (a === b) return true;
  
  // Handle primitives and null
  if (a == null || b == null || typeof a !== 'object' || typeof b !== 'object') {
    return a === b;
  }
  
  // Check for circular references
  if (seenA.has(a)) {
    return seenA.get(a) === b;
  }
  if (seenB.has(b)) {
    return seenB.get(b) === a;
  }
  
  // Mark as seen
  seenA.set(a, b);
  seenB.set(b, a);
  
  // Handle Arrays
  if (Array.isArray(a) && Array.isArray(b)) {
    if (a.length !== b.length) return false;
    
    for (let i = 0; i < a.length; i++) {
      if (!deepEqualCircular(a[i], b[i], seenA, seenB)) return false;
    }
    
    return true;
  }
  
  // Handle objects
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  
  if (keysA.length !== keysB.length) return false;
  
  for (const key of keysA) {
    if (!keysB.includes(key)) return false;
    if (!deepEqualCircular(a[key], b[key], seenA, seenB)) return false;
  }
  
  return true;
}

// Test
console.log('\n=== Circular Reference Handling ===');

// Create circular references
const circular1 = { name: 'obj1' };
circular1.self = circular1;

const circular2 = { name: 'obj1' };
circular2.self = circular2;

console.log('Circular objects:', deepEqualCircular(circular1, circular2)); // true

// Different circular structures
const circular3 = { name: 'obj3' };
circular3.self = circular3;

const circular4 = { name: 'obj4' };
circular4.self = circular4;

console.log('Different circular:', deepEqualCircular(circular3, circular4)); // false

// Complex circular
const a = { x: 1 };
const b = { x: 1 };
a.ref = a;
b.ref = b;

console.log('Complex circular:', deepEqualCircular(a, b)); // true
```

### **Approach 4: Complete Implementation with All Types**
```javascript
/**
 * Comprehensive deep equality check
 */
function isDeepEqual(a, b, seen = new WeakMap()) {
  // Strict equality (handles primitives, same reference)
  if (Object.is(a, b)) return true;
  
  // Different types
  if (typeof a !== typeof b) return false;
  
  // Both are primitives but not equal
  if (typeof a !== 'object' || a === null || b === null) {
    return false;
  }
  
  // Check constructors
  if (a.constructor !== b.constructor) return false;
  
  // Handle Date
  if (a instanceof Date) {
    return a.getTime() === b.getTime();
  }
  
  // Handle RegExp
  if (a instanceof RegExp) {
    return a.source === b.source && a.flags === b.flags;
  }
  
  // Handle Map
  if (a instanceof Map) {
    if (a.size !== b.size) return false;
    
    for (const [key, value] of a) {
      if (!b.has(key) || !isDeepEqual(value, b.get(key), seen)) {
        return false;
      }
    }
    
    return true;
  }
  
  // Handle Set
  if (a instanceof Set) {
    if (a.size !== b.size) return false;
    
    for (const value of a) {
      if (!b.has(value)) return false;
    }
    
    return true;
  }
  
  // Handle ArrayBuffer
  if (a instanceof ArrayBuffer) {
    if (a.byteLength !== b.byteLength) return false;
    const viewA = new Uint8Array(a);
    const viewB = new Uint8Array(b);
    
    for (let i = 0; i < viewA.length; i++) {
      if (viewA[i] !== viewB[i]) return false;
    }
    
    return true;
  }
  
  // Handle TypedArrays
  if (ArrayBuffer.isView(a) && ArrayBuffer.isView(b)) {
    if (a.length !== b.length) return false;
    
    for (let i = 0; i < a.length; i++) {
      if (a[i] !== b[i]) return false;
    }
    
    return true;
  }
  
  // Circular reference check
  if (seen.has(a)) {
    return seen.get(a) === b;
  }
  seen.set(a, b);
  
  // Handle Arrays
  if (Array.isArray(a)) {
    if (a.length !== b.length) return false;
    
    for (let i = 0; i < a.length; i++) {
      if (!isDeepEqual(a[i], b[i], seen)) return false;
    }
    
    return true;
  }
  
  // Handle plain objects
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  
  if (keysA.length !== keysB.length) return false;
  
  // Check own properties
  for (const key of keysA) {
    if (!Object.prototype.hasOwnProperty.call(b, key)) return false;
    if (!isDeepEqual(a[key], b[key], seen)) return false;
  }
  
  return true;
}

// Test
console.log('\n=== Complete Implementation ===');

// Map
const map1 = new Map([['a', 1], ['b', 2]]);
const map2 = new Map([['a', 1], ['b', 2]]);
const map3 = new Map([['b', 2], ['a', 1]]);
console.log('Maps equal:', isDeepEqual(map1, map2)); // true
console.log('Maps different order:', isDeepEqual(map1, map3)); // true

// Set
const set1 = new Set([1, 2, 3]);
const set2 = new Set([1, 2, 3]);
const set3 = new Set([1, 2, 4]);
console.log('Sets equal:', isDeepEqual(set1, set2)); // true
console.log('Sets different:', isDeepEqual(set1, set3)); // false

// TypedArray
const arr1 = new Uint8Array([1, 2, 3]);
const arr2 = new Uint8Array([1, 2, 3]);
const arr3 = new Uint8Array([1, 2, 4]);
console.log('TypedArrays equal:', isDeepEqual(arr1, arr2)); // true
console.log('TypedArrays different:', isDeepEqual(arr1, arr3)); // false

// Complex nested
const complex1 = {
  date: new Date('2024-01-01'),
  regex: /test/gi,
  map: new Map([['key', 'value']]),
  set: new Set([1, 2, 3]),
  nested: { a: 1, b: [2, 3, { c: 4 }] }
};

const complex2 = {
  date: new Date('2024-01-01'),
  regex: /test/gi,
  map: new Map([['key', 'value']]),
  set: new Set([1, 2, 3]),
  nested: { a: 1, b: [2, 3, { c: 4 }] }
};

console.log('Complex nested:', isDeepEqual(complex1, complex2)); // true
```

### **Approach 5: Optimized with Early Exit**
```javascript
/**
 * Optimized version with early exits
 */
function deepEqualOptimized(a, b) {
  // Fast path: same reference
  if (a === b) return true;
  
  // Fast path: different types
  const typeA = typeof a;
  const typeB = typeof b;
  if (typeA !== typeB) return false;
  
  // Fast path: primitives
  if (typeA !== 'object') return false;
  
  // Fast path: null
  if (a === null || b === null) return a === b;
  
  // Fast path: different constructors
  const ctorA = a.constructor;
  const ctorB = b.constructor;
  if (ctorA !== ctorB) return false;
  
  // Arrays
  if (Array.isArray(a)) {
    const len = a.length;
    if (len !== b.length) return false;
    
    for (let i = 0; i < len; i++) {
      if (!deepEqualOptimized(a[i], b[i])) return false;
    }
    
    return true;
  }
  
  // Date
  if (a instanceof Date) {
    return a.getTime() === b.getTime();
  }
  
  // RegExp
  if (a instanceof RegExp) {
    return a.toString() === b.toString();
  }
  
  // Objects
  const keysA = Object.keys(a);
  const len = keysA.length;
  
  // Fast path: different key counts
  if (len !== Object.keys(b).length) return false;
  
  // Check each property
  for (let i = 0; i < len; i++) {
    const key = keysA[i];
    if (!Object.prototype.hasOwnProperty.call(b, key)) return false;
    if (!deepEqualOptimized(a[key], b[key])) return false;
  }
  
  return true;
}

// Test
console.log('\n=== Optimized Deep Equality ===');

const large1 = {
  a: 1, b: 2, c: 3, d: 4, e: 5,
  nested: { x: 1, y: 2, z: 3 },
  array: [1, 2, 3, 4, 5]
};

const large2 = {
  a: 1, b: 2, c: 3, d: 4, e: 5,
  nested: { x: 1, y: 2, z: 3 },
  array: [1, 2, 3, 4, 5]
};

console.time('Optimized');
for (let i = 0; i < 10000; i++) {
  deepEqualOptimized(large1, large2);
}
console.timeEnd('Optimized');

console.log('Result:', deepEqualOptimized(large1, large2)); // true
```

### **Bonus: Shallow Equality**
```javascript
/**
 * Shallow equality (one level deep)
 */
function shallowEqual(a, b) {
  if (a === b) return true;
  
  if (typeof a !== 'object' || typeof b !== 'object' || a === null || b === null) {
    return false;
  }
  
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  
  if (keysA.length !== keysB.length) return false;
  
  for (const key of keysA) {
    if (a[key] !== b[key] || !Object.prototype.hasOwnProperty.call(b, key)) {
      return false;
    }
  }
  
  return true;
}

// Test
console.log('\n=== Shallow Equality ===');

console.log(shallowEqual({ a: 1 }, { a: 1 })); // true
console.log(shallowEqual({ a: {} }, { a: {} })); // false (different references)

const shared = { x: 1 };
console.log(shallowEqual({ a: shared }, { a: shared })); // true (same reference)
```

### **Bonus: Partial Equality (Subset Check)**
```javascript
/**
 * Check if b contains all properties of a (subset)
 */
function isSubset(subset, superset) {
  if (subset === superset) return true;
  
  if (typeof subset !== 'object' || typeof superset !== 'object' ||
      subset === null || superset === null) {
    return subset === superset;
  }
  
  for (const key of Object.keys(subset)) {
    if (!Object.prototype.hasOwnProperty.call(superset, key)) {
      return false;
    }
    
    if (!isSubset(subset[key], superset[key])) {
      return false;
    }
  }
  
  return true;
}

// Test
console.log('\n=== Subset Check ===');

const small = { a: 1, b: 2 };
const large = { a: 1, b: 2, c: 3 };

console.log('Is subset:', isSubset(small, large)); // true
console.log('Is subset (reverse):', isSubset(large, small)); // false

const nested1 = { a: { b: 1 } };
const nested2 = { a: { b: 1, c: 2 }, d: 3 };

console.log('Nested subset:', isSubset(nested1, nested2)); // true
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications of deep equality
 */

// 1. React shouldComponentUpdate
class MyComponent {
  shouldComponentUpdate(nextProps, nextState) {
    return !deepEqual(this.props, nextProps) || 
           !deepEqual(this.state, nextState);
  }
}

// 2. Memoization cache key comparison
const memoCache = new Map();

function memoizeDeep(fn) {
  return function(...args) {
    for (const [cachedArgs, result] of memoCache) {
      if (deepEqual(cachedArgs, args)) {
        console.log('Cache hit!');
        return result;
      }
    }
    
    console.log('Cache miss, computing...');
    const result = fn(...args);
    memoCache.set(args, result);
    return result;
  };
}

const compute = memoizeDeep((obj) => {
  return obj.a + obj.b;
});

console.log('\n=== Memoization ===');
console.log(compute({ a: 1, b: 2 })); // Cache miss
console.log(compute({ a: 1, b: 2 })); // Cache hit!

// 3. Testing/Assertions
function assertEqual(actual, expected, message = '') {
  if (!deepEqual(actual, expected)) {
    throw new Error(
      `Assertion failed: ${message}\n` +
      `Expected: ${JSON.stringify(expected)}\n` +
      `Actual: ${JSON.stringify(actual)}`
    );
  }
  console.log('✓ Test passed:', message);
}

console.log('\n=== Testing ===');
assertEqual({ a: 1, b: [2, 3] }, { a: 1, b: [2, 3] }, 'Objects equal');

// 4. Change detection
class ChangeDetector {
  constructor(data) {
    this.data = data;
    this.snapshot = JSON.parse(JSON.stringify(data));
  }
  
  hasChanged() {
    return !deepEqual(this.data, this.snapshot);
  }
  
  getChanges() {
    if (!this.hasChanged()) return null;
    
    return {
      before: this.snapshot,
      after: this.data
    };
  }
  
  commit() {
    this.snapshot = JSON.parse(JSON.stringify(this.data));
  }
}

console.log('\n=== Change Detection ===');
const detector = new ChangeDetector({ name: 'Alice', age: 30 });
console.log('Changed?', detector.hasChanged()); // false

detector.data.age = 31;
console.log('Changed?', detector.hasChanged()); // true
console.log('Changes:', detector.getChanges());

// 5. Form dirty state
function isFormDirty(initialValues, currentValues) {
  return !deepEqual(initialValues, currentValues);
}

console.log('\n=== Form Dirty State ===');
const initial = { username: 'alice', email: 'alice@example.com' };
const current1 = { username: 'alice', email: 'alice@example.com' };
const current2 = { username: 'alice', email: 'newemail@example.com' };

console.log('Form dirty (no change):', isFormDirty(initial, current1)); // false
console.log('Form dirty (changed):', isFormDirty(initial, current2)); // true
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Primitives
console.log('Numbers:', deepEqual(42, 42)); // true
console.log('Strings:', deepEqual('hello', 'hello')); // true
console.log('Booleans:', deepEqual(true, true)); // true
console.log('undefined:', deepEqual(undefined, undefined)); // true
console.log('null:', deepEqual(null, null)); // true

// Special numbers
console.log('NaN:', deepEqual(NaN, NaN)); // false (use Object.is or special handling)
console.log('Infinity:', deepEqual(Infinity, Infinity)); // true
console.log('+0 vs -0:', deepEqual(0, -0)); // true (use Object.is for distinction)

// Empty structures
console.log('Empty objects:', deepEqual({}, {})); // true
console.log('Empty arrays:', deepEqual([], [])); // true

// Different types
console.log('Number vs String:', deepEqual(1, '1')); // false
console.log('Array vs Object:', deepEqual([], {})); // false
console.log('null vs undefined:', deepEqual(null, undefined)); // false

// Property order (shouldn't matter)
console.log('Property order:', deepEqual(
  { a: 1, b: 2 },
  { b: 2, a: 1 }
)); // true

// Nested arrays
console.log('Nested arrays:', deepEqual(
  [[1, 2], [3, 4]],
  [[1, 2], [3, 4]]
)); // true

// Functions (by reference only)
const fn1 = () => {};
const fn2 = () => {};
console.log('Same function:', deepEqual(fn1, fn1)); // true
console.log('Different functions:', deepEqual(fn1, fn2)); // false

// Symbols
const sym1 = Symbol('test');
const sym2 = Symbol('test');
console.log('Same symbol:', deepEqual(sym1, sym1)); // true
console.log('Different symbols:', deepEqual(sym1, sym2)); // false

// Object with null prototype
const obj1 = Object.create(null);
obj1.a = 1;
const obj2 = Object.create(null);
obj2.a = 1;
console.log('Null prototype:', deepEqual(obj1, obj2)); // true
```

### **Performance Comparison**
```javascript
/**
 * Compare different implementations
 */
console.log('\n=== Performance Comparison ===');

const testObj1 = {
  a: 1, b: 2, c: 3,
  nested: {
    x: [1, 2, 3],
    y: { z: 'hello' }
  }
};

const testObj2 = {
  a: 1, b: 2, c: 3,
  nested: {
    x: [1, 2, 3],
    y: { z: 'hello' }
  }
};

const iterations = 10000;

console.time('Basic deepEqual');
for (let i = 0; i < iterations; i++) {
  deepEqual(testObj1, testObj2);
}
console.timeEnd('Basic deepEqual');

console.time('Optimized deepEqual');
for (let i = 0; i < iterations; i++) {
  deepEqualOptimized(testObj1, testObj2);
}
console.timeEnd('Optimized deepEqual');

console.time('JSON.stringify (not recommended)');
for (let i = 0; i < iterations; i++) {
  JSON.stringify(testObj1) === JSON.stringify(testObj2);
}
console.timeEnd('JSON.stringify (not recommended)');

console.log('\nNote: JSON.stringify can fail with circular refs, different order, etc.');
```

**Interview Tips:**
- Deep equality: recursively compare all nested properties
- Base case: strict equality (===) for primitives and same reference
- For objects: compare keys count, then each property recursively
- For arrays: compare length, then each element recursively
- Handle special types: Date (getTime), RegExp (source + flags), NaN
- Circular references: use WeakMap to track seen objects
- Object.keys() doesn't guarantee order, but that's okay for equality
- Don't use JSON.stringify: fails on circular refs, functions, undefined, Symbol
- Performance: early exit on first difference, cache computed checks
- Constructor check: new Date() !== plain object even if same properties
- Map/Set: compare size, then iterate and check membership
- TypedArrays: compare length and byte-by-byte
- Shallow equality: only compare direct properties (===), not nested
- Use Object.is() for NaN and ±0 distinction
- Applications: React reconciliation, memoization, testing, change detection
- Edge cases: null, undefined, NaN, empty objects, different types, functions
- Clarify: shallow or deep? circular refs? special types? performance critical?
- Follow-ups: implement shallowEqual, handle custom classes, add path tracking

</details>

89. Create a function to flatten a deeply nested object

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Flattening with Dot Notation**
```javascript
/**
 * Flatten nested object using dot notation for keys
 * Time Complexity: O(n) where n is total number of properties
 * Space Complexity: O(n) for the result object
 */
function flattenObject(obj, prefix = '', result = {}) {
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      const newKey = prefix ? `${prefix}.${key}` : key;
      
      if (typeof obj[key] === 'object' && obj[key] !== null && !Array.isArray(obj[key])) {
        flattenObject(obj[key], newKey, result);
      } else {
        result[newKey] = obj[key];
      }
    }
  }
  
  return result;
}

// Test
console.log('=== Basic Flattening ===');

const nested = {
  name: 'John',
  age: 30,
  address: {
    street: '123 Main St',
    city: 'NYC',
    coordinates: {
      lat: 40.7128,
      lng: -74.0060
    }
  },
  hobbies: ['reading', 'coding']
};

console.log('Original:', JSON.stringify(nested, null, 2));
console.log('Flattened:', flattenObject(nested));
// {
//   name: 'John',
//   age: 30,
//   'address.street': '123 Main St',
//   'address.city': 'NYC',
//   'address.coordinates.lat': 40.7128,
//   'address.coordinates.lng': -74.0060,
//   hobbies: ['reading', 'coding']
// }
```

### **Approach 2: Flatten with Custom Separator**
```javascript
/**
 * Flatten with configurable separator
 */
function flattenWithSeparator(obj, separator = '.', prefix = '', result = {}) {
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      const newKey = prefix ? `${prefix}${separator}${key}` : key;
      const value = obj[key];
      
      if (value !== null && typeof value === 'object' && !Array.isArray(value)) {
        flattenWithSeparator(value, separator, newKey, result);
      } else {
        result[newKey] = value;
      }
    }
  }
  
  return result;
}

// Test
console.log('\n=== Custom Separator ===');

const data = {
  user: {
    profile: {
      name: 'Alice',
      email: 'alice@example.com'
    }
  }
};

console.log('Dot separator:', flattenWithSeparator(data, '.'));
// { 'user.profile.name': 'Alice', 'user.profile.email': 'alice@example.com' }

console.log('Underscore separator:', flattenWithSeparator(data, '_'));
// { 'user_profile_name': 'Alice', 'user_profile_email': 'alice@example.com' }

console.log('Slash separator:', flattenWithSeparator(data, '/'));
// { 'user/profile/name': 'Alice', 'user/profile/email': 'alice@example.com' }
```

### **Approach 3: Flatten with Array Handling**
```javascript
/**
 * Flatten including array indices
 */
function flattenWithArrays(obj, prefix = '', result = {}) {
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      const newKey = prefix ? `${prefix}.${key}` : key;
      const value = obj[key];
      
      if (Array.isArray(value)) {
        // Flatten array elements
        value.forEach((item, index) => {
          const arrayKey = `${newKey}[${index}]`;
          
          if (typeof item === 'object' && item !== null) {
            flattenWithArrays(item, arrayKey, result);
          } else {
            result[arrayKey] = item;
          }
        });
      } else if (typeof value === 'object' && value !== null) {
        flattenWithArrays(value, newKey, result);
      } else {
        result[newKey] = value;
      }
    }
  }
  
  return result;
}

// Test
console.log('\n=== With Arrays ===');

const withArrays = {
  name: 'Company',
  employees: [
    { name: 'Alice', role: 'Developer' },
    { name: 'Bob', role: 'Designer' }
  ],
  offices: ['NYC', 'SF', 'London']
};

console.log('Flattened with arrays:', flattenWithArrays(withArrays));
// {
//   name: 'Company',
//   'employees[0].name': 'Alice',
//   'employees[0].role': 'Developer',
//   'employees[1].name': 'Bob',
//   'employees[1].role': 'Designer',
//   'offices[0]': 'NYC',
//   'offices[1]': 'SF',
//   'offices[2]': 'London'
// }
```

### **Approach 4: Unflatten (Reverse Operation)**
```javascript
/**
 * Unflatten a flattened object back to nested structure
 */
function unflattenObject(obj) {
  const result = {};
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      const keys = key.split('.');
      let current = result;
      
      for (let i = 0; i < keys.length - 1; i++) {
        const k = keys[i];
        
        if (!current[k]) {
          current[k] = {};
        }
        
        current = current[k];
      }
      
      current[keys[keys.length - 1]] = obj[key];
    }
  }
  
  return result;
}

// Test
console.log('\n=== Unflatten ===');

const flattened = {
  'name': 'John',
  'address.street': '123 Main St',
  'address.city': 'NYC',
  'address.zip': '10001'
};

console.log('Flattened:', flattened);
console.log('Unflattened:', unflattenObject(flattened));
// {
//   name: 'John',
//   address: {
//     street: '123 Main St',
//     city: 'NYC',
//     zip: '10001'
//   }
// }

// Round trip
const original = { a: { b: { c: 1 } } };
const flat = flattenObject(original);
const restored = unflattenObject(flat);
console.log('Round trip:', JSON.stringify(original) === JSON.stringify(restored)); // true
```

### **Approach 5: Flatten with Max Depth**
```javascript
/**
 * Flatten up to a specified depth
 */
function flattenWithDepth(obj, maxDepth = Infinity, currentDepth = 0, prefix = '', result = {}) {
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      const newKey = prefix ? `${prefix}.${key}` : key;
      const value = obj[key];
      
      if (currentDepth < maxDepth && 
          typeof value === 'object' && 
          value !== null && 
          !Array.isArray(value)) {
        flattenWithDepth(value, maxDepth, currentDepth + 1, newKey, result);
      } else {
        result[newKey] = value;
      }
    }
  }
  
  return result;
}

// Test
console.log('\n=== Flatten with Max Depth ===');

const deep = {
  level1: {
    level2: {
      level3: {
        level4: {
          value: 'deep'
        }
      }
    }
  }
};

console.log('Depth 0:', flattenWithDepth(deep, 0));
// { level1: { level2: { level3: { level4: { value: 'deep' } } } } }

console.log('Depth 1:', flattenWithDepth(deep, 1));
// { 'level1.level2': { level3: { level4: { value: 'deep' } } } }

console.log('Depth 2:', flattenWithDepth(deep, 2));
// { 'level1.level2.level3': { level4: { value: 'deep' } } }

console.log('Depth Infinity:', flattenWithDepth(deep));
// { 'level1.level2.level3.level4.value': 'deep' }
```

### **Bonus: Flatten with Path Array**
```javascript
/**
 * Flatten with path as array instead of string
 */
function flattenToPath(obj, prefix = [], result = {}) {
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      const path = [...prefix, key];
      const value = obj[key];
      
      if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
        flattenToPath(value, path, result);
      } else {
        result[JSON.stringify(path)] = value;
      }
    }
  }
  
  return result;
}

// Test
console.log('\n=== Flatten to Path Array ===');

const obj = {
  a: {
    b: {
      c: 1
    }
  }
};

console.log('Path arrays:', flattenToPath(obj));
// { '["a","b","c"]': 1 }

// Can retrieve value using path
const flatPaths = flattenToPath(obj);
for (const [pathStr, value] of Object.entries(flatPaths)) {
  const path = JSON.parse(pathStr);
  console.log(`Path ${path.join('.')}:`, value);
}
```

### **Bonus: Flatten with Circular Reference Detection**
```javascript
/**
 * Safe flatten that handles circular references
 */
function flattenSafe(obj, prefix = '', result = {}, seen = new WeakSet()) {
  // Check for circular reference
  if (seen.has(obj)) {
    result[prefix] = '[Circular]';
    return result;
  }
  
  // Mark as seen
  if (typeof obj === 'object' && obj !== null) {
    seen.add(obj);
  }
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      const newKey = prefix ? `${prefix}.${key}` : key;
      const value = obj[key];
      
      if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
        flattenSafe(value, newKey, result, seen);
      } else {
        result[newKey] = value;
      }
    }
  }
  
  return result;
}

// Test
console.log('\n=== Safe Flatten (Circular) ===');

const circular = { name: 'obj' };
circular.self = circular;
circular.nested = { ref: circular };

console.log('Circular object:', flattenSafe(circular));
// {
//   name: 'obj',
//   self: '[Circular]',
//   'nested.ref': '[Circular]'
// }
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Convert nested config to environment variables
function toEnvVars(config) {
  const flattened = flattenObject(config);
  const envVars = {};
  
  for (const [key, value] of Object.entries(flattened)) {
    const envKey = key.toUpperCase().replace(/\./g, '_');
    envVars[envKey] = String(value);
  }
  
  return envVars;
}

console.log('\n=== Config to Env Vars ===');
const config = {
  database: {
    host: 'localhost',
    port: 5432,
    credentials: {
      username: 'admin',
      password: 'secret'
    }
  }
};

console.log('Env vars:', toEnvVars(config));
// {
//   DATABASE_HOST: 'localhost',
//   DATABASE_PORT: '5432',
//   DATABASE_CREDENTIALS_USERNAME: 'admin',
//   DATABASE_CREDENTIALS_PASSWORD: 'secret'
// }

// 2. Extract all values for a specific key
function extractKey(obj, targetKey) {
  const values = [];
  
  function traverse(current) {
    for (const key in current) {
      if (current.hasOwnProperty(key)) {
        if (key === targetKey) {
          values.push(current[key]);
        }
        
        if (typeof current[key] === 'object' && current[key] !== null) {
          traverse(current[key]);
        }
      }
    }
  }
  
  traverse(obj);
  return values;
}

console.log('\n=== Extract Specific Key ===');
const data2 = {
  user: { id: 1, name: 'Alice' },
  posts: [
    { id: 101, name: 'Post 1' },
    { id: 102, name: 'Post 2' }
  ]
};

console.log('All ids:', extractKey(data2, 'id')); // [1, 101, 102]
console.log('All names:', extractKey(data2, 'name')); // ['Alice', 'Post 1', 'Post 2']

// 3. Diff two objects
function diffObjects(obj1, obj2) {
  const flat1 = flattenObject(obj1);
  const flat2 = flattenObject(obj2);
  const diff = { added: {}, removed: {}, changed: {} };
  
  // Check for added and changed
  for (const key in flat2) {
    if (!(key in flat1)) {
      diff.added[key] = flat2[key];
    } else if (flat1[key] !== flat2[key]) {
      diff.changed[key] = { from: flat1[key], to: flat2[key] };
    }
  }
  
  // Check for removed
  for (const key in flat1) {
    if (!(key in flat2)) {
      diff.removed[key] = flat1[key];
    }
  }
  
  return diff;
}

console.log('\n=== Object Diff ===');
const v1 = { a: 1, b: { c: 2, d: 3 } };
const v2 = { a: 1, b: { c: 5 }, e: 6 };

console.log('Diff:', diffObjects(v1, v2));
// {
//   added: { e: 6 },
//   removed: { 'b.d': 3 },
//   changed: { 'b.c': { from: 2, to: 5 } }
// }

// 4. Query nested objects
function query(obj, path) {
  const keys = path.split('.');
  let current = obj;
  
  for (const key of keys) {
    if (current && typeof current === 'object' && key in current) {
      current = current[key];
    } else {
      return undefined;
    }
  }
  
  return current;
}

console.log('\n=== Query Nested Object ===');
const userData = {
  user: {
    profile: {
      name: 'Alice',
      settings: {
        theme: 'dark'
      }
    }
  }
};

console.log('Query "user.profile.name":', query(userData, 'user.profile.name')); // 'Alice'
console.log('Query "user.profile.settings.theme":', query(userData, 'user.profile.settings.theme')); // 'dark'
console.log('Query "user.invalid":', query(userData, 'user.invalid')); // undefined

// 5. Set nested value
function setNested(obj, path, value) {
  const keys = path.split('.');
  let current = obj;
  
  for (let i = 0; i < keys.length - 1; i++) {
    const key = keys[i];
    
    if (!(key in current) || typeof current[key] !== 'object') {
      current[key] = {};
    }
    
    current = current[key];
  }
  
  current[keys[keys.length - 1]] = value;
}

console.log('\n=== Set Nested Value ===');
const target = {};
setNested(target, 'user.profile.name', 'Bob');
setNested(target, 'user.settings.theme', 'light');
console.log('Result:', target);
// { user: { profile: { name: 'Bob' }, settings: { theme: 'light' } } }
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty object
console.log('Empty:', flattenObject({})); // {}

// Null values
console.log('With null:', flattenObject({ a: null, b: { c: null } }));
// { a: null, 'b.c': null }

// Undefined values
console.log('With undefined:', flattenObject({ a: undefined, b: { c: 1 } }));
// { a: undefined, 'b.c': 1 }

// Arrays as values
console.log('Arrays:', flattenObject({ a: [1, 2], b: { c: [3, 4] } }));
// { a: [1, 2], 'b.c': [3, 4] }

// Single level
console.log('Single level:', flattenObject({ a: 1, b: 2 }));
// { a: 1, b: 2 }

// Deep nesting
const veryDeep = { a: { b: { c: { d: { e: { f: 1 } } } } } };
console.log('Very deep:', flattenObject(veryDeep));
// { 'a.b.c.d.e.f': 1 }

// Special characters in keys
console.log('Special keys:', flattenObject({ 'key.with.dots': { value: 1 } }));
// { 'key.with.dots.value': 1 } - may need escaping
```

### **Performance Considerations**
```javascript
/**
 * Benchmark flattening
 */
console.log('\n=== Performance ===');

const largeObj = {};
for (let i = 0; i < 100; i++) {
  largeObj[`key${i}`] = {
    nested: {
      value: i,
      data: { x: i * 2 }
    }
  };
}

console.time('Flatten 100 nested objects');
const flattened2 = flattenObject(largeObj);
console.timeEnd('Flatten 100 nested objects');

console.log('Flattened keys:', Object.keys(flattened2).length);

console.time('Unflatten back');
const unflattened = unflattenObject(flattened2);
console.timeEnd('Unflatten back');

console.log('Restored keys:', Object.keys(unflattened).length);
```

**Interview Tips:**
- Flatten: convert nested object to single-level with composite keys
- Use dot notation for nested keys: "a.b.c"
- Recursive approach: traverse tree, build flat result
- Base case: primitive or array → add to result
- Recursive case: object → recurse with updated prefix
- Arrays: can treat as values or flatten with indices [0], [1]
- Unflatten: reverse operation, split keys by delimiter
- Custom separator: . (dot), _ (underscore), / (slash)
- Max depth: control flattening level, useful for partial flatten
- Circular references: use WeakSet to track visited objects
- Applications: env vars, config files, form data, diff/patch, query paths
- Object.keys vs for...in: keys better (only own properties)
- hasOwnProperty check prevents prototype pollution
- Path arrays alternative: store paths as arrays instead of strings
- Query/set helpers: navigate nested structures by path string
- Edge cases: empty object, null, undefined, arrays, single level, special chars
- Performance: O(n) time, O(n) space where n = total properties
- Clarify: handle arrays? max depth? separator? circular refs? preserve types?
- Follow-ups: implement with path arrays, add type preservation, streaming flatten

</details>

90. Implement call, apply, and bind from scratch

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Implement call()**
```javascript
/**
 * Implement Function.prototype.call()
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * call() invokes function with specific 'this' context and arguments
 */
Function.prototype.myCall = function(context, ...args) {
  // Handle null/undefined context (becomes globalThis in non-strict mode)
  context = context || globalThis;
  
  // Create unique property to avoid name collision
  const fnSymbol = Symbol('fn');
  
  // Assign function to context
  context[fnSymbol] = this;
  
  // Call function with context
  const result = context[fnSymbol](...args);
  
  // Clean up
  delete context[fnSymbol];
  
  return result;
};

// Test
console.log('=== myCall() Implementation ===');

function greet(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

const person = { name: 'Alice' };

console.log(greet.myCall(person, 'Hello', '!')); // "Hello, Alice!"
console.log(greet.myCall(person, 'Hi', '.')); // "Hi, Alice."

// Compare with native
console.log(greet.call(person, 'Hello', '!')); // "Hello, Alice!"

// With different context
const person2 = { name: 'Bob' };
console.log(greet.myCall(person2, 'Hey', '~')); // "Hey, Bob~"
```

### **Approach 2: Implement apply()**
```javascript
/**
 * Implement Function.prototype.apply()
 * Similar to call but takes arguments as array
 */
Function.prototype.myApply = function(context, argsArray = []) {
  // Handle null/undefined context
  context = context || globalThis;
  
  // Validate argsArray
  if (!Array.isArray(argsArray) && argsArray !== null && argsArray !== undefined) {
    throw new TypeError('Second argument must be an array or array-like object');
  }
  
  // Create unique property
  const fnSymbol = Symbol('fn');
  
  // Assign function to context
  context[fnSymbol] = this;
  
  // Call function with spread arguments
  const result = context[fnSymbol](...(argsArray || []));
  
  // Clean up
  delete context[fnSymbol];
  
  return result;
};

// Test
console.log('\n=== myApply() Implementation ===');

function introduce(greeting, hobby, age) {
  return `${greeting}, I'm ${this.name}. I like ${hobby} and I'm ${age} years old.`;
}

const user = { name: 'Charlie' };

console.log(introduce.myApply(user, ['Hi', 'coding', 25]));
// "Hi, I'm Charlie. I like coding and I'm 25 years old."

// Compare with native
console.log(introduce.apply(user, ['Hi', 'coding', 25]));

// With Math.max example
const numbers = [5, 6, 2, 3, 7];
console.log('Max with myApply:', Math.max.myApply(null, numbers)); // 7
console.log('Max with apply:', Math.max.apply(null, numbers)); // 7
```

### **Approach 3: Implement bind()**
```javascript
/**
 * Implement Function.prototype.bind()
 * Returns new function with bound context
 */
Function.prototype.myBind = function(context, ...boundArgs) {
  const originalFunction = this;
  
  // Return new function
  return function(...args) {
    // Combine bound args with new args
    return originalFunction.myApply(context, [...boundArgs, ...args]);
  };
};

// Test
console.log('\n=== myBind() Implementation ===');

function multiply(a, b) {
  return `${this.name}: ${a} × ${b} = ${a * b}`;
}

const calculator = { name: 'Calculator' };

const boundMultiply = multiply.myBind(calculator);
console.log(boundMultiply(5, 3)); // "Calculator: 5 × 3 = 15"

const boundMultiplyWith10 = multiply.myBind(calculator, 10);
console.log(boundMultiplyWith10(5)); // "Calculator: 10 × 5 = 50"

// Compare with native
const nativeBound = multiply.bind(calculator, 10);
console.log(nativeBound(5)); // "Calculator: 10 × 5 = 50"
```

### **Approach 4: Handle Constructor Functions with bind()**
```javascript
/**
 * Enhanced bind that handles constructor invocation
 */
Function.prototype.myBindAdvanced = function(context, ...boundArgs) {
  const originalFunction = this;
  
  const boundFunction = function(...args) {
    // Check if called with 'new'
    if (this instanceof boundFunction) {
      // Called as constructor
      return new originalFunction(...boundArgs, ...args);
    } else {
      // Called as regular function
      return originalFunction.apply(context, [...boundArgs, ...args]);
    }
  };
  
  // Maintain prototype chain
  if (originalFunction.prototype) {
    boundFunction.prototype = Object.create(originalFunction.prototype);
  }
  
  return boundFunction;
};

// Test
console.log('\n=== Advanced bind() with Constructor ===');

function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function() {
  return `Hi, I'm ${this.name} and I'm ${this.age}`;
};

const BoundPerson = Person.myBindAdvanced(null, 'Alice');

// Used as constructor
const alice = new BoundPerson(30);
console.log(alice.name); // "Alice"
console.log(alice.age); // 30
console.log(alice.greet()); // "Hi, I'm Alice and I'm 30"
console.log(alice instanceof Person); // true

// Regular function
function regularFunc() {
  return this.value;
}

const obj = { value: 42 };
const bound = regularFunc.myBindAdvanced(obj);
console.log(bound()); // 42
```

### **Approach 5: Complete Implementation with Edge Cases**
```javascript
/**
 * Production-ready implementations
 */

// Complete call implementation
Function.prototype.myCallComplete = function(context, ...args) {
  // Check if called on a function
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.call called on non-function');
  }
  
  // Handle primitive contexts
  if (context === null || context === undefined) {
    context = globalThis;
  } else {
    context = Object(context); // Convert primitives to objects
  }
  
  const fnSymbol = Symbol('fn');
  context[fnSymbol] = this;
  
  const result = context[fnSymbol](...args);
  delete context[fnSymbol];
  
  return result;
};

// Complete apply implementation
Function.prototype.myApplyComplete = function(context, argsArray) {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.apply called on non-function');
  }
  
  if (context === null || context === undefined) {
    context = globalThis;
  } else {
    context = Object(context);
  }
  
  // Validate argsArray
  if (argsArray !== null && argsArray !== undefined) {
    if (typeof argsArray !== 'object') {
      throw new TypeError('Second argument must be an array or array-like object');
    }
  }
  
  const fnSymbol = Symbol('fn');
  context[fnSymbol] = this;
  
  const result = argsArray
    ? context[fnSymbol](...Array.from(argsArray))
    : context[fnSymbol]();
  
  delete context[fnSymbol];
  
  return result;
};

// Complete bind implementation
Function.prototype.myBindComplete = function(context, ...boundArgs) {
  if (typeof this !== 'function') {
    throw new TypeError('Function.prototype.bind called on non-function');
  }
  
  const originalFunction = this;
  
  const boundFunction = function(...args) {
    const isConstructor = this instanceof boundFunction;
    
    return originalFunction.myApplyComplete(
      isConstructor ? this : context,
      [...boundArgs, ...args]
    );
  };
  
  // Preserve prototype
  if (originalFunction.prototype) {
    boundFunction.prototype = Object.create(originalFunction.prototype);
  }
  
  // Set length property
  Object.defineProperty(boundFunction, 'length', {
    value: Math.max(0, originalFunction.length - boundArgs.length),
    configurable: true
  });
  
  // Set name property
  Object.defineProperty(boundFunction, 'name', {
    value: 'bound ' + (originalFunction.name || 'anonymous'),
    configurable: true
  });
  
  return boundFunction;
};

// Test
console.log('\n=== Complete Implementations ===');

function testFunc(a, b, c) {
  return { name: this.name, args: [a, b, c] };
}

const context = { name: 'TestContext' };

// Test call
console.log('myCallComplete:', testFunc.myCallComplete(context, 1, 2, 3));

// Test apply
console.log('myApplyComplete:', testFunc.myApplyComplete(context, [4, 5, 6]));

// Test bind
const boundFunc = testFunc.myBindComplete(context, 7);
console.log('myBindComplete:', boundFunc(8, 9));
console.log('Bound name:', boundFunc.name); // "bound testFunc"
console.log('Bound length:', boundFunc.length); // 2 (3 - 1 bound arg)
```

### **Bonus: Soft Bind**
```javascript
/**
 * Soft bind: allows overriding context with explicit call/apply
 */
Function.prototype.softBind = function(context, ...boundArgs) {
  const originalFunction = this;
  
  return function(...args) {
    // Use 'this' if it's defined and not global, otherwise use bound context
    const effectiveContext = (!this || this === globalThis) ? context : this;
    
    return originalFunction.apply(effectiveContext, [...boundArgs, ...args]);
  };
};

// Test
console.log('\n=== Soft Bind ===');

function showName() {
  return this.name;
}

const obj1 = { name: 'Object 1' };
const obj2 = { name: 'Object 2' };

// Hard bind (normal bind)
const hardBound = showName.myBind(obj1);
console.log('Hard bound:', hardBound()); // "Object 1"
console.log('Hard bound with call:', hardBound.call(obj2)); // Still "Object 1"

// Soft bind
const softBound = showName.softBind(obj1);
console.log('Soft bound:', softBound()); // "Object 1"
console.log('Soft bound with call:', softBound.call(obj2)); // "Object 2" (overridden!)
```

### **Bonus: Partial Application**
```javascript
/**
 * Partial application using bind
 */
function partial(fn, ...presetArgs) {
  return fn.myBind(null, ...presetArgs);
}

// Test
console.log('\n=== Partial Application ===');

function volume(length, width, height) {
  return length * width * height;
}

const volumeWith5 = partial(volume, 5);
console.log('Volume with 5:', volumeWith5(10, 2)); // 100

const volumeWith5And10 = partial(volume, 5, 10);
console.log('Volume with 5 and 10:', volumeWith5And10(2)); // 100

// Practical example: logging
function log(level, timestamp, message) {
  return `[${level}] ${timestamp}: ${message}`;
}

const errorLog = partial(log, 'ERROR');
console.log(errorLog('2024-01-01', 'System failure'));

const errorLogWithTime = partial(log, 'ERROR', '2024-01-01');
console.log(errorLogWithTime('Database connection lost'));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Event handlers with context
class Button {
  constructor(label) {
    this.label = label;
    this.clicks = 0;
  }
  
  handleClick() {
    this.clicks++;
    console.log(`${this.label} clicked ${this.clicks} times`);
  }
  
  attachTo(element) {
    // Without bind, 'this' would be the element
    element.addEventListener('click', this.handleClick.myBind(this));
  }
}

console.log('\n=== Event Handler Example ===');
const button = new Button('Submit');
const mockElement = {
  addEventListener(event, handler) {
    console.log('Attached handler for:', event);
    handler(); // Simulate click
  }
};
button.attachTo(mockElement);

// 2. Borrowing methods
const person1 = {
  name: 'Alice',
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};

const person2 = { name: 'Bob' };

console.log('\n=== Borrowing Methods ===');
console.log('Borrowed with call:', person1.greet.myCall(person2)); // "Hello, I'm Bob"

// Borrowing array methods
const arrayLike = { 0: 'a', 1: 'b', 2: 'c', length: 3 };
const array = Array.prototype.slice.myCall(arrayLike);
console.log('Array-like to array:', array); // ['a', 'b', 'c']

// 3. Function currying with bind
function add(a, b) {
  return a + b;
}

console.log('\n=== Currying with Bind ===');
const add5 = add.myBind(null, 5);
console.log('add5(3):', add5(3)); // 8
console.log('add5(10):', add5(10)); // 15

// 4. Debounce with context preservation
function debounce(func, delay) {
  let timeoutId;
  
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      func.myApply(this, args);
    }, delay);
  };
}

console.log('\n=== Debounce Example ===');
const obj3 = {
  value: 0,
  increment() {
    this.value++;
    console.log('Value:', this.value);
  }
};

const debouncedIncrement = debounce(obj3.increment, 100);
debouncedIncrement.myCall(obj3);
setTimeout(() => debouncedIncrement.myCall(obj3), 50);
setTimeout(() => debouncedIncrement.myCall(obj3), 150); // Only this fires

// 5. Method chaining
class Calculator {
  constructor(value = 0) {
    this.value = value;
  }
  
  add(n) {
    this.value += n;
    return this;
  }
  
  multiply(n) {
    this.value *= n;
    return this;
  }
  
  getResult() {
    return this.value;
  }
}

console.log('\n=== Method Chaining ===');
const calc = new Calculator(5);
const result = calc.add(10).multiply(2).getResult();
console.log('Result:', result); // 30

// Using bind to create pre-configured calculators
const addTen = calc.add.myBind(calc, 10);
addTen();
console.log('After addTen:', calc.value); // 40
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Primitive context
function showThis() {
  return this;
}

console.log('With number:', typeof showThis.myCall(5)); // "object" (Number wrapper)
console.log('With string:', typeof showThis.myCall('hello')); // "object" (String wrapper)
console.log('With boolean:', typeof showThis.myCall(true)); // "object" (Boolean wrapper)

// null/undefined context
function getGlobal() {
  return this === globalThis;
}

console.log('null context:', getGlobal.myCall(null)); // true
console.log('undefined context:', getGlobal.myCall(undefined)); // true

// No arguments
function noArgs() {
  return 'no arguments';
}

console.log('myCall no args:', noArgs.myCall({})); // "no arguments"
console.log('myApply no args:', noArgs.myApply({})); // "no arguments"

// Empty array
function withArgs(...args) {
  return args.length;
}

console.log('myApply empty array:', withArgs.myApply({}, [])); // 0

// Return value
function returnValue(x) {
  return x * 2;
}

console.log('myCall return:', returnValue.myCall(null, 21)); // 42
console.log('myApply return:', returnValue.myApply(null, [21])); // 42

const boundReturn = returnValue.myBind(null, 21);
console.log('myBind return:', boundReturn()); // 42

// Arrow functions (don't have own 'this')
const arrowFunc = () => this;
const boundArrow = arrowFunc.myBind({ name: 'test' });
// Arrow functions ignore bind's context

// Binding already bound function
function original() {
  return this.value;
}

const bound1 = original.myBind({ value: 1 });
const bound2 = bound1.myBind({ value: 2 });
console.log('Double bind:', bound2()); // 1 (first bind wins)
```

### **Performance Comparison**
```javascript
/**
 * Benchmark implementations
 */
console.log('\n=== Performance Comparison ===');

function testFunc2(a, b) {
  return this.x + a + b;
}

const ctx = { x: 10 };
const iterations = 100000;

// Native call
console.time('Native call');
for (let i = 0; i < iterations; i++) {
  testFunc2.call(ctx, 5, 3);
}
console.timeEnd('Native call');

// Custom call
console.time('Custom call');
for (let i = 0; i < iterations; i++) {
  testFunc2.myCall(ctx, 5, 3);
}
console.timeEnd('Custom call');

// Native bind
console.time('Native bind');
const nativeBound2 = testFunc2.bind(ctx);
for (let i = 0; i < iterations; i++) {
  nativeBound2(5, 3);
}
console.timeEnd('Native bind');

// Custom bind
console.time('Custom bind');
const customBound2 = testFunc2.myBind(ctx);
for (let i = 0; i < iterations; i++) {
  customBound2(5, 3);
}
console.timeEnd('Custom bind');

console.log('\nNote: Native implementations are optimized at engine level');
```

### **Comparison Table**
```javascript
console.log('\n=== call vs apply vs bind ===');
console.log(`
┌──────────┬────────────────────┬──────────────┬────────────────┐
│ Method   │ Arguments          │ Returns      │ Invokes Now    │
├──────────┼────────────────────┼──────────────┼────────────────┤
│ call     │ Individual args    │ Result       │ Yes            │
│ apply    │ Array of args      │ Result       │ Yes            │
│ bind     │ Individual args    │ New function │ No (later)     │
└──────────┴────────────────────┴──────────────┴────────────────┘

Usage:
  func.call(context, arg1, arg2, arg3)
  func.apply(context, [arg1, arg2, arg3])
  func.bind(context, arg1, arg2) // returns function
`);

// Examples of when to use each
function example(a, b, c) {
  return `${this.name}: ${a}, ${b}, ${c}`;
}

const ctx2 = { name: 'Example' };

console.log('call:', example.call(ctx2, 1, 2, 3));
console.log('apply:', example.apply(ctx2, [1, 2, 3]));

const bound3 = example.bind(ctx2, 1);
console.log('bind:', bound3(2, 3));
```

**Interview Tips:**
- call: invoke function with specific 'this' and individual arguments
- apply: same as call but takes arguments as array
- bind: returns new function with bound context (doesn't invoke immediately)
- Implementation strategy: assign function to context as temporary property
- Use Symbol to avoid property name collision
- Clean up temporary property after execution
- Handle null/undefined context (becomes globalThis)
- Convert primitive contexts to objects (Object(context))
- bind must handle constructor invocation with 'new'
- Preserve prototype chain for bound constructors
- bind can do partial application (preset arguments)
- Difference: call/apply invoke immediately, bind returns function
- Use call when you know arguments count
- Use apply with array/array-like or unknown argument count (e.g., Math.max)
- Use bind for event handlers, callbacks to preserve 'this'
- Soft bind: allows re-binding context later
- Edge cases: primitives, null, undefined, no args, arrow functions
- Arrow functions ignore bind/call/apply context
- Double binding: first bind wins, can't re-bind
- Applications: borrowing methods, event handlers, currying, debounce
- Clarify: handle constructors? preserve prototype? partial application?
- Follow-ups: implement softBind, add validation, optimize performance

</details>

91. Create a memoization function for recursive functions

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Memoization**
```javascript
/**
 * Simple memoization for any function
 * Time Complexity: O(1) for cache hit, O(f(n)) for cache miss
 * Space Complexity: O(n) where n is number of unique inputs
 */
function memoize(fn) {
  const cache = {};
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (key in cache) {
      console.log('Cache hit:', key);
      return cache[key];
    }
    
    console.log('Cache miss, computing:', key);
    const result = fn.apply(this, args);
    cache[key] = result;
    
    return result;
  };
}

// Test
console.log('=== Basic Memoization ===');

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

const memoizedFib = memoize(fibonacci);

console.log('fib(10):', memoizedFib(10)); // Cache miss
console.log('fib(10) again:', memoizedFib(10)); // Cache hit

// Problem: recursive calls inside aren't memoized!
console.log('\nNote: Internal recursive calls not cached');
```

### **Approach 2: Memoization for Recursive Functions**
```javascript
/**
 * Proper memoization that caches recursive calls
 */
function memoizeRecursive(fn) {
  const cache = {};
  
  const memoized = function(...args) {
    const key = JSON.stringify(args);
    
    if (key in cache) {
      return cache[key];
    }
    
    // Pass memoized version to function so recursive calls are cached
    const result = fn.call(this, memoized, ...args);
    cache[key] = result;
    
    return result;
  };
  
  return memoized;
}

// Test
console.log('\n=== Recursive Memoization ===');

// Function must accept itself as first parameter
const fibRecursive = memoizeRecursive((fib, n) => {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2); // Use passed memoized version
});

console.time('fib(40) first call');
console.log('fib(40):', fibRecursive(40));
console.timeEnd('fib(40) first call');

console.time('fib(40) second call');
console.log('fib(40) again:', fibRecursive(40));
console.timeEnd('fib(40) second call');

console.time('fib(50)');
console.log('fib(50):', fibRecursive(50)); // Uses cached fib(40), fib(39), etc.
console.timeEnd('fib(50)');
```

### **Approach 3: Auto-Memoizing Recursive Functions**
```javascript
/**
 * Automatically memoize recursive function without modifying it
 */
function memoizeAuto(fn) {
  const cache = new Map();
  
  const memoized = function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    // Temporarily replace original function in scope
    const originalFn = fn;
    const context = this;
    
    // Replace function with memoized version during execution
    const result = fn.apply(context, args);
    
    cache.set(key, result);
    return result;
  };
  
  return memoized;
}

// Better approach: Y Combinator style
function memoizeY(fn) {
  const cache = new Map();
  
  return function memoized(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    // Create wrapper that replaces function name with memoized version
    const wrapper = fn.bind(this, memoized);
    const result = wrapper(...args);
    
    cache.set(key, result);
    return result;
  };
}

// Test
console.log('\n=== Auto-Memoizing ===');

const factorial = memoizeY((fact, n) => {
  if (n <= 1) return 1;
  return n * fact(n - 1);
});

console.log('factorial(10):', factorial(10)); // 3628800
console.log('factorial(5):', factorial(5)); // Uses cached values
console.log('factorial(15):', factorial(15)); // Uses cached 10!
```

### **Approach 4: With Cache Management**
```javascript
/**
 * Memoization with cache size limit and statistics
 */
function memoizeAdvanced(fn, options = {}) {
  const { maxSize = 100, ttl = Infinity } = options;
  const cache = new Map();
  let hits = 0;
  let misses = 0;
  
  const memoized = function(...args) {
    const key = JSON.stringify(args);
    
    // Check cache
    if (cache.has(key)) {
      const entry = cache.get(key);
      
      // Check TTL
      if (Date.now() - entry.timestamp < ttl) {
        hits++;
        
        // Move to end (LRU)
        cache.delete(key);
        cache.set(key, entry);
        
        return entry.value;
      } else {
        cache.delete(key);
      }
    }
    
    misses++;
    
    // Compute result
    const result = fn.call(this, memoized, ...args);
    
    // Evict oldest if at capacity (LRU)
    if (cache.size >= maxSize) {
      const firstKey = cache.keys().next().value;
      cache.delete(firstKey);
    }
    
    // Store with timestamp
    cache.set(key, {
      value: result,
      timestamp: Date.now()
    });
    
    return result;
  };
  
  // Add utility methods
  memoized.clear = () => cache.clear();
  memoized.delete = (...args) => cache.delete(JSON.stringify(args));
  memoized.stats = () => ({
    hits,
    misses,
    hitRate: hits / (hits + misses) || 0,
    size: cache.size
  });
  
  return memoized;
}

// Test
console.log('\n=== Advanced Memoization ===');

const expensiveFib = memoizeAdvanced(
  (fib, n) => {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
  },
  { maxSize: 50, ttl: 5000 }
);

expensiveFib(20);
expensiveFib(25);
expensiveFib(20); // Hit

console.log('Stats:', expensiveFib.stats());
// { hits: ..., misses: ..., hitRate: ..., size: ... }
```

### **Approach 5: Multi-Argument Hash Function**
```javascript
/**
 * Custom hash function for complex arguments
 */
function memoizeWithHash(fn, hashFn) {
  const cache = new Map();
  
  const memoized = function(...args) {
    const key = hashFn ? hashFn(...args) : JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.call(this, memoized, ...args);
    cache.set(key, result);
    
    return result;
  };
  
  return memoized;
}

// Test
console.log('\n=== Custom Hash Function ===');

// For objects, use specific property
function expensiveCompute(obj, n) {
  // Some expensive operation
  return obj.id * n;
}

const memoizedCompute = memoizeWithHash(
  (fn, obj, n) => fn.call(this, obj, n),
  (obj, n) => `${obj.id}-${n}` // Hash by id only
);

const obj1 = { id: 1, data: 'large data' };
const obj2 = { id: 1, data: 'different data' };

console.log(memoizedCompute(obj1, 10)); // Cache miss
console.log(memoizedCompute(obj2, 10)); // Cache hit! (same id)

// Custom hash for arrays
const sumMemoized = memoizeWithHash(
  (fn, arr) => arr.reduce((a, b) => a + b, 0),
  (arr) => arr.join(',')
);

console.log(sumMemoized([1, 2, 3])); // Cache miss
console.log(sumMemoized([1, 2, 3])); // Cache hit
```

### **Bonus: Memoize Multiple Functions Together**
```javascript
/**
 * Shared cache across multiple functions
 */
function createMemoCache() {
  const cache = new Map();
  
  return function memoize(fn) {
    return function(...args) {
      const key = fn.name + JSON.stringify(args);
      
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      const result = fn(...args);
      cache.set(key, result);
      
      return result;
    };
  };
}

// Test
console.log('\n=== Shared Cache ===');

const sharedMemoize = createMemoCache();

function double(n) {
  return n * 2;
}

function triple(n) {
  return n * 3;
}

const memoDouble = sharedMemoize(double);
const memoTriple = sharedMemoize(triple);

memoDouble(5);
memoTriple(5);

console.log('Both functions share same cache');
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Dynamic Programming - Longest Common Subsequence
const lcs = memoizeRecursive((lcs, s1, s2, i = s1.length - 1, j = s2.length - 1) => {
  if (i < 0 || j < 0) return 0;
  
  if (s1[i] === s2[i]) {
    return 1 + lcs(s1, s2, i - 1, j - 1);
  }
  
  return Math.max(
    lcs(s1, s2, i - 1, j),
    lcs(s1, s2, i, j - 1)
  );
});

console.log('\n=== LCS Example ===');
console.log('LCS("ABCDGH", "AEDFHR"):', lcs('ABCDGH', 'AEDFHR')); // 3 (ADH)

// 2. Path finding - Count paths in grid
const countPaths = memoizeRecursive((count, m, n) => {
  if (m === 1 || n === 1) return 1;
  return count(m - 1, n) + count(m, n - 1);
});

console.log('\n=== Grid Paths ===');
console.log('Paths in 3x3 grid:', countPaths(3, 3)); // 6

// 3. Coin change problem
const coinChange = memoizeRecursive((change, coins, amount) => {
  if (amount === 0) return 1;
  if (amount < 0 || coins.length === 0) return 0;
  
  return change(coins, amount - coins[0]) + 
         change(coins.slice(1), amount);
});

console.log('\n=== Coin Change ===');
console.log('Ways to make 5:', coinChange([1, 2, 5], 5)); // 4 ways

// 4. API request caching
function createApiCache(ttl = 60000) {
  return memoizeAdvanced(
    async (fn, endpoint) => {
      const response = await fetch(endpoint);
      return response.json();
    },
    { ttl }
  );
}

console.log('\n=== API Caching ===');
console.log('API requests cached for', 60, 'seconds');

// 5. Expensive calculations
const calculatePrimes = memoizeRecursive((calc, n) => {
  if (n < 2) return [];
  
  const primes = [];
  for (let i = 2; i <= n; i++) {
    let isPrime = true;
    for (let j = 2; j * j <= i; j++) {
      if (i % j === 0) {
        isPrime = false;
        break;
      }
    }
    if (isPrime) primes.push(i);
  }
  
  return primes;
});

console.log('\n=== Prime Calculation ===');
console.time('Primes up to 1000');
console.log('Count:', calculatePrimes(1000).length);
console.timeEnd('Primes up to 1000');

console.time('Primes up to 1000 (cached)');
console.log('Count:', calculatePrimes(1000).length);
console.timeEnd('Primes up to 1000 (cached)');
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// No arguments
const noArgs = memoize(() => Math.random());
console.log('No args 1:', noArgs()); // Random
console.log('No args 2:', noArgs()); // Same (cached)

// Different argument types
const typeTest = memoize((a, b) => `${a}-${b}`);
console.log(typeTest(1, 2)); // "1-2"
console.log(typeTest('1', '2')); // Different key, new computation
console.log(typeTest(1, 2)); // Cached

// Object arguments
const objMemo = memoize((obj) => obj.value * 2);
console.log(objMemo({ value: 5 })); // 10
console.log(objMemo({ value: 5 })); // Cached (same JSON)

// undefined vs null
const nullTest = memoize((x) => x);
console.log(nullTest(null)); // null
console.log(nullTest(undefined)); // undefined (different key)

// Array order matters
const arrayMemo = memoize((arr) => arr.reduce((a, b) => a + b, 0));
console.log(arrayMemo([1, 2, 3])); // 6
console.log(arrayMemo([3, 2, 1])); // Not cached (different order)
```

### **Performance Comparison**
```javascript
/**
 * Benchmark memoization impact
 */
console.log('\n=== Performance Impact ===');

function slowFib(n) {
  if (n <= 1) return n;
  return slowFib(n - 1) + slowFib(n - 2);
}

const fastFib = memoizeRecursive((fib, n) => {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
});

console.time('Without memoization fib(35)');
slowFib(35);
console.timeEnd('Without memoization fib(35)');

console.time('With memoization fib(35)');
fastFib(35);
console.timeEnd('With memoization fib(35)');

console.time('With memoization fib(100)');
fastFib(100);
console.timeEnd('With memoization fib(100)');

console.log('\nMemoization reduces O(2^n) to O(n) for recursive functions!');
```

**Interview Tips:**
- Memoization: cache function results to avoid recomputation
- Key challenge: recursive calls inside function must also be cached
- Solution: pass memoized version as argument to function
- Function signature: (memoizedSelf, ...originalArgs)
- Cache key: JSON.stringify for simple types, custom hash for complex
- Map better than object for non-string keys
- Handle cache size: implement LRU eviction
- TTL for time-sensitive data (API responses)
- Clear/delete methods for cache management
- Statistics: track hits/misses for optimization
- Transforms O(2^n) to O(n) for recursive algorithms
- Perfect for: DP problems, Fibonacci, path finding, expensive calculations
- Not suitable for: side effects, I/O operations, time-sensitive functions
- JSON.stringify limitations: order matters, circular refs fail
- Custom hash for objects: use specific properties (id)
- Trade-off: memory for speed
- Applications: DP, API caching, expensive math, recursive algorithms
- Edge cases: no args, different types, objects, arrays, null/undefined
- Clarify: cache size? TTL? LRU? stats? custom hash?
- Follow-ups: implement weak memoization, add cache warming, serialize to disk

</details>

92. Implement a function to convert callback-based functions to promises (promisify)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Promisify**
```javascript
/**
 * Convert callback-based function to promise-based
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * Assumes callback follows Node.js convention: (err, result)
 */
function promisify(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, result) => {
        if (err) {
          reject(err);
        } else {
          resolve(result);
        }
      });
    });
  };
}

// Test
console.log('=== Basic Promisify ===');

// Simulate Node.js-style callback function
function readFile(filename, callback) {
  setTimeout(() => {
    if (filename === 'error.txt') {
      callback(new Error('File not found'), null);
    } else {
      callback(null, `Contents of ${filename}`);
    }
  }, 100);
}

const readFilePromise = promisify(readFile);

(async () => {
  try {
    const content = await readFilePromise('test.txt');
    console.log('Success:', content); // "Contents of test.txt"
  } catch (error) {
    console.error('Error:', error.message);
  }
  
  try {
    await readFilePromise('error.txt');
  } catch (error) {
    console.error('Expected error:', error.message); // "File not found"
  }
})();
```

### **Approach 2: Promisify with Context Binding**
```javascript
/**
 * Promisify that preserves 'this' context
 */
function promisifyWithContext(fn, context) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn.call(context, ...args, (err, result) => {
        if (err) {
          reject(err);
        } else {
          resolve(result);
        }
      });
    });
  };
}

// Test
console.log('\n=== Promisify with Context ===');

const db = {
  name: 'MyDatabase',
  query(sql, callback) {
    console.log(`${this.name} executing: ${sql}`);
    setTimeout(() => {
      callback(null, { rows: [{ id: 1, name: 'Alice' }] });
    }, 100);
  }
};

const queryPromise = promisifyWithContext(db.query, db);

(async () => {
  const result = await queryPromise('SELECT * FROM users');
  console.log('Query result:', result);
})();
```

### **Approach 3: Promisify with Multiple Results**
```javascript
/**
 * Handle callbacks with multiple success values
 */
function promisifyMulti(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, ...results) => {
        if (err) {
          reject(err);
        } else {
          // If multiple results, return as array
          resolve(results.length <= 1 ? results[0] : results);
        }
      });
    });
  };
}

// Test
console.log('\n=== Multiple Results ===');

function getCoordinates(address, callback) {
  setTimeout(() => {
    callback(null, 40.7128, -74.0060); // lat, lng
  }, 100);
}

const getCoordsPromise = promisifyMulti(getCoordinates);

(async () => {
  const coords = await getCoordsPromise('New York');
  console.log('Coordinates:', coords); // [40.7128, -74.0060]
})();
```

### **Approach 4: Promisify.all() for Entire Module**
```javascript
/**
 * Promisify all methods of an object
 */
function promisifyAll(obj, options = {}) {
  const { suffix = 'Async', exclude = [] } = options;
  const promisified = {};
  
  for (const key in obj) {
    if (typeof obj[key] === 'function' && !exclude.includes(key)) {
      // Create promisified version with suffix
      promisified[key + suffix] = promisify(obj[key].bind(obj));
      // Keep original
      promisified[key] = obj[key].bind(obj);
    } else {
      promisified[key] = obj[key];
    }
  }
  
  return promisified;
}

// Test
console.log('\n=== Promisify All ===');

const fs = {
  readFile(path, callback) {
    setTimeout(() => callback(null, `Content of ${path}`), 50);
  },
  writeFile(path, content, callback) {
    setTimeout(() => callback(null, `Written to ${path}`), 50);
  },
  deleteFile(path, callback) {
    setTimeout(() => callback(null, `Deleted ${path}`), 50);
  }
};

const fsPromise = promisifyAll(fs);

(async () => {
  const content = await fsPromise.readFileAsync('test.txt');
  console.log(content);
  
  const writeResult = await fsPromise.writeFileAsync('new.txt', 'data');
  console.log(writeResult);
  
  // Original callback version still available
  fsPromise.readFile('test.txt', (err, data) => {
    console.log('Callback version:', data);
  });
})();
```

### **Approach 5: Custom Callback Format**
```javascript
/**
 * Promisify with custom callback position and format
 */
function promisifyCustom(fn, options = {}) {
  const {
    callbackPosition = -1, // -1 means last argument
    errorFirst = true,
    multiArgs = false
  } = options;
  
  return function(...args) {
    return new Promise((resolve, reject) => {
      const callback = errorFirst
        ? (err, ...results) => {
            if (err) {
              reject(err);
            } else {
              resolve(multiArgs ? results : results[0]);
            }
          }
        : (...results) => {
            resolve(multiArgs ? results : results[0]);
          };
      
      if (callbackPosition === -1) {
        args.push(callback);
      } else {
        args.splice(callbackPosition, 0, callback);
      }
      
      fn(...args);
    });
  };
}

// Test
console.log('\n=== Custom Callback Format ===');

// Non-error-first callback
function setTimeout2(delay, callback) {
  setTimeout(() => callback('Done!'), delay);
}

const sleep = promisifyCustom(setTimeout2, { errorFirst: false });

(async () => {
  const result = await sleep(100);
  console.log('Sleep result:', result); // "Done!"
})();

// Callback in middle position
function processData(data, callback, options) {
  setTimeout(() => callback(null, `Processed: ${data}`), 50);
}

const processPromise = promisifyCustom(processData, { callbackPosition: 1 });

(async () => {
  const result = await processPromise('data', { timeout: 1000 });
  console.log('Process result:', result);
})();
```

### **Bonus: Promisify with Timeout**
```javascript
/**
 * Promisify with automatic timeout
 */
function promisifyWithTimeout(fn, defaultTimeout = 5000) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      let timeoutId;
      let completed = false;
      
      const callback = (err, result) => {
        if (completed) return;
        completed = true;
        clearTimeout(timeoutId);
        
        if (err) {
          reject(err);
        } else {
          resolve(result);
        }
      };
      
      // Set timeout
      timeoutId = setTimeout(() => {
        if (!completed) {
          completed = true;
          reject(new Error('Operation timed out'));
        }
      }, defaultTimeout);
      
      fn(...args, callback);
    });
  };
}

// Test
console.log('\n=== Promisify with Timeout ===');

function slowOperation(data, callback) {
  setTimeout(() => {
    callback(null, `Result: ${data}`);
  }, 3000);
}

const slowPromise = promisifyWithTimeout(slowOperation, 1000);

(async () => {
  try {
    await slowPromise('test');
  } catch (error) {
    console.log('Timeout error:', error.message); // "Operation timed out"
  }
})();
```

### **Bonus: Reverse - Callbackify**
```javascript
/**
 * Convert promise-based function to callback-based
 */
function callbackify(fn) {
  return function(...args) {
    const callback = args.pop();
    
    if (typeof callback !== 'function') {
      throw new TypeError('Last argument must be a callback function');
    }
    
    fn(...args)
      .then(result => callback(null, result))
      .catch(err => callback(err));
  };
}

// Test
console.log('\n=== Callbackify (Reverse) ===');

async function fetchData(url) {
  await new Promise(resolve => setTimeout(resolve, 100));
  return `Data from ${url}`;
}

const fetchDataCallback = callbackify(fetchData);

fetchDataCallback('http://api.example.com', (err, data) => {
  if (err) {
    console.error('Error:', err);
  } else {
    console.log('Callback result:', data);
  }
});
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Node.js fs module
console.log('\n=== File System Example ===');

// Simulated fs functions
const fsModule = {
  readFile(path, encoding, callback) {
    setTimeout(() => {
      callback(null, `File content from ${path}`);
    }, 100);
  },
  
  writeFile(path, data, callback) {
    setTimeout(() => {
      callback(null);
    }, 100);
  },
  
  stat(path, callback) {
    setTimeout(() => {
      callback(null, { size: 1024, isFile: () => true });
    }, 100);
  }
};

const fs2 = promisifyAll(fsModule);

(async () => {
  const content = await fs2.readFileAsync('file.txt', 'utf8');
  console.log('Read:', content);
  
  await fs2.writeFileAsync('output.txt', content);
  console.log('Written successfully');
  
  const stats = await fs2.statAsync('file.txt');
  console.log('File size:', stats.size);
})();

// 2. Database queries
console.log('\n=== Database Example ===');

const database = {
  connect(config, callback) {
    setTimeout(() => callback(null, 'Connected'), 50);
  },
  
  query(sql, params, callback) {
    setTimeout(() => {
      callback(null, [{ id: 1 }, { id: 2 }]);
    }, 50);
  },
  
  close(callback) {
    setTimeout(() => callback(null), 50);
  }
};

const db2 = promisifyAll(database);

(async () => {
  await db2.connectAsync({ host: 'localhost' });
  console.log('Database connected');
  
  const results = await db2.queryAsync('SELECT * FROM users', []);
  console.log('Query results:', results);
  
  await db2.closeAsync();
  console.log('Database closed');
})();

// 3. HTTP requests
console.log('\n=== HTTP Request Example ===');

function httpGet(url, callback) {
  setTimeout(() => {
    if (url.includes('error')) {
      callback(new Error('HTTP 404'));
    } else {
      callback(null, { status: 200, body: 'Response data' });
    }
  }, 100);
}

const httpGetPromise = promisify(httpGet);

(async () => {
  try {
    const response = await httpGetPromise('http://api.example.com/users');
    console.log('HTTP response:', response);
  } catch (error) {
    console.error('HTTP error:', error.message);
  }
})();

// 4. setTimeout promisified (delay utility)
console.log('\n=== Delay Utility ===');

const delay = (ms) => {
  return new Promise(resolve => setTimeout(resolve, ms));
};

(async () => {
  console.log('Waiting 500ms...');
  await delay(500);
  console.log('Done waiting!');
})();

// 5. Event-based to promise
console.log('\n=== Event to Promise ===');

function eventToPromise(emitter, successEvent, errorEvent) {
  return new Promise((resolve, reject) => {
    emitter.once(successEvent, resolve);
    if (errorEvent) {
      emitter.once(errorEvent, reject);
    }
  });
}

// Simulated event emitter
const emitter = {
  listeners: {},
  once(event, handler) {
    this.listeners[event] = handler;
  },
  emit(event, data) {
    if (this.listeners[event]) {
      this.listeners[event](data);
    }
  }
};

(async () => {
  setTimeout(() => emitter.emit('complete', 'Success!'), 200);
  
  const result = await eventToPromise(emitter, 'complete', 'error');
  console.log('Event result:', result);
})();
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// No arguments
function noArgsCallback(callback) {
  setTimeout(() => callback(null, 'result'), 50);
}

const noArgsPromise = promisify(noArgsCallback);

(async () => {
  const result = await noArgsPromise();
  console.log('No args:', result);
})();

// Multiple error types
function customError(callback) {
  setTimeout(() => {
    const error = new TypeError('Custom error');
    error.code = 'CUSTOM_ERROR';
    callback(error);
  }, 50);
}

const customErrorPromise = promisify(customError);

(async () => {
  try {
    await customErrorPromise();
  } catch (error) {
    console.log('Custom error type:', error.constructor.name); // TypeError
    console.log('Error code:', error.code); // CUSTOM_ERROR
  }
})();

// Sync callback (edge case)
function syncCallback(callback) {
  callback(null, 'immediate');
}

const syncPromise = promisify(syncCallback);

(async () => {
  const result = await syncPromise();
  console.log('Sync callback:', result);
})();

// Multiple calls
function counter(callback) {
  counter.count = (counter.count || 0) + 1;
  setTimeout(() => callback(null, counter.count), 50);
}

const counterPromise = promisify(counter);

(async () => {
  const [r1, r2, r3] = await Promise.all([
    counterPromise(),
    counterPromise(),
    counterPromise()
  ]);
  console.log('Multiple calls:', r1, r2, r3);
})();

// Callback called multiple times (error case)
function badCallback(callback) {
  setTimeout(() => callback(null, 'first'), 50);
  setTimeout(() => callback(null, 'second'), 100); // Bad! Called twice
}

const badPromise = promisify(badCallback);

(async () => {
  const result = await badPromise();
  console.log('Bad callback (only first resolves):', result); // "first"
})();
```

### **Advanced: Promisify with Cancelation**
```javascript
/**
 * Promisify with ability to cancel
 */
function promisifyCancelable(fn) {
  return function(...args) {
    let cancelled = false;
    
    const promise = new Promise((resolve, reject) => {
      fn(...args, (err, result) => {
        if (cancelled) return;
        
        if (err) {
          reject(err);
        } else {
          resolve(result);
        }
      });
    });
    
    promise.cancel = () => {
      cancelled = true;
    };
    
    return promise;
  };
}

// Test
console.log('\n=== Cancelable Promise ===');

function longOperation(data, callback) {
  const timeoutId = setTimeout(() => {
    callback(null, `Result: ${data}`);
  }, 2000);
  
  // Return cleanup function
  return () => clearTimeout(timeoutId);
}

const cancelableOp = promisifyCancelable(longOperation);

(async () => {
  const promise = cancelableOp('test');
  
  setTimeout(() => {
    console.log('Canceling operation...');
    promise.cancel();
  }, 500);
  
  try {
    const result = await promise;
    console.log('Result:', result);
  } catch (error) {
    console.log('Canceled or error');
  }
})();
```

### **Performance Considerations**
```javascript
/**
 * Compare callback vs promise performance
 */
console.log('\n=== Performance Comparison ===');

function callbackVersion(x, callback) {
  callback(null, x * 2);
}

const promiseVersion = promisify(callbackVersion);

// Callback benchmark
console.time('Callbacks (10000 calls)');
let callbackCount = 0;
for (let i = 0; i < 10000; i++) {
  callbackVersion(i, (err, result) => {
    callbackCount++;
  });
}
console.timeEnd('Callbacks (10000 calls)');

// Promise benchmark
console.time('Promises (10000 calls)');
(async () => {
  for (let i = 0; i < 10000; i++) {
    await promiseVersion(i);
  }
  console.timeEnd('Promises (10000 calls)');
  
  console.log('\nNote: Promises have overhead but provide better error handling and async/await syntax');
})();
```

### **Node.js util.promisify Comparison**
```javascript
/**
 * Compare with Node.js built-in
 */
console.log('\n=== Comparison with Node.js util.promisify ===');

// Our implementation
function customPromisify(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, result) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  };
}

// Features comparison
console.log(`
Feature Comparison:
┌─────────────────────────┬─────────────┬──────────────┐
│ Feature                 │ Custom      │ util.promisify│
├─────────────────────────┼─────────────┼──────────────┤
│ Basic promisify         │ ✓           │ ✓            │
│ Context preservation    │ Manual      │ Automatic    │
│ Custom symbols          │ ✗           │ ✓            │
│ Multiple results        │ Manual      │ ✗            │
│ Timeout                 │ Manual      │ ✗            │
│ Cancelation             │ Manual      │ ✗            │
└─────────────────────────┴─────────────┴──────────────┘

Usage:
  const util = require('util');
  const readFilePromise = util.promisify(fs.readFile);
  
  // With custom promisify symbol
  function customFn(callback) {
    // ...
  }
  customFn[util.promisify.custom] = () => {
    return Promise.resolve('custom behavior');
  };
`);
```

**Interview Tips:**
- Promisify: convert callback-based function to promise-based
- Node.js convention: callback is last parameter, format (err, result)
- Return new function that returns a Promise
- Callback invokes resolve(result) on success, reject(err) on error
- Preserve 'this' context: use Function.call or bind
- Handle multiple results: return array if callback has >1 success params
- promisifyAll: convert entire module/object at once
- Suffix convention: add 'Async' to promisified method names
- Custom formats: adjust for non-error-first or different callback positions
- Timeout: wrap with timeout to prevent hanging
- Callbackify: reverse operation (promise → callback)
- Edge cases: sync callbacks, no args, multiple calls, errors
- Cancelation: track state to ignore callback after cancel
- Performance: promises have overhead but better ergonomics
- util.promisify in Node.js: built-in with custom symbol support
- Applications: fs, database, HTTP, setTimeout, events
- Modern approach: prefer async/await over callbacks
- Error handling: promise rejections easier to catch than callback errors
- Clarify: error-first? context preservation? multiple results? timeout?
- Follow-ups: add util.promisify.custom symbol, implement throttling, add retry logic

</details>

93. Create a once() function that executes only once

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic once() Function**
```javascript
/**
 * Execute function only once, subsequent calls return cached result
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 */
function once(fn) {
  let called = false;
  let result;
  
  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    
    return result;
  };
}

// Test
console.log('=== Basic once() ===');

function expensiveOperation(x) {
  console.log('Computing...');
  return x * 2;
}

const compute = once(expensiveOperation);

console.log('First call:', compute(5)); // Computing... 10
console.log('Second call:', compute(10)); // 10 (uses cached result, ignores new arg)
console.log('Third call:', compute(15)); // 10 (still cached)
```

### **Approach 2: once() with Context Preservation**
```javascript
/**
 * Preserve 'this' context in once function
 */
function onceWithContext(fn) {
  let called = false;
  let result;
  
  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    
    return result;
  };
}

// Test
console.log('\n=== once() with Context ===');

const obj = {
  value: 10,
  multiply: onceWithContext(function(x) {
    console.log('Multiplying...');
    return this.value * x;
  })
};

console.log('First:', obj.multiply(3)); // Multiplying... 30
console.log('Second:', obj.multiply(5)); // 30 (cached)
```

### **Approach 3: once() that Returns Undefined After First Call**
```javascript
/**
 * Alternative: subsequent calls return undefined
 */
function onceUndefined(fn) {
  let called = false;
  
  return function(...args) {
    if (!called) {
      called = true;
      return fn.apply(this, args);
    }
    
    return undefined;
  };
}

// Test
console.log('\n=== once() Returns Undefined ===');

const greet = onceUndefined((name) => {
  console.log(`Hello, ${name}!`);
  return 'greeted';
});

console.log('First:', greet('Alice')); // Hello, Alice! "greeted"
console.log('Second:', greet('Bob')); // undefined
console.log('Third:', greet('Charlie')); // undefined
```

### **Approach 4: once() with Reset Capability**
```javascript
/**
 * once() with ability to reset
 */
function onceResettable(fn) {
  let called = false;
  let result;
  
  const wrapped = function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    
    return result;
  };
  
  wrapped.reset = function() {
    called = false;
    result = undefined;
  };
  
  wrapped.hasBeenCalled = function() {
    return called;
  };
  
  return wrapped;
}

// Test
console.log('\n=== Resettable once() ===');

const counter = onceResettable((start) => {
  console.log(`Starting from ${start}`);
  return start;
});

console.log('Call 1:', counter(1)); // Starting from 1, returns 1
console.log('Call 2:', counter(2)); // returns 1 (cached)
console.log('Has been called?', counter.hasBeenCalled()); // true

counter.reset();
console.log('After reset, has been called?', counter.hasBeenCalled()); // false

console.log('Call 3:', counter(10)); // Starting from 10, returns 10
console.log('Call 4:', counter(20)); // returns 10 (cached again)
```

### **Approach 5: once() for Async Functions**
```javascript
/**
 * once() for async functions with promise caching
 */
function onceAsync(fn) {
  let called = false;
  let promise;
  
  return function(...args) {
    if (!called) {
      called = true;
      promise = Promise.resolve(fn.apply(this, args));
    }
    
    return promise;
  };
}

// Test
console.log('\n=== Async once() ===');

const fetchData = onceAsync(async (url) => {
  console.log(`Fetching from ${url}...`);
  await new Promise(resolve => setTimeout(resolve, 500));
  return `Data from ${url}`;
});

(async () => {
  console.log('Request 1 started');
  const result1 = await fetchData('http://api.example.com');
  console.log('Result 1:', result1);
  
  console.log('Request 2 started');
  const result2 = await fetchData('http://different.com'); // Uses cached promise
  console.log('Result 2:', result2); // Same as result1
})();
```

### **Bonus: before() - Execute N Times**
```javascript
/**
 * Execute function up to N times
 */
function before(n, fn) {
  let count = 0;
  let lastResult;
  
  return function(...args) {
    if (count < n) {
      count++;
      lastResult = fn.apply(this, args);
    }
    
    return lastResult;
  };
}

// Test
console.log('\n=== before() - N Times ===');

const greet3Times = before(3, (name) => {
  console.log(`Hello, ${name}!`);
  return `Greeted ${name}`;
});

console.log('1:', greet3Times('Alice')); // Hello, Alice!
console.log('2:', greet3Times('Bob')); // Hello, Bob!
console.log('3:', greet3Times('Charlie')); // Hello, Charlie!
console.log('4:', greet3Times('David')); // (no output, returns last result)
```

### **Bonus: after() - Execute After N Calls**
```javascript
/**
 * Execute function only after N calls
 */
function after(n, fn) {
  let count = 0;
  
  return function(...args) {
    count++;
    
    if (count >= n) {
      return fn.apply(this, args);
    }
  };
}

// Test
console.log('\n=== after() - After N Calls ===');

const afterThreeCalls = after(3, () => {
  console.log('Finally executed after 3 calls!');
  return 'result';
});

console.log('Call 1:', afterThreeCalls()); // undefined
console.log('Call 2:', afterThreeCalls()); // undefined
console.log('Call 3:', afterThreeCalls()); // Finally executed! "result"
console.log('Call 4:', afterThreeCalls()); // Finally executed! "result"
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Initialize once
console.log('\n=== Initialization Example ===');

class Database {
  constructor() {
    this.connection = null;
    this.connect = once(this._connect.bind(this));
  }
  
  _connect() {
    console.log('Establishing database connection...');
    this.connection = { connected: true };
    return this.connection;
  }
  
  query(sql) {
    this.connect(); // Safe to call multiple times
    console.log(`Executing: ${sql}`);
    return 'query result';
  }
}

const db = new Database();
db.query('SELECT * FROM users'); // Establishes connection
db.query('SELECT * FROM posts'); // Uses existing connection

// 2. Event listener that fires once
console.log('\n=== Event Listener Once ===');

class EventEmitter {
  constructor() {
    this.listeners = {};
  }
  
  on(event, handler) {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event].push(handler);
  }
  
  once(event, handler) {
    const onceHandler = once((...args) => {
      handler(...args);
      this.off(event, onceHandler);
    });
    
    this.on(event, onceHandler);
  }
  
  off(event, handler) {
    if (this.listeners[event]) {
      this.listeners[event] = this.listeners[event].filter(h => h !== handler);
    }
  }
  
  emit(event, ...args) {
    if (this.listeners[event]) {
      this.listeners[event].forEach(handler => handler(...args));
    }
  }
}

const emitter = new EventEmitter();

emitter.once('start', (data) => {
  console.log('Started once:', data);
});

emitter.emit('start', 'first'); // Started once: first
emitter.emit('start', 'second'); // (no output)

// 3. Lazy initialization
console.log('\n=== Lazy Initialization ===');

class Config {
  constructor() {
    this.getConfig = once(() => {
      console.log('Loading configuration...');
      return {
        apiKey: 'secret-key',
        endpoint: 'https://api.example.com'
      };
    });
  }
  
  get(key) {
    const config = this.getConfig();
    return config[key];
  }
}

const config = new Config();
console.log('API Key:', config.get('apiKey')); // Loads config
console.log('Endpoint:', config.get('endpoint')); // Uses cached config

// 4. Payment processing (prevent double submission)
console.log('\n=== Payment Processing ===');

class PaymentProcessor {
  processPayment(amount) {
    console.log(`Processing payment of $${amount}...`);
    return { success: true, transactionId: Math.random() };
  }
  
  createOneTimeProcessor(amount) {
    return once(() => this.processPayment(amount));
  }
}

const processor = new PaymentProcessor();
const payOnce = processor.createOneTimeProcessor(100);

console.log('Submit 1:', payOnce()); // Processes payment
console.log('Submit 2:', payOnce()); // Returns cached result (no double charge)

// 5. Resource cleanup
console.log('\n=== Resource Cleanup ===');

class Resource {
  constructor() {
    this.active = true;
    this.cleanup = once(() => {
      console.log('Cleaning up resources...');
      this.active = false;
      return 'cleaned';
    });
  }
  
  use() {
    if (this.active) {
      console.log('Using resource');
    }
  }
  
  close() {
    this.cleanup(); // Safe to call multiple times
  }
}

const resource = new Resource();
resource.use();
resource.close(); // Cleaning up resources...
resource.close(); // (no output, already cleaned)
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// No arguments
const noArgs = once(() => {
  console.log('No arguments');
  return 'result';
});

console.log(noArgs()); // No arguments, "result"
console.log(noArgs()); // "result" (cached)

// Returning undefined
const returnsUndefined = once(() => {
  console.log('Returns undefined');
  return undefined;
});

console.log('First:', returnsUndefined()); // Returns undefined, undefined
console.log('Second:', returnsUndefined()); // undefined (cached)

// Throwing error
const throwsError = once(() => {
  console.log('Throwing error...');
  throw new Error('Once error');
});

try {
  throwsError();
} catch (e) {
  console.log('Caught:', e.message);
}

try {
  throwsError(); // Still cached!
} catch (e) {
  console.log('Second call caught:', e.message);
}

// Different arguments ignored
const adder = once((a, b) => {
  console.log(`Adding ${a} + ${b}`);
  return a + b;
});

console.log('5 + 3 =', adder(5, 3)); // Adding 5 + 3, 8
console.log('10 + 20 =', adder(10, 20)); // 8 (cached, args ignored)

// With 'this' context
const obj2 = {
  value: 100,
  getValue: once(function() {
    console.log('Getting value...');
    return this.value;
  })
};

console.log('Value:', obj2.getValue()); // Getting value... 100
console.log('Value again:', obj2.getValue()); // 100 (cached)

// Async error handling
const asyncError = onceAsync(async () => {
  console.log('Async operation failing...');
  throw new Error('Async error');
});

(async () => {
  try {
    await asyncError();
  } catch (e) {
    console.log('Async error 1:', e.message);
  }
  
  try {
    await asyncError(); // Same promise, same error
  } catch (e) {
    console.log('Async error 2:', e.message);
  }
})();
```

### **Comparison with Lodash**
```javascript
/**
 * Compare with lodash.once
 */
console.log('\n=== Comparison ===');

console.log(`
Feature Comparison with lodash.once:
┌────────────────────────┬──────────┬─────────┐
│ Feature                │ Custom   │ Lodash  │
├────────────────────────┼──────────┼─────────┤
│ Execute once           │ ✓        │ ✓       │
│ Cache result           │ ✓        │ ✓       │
│ Context preservation   │ ✓        │ ✓       │
│ Reset capability       │ Manual   │ ✗       │
│ Check if called        │ Manual   │ ✗       │
│ Async support          │ Manual   │ ✗       │
└────────────────────────┴──────────┴─────────┘

Usage:
  import once from 'lodash/once';
  
  const initialize = once(() => {
    console.log('initialized');
  });
  
  initialize(); // 'initialized'
  initialize(); // (no output)
`);

// Implementation comparison
function customOnce(fn) {
  let called = false;
  let result;
  
  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}

// Lodash-like implementation
function lodashStyleOnce(fn) {
  let result;
  
  function onceWrapper(...args) {
    if (fn) {
      result = fn.apply(this, args);
      fn = null; // Clear reference
    }
    return result;
  }
  
  return onceWrapper;
}

const custom = customOnce(() => 'custom');
const lodashStyle = lodashStyleOnce(() => 'lodash');

console.log('Custom:', custom(), custom());
console.log('Lodash style:', lodashStyle(), lodashStyle());
```

### **Performance Considerations**
```javascript
/**
 * Benchmark once() overhead
 */
console.log('\n=== Performance ===');

function regularFunction() {
  return 42;
}

const oncedFunction = once(() => 42);

console.time('Regular (100000 calls)');
for (let i = 0; i < 100000; i++) {
  regularFunction();
}
console.timeEnd('Regular (100000 calls)');

console.time('Onced (100000 calls)');
for (let i = 0; i < 100000; i++) {
  oncedFunction();
}
console.timeEnd('Onced (100000 calls)');

console.log('\nNote: once() adds minimal overhead, mostly on first call');
```

**Interview Tips:**
- once(): execute function only on first call, subsequent calls return cached result
- Implementation: use closure with boolean flag and result variable
- Key insight: check flag, if false execute and cache, always return cached result
- Preserve 'this' context: use Function.apply(this, args)
- Arguments on subsequent calls are ignored (uses first call's result)
- Variants: return undefined after first, reset capability, async support
- before(n): execute up to n times, then return last result
- after(n): execute only after n calls
- Applications: initialization, event listeners, lazy loading, payment processing, cleanup
- Use cases: database connection, config loading, prevent double submission
- Edge cases: no args, undefined return, errors, different args, 'this' binding
- Error handling: cached errors will be re-thrown on subsequent calls
- Async: cache the promise, not the resolved value
- Lodash once(): similar but clears function reference after first call
- Memory: stores one result, minimal overhead
- Performance: minimal overhead after first call
- Similar to memoization but simpler (no argument-based keys)
- Clarify: cache result or undefined? reset capability? async? error handling?
- Follow-ups: implement before/after, add telemetry, make thread-safe

</details>

94. Implement a sleep/delay function using promises

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Sleep Function**
```javascript
/**
 * Simple sleep/delay using setTimeout and Promise
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 */
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Test
console.log('=== Basic Sleep ===');

(async () => {
  console.log('Start:', new Date().toLocaleTimeString());
  
  await sleep(1000); // Sleep for 1 second
  
  console.log('After 1s:', new Date().toLocaleTimeString());
  
  await sleep(500);
  
  console.log('After 1.5s:', new Date().toLocaleTimeString());
})();
```

### **Approach 2: Sleep with Return Value**
```javascript
/**
 * Sleep that returns a value after delay
 */
function sleepWithValue(ms, value) {
  return new Promise(resolve => setTimeout(() => resolve(value), ms));
}

// Test
console.log('\n=== Sleep with Value ===');

(async () => {
  const result = await sleepWithValue(500, 'Hello after 500ms');
  console.log('Result:', result);
  
  const data = await sleepWithValue(300, { status: 'ready', code: 200 });
  console.log('Data:', data);
})();
```

### **Approach 3: Cancelable Sleep**
```javascript
/**
 * Sleep that can be canceled
 */
function cancelableSleep(ms) {
  let timeoutId;
  let rejectFn;
  
  const promise = new Promise((resolve, reject) => {
    rejectFn = reject;
    timeoutId = setTimeout(resolve, ms);
  });
  
  promise.cancel = () => {
    clearTimeout(timeoutId);
    rejectFn(new Error('Sleep canceled'));
  };
  
  return promise;
}

// Test
console.log('\n=== Cancelable Sleep ===');

(async () => {
  const sleepPromise = cancelableSleep(2000);
  
  setTimeout(() => {
    console.log('Canceling sleep...');
    sleepPromise.cancel();
  }, 500);
  
  try {
    await sleepPromise;
    console.log('Sleep completed');
  } catch (error) {
    console.log('Caught:', error.message);
  }
})();
```

### **Approach 4: Sleep with Progress Callback**
```javascript
/**
 * Sleep with periodic progress updates
 */
function sleepWithProgress(ms, interval = 100, onProgress) {
  return new Promise((resolve) => {
    const startTime = Date.now();
    
    const progressInterval = setInterval(() => {
      const elapsed = Date.now() - startTime;
      const progress = Math.min((elapsed / ms) * 100, 100);
      
      if (onProgress) {
        onProgress(progress, elapsed);
      }
      
      if (elapsed >= ms) {
        clearInterval(progressInterval);
        resolve();
      }
    }, interval);
  });
}

// Test
console.log('\n=== Sleep with Progress ===');

(async () => {
  console.log('Starting sleep with progress...');
  
  await sleepWithProgress(1000, 200, (progress, elapsed) => {
    console.log(`Progress: ${progress.toFixed(1)}% (${elapsed}ms)`);
  });
  
  console.log('Sleep complete!');
})();
```

### **Approach 5: Sleep with Timeout Racing**
```javascript
/**
 * Utility to race a promise against a timeout
 */
function withTimeout(promise, ms, timeoutError = 'Operation timed out') {
  const timeout = new Promise((_, reject) => 
    setTimeout(() => reject(new Error(timeoutError)), ms)
  );
  
  return Promise.race([promise, timeout]);
}

// Test
console.log('\n=== Timeout Racing ===');

async function longOperation() {
  await sleep(2000);
  return 'Operation complete';
}

(async () => {
  try {
    const result = await withTimeout(longOperation(), 1000);
    console.log('Result:', result);
  } catch (error) {
    console.log('Timeout:', error.message);
  }
})();
```

### **Bonus: Delay Decorator**
```javascript
/**
 * Decorator to add delay before function execution
 */
function delayed(ms) {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function(...args) {
      await sleep(ms);
      return originalMethod.apply(this, args);
    };
    
    return descriptor;
  };
}

// Functional version
function addDelay(fn, ms) {
  return async function(...args) {
    await sleep(ms);
    return fn.apply(this, args);
  };
}

// Test
console.log('\n=== Delayed Function ===');

function greet(name) {
  console.log(`Hello, ${name}!`);
  return `Greeted ${name}`;
}

const delayedGreet = addDelay(greet, 500);

(async () => {
  console.log('Calling delayed greet...');
  const result = await delayedGreet('Alice');
  console.log('Result:', result);
})();
```

### **Bonus: Retry with Exponential Backoff**
```javascript
/**
 * Retry operation with exponential backoff delays
 */
async function retryWithBackoff(fn, options = {}) {
  const {
    maxRetries = 3,
    initialDelay = 1000,
    maxDelay = 10000,
    factor = 2
  } = options;
  
  let lastError;
  
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      
      if (attempt < maxRetries) {
        const delay = Math.min(initialDelay * Math.pow(factor, attempt), maxDelay);
        console.log(`Attempt ${attempt + 1} failed, retrying in ${delay}ms...`);
        await sleep(delay);
      }
    }
  }
  
  throw lastError;
}

// Test
console.log('\n=== Retry with Backoff ===');

let attemptCount = 0;

async function unreliableOperation() {
  attemptCount++;
  console.log(`Attempt ${attemptCount}`);
  
  if (attemptCount < 3) {
    throw new Error('Operation failed');
  }
  
  return 'Success!';
}

(async () => {
  try {
    const result = await retryWithBackoff(unreliableOperation, {
      maxRetries: 3,
      initialDelay: 100,
      factor: 2
    });
    console.log('Final result:', result);
  } catch (error) {
    console.log('All retries failed:', error.message);
  }
})();
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Rate limiting / Throttling
console.log('\n=== Rate Limiting ===');

async function rateLimitedFetch(urls, delayBetween = 1000) {
  const results = [];
  
  for (const url of urls) {
    console.log(`Fetching ${url}...`);
    // Simulated fetch
    results.push({ url, data: `Data from ${url}` });
    
    if (urls.indexOf(url) < urls.length - 1) {
      await sleep(delayBetween);
    }
  }
  
  return results;
}

(async () => {
  const urls = ['url1', 'url2', 'url3'];
  const results = await rateLimitedFetch(urls, 300);
  console.log('All fetched:', results.length);
})();

// 2. Polling with delay
console.log('\n=== Polling ===');

async function pollUntilReady(checkFn, options = {}) {
  const { interval = 1000, maxAttempts = 10 } = options;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    console.log(`Poll attempt ${attempt}...`);
    
    const result = await checkFn();
    
    if (result) {
      return result;
    }
    
    if (attempt < maxAttempts) {
      await sleep(interval);
    }
  }
  
  throw new Error('Polling timeout');
}

(async () => {
  let ready = false;
  setTimeout(() => { ready = true; }, 2000);
  
  try {
    const result = await pollUntilReady(
      () => ready ? 'Ready!' : false,
      { interval: 500, maxAttempts: 5 }
    );
    console.log('Poll result:', result);
  } catch (error) {
    console.log('Poll failed:', error.message);
  }
})();

// 3. Debounce with delay
console.log('\n=== Debounce ===');

function debounce(fn, delay) {
  let timeoutId;
  
  return function(...args) {
    clearTimeout(timeoutId);
    
    return new Promise((resolve) => {
      timeoutId = setTimeout(() => {
        resolve(fn.apply(this, args));
      }, delay);
    });
  };
}

const search = debounce((query) => {
  console.log(`Searching for: ${query}`);
  return `Results for ${query}`;
}, 500);

(async () => {
  search('a'); // Won't execute
  search('ab'); // Won't execute
  const result = await search('abc'); // Executes after 500ms
  console.log('Search result:', result);
})();

// 4. Animation frames
console.log('\n=== Animation ===');

async function animate(duration, onFrame) {
  const startTime = Date.now();
  const fps = 60;
  const frameDelay = 1000 / fps;
  
  while (Date.now() - startTime < duration) {
    const elapsed = Date.now() - startTime;
    const progress = elapsed / duration;
    
    onFrame(progress);
    
    await sleep(frameDelay);
  }
  
  onFrame(1); // Final frame
}

(async () => {
  console.log('Starting animation...');
  
  await animate(1000, (progress) => {
    const bar = '█'.repeat(Math.floor(progress * 20));
    console.log(`[${bar.padEnd(20, '░')}] ${(progress * 100).toFixed(0)}%`);
  });
  
  console.log('Animation complete!');
})();

// 5. Sequential execution with delays
console.log('\n=== Sequential Tasks ===');

async function executeSequentially(tasks, delayBetween = 500) {
  const results = [];
  
  for (const [index, task] of tasks.entries()) {
    console.log(`Executing task ${index + 1}...`);
    const result = await task();
    results.push(result);
    
    if (index < tasks.length - 1) {
      await sleep(delayBetween);
    }
  }
  
  return results;
}

(async () => {
  const tasks = [
    () => 'Task 1 done',
    () => 'Task 2 done',
    () => 'Task 3 done'
  ];
  
  const results = await executeSequentially(tasks, 200);
  console.log('All tasks:', results);
})();
```

### **Advanced Patterns**
```javascript
/**
 * Complex delay patterns
 */

// 1. Jittered delay (randomized for distributed systems)
function sleepWithJitter(ms, jitterPercent = 0.1) {
  const jitter = ms * jitterPercent * (Math.random() * 2 - 1);
  const actualDelay = Math.max(0, ms + jitter);
  return sleep(actualDelay);
}

console.log('\n=== Jittered Sleep ===');

(async () => {
  console.time('Jittered sleep');
  await sleepWithJitter(1000, 0.2); // 800-1200ms
  console.timeEnd('Jittered sleep');
})();

// 2. Minimum delay wrapper
function withMinDelay(fn, minDelay) {
  return async function(...args) {
    const start = Date.now();
    const result = await fn(...args);
    const elapsed = Date.now() - start;
    
    if (elapsed < minDelay) {
      await sleep(minDelay - elapsed);
    }
    
    return result;
  };
}

console.log('\n=== Minimum Delay ===');

const fastFunction = () => 'instant';
const slowFunction = withMinDelay(fastFunction, 500);

(async () => {
  console.time('With min delay');
  const result = await slowFunction();
  console.timeEnd('With min delay');
  console.log('Result:', result);
})();

// 3. Delay between operations
async function delayBetween(operations, delay) {
  const results = [];
  
  for (let i = 0; i < operations.length; i++) {
    results.push(await operations[i]());
    
    if (i < operations.length - 1) {
      await sleep(delay);
    }
  }
  
  return results;
}

// 4. Batch with delays
async function batchWithDelay(items, batchSize, processBatch, delayBetween = 1000) {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    console.log(`Processing batch ${Math.floor(i / batchSize) + 1}...`);
    
    const batchResults = await processBatch(batch);
    results.push(...batchResults);
    
    if (i + batchSize < items.length) {
      await sleep(delayBetween);
    }
  }
  
  return results;
}

console.log('\n=== Batch Processing ===');

(async () => {
  const items = Array.from({ length: 10 }, (_, i) => i + 1);
  
  const results = await batchWithDelay(
    items,
    3,
    async (batch) => {
      console.log('  Batch:', batch);
      return batch.map(x => x * 2);
    },
    200
  );
  
  console.log('Final results:', results);
})();

// 5. Conditional delay
async function delayIf(condition, ms) {
  if (condition) {
    await sleep(ms);
  }
}

async function sleepUnless(condition, ms) {
  if (!condition) {
    await sleep(ms);
  }
}
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Zero delay
(async () => {
  console.log('Before zero delay');
  await sleep(0);
  console.log('After zero delay (still async)');
})();

// Negative delay (treated as 0)
(async () => {
  await sleep(-100);
  console.log('Negative delay handled');
})();

// Very long delay
(async () => {
  const longSleep = sleep(5000);
  console.log('Started long sleep');
  
  // Can cancel by not awaiting
  setTimeout(() => console.log('Meanwhile, doing other work'), 100);
})();

// Chaining delays
(async () => {
  console.log('Chaining delays...');
  
  await sleep(100)
    .then(() => console.log('After 100ms'))
    .then(() => sleep(100))
    .then(() => console.log('After 200ms'));
})();

// Promise.all with delays
(async () => {
  console.log('Parallel delays starting...');
  
  await Promise.all([
    sleep(100).then(() => console.log('Delay 1 done')),
    sleep(200).then(() => console.log('Delay 2 done')),
    sleep(150).then(() => console.log('Delay 3 done'))
  ]);
  
  console.log('All parallel delays done');
})();
```

### **Performance Considerations**
```javascript
/**
 * Memory and timing accuracy
 */
console.log('\n=== Performance Notes ===');

console.log(`
Performance Characteristics:
• setTimeout accuracy: ~4ms minimum in browsers (HTML5 spec)
• Node.js: ~1ms minimum
• Actual delay may be longer due to event loop
• Not suitable for precise timing
• Multiple timers share event loop queue
• Heavy computations block sleep resolution

Best Practices:
• Use sleep for delays, not precise timing
• Combine with requestAnimationFrame for animations
• Clear timers if operation is canceled
• Avoid creating too many concurrent timers
• Consider Worker threads for CPU-intensive work
`);

// Measuring accuracy
async function measureAccuracy(targetDelay, iterations = 5) {
  console.log(`\nMeasuring ${targetDelay}ms sleep accuracy:`);
  
  for (let i = 0; i < iterations; i++) {
    const start = Date.now();
    await sleep(targetDelay);
    const actual = Date.now() - start;
    const diff = actual - targetDelay;
    
    console.log(`  Iteration ${i + 1}: ${actual}ms (diff: +${diff}ms)`);
  }
}

setTimeout(() => measureAccuracy(100, 3), 3000);
```

### **Comparison with Alternatives**
```javascript
/**
 * Compare different delay methods
 */
console.log('\n=== Alternatives Comparison ===');

console.log(`
Method Comparison:
┌─────────────────────┬────────────┬──────────────┬────────────┐
│ Method              │ Async      │ Cancelable   │ Accuracy   │
├─────────────────────┼────────────┼──────────────┼────────────┤
│ Promise + setTimeout│ Yes        │ With wrapper │ ~4ms       │
│ async/await + sleep │ Yes        │ With wrapper │ ~4ms       │
│ setTimeout callback │ No         │ clearTimeout │ ~4ms       │
│ setInterval         │ No         │ clearInterval│ ~4ms       │
│ requestAnimationFrame│ Yes       │ cancelAnimationFrame│ 16ms│
│ process.nextTick    │ Yes (Node) │ No           │ Immediate  │
│ setImmediate        │ Yes (Node) │ clearImmediate│ Next tick│
└─────────────────────┴────────────┴──────────────┴────────────┘

Usage Guidelines:
• sleep(): General delays, rate limiting
• requestAnimationFrame(): Smooth animations
• setImmediate/nextTick: Non-blocking I/O (Node.js)
• setTimeout: Callbacks, legacy code
• setInterval: Repeated tasks (prefer recursive setTimeout)
`);
```

### **Utility Functions**
```javascript
/**
 * Collection of delay utilities
 */

// Create sleep instance with methods
const Sleep = {
  // Basic sleep
  delay: (ms) => new Promise(resolve => setTimeout(resolve, ms)),
  
  // Sleep with value
  value: (ms, val) => new Promise(resolve => setTimeout(() => resolve(val), ms)),
  
  // Sleep until condition
  until: async (condition, checkInterval = 100, timeout = 5000) => {
    const start = Date.now();
    while (!condition() && Date.now() - start < timeout) {
      await Sleep.delay(checkInterval);
    }
    if (!condition()) throw new Error('Timeout waiting for condition');
  },
  
  // Random sleep
  random: (min, max) => Sleep.delay(min + Math.random() * (max - min)),
  
  // Sleep with callback
  then: (ms, callback) => Sleep.delay(ms).then(callback)
};

console.log('\n=== Sleep Utilities ===');

(async () => {
  await Sleep.delay(100);
  console.log('Basic delay done');
  
  const val = await Sleep.value(100, 'returned value');
  console.log('Value:', val);
  
  await Sleep.random(50, 150);
  console.log('Random delay done');
})();
```

**Interview Tips:**
- sleep/delay: pause execution using setTimeout wrapped in Promise
- Basic pattern: `new Promise(resolve => setTimeout(resolve, ms))`
- async/await makes it look synchronous: `await sleep(1000)`
- No blocking: JavaScript is single-threaded, sleep uses event loop
- Cancelable: store timeout ID, clear on cancel, reject promise
- Return value: setTimeout callback can resolve with value
- Progress: use setInterval for periodic updates during delay
- Timeout: race promise against setTimeout for max wait time
- Applications: rate limiting, polling, retries, animations, sequential tasks
- Exponential backoff: increase delay between retries (1s, 2s, 4s, 8s)
- Jitter: add randomness to avoid thundering herd in distributed systems
- Minimum delay: ensure operation takes at least N ms
- Batch processing: delay between batches to avoid overwhelming systems
- Not precise: setTimeout accuracy ~4ms in browsers, affected by event loop
- Zero delay: still async, allows other events to process
- Negative delay: typically treated as 0
- Edge cases: zero delay, very long delays, multiple concurrent delays
- Alternatives: requestAnimationFrame for animations, setImmediate for Node.js
- Clarify: cancelable? progress updates? return value? timeout? accuracy requirements?
- Follow-ups: implement with AbortController, add high-resolution timing, create sleep class

</details>

95. Build a custom event emitter (pub/sub system)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Event Emitter**
```javascript
/**
 * Simple event emitter with on/emit/off
 * Time Complexity: O(n) for emit where n is number of listeners
 * Space Complexity: O(n) for stored listeners
 */
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
    return this;
  }
  
  emit(event, ...args) {
    if (!this.events[event]) {
      return false;
    }
    
    this.events[event].forEach(listener => {
      listener(...args);
    });
    
    return true;
  }
  
  off(event, listenerToRemove) {
    if (!this.events[event]) {
      return this;
    }
    
    this.events[event] = this.events[event].filter(
      listener => listener !== listenerToRemove
    );
    
    return this;
  }
}

// Test
console.log('=== Basic Event Emitter ===');

const emitter = new EventEmitter();

function onMessage(data) {
  console.log('Received:', data);
}

emitter.on('message', onMessage);
emitter.on('message', (data) => console.log('Also received:', data));

emitter.emit('message', 'Hello!'); // Both listeners fire

emitter.off('message', onMessage);
emitter.emit('message', 'World!'); // Only second listener fires
```

### **Approach 2: Event Emitter with once()**
```javascript
/**
 * Event emitter with one-time listeners
 */
class EventEmitterWithOnce extends EventEmitter {
  once(event, listener) {
    const onceWrapper = (...args) => {
      listener(...args);
      this.off(event, onceWrapper);
    };
    
    // Store original listener reference for removal
    onceWrapper.listener = listener;
    
    return this.on(event, onceWrapper);
  }
  
  off(event, listenerToRemove) {
    if (!this.events[event]) {
      return this;
    }
    
    this.events[event] = this.events[event].filter(listener => {
      return listener !== listenerToRemove && 
             listener.listener !== listenerToRemove;
    });
    
    return this;
  }
}

// Test
console.log('\n=== Event Emitter with once() ===');

const emitter2 = new EventEmitterWithOnce();

emitter2.once('startup', () => {
  console.log('App started (once)');
});

emitter2.on('startup', () => {
  console.log('App started (always)');
});

emitter2.emit('startup'); // Both fire
emitter2.emit('startup'); // Only 'always' fires
```

### **Approach 3: Event Emitter with Priority**
```javascript
/**
 * Listeners can have priority levels
 */
class PriorityEventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener, priority = 0) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    
    this.events[event].push({ listener, priority });
    
    // Sort by priority (highest first)
    this.events[event].sort((a, b) => b.priority - a.priority);
    
    return this;
  }
  
  emit(event, ...args) {
    if (!this.events[event]) {
      return false;
    }
    
    this.events[event].forEach(({ listener }) => {
      listener(...args);
    });
    
    return true;
  }
  
  off(event, listenerToRemove) {
    if (!this.events[event]) {
      return this;
    }
    
    this.events[event] = this.events[event].filter(
      ({ listener }) => listener !== listenerToRemove
    );
    
    return this;
  }
}

// Test
console.log('\n=== Priority Event Emitter ===');

const emitter3 = new PriorityEventEmitter();

emitter3.on('task', () => console.log('Priority 0 (default)'), 0);
emitter3.on('task', () => console.log('Priority 10 (high)'), 10);
emitter3.on('task', () => console.log('Priority 5 (medium)'), 5);

emitter3.emit('task'); // Fires in order: 10, 5, 0
```

### **Approach 4: Async Event Emitter**
```javascript
/**
 * Support async listeners with Promise.all
 */
class AsyncEventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
    return this;
  }
  
  async emit(event, ...args) {
    if (!this.events[event]) {
      return false;
    }
    
    await Promise.all(
      this.events[event].map(listener => listener(...args))
    );
    
    return true;
  }
  
  off(event, listenerToRemove) {
    if (!this.events[event]) {
      return this;
    }
    
    this.events[event] = this.events[event].filter(
      listener => listener !== listenerToRemove
    );
    
    return this;
  }
  
  // Sequential async emit
  async emitSerial(event, ...args) {
    if (!this.events[event]) {
      return false;
    }
    
    for (const listener of this.events[event]) {
      await listener(...args);
    }
    
    return true;
  }
}

// Test
console.log('\n=== Async Event Emitter ===');

const asyncEmitter = new AsyncEventEmitter();

asyncEmitter.on('process', async (data) => {
  await new Promise(r => setTimeout(r, 100));
  console.log('Async listener 1:', data);
});

asyncEmitter.on('process', async (data) => {
  await new Promise(r => setTimeout(r, 50));
  console.log('Async listener 2:', data);
});

(async () => {
  console.log('Emitting...');
  await asyncEmitter.emit('process', 'data');
  console.log('All async listeners completed');
})();
```

### **Approach 5: Feature-Complete Event Emitter**
```javascript
/**
 * Production-ready event emitter with all features
 */
class AdvancedEventEmitter {
  constructor(options = {}) {
    this.events = {};
    this.maxListeners = options.maxListeners || 10;
    this.wildcard = options.wildcard || false;
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    
    if (this.events[event].length >= this.maxListeners) {
      console.warn(`Warning: Max listeners (${this.maxListeners}) exceeded for event "${event}"`);
    }
    
    this.events[event].push(listener);
    this.emit('newListener', event, listener);
    
    return this;
  }
  
  once(event, listener) {
    const onceWrapper = (...args) => {
      listener(...args);
      this.off(event, onceWrapper);
    };
    
    onceWrapper.listener = listener;
    return this.on(event, onceWrapper);
  }
  
  emit(event, ...args) {
    let fired = false;
    
    // Fire specific event listeners
    if (this.events[event]) {
      this.events[event].forEach(listener => {
        listener(...args);
      });
      fired = true;
    }
    
    // Fire wildcard listeners
    if (this.wildcard && this.events['*']) {
      this.events['*'].forEach(listener => {
        listener(event, ...args);
      });
      fired = true;
    }
    
    return fired;
  }
  
  off(event, listenerToRemove) {
    if (!listenerToRemove) {
      // Remove all listeners for event
      delete this.events[event];
      return this;
    }
    
    if (!this.events[event]) {
      return this;
    }
    
    this.events[event] = this.events[event].filter(listener => {
      const match = listener === listenerToRemove || 
                    listener.listener === listenerToRemove;
      
      if (match) {
        this.emit('removeListener', event, listener);
      }
      
      return !match;
    });
    
    return this;
  }
  
  removeAllListeners(event) {
    if (event) {
      delete this.events[event];
    } else {
      this.events = {};
    }
    
    return this;
  }
  
  listenerCount(event) {
    return this.events[event] ? this.events[event].length : 0;
  }
  
  listeners(event) {
    return this.events[event] ? [...this.events[event]] : [];
  }
  
  eventNames() {
    return Object.keys(this.events);
  }
  
  setMaxListeners(n) {
    this.maxListeners = n;
    return this;
  }
}

// Test
console.log('\n=== Advanced Event Emitter ===');

const advanced = new AdvancedEventEmitter({ wildcard: true });

advanced.on('newListener', (event) => {
  console.log(`New listener added for: ${event}`);
});

advanced.on('data', (data) => console.log('Data:', data));
advanced.on('data', (data) => console.log('Data again:', data));

console.log('Listener count:', advanced.listenerCount('data'));
console.log('Event names:', advanced.eventNames());

advanced.emit('data', 'test');

// Wildcard
advanced.on('*', (event, ...args) => {
  console.log(`Wildcard caught: ${event}`, args);
});

advanced.emit('custom', 'anything');
```

### **Bonus: Pub/Sub Pattern**
```javascript
/**
 * Pub/Sub with topics and subscriptions
 */
class PubSub {
  constructor() {
    this.subscribers = {};
    this.subId = 0;
  }
  
  subscribe(topic, callback) {
    if (!this.subscribers[topic]) {
      this.subscribers[topic] = {};
    }
    
    const id = this.subId++;
    this.subscribers[topic][id] = callback;
    
    // Return unsubscribe function
    return () => {
      delete this.subscribers[topic][id];
    };
  }
  
  publish(topic, data) {
    if (!this.subscribers[topic]) {
      return 0;
    }
    
    let count = 0;
    
    Object.values(this.subscribers[topic]).forEach(callback => {
      callback(data);
      count++;
    });
    
    return count;
  }
  
  clear(topic) {
    if (topic) {
      delete this.subscribers[topic];
    } else {
      this.subscribers = {};
    }
  }
}

// Test
console.log('\n=== Pub/Sub Pattern ===');

const pubsub = new PubSub();

const unsubscribe1 = pubsub.subscribe('news', (data) => {
  console.log('Subscriber 1 got:', data);
});

const unsubscribe2 = pubsub.subscribe('news', (data) => {
  console.log('Subscriber 2 got:', data);
});

pubsub.publish('news', 'Breaking news!'); // Both receive

unsubscribe1(); // Unsubscribe first

pubsub.publish('news', 'More news'); // Only subscriber 2 receives
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Application state management
console.log('\n=== State Management ===');

class Store extends EventEmitter {
  constructor(initialState = {}) {
    super();
    this.state = initialState;
  }
  
  getState() {
    return { ...this.state };
  }
  
  setState(updates) {
    const oldState = this.getState();
    this.state = { ...this.state, ...updates };
    this.emit('stateChange', this.state, oldState);
  }
}

const store = new Store({ count: 0 });

store.on('stateChange', (newState, oldState) => {
  console.log('State changed:', { old: oldState.count, new: newState.count });
});

store.setState({ count: 1 });
store.setState({ count: 2 });

// 2. HTTP Server-like interface
console.log('\n=== Server Interface ===');

class Server extends EventEmitter {
  constructor() {
    super();
    this.running = false;
  }
  
  listen(port) {
    this.running = true;
    this.emit('listening', port);
    
    // Simulate requests
    setTimeout(() => this.emit('request', { url: '/api/users', method: 'GET' }), 100);
  }
  
  close() {
    this.running = false;
    this.emit('close');
  }
}

const server = new Server();

server.on('listening', (port) => {
  console.log(`Server listening on port ${port}`);
});

server.on('request', (req) => {
  console.log(`Request: ${req.method} ${req.url}`);
});

server.on('close', () => {
  console.log('Server closed');
});

server.listen(3000);
setTimeout(() => server.close(), 500);

// 3. Real-time chat
console.log('\n=== Chat System ===');

class ChatRoom extends EventEmitter {
  constructor(name) {
    super();
    this.name = name;
    this.users = [];
  }
  
  join(user) {
    this.users.push(user);
    this.emit('userJoined', user);
  }
  
  leave(user) {
    this.users = this.users.filter(u => u !== user);
    this.emit('userLeft', user);
  }
  
  sendMessage(user, message) {
    this.emit('message', { user, message, timestamp: Date.now() });
  }
}

const room = new ChatRoom('General');

room.on('userJoined', (user) => {
  console.log(`${user} joined the chat`);
});

room.on('message', ({ user, message }) => {
  console.log(`${user}: ${message}`);
});

room.join('Alice');
room.join('Bob');
room.sendMessage('Alice', 'Hello everyone!');
room.sendMessage('Bob', 'Hi Alice!');

// 4. File watcher simulation
console.log('\n=== File Watcher ===');

class FileWatcher extends EventEmitter {
  constructor() {
    super();
    this.watching = new Set();
  }
  
  watch(filename) {
    this.watching.add(filename);
    this.emit('watching', filename);
  }
  
  simulateChange(filename, type = 'change') {
    if (this.watching.has(filename)) {
      this.emit(type, filename);
    }
  }
  
  unwatch(filename) {
    this.watching.delete(filename);
    this.emit('unwatched', filename);
  }
}

const watcher = new FileWatcher();

watcher.on('change', (file) => {
  console.log(`File changed: ${file}`);
});

watcher.on('delete', (file) => {
  console.log(`File deleted: ${file}`);
});

watcher.watch('app.js');
watcher.simulateChange('app.js', 'change');
watcher.simulateChange('app.js', 'delete');

// 5. Task queue with events
console.log('\n=== Task Queue ===');

class TaskQueue extends EventEmitter {
  constructor() {
    super();
    this.queue = [];
    this.processing = false;
  }
  
  add(task) {
    this.queue.push(task);
    this.emit('taskAdded', task);
    this.process();
  }
  
  async process() {
    if (this.processing || this.queue.length === 0) {
      return;
    }
    
    this.processing = true;
    
    while (this.queue.length > 0) {
      const task = this.queue.shift();
      
      this.emit('taskStart', task);
      
      try {
        await task();
        this.emit('taskComplete', task);
      } catch (error) {
        this.emit('taskError', task, error);
      }
    }
    
    this.processing = false;
    this.emit('queueEmpty');
  }
}

const queue = new TaskQueue();

queue.on('taskComplete', () => console.log('Task completed'));
queue.on('queueEmpty', () => console.log('Queue empty'));

queue.add(async () => {
  await new Promise(r => setTimeout(r, 100));
  console.log('Task 1 executed');
});

queue.add(async () => {
  console.log('Task 2 executed');
});
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

const edge = new EventEmitter();

// No listeners
console.log('Emit with no listeners:', edge.emit('nonexistent')); // false

// Multiple off calls
const listener = () => console.log('test');
edge.on('test', listener);
edge.off('test', listener);
edge.off('test', listener); // No error

// Remove all for event
edge.on('test', () => console.log('1'));
edge.on('test', () => console.log('2'));
edge.events.test = [];
edge.emit('test'); // No output

// Emit during emit
edge.on('recursive', function() {
  console.log('Recursive emit');
  if (!this.recursiveCount) this.recursiveCount = 0;
  if (this.recursiveCount++ < 2) {
    edge.emit('recursive');
  }
});

edge.emit('recursive');

// Listener modifies listeners array
edge.on('modify', () => {
  console.log('Modifying during emit');
  edge.off('modify');
});

edge.emit('modify');
edge.emit('modify'); // No output

// Error in listener
edge.on('error', () => {
  throw new Error('Listener error');
});

try {
  edge.emit('error');
} catch (e) {
  console.log('Caught listener error:', e.message);
}
```

### **Performance Considerations**
```javascript
/**
 * Benchmark event emitter
 */
console.log('\n=== Performance ===');

const perfEmitter = new EventEmitter();

// Add many listeners
for (let i = 0; i < 100; i++) {
  perfEmitter.on('test', () => {});
}

console.time('Emit with 100 listeners');
for (let i = 0; i < 1000; i++) {
  perfEmitter.emit('test');
}
console.timeEnd('Emit with 100 listeners');

console.log(`
Performance Notes:
• O(n) time for emit (n = number of listeners)
• Array iteration is fast for small n
• Consider Set for large listener counts
• WeakMap for memory efficiency with objects
• Async listeners add Promise overhead
• Deep event hierarchies slow down wildcards
• Memory leaks if listeners not removed
`);
```

**Interview Tips:**
- Event emitter: pub/sub pattern for decoupled communication
- Core methods: on (subscribe), emit (publish), off (unsubscribe)
- Store listeners in object: { eventName: [listener1, listener2] }
- on(): add listener to array for event name
- emit(): call all listeners for event with provided arguments
- off(): filter out listener from array
- once(): wrapper that removes itself after first call
- Return 'this' for method chaining
- Priority: sort listeners by priority before emit
- Async: use Promise.all or sequential await
- Wildcard: '*' event catches all emissions
- Max listeners: warn when too many (potential memory leak)
- Applications: state management, servers, chat, file watchers, queues
- Node.js EventEmitter: built-in with many features
- Pub/Sub: similar but with topic-based subscriptions
- Edge cases: no listeners, remove during emit, errors in listeners
- Memory leaks: always remove listeners when done
- Performance: O(n) for emit, consider Set for large listener counts
- Clarify: sync or async? priority? wildcard? max listeners? Node.js compatible?
- Follow-ups: add namespaces, implement middleware, add error event, make thread-safe

</details>
