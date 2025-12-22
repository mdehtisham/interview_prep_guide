# JavaScript Coding Interview Questions - Medium Level

This section contains a collection of medium-level JavaScript coding questions.

---

## **MEDIUM LEVEL (31-60)**

### **Arrays & Strings**

31. **Rotate Array**: Given an array, rotate the array to the right by `k` steps, where `k` is non-negative.

<details>
<summary><b>Solution</b></summary>

**Approach 1: Using `pop()` and `unshift()` (Brute Force)**
- **Time Complexity**: O(n * k) - `unshift` is O(n) and we do it `k` times.
- **Space Complexity**: O(1) - In-place modification.

```javascript
function rotateArray(nums, k) {
  if (!nums || nums.length === 0) return;

  // Reduce k to be within the bounds of the array length
  k = k % nums.length;

  for (let i = 0; i < k; i++) {
    // Take the last element and add it to the front
    nums.unshift(nums.pop());
  }
  return nums;
}

// Example:
console.log(rotateArray([1, 2, 3, 4, 5, 6, 7], 3)); // [5, 6, 7, 1, 2, 3, 4]
```

**Approach 2: Using `slice()` and Spread Operator (More Efficient)**
- **Time Complexity**: O(n) - `slice` and spread are both O(n).
- **Space Complexity**: O(n) - A new array is created.

```javascript
function rotateArray(nums, k) {
  if (!nums || nums.length === 0) return [];
  
  const n = nums.length;
  k = k % n;
  
  // Slice the array into two parts and concatenate them in reverse order
  const rotatedPart = nums.slice(n - k);
  const remainingPart = nums.slice(0, n - k);
  
  return [...rotatedPart, ...remainingPart];
}
```

**Approach 3: Reverse Technique (Most Optimal)**
- **Time Complexity**: O(n) - Three passes over the array.
- **Space Complexity**: O(1) - In-place modification.

This is the most common and expected solution in an interview.
1. Reverse the entire array.
2. Reverse the first `k` elements.
3. Reverse the remaining `n-k` elements.

```javascript
function reverse(arr, start, end) {
  while (start < end) {
    [arr[start], arr[end]] = [arr[end], arr[start]]; // Swap elements
    start++;
    end--;
  }
}

function rotateArray(nums, k) {
  const n = nums.length;
  if (n === 0) return;
  k = k % n;

  // 1. Reverse the entire array
  reverse(nums, 0, n - 1);
  // 2. Reverse the first k elements
  reverse(nums, 0, k - 1);
  // 3. Reverse the rest of the array
  reverse(nums, k, n - 1);

  return nums;
}
```
</details>

32. **Group Anagrams**: Given an array of strings, group the anagrams together.

<details>
<summary><b>Solution</b></summary>

**Approach: Using a Hash Map with Sorted Strings as Keys**
- **Time Complexity**: O(n * k log k), where `n` is the number of strings and `k` is the maximum length of a string. Sorting each string takes `k log k`.
- **Space Complexity**: O(n * k) - to store the grouped anagrams.

```javascript
function groupAnagrams(strs) {
  const anagramMap = new Map();

  for (const str of strs) {
    // Sort the string to create a unique key for anagrams
    const sortedStr = str.split('').sort().join('');
    
    // If the key doesn't exist, initialize it with an empty array
    if (!anagramMap.has(sortedStr)) {
      anagramMap.set(sortedStr, []);
    }
    
    // Push the original string to the corresponding group
    anagramMap.get(sortedStr).push(str);
  }

  // The values of the map are the grouped anagrams
  return Array.from(anagramMap.values());
}

// Example:
console.log(groupAnagrams(["eat", "tea", "tan", "ate", "nat", "bat"]));
// Output: [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
```
</details>

33. **Longest Substring Without Repeating Characters**: Find the length of the longest substring without repeating characters.

<details>
<summary><b>Solution</b></summary>

**Approach: Sliding Window with a Set**
- **Time Complexity**: O(n) - Each character is visited at most twice (once by `right` pointer, once by `left` pointer).
- **Space Complexity**: O(min(n, m)) - Where `n` is the string length and `m` is the size of the character set. The space is for the `Set`.

```javascript
function lengthOfLongestSubstring(s) {
  let maxLength = 0;
  let left = 0; // Left pointer of the window
  const charSet = new Set();

  for (let right = 0; right < s.length; right++) {
    // While the character is already in the set, shrink the window from the left
    while (charSet.has(s[right])) {
      charSet.delete(s[left]);
      left++;
    }
    
    // Add the current character to the set and expand the window
    charSet.add(s[right]);
    
    // Update the max length found so far
    maxLength = Math.max(maxLength, right - left + 1);
  }

  return maxLength;
}

// Example:
console.log(lengthOfLongestSubstring("abcabcbb")); // 3 (for "abc")
console.log(lengthOfLongestSubstring("bbbbb"));    // 1 (for "b")
console.log(lengthOfLongestSubstring("pwwkew"));   // 3 (for "wke")
```
</details>

