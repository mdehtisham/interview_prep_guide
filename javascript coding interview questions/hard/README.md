# JavaScript Coding Interview Questions - Hard Level

This section contains a collection of hard-level JavaScript coding questions.

---

## **HARD LEVEL (61-90)**

### **Advanced Algorithms**

61. **Trapping Rain Water**: Given `n` non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it can trap after raining.

<details>
<summary><b>Solution</b></summary>

**Approach 1: Brute Force**
- For each element, find the highest bar on its left and right.
- The water trapped at the current bar is `min(leftMax, rightMax) - height[i]`.
- **Time Complexity**: O(n^2) - For each bar, we iterate to find left and right maxes.
- **Space Complexity**: O(1)

```javascript
function trapBruteForce(height) {
  let totalWater = 0;
  for (let i = 0; i < height.length; i++) {
    let leftMax = 0;
    for (let j = 0; j <= i; j++) {
      leftMax = Math.max(leftMax, height[j]);
    }
    
    let rightMax = 0;
    for (let j = i; j < height.length; j++) {
      rightMax = Math.max(rightMax, height[j]);
    }
    
    totalWater += Math.min(leftMax, rightMax) - height[i];
  }
  return totalWater;
}
```

**Approach 2: Dynamic Programming (Pre-computation)**
- Pre-compute `leftMax` and `rightMax` for each position in two separate arrays.
- **Time Complexity**: O(n) - Three passes (one for left maxes, one for right maxes, one for calculation).
- **Space Complexity**: O(n) - For the two DP arrays.

```javascript
function trapDP(height) {
  if (!height || height.length === 0) return 0;
  
  const n = height.length;
  const leftMax = new Array(n).fill(0);
  const rightMax = new Array(n).fill(0);
  let totalWater = 0;
  
  // Fill leftMax array
  leftMax[0] = height[0];
  for (let i = 1; i < n; i++) {
    leftMax[i] = Math.max(height[i], leftMax[i - 1]);
  }
  
  // Fill rightMax array
  rightMax[n - 1] = height[n - 1];
  for (let i = n - 2; i >= 0; i--) {
    rightMax[i] = Math.max(height[i], rightMax[i + 1]);
  }
  
  // Calculate trapped water
  for (let i = 0; i < n; i++) {
    totalWater += Math.min(leftMax[i], rightMax[i]) - height[i];
  }
  
  return totalWater;
}
```

**Approach 3: Two Pointers (Optimal)**
- Use two pointers, `left` and `right`, starting from the ends.
- Keep track of `leftMax` and `rightMax` as you move the pointers.
- If `height[left]` is smaller than `height[right]`, the bottleneck is the left side. The water trapped depends on `leftMax`. Move `left`.
- Otherwise, the bottleneck is the right side. The water trapped depends on `rightMax`. Move `right`.
- **Time Complexity**: O(n) - Single pass.
- **Space Complexity**: O(1)

```javascript
function trap(height) {
  let left = 0, right = height.length - 1;
  let leftMax = 0, rightMax = 0;
  let totalWater = 0;

  while (left < right) {
    if (height[left] < height[right]) {
      // Water level is determined by the shorter wall
      if (height[left] >= leftMax) {
        leftMax = height[left];
      } else {
        totalWater += leftMax - height[left];
      }
      left++;
    } else {
      if (height[right] >= rightMax) {
        rightMax = height[right];
      } else {
        totalWater += rightMax - height[right];
      }
      right--;
    }
  }
  return totalWater;
}

// Example:
console.log(trap([0,1,0,2,1,0,1,3,2,1,2,1])); // 6
```
</details>

62. **Median of Two Sorted Arrays**: Find the median of two sorted arrays. The overall run time complexity should be O(log (m+n)).

<details>
<summary><b>Solution</b></summary>

**Approach: Binary Search on the Smaller Array**
- This is a very complex algorithm, often considered one of the hardest LeetCode-style questions.
- The key idea is to partition both arrays such that all elements in the "left part" are less than or equal to all elements in the "right part".
- The median will be derived from the boundary elements of these partitions.
- We perform a binary search on the smaller array to find the correct partition point.

- **Time Complexity**: O(log(min(m, n)))
- **Space Complexity**: O(1)

```javascript
function findMedianSortedArrays(nums1, nums2) {
  // Ensure nums1 is the smaller array to optimize binary search
  if (nums1.length > nums2.length) {
    [nums1, nums2] = [nums2, nums1];
  }

  const m = nums1.length;
  const n = nums2.length;
  let low = 0;
  let high = m;
  const halfLen = Math.floor((m + n + 1) / 2);

  while (low <= high) {
    const partitionX = Math.floor((low + high) / 2);
    const partitionY = halfLen - partitionX;

    // Get the boundary elements
    const maxX = (partitionX === 0) ? -Infinity : nums1[partitionX - 1];
    const minX = (partitionX === m) ? Infinity : nums1[partitionX];

    const maxY = (partitionY === 0) ? -Infinity : nums2[partitionY - 1];
    const minY = (partitionY === n) ? Infinity : nums2[partitionY];

    if (maxX <= minY && maxY <= minX) {
      // Correct partition found, calculate median
      if ((m + n) % 2 === 0) {
        // Even number of elements
        return (Math.max(maxX, maxY) + Math.min(minX, minY)) / 2;
      } else {
        // Odd number of elements
        return Math.max(maxX, maxY);
      }
    } else if (maxX > minY) {
      // PartitionX is too large, move left
      high = partitionX - 1;
    } else {
      // PartitionX is too small, move right
      low = partitionX + 1;
    }
  }
  
  throw new Error("Input arrays are not sorted.");
}

// Example:
console.log(findMedianSortedArrays([1, 3], [2])); // 2.0
console.log(findMedianSortedArrays([1, 2], [3, 4])); // 2.5
```
</details>