### **Linked Lists**

34. **Reverse a Linked List**: Reverse a singly linked list.

<details>
<summary><b>Solution</b></summary>

**Approach 1: Iterative**
- **Time Complexity**: O(n)
- **Space Complexity**: O(1)

```javascript
class ListNode {
  constructor(val, next = null) {
    this.val = val;
    this.next = next;
  }
}

function reverseList(head) {
  let prev = null;
  let current = head;

  while (current !== null) {
    const nextTemp = current.next; // Store the next node
    current.next = prev;           // Reverse the link
    prev = current;                // Move prev to current
    current = nextTemp;            // Move to the next node in the original list
  }

  return prev; // `prev` is the new head
}
```

**Approach 2: Recursive**
- **Time Complexity**: O(n)
- **Space Complexity**: O(n) - due to the recursion stack.

```javascript
function reverseListRecursive(head) {
  // Base case: if head is null or it's the last node
  if (head === null || head.next === null) {
    return head;
  }
  
  // Recursively reverse the rest of the list
  const reversedHead = reverseListRecursive(head.next);
  
  // Reverse the link for the current node
  head.next.next = head;
  head.next = null;
  
  return reversedHead;
}
```
</details>

35. **Detect Cycle in Linked List**: Determine if a linked list has a cycle in it.

<details>
<summary><b>Solution</b></summary>

**Approach: Floyd's Tortoise and Hare Algorithm**
- **Time Complexity**: O(n)
- **Space Complexity**: O(1)

This algorithm uses two pointers, one moving slow (one step at a time) and one moving fast (two steps at a time). If there's a cycle, they will eventually meet.

```javascript
function hasCycle(head) {
  if (!head || !head.next) {
    return false;
  }

  let slow = head;
  let fast = head.next;

  while (slow !== fast) {
    // If fast reaches the end, there's no cycle
    if (!fast || !fast.next) {
      return false;
    }
    slow = slow.next;
    fast = fast.next.next;
  }

  return true; // Pointers met, so there's a cycle
}
```
</details>

### **Trees**

36. **Invert Binary Tree**: Invert a binary tree (mirror it).

<details>
<summary><b>Solution</b></summary>

**Approach 1: Recursive (DFS)**
- **Time Complexity**: O(n) - visiting each node once.
- **Space Complexity**: O(h) - where `h` is the height of the tree, for the recursion stack.

```javascript
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

function invertTree(root) {
  if (root === null) {
    return null;
  }

  // Swap the left and right children
  const temp = root.left;
  root.left = root.right;
  root.right = temp;

  // Recursively invert the left and right subtrees
  invertTree(root.left);
  invertTree(root.right);

  return root;
}
```

**Approach 2: Iterative (BFS)**
- **Time Complexity**: O(n)
- **Space Complexity**: O(w) - where `w` is the maximum width of the tree, for the queue.

```javascript
function invertTreeIterative(root) {
  if (root === null) return null;
  
  const queue = [root];

  while (queue.length > 0) {
    const node = queue.shift();

    // Swap children
    [node.left, node.right] = [node.right, node.left];

    // Add children to the queue if they exist
    if (node.left) queue.push(node.left);
    if (node.right) queue.push(node.right);
  }

  return root;
}
```
</details>

37. **Maximum Depth of Binary Tree**: Find the maximum depth (height) of a binary tree.

<details>
<summary><b>Solution</b></summary>

**Approach 1: Recursive (DFS)**
- **Time Complexity**: O(n)
- **Space Complexity**: O(h) - for recursion stack.

```javascript
function maxDepth(root) {
  if (root === null) {
    return 0; // Base case
  }
  
  const leftDepth = maxDepth(root.left);
  const rightDepth = maxDepth(root.right);
  
  // Depth is 1 (for the current node) + max of depths of subtrees
  return Math.max(leftDepth, rightDepth) + 1;
}
```

**Approach 2: Iterative (BFS)**
- **Time Complexity**: O(n)
- **Space Complexity**: O(w) - for the queue.

```javascript
function maxDepthBFS(root) {
  if (!root) return 0;
  
  let depth = 0;
  const queue = [root];
  
  while (queue.length > 0) {
    const levelSize = queue.length;
    
    // Process all nodes at the current level
    for (let i = 0; i < levelSize; i++) {
      const node = queue.shift();
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    
    depth++; // Increment depth after processing each level
  }
  
  return depth;
}
```
</details>