63. **Word Ladder**: Find the length of the shortest transformation sequence from a `beginWord` to an `endWord`, such that only one letter can be changed at a time, and each transformed word must exist in a `wordList`.

<details>
<summary><b>Solution</b></summary>

**Approach: Breadth-First Search (BFS)**
- This is a shortest path problem on an unweighted graph, so BFS is the ideal algorithm.
- The "nodes" of the graph are the words. An "edge" exists between two words if they are one letter apart.
- We start a BFS from the `beginWord`, exploring its neighbors, and their neighbors, level by level, until we find the `endWord`.

- **Time Complexity**: O(M^2 * N), where M is the length of each word and N is the number of words in the list. For each word, we generate M new words by changing one character, which takes O(M).
- **Space Complexity**: O(M * N) for the queue and visited set.

```javascript
function ladderLength(beginWord, endWord, wordList) {
  const wordSet = new Set(wordList);
  if (!wordSet.has(endWord)) return 0;

  const queue = [[beginWord, 1]]; // [word, length]
  const visited = new Set([beginWord]);

  while (queue.length > 0) {
    const [currentWord, level] = queue.shift();

    if (currentWord === endWord) {
      return level;
    }

    // Find all possible next words (neighbors)
    for (let i = 0; i < currentWord.length; i++) {
      for (let j = 0; j < 26; j++) {
        const newChar = String.fromCharCode(97 + j); // 'a' to 'z'
        const nextWord = currentWord.slice(0, i) + newChar + currentWord.slice(i + 1);

        if (wordSet.has(nextWord) && !visited.has(nextWord)) {
          visited.add(nextWord);
          queue.push([nextWord, level + 1]);
        }
      }
    }
  }

  return 0; // End word not reachable
}

// Example:
const beginWord = "hit";
const endWord = "cog";
const wordList = ["hot","dot","dog","lot","log","cog"];
console.log(ladderLength(beginWord, endWord, wordList)); // 5 (hit -> hot -> dot -> dog -> cog)
```
</details>

### **System Design & Concurrency**

64. **Implement `Promise.all()`**: Implement a function that behaves like `Promise.all()`.

<details>
<summary><b>Solution</b></summary>

**Logic**
- The function takes an array of promises.
- It returns a single promise that resolves when all input promises have resolved. The resolved value is an array of the results.
- If any input promise rejects, the returned promise immediately rejects with the reason of the first promise that rejected.

```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completedCount = 0;
    
    if (promises.length === 0) {
      resolve(results);
      return;
    }

    promises.forEach((promise, index) => {
      // Ensure items are treated as promises
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completedCount++;

          // If all promises are resolved, resolve the main promise
          if (completedCount === promises.length) {
            resolve(results);
          }
        })
        .catch(error => {
          // If any promise rejects, reject the main promise immediately
          reject(error);
        });
    });
  });
}

// Example Usage:
const p1 = Promise.resolve(3);
const p2 = 42;
const p3 = new Promise((resolve) => setTimeout(() => resolve("foo"), 100));

promiseAll([p1, p2, p3]).then(values => {
  console.log(values); // [3, 42, "foo"]
});

const p4 = Promise.reject("Error!");
promiseAll([p1, p4, p3]).catch(error => {
  console.error(error); // "Error!"
});
```
</details>

65. **Implement a Debounce Function**: Create a `debounce` function.

<details>
<summary><b>Solution</b></summary>

**Logic**
- Debounce ensures that a function is not called until a certain amount of time has passed without it being called again.
- It's useful for performance-heavy tasks like handling search input, window resizing, etc.
- The function will take another function (`func`) and a delay (`delay`) as arguments.
- It returns a new function that, when called, will reset a timer. The original `func` is only executed after the timer completes.

```javascript
function debounce(func, delay) {
  let timeoutId;

  // Return a new function that wraps the original function
  return function(...args) {
    // `this` context and arguments of the last call
    const context = this; 

    // Clear the previous timeout to reset the delay
    clearTimeout(timeoutId);

    // Set a new timeout
    timeoutId = setTimeout(() => {
      // Execute the original function with the saved context and arguments
      func.apply(context, args);
    }, delay);
  };
}

// Example Usage:
function handleInput(e) {
  console.log("Searching for:", e.target.value);
}

const debouncedHandleInput = debounce(handleInput, 500);

// In a real scenario, you would attach this to an input's 'keyup' or 'input' event
// For demonstration:
// inputElement.addEventListener('input', debouncedHandleInput);
```
</details>

66. **Implement a Throttle Function**: Create a `throttle` function.

<details>
<summary><b>Solution</b></summary>

**Logic**
- Throttle ensures that a function is called at most once in a specified time period.
- It's useful for rate-limiting events like scrolling or mouse movement.
- The function will take another function (`func`) and a limit (`limit`) as arguments.
- It returns a new function that, when called, will only execute `func` if the time limit has passed since the last execution.

```javascript
function throttle(func, limit) {
  let inThrottle = false;

  return function(...args) {
    const context = this;

    if (!inThrottle) {
      // Execute the function
      func.apply(context, args);
      
      // Set the flag to prevent further executions
      inThrottle = true;
      
      // After the limit, reset the flag
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Example Usage:
function handleScroll() {
  console.log("Scroll event processed!");
}

const throttledHandleScroll = throttle(handleScroll, 1000);

// In a real scenario, you would attach this to the window's 'scroll' event
// For demonstration:
// window.addEventListener('scroll', throttledHandleScroll);
```
</details>
