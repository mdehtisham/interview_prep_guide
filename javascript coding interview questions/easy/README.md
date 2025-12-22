# JavaScript Coding Interview Questions - Easy Level

This section contains a collection of easy-level JavaScript coding questions.

---

## **EASY LEVEL (1-30)**

### **Arrays & Strings**

1. Reverse a string without using built-in reverse method

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using a Loop (Brute Force)**
```javascript
/**
 * Reverse a string using traditional for loop
 * Time Complexity: O(n) - iterate through entire string
 * Space Complexity: O(n) - create new string
 * 
 * @param {string} str - Input string to reverse
 * @returns {string} - Reversed string
 */
function reverseString(str) {
  // Handle edge cases
  if (!str || str.length === 0) return str;
  
  let reversed = '';
  
  // Iterate from end to beginning
  for (let i = str.length - 1; i >= 0; i--) {
    reversed += str[i];
  }
  
  return reversed;
}

// Test cases
console.log(reverseString('hello'));        // 'olleh'
console.log(reverseString('JavaScript'));   // 'tpircSavaJ'
console.log(reverseString(''));             // ''
console.log(reverseString('a'));            // 'a'
```

### **Approach 2: Using Array Methods (Optimized)**
```javascript
/**
 * Reverse string using array conversion
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Most readable and commonly used in production
 */
function reverseString(str) {
  if (!str) return str;
  
  // Split to array, reverse, join back
  return str.split('').reverse().join('');
}

// Test cases
console.log(reverseString('hello'));        // 'olleh'
console.log(reverseString('JavaScript'));   // 'tpircSavaJ'
```

### **Approach 3: Using Recursion**
```javascript
/**
 * Reverse string using recursion
 * Time Complexity: O(n)
 * Space Complexity: O(n) - due to call stack
 * 
 * Good for demonstrating recursion understanding
 */
function reverseString(str) {
  // Base case: empty or single character
  if (str.length <= 1) return str;
  
  // Recursive case: last char + reverse of remaining string
  return str[str.length - 1] + reverseString(str.slice(0, -1));
}

// Test cases
console.log(reverseString('hello'));        // 'olleh'
console.log(reverseString('JavaScript'));   // 'tpircSavaJ'
```

### **Approach 4: Two-Pointer In-Place (Most Efficient for Arrays)**
```javascript
/**
 * Reverse string in-place using two pointers
 * Time Complexity: O(n)
 * Space Complexity: O(1) - if working with array
 * 
 * Most efficient for character arrays
 */
function reverseString(str) {
  if (!str) return str;
  
  // Convert to array for in-place modification
  const arr = str.split('');
  let left = 0;
  let right = arr.length - 1;
  
  // Swap characters from both ends moving towards center
  while (left < right) {
    // Swap using destructuring
    [arr[left], arr[right]] = [arr[right], arr[left]];
    left++;
    right--;
  }
  
  return arr.join('');
}

// Test cases
console.log(reverseString('hello'));        // 'olleh'
console.log(reverseString('JavaScript'));   // 'tpircSavaJ'
```

### **Production-Grade Solution with Validation**
```javascript
/**
 * Production-ready string reversal with comprehensive error handling
 * 
 * @param {string} str - Input string to reverse
 * @returns {string} - Reversed string
 * @throws {TypeError} - If input is not a string
 */
function reverseString(str) {
  // Type validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  // Handle empty string
  if (str.length === 0) return str;
  
  // Handle single character (optimization)
  if (str.length === 1) return str;
  
  // Use built-in methods for best performance and readability
  return str.split('').reverse().join('');
}

// Comprehensive test suite
try {
  console.log(reverseString('hello'));           // 'olleh'
  console.log(reverseString('JavaScript'));      // 'tpircSavaJ'
  console.log(reverseString(''));                // ''
  console.log(reverseString('a'));               // 'a'
  console.log(reverseString('A man a plan'));    // 'nalp a nam A'
  console.log(reverseString(123));               // TypeError
} catch (error) {
  console.error(error.message);
}
```

**Interview Tips:**
- Start with brute force approach, then optimize
- Mention that strings are immutable in JavaScript
- Discuss trade-offs: readability vs performance
- For production, use `split().reverse().join()` for clarity
- For interviews, two-pointer approach shows algorithm knowledge

</details>

2. Check if a string is a palindrome

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Reverse and Compare (Simple)**
```javascript
/**
 * Check palindrome by reversing string
 * Time Complexity: O(n)
 * Space Complexity: O(n) - creates reversed copy
 * 
 * @param {string} str - Input string
 * @returns {boolean} - True if palindrome
 */
function isPalindrome(str) {
  // Normalize: lowercase and remove non-alphanumeric
  const normalized = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  
  // Compare with reversed version
  const reversed = normalized.split('').reverse().join('');
  
  return normalized === reversed;
}

// Test cases
console.log(isPalindrome('racecar'));              // true
console.log(isPalindrome('hello'));                // false
console.log(isPalindrome('A man a plan a canal Panama')); // true
console.log(isPalindrome('Was it a car or a cat I saw')); // true
console.log(isPalindrome(''));                     // true
```

### **Approach 2: Two-Pointer (Optimized)**
```javascript
/**
 * Check palindrome using two pointers
 * Time Complexity: O(n)
 * Space Complexity: O(1) - only uses pointers
 * 
 * Most efficient approach - stops at first mismatch
 */
function isPalindrome(str) {
  // Normalize string
  const normalized = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  
  let left = 0;
  let right = normalized.length - 1;
  
  // Compare characters from both ends
  while (left < right) {
    if (normalized[left] !== normalized[right]) {
      return false; // Early exit on mismatch
    }
    left++;
    right--;
  }
  
  return true;
}

// Test cases
console.log(isPalindrome('racecar'));              // true
console.log(isPalindrome('hello'));                // false
console.log(isPalindrome('A man a plan a canal Panama')); // true
```

### **Approach 3: Recursive Solution**
```javascript
/**
 * Check palindrome using recursion
 * Time Complexity: O(n)
 * Space Complexity: O(n) - call stack
 */
function isPalindrome(str, left = 0, right = str.length - 1) {
  // Normalize on first call
  if (left === 0 && right === str.length - 1) {
    str = str.toLowerCase().replace(/[^a-z0-9]/g, '');
    right = str.length - 1;
  }
  
  // Base case: pointers met or crossed
  if (left >= right) return true;
  
  // Recursive case: check current chars and recurse
  if (str[left] !== str[right]) return false;
  
  return isPalindrome(str, left + 1, right - 1);
}

// Test cases
console.log(isPalindrome('racecar'));              // true
console.log(isPalindrome('hello'));                // false
```

### **Approach 4: Using Every Method (Functional)**
```javascript
/**
 * Functional approach using array methods
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function isPalindrome(str) {
  const normalized = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  const arr = normalized.split('');
  
  return arr.every((char, index) => {
    return char === arr[arr.length - 1 - index];
  });
}

// Test cases
console.log(isPalindrome('racecar'));              // true
console.log(isPalindrome('hello'));                // false
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready palindrome checker with options
 * 
 * @param {string} str - Input string to check
 * @param {Object} options - Configuration options
 * @param {boolean} options.caseSensitive - Whether to consider case (default: false)
 * @param {boolean} options.ignoreSpaces - Whether to ignore spaces (default: true)
 * @param {boolean} options.alphanumericOnly - Only consider alphanumeric chars (default: true)
 * @returns {boolean} - True if string is palindrome
 */
function isPalindrome(str, options = {}) {
  const {
    caseSensitive = false,
    ignoreSpaces = true,
    alphanumericOnly = true
  } = options;
  
  // Type validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  // Normalize based on options
  let normalized = str;
  
  if (!caseSensitive) {
    normalized = normalized.toLowerCase();
  }
  
  if (alphanumericOnly) {
    normalized = normalized.replace(/[^a-z0-9]/gi, '');
  } else if (ignoreSpaces) {
    normalized = normalized.replace(/\s/g, '');
  }
  
  // Two-pointer check
  let left = 0;
  let right = normalized.length - 1;
  
  while (left < right) {
    if (normalized[left] !== normalized[right]) {
      return false;
    }
    left++;
    right--;
  }
  
  return true;
}

// Comprehensive test suite
console.log(isPalindrome('racecar'));                           // true
console.log(isPalindrome('RaceCar'));                           // true
console.log(isPalindrome('A man a plan a canal Panama'));       // true
console.log(isPalindrome('hello'));                             // false
console.log(isPalindrome('Was it a car or a cat I saw'));       // true
console.log(isPalindrome('Madam'));                             // true
console.log(isPalindrome('12321'));                             // true
console.log(isPalindrome('12345'));                             // false

// With options
console.log(isPalindrome('RaceCar', { caseSensitive: true }));  // false
console.log(isPalindrome('race car', { ignoreSpaces: false })); // false
```

**Interview Tips:**
- Two-pointer is the most efficient (O(1) space)
- Always clarify requirements: case sensitivity, special characters
- Mention early exit optimization
- Show understanding of string normalization
- Discuss Unicode considerations for international characters

</details>

3. Find the largest number in an array

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Loop (Brute Force)**
```javascript
/**
 * Find largest number using traditional loop
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * @param {number[]} arr - Array of numbers
 * @returns {number} - Largest number in array
 */
function findLargest(arr) {
  // Handle edge cases
  if (!arr || arr.length === 0) {
    throw new Error('Array is empty or undefined');
  }
  
  // Initialize with first element
  let largest = arr[0];
  
  // Iterate through array starting from second element
  for (let i = 1; i < arr.length; i++) {
    if (arr[i] > largest) {
      largest = arr[i];
    }
  }
  
  return largest;
}

// Test cases
console.log(findLargest([3, 7, 2, 9, 1]));           // 9
console.log(findLargest([1]));                       // 1
console.log(findLargest([-5, -2, -10, -1]));         // -1
console.log(findLargest([100, 50, 200, 75]));        // 200
```

### **Approach 2: Using Math.max with Spread (Clean)**
```javascript
/**
 * Find largest using Math.max
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Most concise approach
 */
function findLargest(arr) {
  if (!arr || arr.length === 0) {
    throw new Error('Array is empty or undefined');
  }
  
  // Spread array into Math.max
  return Math.max(...arr);
}

// Test cases
console.log(findLargest([3, 7, 2, 9, 1]));           // 9
console.log(findLargest([-5, -2, -10, -1]));         // -1
```

### **Approach 3: Using Reduce (Functional)**
```javascript
/**
 * Find largest using array reduce
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Functional programming approach
 */
function findLargest(arr) {
  if (!arr || arr.length === 0) {
    throw new Error('Array is empty or undefined');
  }
  
  return arr.reduce((max, current) => {
    return current > max ? current : max;
  });
}

// Alternative: with initial value
function findLargest(arr) {
  if (!arr || arr.length === 0) {
    throw new Error('Array is empty or undefined');
  }
  
  return arr.reduce((max, current) => Math.max(max, current), arr[0]);
}

// Test cases
console.log(findLargest([3, 7, 2, 9, 1]));           // 9
console.log(findLargest([-5, -2, -10, -1]));         // -1
```

### **Approach 4: Using Sort (Less Efficient)**
```javascript
/**
 * Find largest by sorting
 * Time Complexity: O(n log n) - due to sorting
 * Space Complexity: O(1) or O(n) depending on sort implementation
 * 
 * NOT recommended for just finding max, but useful if sorted array is needed
 */
function findLargest(arr) {
  if (!arr || arr.length === 0) {
    throw new Error('Array is empty or undefined');
  }
  
  // Create copy to avoid mutating original
  const sorted = [...arr].sort((a, b) => b - a);
  
  return sorted[0];
}

// Test cases
console.log(findLargest([3, 7, 2, 9, 1]));           // 9
console.log(findLargest([-5, -2, -10, -1]));         // -1
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready function to find largest number with comprehensive error handling
 * 
 * @param {number[]} arr - Array of numbers
 * @param {Object} options - Configuration options
 * @param {boolean} options.ignoreNaN - Whether to ignore NaN values (default: true)
 * @param {boolean} options.ignoreInfinity - Whether to ignore Infinity (default: false)
 * @returns {number} - Largest number in array
 * @throws {TypeError} - If input is not an array
 * @throws {Error} - If array is empty or has no valid numbers
 */
function findLargest(arr, options = {}) {
  const { ignoreNaN = true, ignoreInfinity = false } = options;
  
  // Type validation
  if (!Array.isArray(arr)) {
    throw new TypeError('Input must be an array');
  }
  
  if (arr.length === 0) {
    throw new Error('Array cannot be empty');
  }
  
  // Filter based on options
  let validNumbers = arr.filter(num => {
    if (typeof num !== 'number') return false;
    if (ignoreNaN && Number.isNaN(num)) return false;
    if (ignoreInfinity && !Number.isFinite(num)) return false;
    return true;
  });
  
  if (validNumbers.length === 0) {
    throw new Error('Array contains no valid numbers');
  }
  
  // Use Math.max for best performance and clarity
  return Math.max(...validNumbers);
}

// Comprehensive test suite
console.log(findLargest([3, 7, 2, 9, 1]));                    // 9
console.log(findLargest([-5, -2, -10, -1]));                  // -1
console.log(findLargest([100, 50, 200, 75]));                 // 200
console.log(findLargest([0, -0, 0.1, -0.1]));                 // 0.1
console.log(findLargest([1, NaN, 3], { ignoreNaN: true }));   // 3
console.log(findLargest([1, Infinity], { ignoreInfinity: true })); // 1

// Edge cases
try {
  console.log(findLargest([]));                               // Error
} catch (e) {
  console.error(e.message);
}

try {
  console.log(findLargest([NaN, NaN]));                       // Error
} catch (e) {
  console.error(e.message);
}
```

### **Bonus: Find N Largest Numbers**
```javascript
/**
 * Find N largest numbers in array
 * Time Complexity: O(n log n) for sorting approach
 * Space Complexity: O(n)
 */
function findNLargest(arr, n) {
  if (!Array.isArray(arr) || arr.length === 0) {
    throw new Error('Invalid array');
  }
  
  if (n <= 0) return [];
  if (n >= arr.length) return [...arr].sort((a, b) => b - a);
  
  // Sort in descending order and take first n elements
  return [...arr].sort((a, b) => b - a).slice(0, n);
}

// Test cases
console.log(findNLargest([3, 7, 2, 9, 1, 8, 5], 3));  // [9, 8, 7]
console.log(findNLargest([10, 5, 15, 20], 2));        // [20, 15]
```

**Interview Tips:**
- Math.max with spread is most concise for production
- Mention O(n) vs O(n log n) complexity
- Discuss handling of NaN, Infinity, and mixed types
- For large arrays, discuss memory limitations of spread operator
- Mention that sort mutates array (show array copy technique)

</details>

4. Remove duplicates from an array

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Set (Modern & Efficient)**
```javascript
/**
 * Remove duplicates using ES6 Set
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Best approach for most cases
 * 
 * @param {Array} arr - Input array with potential duplicates
 * @returns {Array} - Array with unique values
 */
function removeDuplicates(arr) {
  // Set automatically removes duplicates
  return [...new Set(arr)];
}

// Alternative using Array.from
function removeDuplicates(arr) {
  return Array.from(new Set(arr));
}

// Test cases
console.log(removeDuplicates([1, 2, 2, 3, 4, 4, 5]));           // [1, 2, 3, 4, 5]
console.log(removeDuplicates(['a', 'b', 'a', 'c']));            // ['a', 'b', 'c']
console.log(removeDuplicates([1, '1', 2, '2']));                // [1, '1', 2, '2'] - maintains types
console.log(removeDuplicates([]));                              // []
```

### **Approach 2: Using Filter with indexOf**
```javascript
/**
 * Remove duplicates using filter
 * Time Complexity: O(n²) - indexOf is O(n) inside filter
 * Space Complexity: O(n)
 * 
 * Works in older browsers without Set support
 */
function removeDuplicates(arr) {
  return arr.filter((item, index) => {
    // Keep item if it's the first occurrence
    return arr.indexOf(item) === index;
  });
}

// Test cases
console.log(removeDuplicates([1, 2, 2, 3, 4, 4, 5]));           // [1, 2, 3, 4, 5]
console.log(removeDuplicates(['a', 'b', 'a', 'c']));            // ['a', 'b', 'c']
```

### **Approach 3: Using Reduce with Object**
```javascript
/**
 * Remove duplicates using reduce with accumulator object
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Good for showing understanding of reduce
 */
function removeDuplicates(arr) {
  return arr.reduce((unique, item) => {
    // Add item if not already in unique array
    return unique.includes(item) ? unique : [...unique, item];
  }, []);
}

// More efficient with object lookup
function removeDuplicates(arr) {
  const seen = {};
  return arr.reduce((unique, item) => {
    if (!seen[item]) {
      seen[item] = true;
      unique.push(item);
    }
    return unique;
  }, []);
}

// Test cases
console.log(removeDuplicates([1, 2, 2, 3, 4, 4, 5]));           // [1, 2, 3, 4, 5]
console.log(removeDuplicates(['a', 'b', 'a', 'c']));            // ['a', 'b', 'c']
```

### **Approach 4: Using Loop with Hash Map (Most Control)**
```javascript
/**
 * Remove duplicates using traditional loop
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Most control over the process
 */
function removeDuplicates(arr) {
  const seen = new Map();
  const result = [];
  
  for (let i = 0; i < arr.length; i++) {
    const item = arr[i];
    
    // Check if we've seen this item before
    if (!seen.has(item)) {
      seen.set(item, true);
      result.push(item);
    }
  }
  
  return result;
}

// Test cases
console.log(removeDuplicates([1, 2, 2, 3, 4, 4, 5]));           // [1, 2, 3, 4, 5]
console.log(removeDuplicates(['a', 'b', 'a', 'c']));            // ['a', 'b', 'c']
```

### **Approach 5: In-Place Removal for Sorted Array**
```javascript
/**
 * Remove duplicates in-place from SORTED array
 * Time Complexity: O(n)
 * Space Complexity: O(1) - modifies array in place
 * 
 * LeetCode-style problem (modify array in place)
 * Returns length of unique portion
 */
function removeDuplicatesInPlace(arr) {
  if (arr.length === 0) return 0;
  
  let writeIndex = 1; // Position to write next unique element
  
  for (let readIndex = 1; readIndex < arr.length; readIndex++) {
    // If current element is different from previous
    if (arr[readIndex] !== arr[readIndex - 1]) {
      arr[writeIndex] = arr[readIndex];
      writeIndex++;
    }
  }
  
  // Truncate array to unique length
  arr.length = writeIndex;
  
  return writeIndex;
}

// Test cases
const arr1 = [1, 1, 2, 2, 3, 4, 4, 5];
const length1 = removeDuplicatesInPlace(arr1);
console.log(arr1, length1);  // [1, 2, 3, 4, 5], 5

const arr2 = [1, 1, 1, 1];
const length2 = removeDuplicatesInPlace(arr2);
console.log(arr2, length2);  // [1], 1
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready duplicate removal with options
 * 
 * @param {Array} arr - Input array
 * @param {Object} options - Configuration options
 * @param {boolean} options.preserveOrder - Maintain original order (default: true)
 * @param {Function} options.comparator - Custom comparison function
 * @param {boolean} options.caseInsensitive - For strings, ignore case (default: false)
 * @returns {Array} - Array with duplicates removed
 */
function removeDuplicates(arr, options = {}) {
  const {
    preserveOrder = true,
    comparator = null,
    caseInsensitive = false
  } = options;
  
  // Type validation
  if (!Array.isArray(arr)) {
    throw new TypeError('Input must be an array');
  }
  
  if (arr.length === 0) return [];
  
  // Custom comparator logic
  if (comparator) {
    const result = [];
    for (const item of arr) {
      const isDuplicate = result.some(existing => comparator(existing, item));
      if (!isDuplicate) {
        result.push(item);
      }
    }
    return result;
  }
  
  // Case-insensitive string handling
  if (caseInsensitive && arr.every(item => typeof item === 'string')) {
    const seen = new Set();
    const result = [];
    
    for (const item of arr) {
      const normalized = item.toLowerCase();
      if (!seen.has(normalized)) {
        seen.add(normalized);
        result.push(item); // Keep original case
      }
    }
    return result;
  }
  
  // Default: use Set (fastest and most reliable)
  return [...new Set(arr)];
}

// Comprehensive test suite
console.log(removeDuplicates([1, 2, 2, 3, 4, 4, 5]));
// [1, 2, 3, 4, 5]

console.log(removeDuplicates(['a', 'b', 'A', 'c'], { caseInsensitive: true }));
// ['a', 'b', 'c']

console.log(removeDuplicates(
  [{ id: 1 }, { id: 2 }, { id: 1 }],
  { comparator: (a, b) => a.id === b.id }
));
// [{ id: 1 }, { id: 2 }]

console.log(removeDuplicates([NaN, NaN, 1, 2, 1]));
// [NaN, 1, 2] - Set handles NaN correctly

console.log(removeDuplicates([1, '1', true, 1, 'true']));
// [1, '1', true, 'true'] - maintains types
```

### **Bonus: Remove Duplicates from Array of Objects**
```javascript
/**
 * Remove duplicate objects based on a property
 */
function removeDuplicatesByProperty(arr, property) {
  const seen = new Set();
  return arr.filter(item => {
    const value = item[property];
    if (seen.has(value)) {
      return false;
    }
    seen.add(value);
    return true;
  });
}

// Test
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 1, name: 'John Doe' },
  { id: 3, name: 'Bob' }
];

console.log(removeDuplicatesByProperty(users, 'id'));
// [{ id: 1, name: 'John' }, { id: 2, name: 'Jane' }, { id: 3, name: 'Bob' }]
```

**Interview Tips:**
- Set is the modern, preferred approach (O(n) time)
- Mention that Set uses SameValueZero comparison (NaN === NaN)
- Discuss trade-offs: readability vs browser support
- For objects, need custom comparison logic
- In-place solution demonstrates algorithmic thinking
- Mention that order is preserved with Set

</details>

5. Check if two strings are anagrams

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Sort and Compare (Simple)**
```javascript
/**
 * Check anagram by sorting both strings
 * Time Complexity: O(n log n) - dominated by sorting
 * Space Complexity: O(n) - for sorted strings
 * 
 * @param {string} str1 - First string
 * @param {string} str2 - Second string
 * @returns {boolean} - True if strings are anagrams
 */
function isAnagram(str1, str2) {
  // Quick check: different lengths can't be anagrams
  if (str1.length !== str2.length) return false;
  
  // Normalize: lowercase and remove spaces
  const normalize = (str) => 
    str.toLowerCase().replace(/\s/g, '').split('').sort().join('');
  
  return normalize(str1) === normalize(str2);
}

// Test cases
console.log(isAnagram('listen', 'silent'));          // true
console.log(isAnagram('hello', 'world'));            // false
console.log(isAnagram('Dormitory', 'Dirty room'));   // true
console.log(isAnagram('The eyes', 'They see'));      // true
console.log(isAnagram('abc', 'def'));                // false
```

### **Approach 2: Character Frequency Map (Optimized)**
```javascript
/**
 * Check anagram using character frequency counting
 * Time Complexity: O(n)
 * Space Complexity: O(k) where k is unique characters (max 26 for English)
 * 
 * Most efficient approach
 */
function isAnagram(str1, str2) {
  // Early exit: different lengths
  if (str1.length !== str2.length) return false;
  
  // Normalize strings
  str1 = str1.toLowerCase().replace(/\s/g, '');
  str2 = str2.toLowerCase().replace(/\s/g, '');
  
  // Build frequency map for first string
  const charCount = {};
  
  for (const char of str1) {
    charCount[char] = (charCount[char] || 0) + 1;
  }
  
  // Verify second string matches frequencies
  for (const char of str2) {
    if (!charCount[char]) {
      return false; // Character not in first string or count exhausted
    }
    charCount[char]--;
  }
  
  // All counts should be zero
  return Object.values(charCount).every(count => count === 0);
}

// Test cases
console.log(isAnagram('listen', 'silent'));          // true
console.log(isAnagram('hello', 'world'));            // false
console.log(isAnagram('anagram', 'nagaram'));        // true
```

### **Approach 3: Using Map (ES6)**
```javascript
/**
 * Check anagram using ES6 Map
 * Time Complexity: O(n)
 * Space Complexity: O(k)
 * 
 * Modern JavaScript approach
 */
function isAnagram(str1, str2) {
  if (str1.length !== str2.length) return false;
  
  // Normalize
  str1 = str1.toLowerCase().replace(/\s/g, '');
  str2 = str2.toLowerCase().replace(/\s/g, '');
  
  const charMap = new Map();
  
  // Count characters in first string
  for (const char of str1) {
    charMap.set(char, (charMap.get(char) || 0) + 1);
  }
  
  // Decrement counts with second string
  for (const char of str2) {
    if (!charMap.has(char)) return false;
    
    const count = charMap.get(char) - 1;
    if (count === 0) {
      charMap.delete(char);
    } else {
      charMap.set(char, count);
    }
  }
  
  // Map should be empty if anagram
  return charMap.size === 0;
}

// Test cases
console.log(isAnagram('listen', 'silent'));          // true
console.log(isAnagram('hello', 'world'));            // false
```

### **Approach 4: Single Pass with Array (For Lowercase Letters Only)**
```javascript
/**
 * Optimized for lowercase English letters only
 * Time Complexity: O(n)
 * Space Complexity: O(1) - fixed size array of 26
 * 
 * Best performance for English alphabet
 */
function isAnagram(str1, str2) {
  if (str1.length !== str2.length) return false;
  
  // Array to count character frequencies (a-z)
  const counts = new Array(26).fill(0);
  
  str1 = str1.toLowerCase().replace(/\s/g, '');
  str2 = str2.toLowerCase().replace(/\s/g, '');
  
  // Increment for str1, decrement for str2
  for (let i = 0; i < str1.length; i++) {
    counts[str1.charCodeAt(i) - 97]++; // 'a' is 97
    counts[str2.charCodeAt(i) - 97]--;
  }
  
  // All counts should be zero
  return counts.every(count => count === 0);
}

// Test cases
console.log(isAnagram('listen', 'silent'));          // true
console.log(isAnagram('hello', 'world'));            // false
console.log(isAnagram('anagram', 'nagaram'));        // true
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready anagram checker with comprehensive options
 * 
 * @param {string} str1 - First string
 * @param {string} str2 - Second string
 * @param {Object} options - Configuration options
 * @param {boolean} options.caseSensitive - Consider case (default: false)
 * @param {boolean} options.ignoreSpaces - Ignore spaces (default: true)
 * @param {boolean} options.ignoreSpecialChars - Ignore non-alphanumeric (default: true)
 * @returns {boolean} - True if strings are anagrams
 */
function isAnagram(str1, str2, options = {}) {
  const {
    caseSensitive = false,
    ignoreSpaces = true,
    ignoreSpecialChars = true
  } = options;
  
  // Type validation
  if (typeof str1 !== 'string' || typeof str2 !== 'string') {
    throw new TypeError('Both inputs must be strings');
  }
  
  // Normalize strings based on options
  const normalize = (str) => {
    let normalized = str;
    
    if (!caseSensitive) {
      normalized = normalized.toLowerCase();
    }
    
    if (ignoreSpaces) {
      normalized = normalized.replace(/\s/g, '');
    }
    
    if (ignoreSpecialChars) {
      normalized = normalized.replace(/[^a-z0-9]/gi, '');
    }
    
    return normalized;
  };
  
  const normalized1 = normalize(str1);
  const normalized2 = normalize(str2);
  
  // Quick check: different lengths
  if (normalized1.length !== normalized2.length) {
    return false;
  }
  
  // Use frequency map for optimal performance
  const charCount = new Map();
  
  // Count characters in first string
  for (const char of normalized1) {
    charCount.set(char, (charCount.get(char) || 0) + 1);
  }
  
  // Verify with second string
  for (const char of normalized2) {
    const count = charCount.get(char);
    
    if (!count) { // Character missing or count exhausted
      return false;
    }
    
    if (count === 1) {
      charCount.delete(char);
    } else {
      charCount.set(char, count - 1);
    }
  }
  
  return charCount.size === 0;
}

// Comprehensive test suite
console.log(isAnagram('listen', 'silent'));                    // true
console.log(isAnagram('hello', 'world'));                      // false
console.log(isAnagram('Dormitory', 'Dirty room'));             // true
console.log(isAnagram('The eyes', 'They see'));                // true
console.log(isAnagram('astronomer', 'moon starer'));           // true
console.log(isAnagram('abc', 'def'));                          // false

// With options
console.log(isAnagram('Listen', 'Silent', { caseSensitive: true }));  // false
console.log(isAnagram('hello!', 'olleh', { ignoreSpecialChars: false })); // false

// Edge cases
console.log(isAnagram('', ''));                                // true
console.log(isAnagram('a', 'a'));                              // true
console.log(isAnagram('ab', 'ba'));                            // true
```

### **Bonus: Find All Anagrams in String**
```javascript
/**
 * Find all starting indices of anagrams of pattern in string
 * (Sliding Window approach)
 * Time Complexity: O(n) where n is length of string
 */
function findAnagrams(str, pattern) {
  const result = [];
  const patternMap = new Map();
  const windowMap = new Map();
  
  // Build pattern frequency map
  for (const char of pattern) {
    patternMap.set(char, (patternMap.get(char) || 0) + 1);
  }
  
  let left = 0;
  let right = 0;
  let count = patternMap.size; // Unique characters to match
  
  while (right < str.length) {
    // Expand window
    const rightChar = str[right];
    if (patternMap.has(rightChar)) {
      windowMap.set(rightChar, (windowMap.get(rightChar) || 0) + 1);
      if (windowMap.get(rightChar) === patternMap.get(rightChar)) {
        count--;
      }
    }
    right++;
    
    // When all characters matched
    while (count === 0) {
      // Found an anagram
      if (right - left === pattern.length) {
        result.push(left);
      }
      
      // Shrink window
      const leftChar = str[left];
      if (patternMap.has(leftChar)) {
        if (windowMap.get(leftChar) === patternMap.get(leftChar)) {
          count++;
        }
        windowMap.set(leftChar, windowMap.get(leftChar) - 1);
      }
      left++;
    }
  }
  
  return result;
}

// Test
console.log(findAnagrams('cbaebabacd', 'abc'));  // [0, 6] - "cba" and "bac"
console.log(findAnagrams('abab', 'ab'));         // [0, 1, 2] - "ab", "ba", "ab"
```

**Interview Tips:**
- Sorting approach is O(n log n), frequency map is O(n)
- Frequency map is optimal for performance
- Always clarify: case sensitivity, spaces, special characters
- For English alphabet only, can use fixed-size array (26)
- Mention early exit optimization (length check)
- Discuss Unicode considerations for international strings

</details>

6. Find the first non-repeating character in a string

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Two-Pass with Object (Straightforward)**
```javascript
/**
 * Find first non-repeating character using frequency map
 * Time Complexity: O(n) - two passes through string
 * Space Complexity: O(k) - k is unique characters
 * 
 * @param {string} str - Input string
 * @returns {string|null} - First non-repeating character or null
 */
function firstNonRepeating(str) {
  if (!str) return null;
  
  // First pass: count character frequencies
  const charCount = {};
  for (const char of str) {
    charCount[char] = (charCount[char] || 0) + 1;
  }
  
  // Second pass: find first character with count 1
  for (const char of str) {
    if (charCount[char] === 1) {
      return char;
    }
  }
  
  return null; // All characters repeat
}

// Test cases
console.log(firstNonRepeating('leetcode'));      // 'l'
console.log(firstNonRepeating('loveleetcode'));  // 'v'
console.log(firstNonRepeating('aabb'));          // null
console.log(firstNonRepeating(''));              // null
console.log(firstNonRepeating('a'));             // 'a'
```

### **Approach 2: Using Map (ES6)**
```javascript
/**
 * Find first non-repeating character using ES6 Map
 * Time Complexity: O(n)
 * Space Complexity: O(k)
 * 
 * Maintains insertion order naturally
 */
function firstNonRepeating(str) {
  if (!str) return null;
  
  const charMap = new Map();
  
  // Count frequencies
  for (const char of str) {
    charMap.set(char, (charMap.get(char) || 0) + 1);
  }
  
  // Find first with count 1 (Map maintains insertion order)
  for (const [char, count] of charMap) {
    if (count === 1) {
      return char;
    }
  }
  
  return null;
}

// Test cases
console.log(firstNonRepeating('leetcode'));      // 'l'
console.log(firstNonRepeating('loveleetcode'));  // 'v'
console.log(firstNonRepeating('aabb'));          // null
```

### **Approach 3: Using indexOf and lastIndexOf**
```javascript
/**
 * Simple approach using built-in string methods
 * Time Complexity: O(n²) - indexOf/lastIndexOf are O(n)
 * Space Complexity: O(1)
 * 
 * Less efficient but very concise
 */
function firstNonRepeating(str) {
  if (!str) return null;
  
  for (let i = 0; i < str.length; i++) {
    const char = str[i];
    // If first and last occurrence are same, it's unique
    if (str.indexOf(char) === str.lastIndexOf(char)) {
      return char;
    }
  }
  
  return null;
}

// Test cases
console.log(firstNonRepeating('leetcode'));      // 'l'
console.log(firstNonRepeating('loveleetcode'));  // 'v'
console.log(firstNonRepeating('aabb'));          // null
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready first non-repeating character finder
 * 
 * @param {string} str - Input string
 * @param {Object} options - Configuration options
 * @param {boolean} options.caseSensitive - Consider case (default: false)
 * @param {boolean} options.ignoreSpaces - Ignore spaces (default: true)
 * @param {boolean} options.returnIndex - Return index instead of char (default: false)
 * @returns {string|number|null} - First non-repeating character, its index, or null
 */
function firstNonRepeating(str, options = {}) {
  const {
    caseSensitive = false,
    ignoreSpaces = true,
    returnIndex = false
  } = options;
  
  // Type validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  if (str.length === 0) return null;
  
  // Normalize if needed
  let processedStr = str;
  if (!caseSensitive) {
    processedStr = str.toLowerCase();
  }
  
  // Build frequency map with original indices
  const charMap = new Map();
  
  for (let i = 0; i < processedStr.length; i++) {
    const char = processedStr[i];
    
    // Skip spaces if ignoreSpaces is true
    if (ignoreSpaces && char === ' ') continue;
    
    if (charMap.has(char)) {
      charMap.get(char).count++;
    } else {
      charMap.set(char, {
        count: 1,
        originalIndex: i,
        originalChar: str[i] // Keep original case
      });
    }
  }
  
  // Find first non-repeating character
  let result = null;
  let minIndex = Infinity;
  
  for (const [char, data] of charMap) {
    if (data.count === 1 && data.originalIndex < minIndex) {
      minIndex = data.originalIndex;
      result = returnIndex ? data.originalIndex : data.originalChar;
    }
  }
  
  return result;
}

// Comprehensive test suite
console.log(firstNonRepeating('leetcode'));                    // 'l'
console.log(firstNonRepeating('loveleetcode'));                // 'v'
console.log(firstNonRepeating('aabb'));                        // null
console.log(firstNonRepeating(''));                            // null
console.log(firstNonRepeating('AaBbCc', { caseSensitive: true })); // 'A'
console.log(firstNonRepeating('AaBbCc', { caseSensitive: false })); // null
console.log(firstNonRepeating('a b c b a', { ignoreSpaces: true })); // 'c'
console.log(firstNonRepeating('leetcode', { returnIndex: true })); // 0
```

**Interview Tips:**
- Two-pass solution is most straightforward and efficient O(n)
- indexOf/lastIndexOf is simple but O(n²)
- Map maintains insertion order (ES6+)
- Clarify if case matters and how to handle spaces
- Mention that returning null vs empty string vs -1 (index) should be clarified

</details>

7. Merge two sorted arrays

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Two-Pointer Merge (Optimal)**
```javascript
/**
 * Merge two sorted arrays using two-pointer technique
 * Time Complexity: O(n + m) where n, m are array lengths
 * Space Complexity: O(n + m) for result array
 * 
 * Classic merge sort merge step
 * 
 * @param {number[]} arr1 - First sorted array
 * @param {number[]} arr2 - Second sorted array
 * @returns {number[]} - Merged sorted array
 */
function mergeSortedArrays(arr1, arr2) {
  const result = [];
  let i = 0; // Pointer for arr1
  let j = 0; // Pointer for arr2
  
  // Compare elements from both arrays
  while (i < arr1.length && j < arr2.length) {
    if (arr1[i] <= arr2[j]) {
      result.push(arr1[i]);
      i++;
    } else {
      result.push(arr2[j]);
      j++;
    }
  }
  
  // Add remaining elements from arr1 (if any)
  while (i < arr1.length) {
    result.push(arr1[i]);
    i++;
  }
  
  // Add remaining elements from arr2 (if any)
  while (j < arr2.length) {
    result.push(arr2[j]);
    j++;
  }
  
  return result;
}

// Test cases
console.log(mergeSortedArrays([1, 3, 5], [2, 4, 6]));
// [1, 2, 3, 4, 5, 6]
```

### **Approach 2: Using Spread and Sort (Simple but Less Efficient)**
```javascript
/**
 * Merge by concatenating and sorting
 * Time Complexity: O((n + m) log(n + m)) - due to sorting
 * Space Complexity: O(n + m)
 */
function mergeSortedArrays(arr1, arr2) {
  return [...arr1, ...arr2].sort((a, b) => a - b);
}
```

**Interview Tips:**
- Two-pointer is optimal: O(n + m) time
- Always handle edge cases: empty arrays
- This is the "merge" step in merge sort

</details>

8. Find the second largest number in an array

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Single Pass (Most Efficient)**
```javascript
/**
 * Find second largest using single pass
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * @param {number[]} arr - Array of numbers
 * @returns {number|null} - Second largest number or null
 */
function findSecondLargest(arr) {
  if (!arr || arr.length < 2) {
    return null;
  }
  
  let largest = -Infinity;
  let secondLargest = -Infinity;
  
  for (const num of arr) {
    if (num > largest) {
      secondLargest = largest;
      largest = num;
    } else if (num > secondLargest && num < largest) {
      secondLargest = num;
    }
  }
  
  return secondLargest === -Infinity ? null : secondLargest;
}

// Test cases
console.log(findSecondLargest([3, 7, 2, 9, 1]));        // 7
console.log(findSecondLargest([5, 5, 5, 5]));           // null
```

### **Approach 2: Using Set to Remove Duplicates**
```javascript
/**
 * Remove duplicates then find second max
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function findSecondLargest(arr) {
  if (!arr || arr.length < 2) return null;
  
  const unique = [...new Set(arr)];
  if (unique.length < 2) return null;
  
  let largest = Math.max(unique[0], unique[1]);
  let secondLargest = Math.min(unique[0], unique[1]);
  
  for (let i = 2; i < unique.length; i++) {
    if (unique[i] > largest) {
      secondLargest = largest;
      largest = unique[i];
    } else if (unique[i] > secondLargest) {
      secondLargest = unique[i];
    }
  }
  
  return secondLargest;
}
```

**Interview Tips:**
- Single-pass solution is optimal: O(n) time, O(1) space
- Clarify if duplicates count (e.g., [5,5,3] - is second largest 5 or 3?)
- Be careful with initialization: use -Infinity

</details>

9. Count the occurrence of each character in a string

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Object (Classic)**
```javascript
/**
 * Count character occurrences using plain object
 * Time Complexity: O(n)
 * Space Complexity: O(k) where k is unique characters
 * 
 * @param {string} str - Input string
 * @returns {Object} - Character frequency map
 */
function countCharacters(str) {
  const charCount = {};
  
  for (const char of str) {
    charCount[char] = (charCount[char] || 0) + 1;
  }
  
  return charCount;
}

// Test cases
console.log(countCharacters('hello'));
// { h: 1, e: 1, l: 2, o: 1 }

console.log(countCharacters('mississippi'));
// { m: 1, i: 4, s: 4, p: 2 }
```

### **Approach 2: Using Map (ES6)**
```javascript
/**
 * Count characters using ES6 Map
 * Time Complexity: O(n)
 * Space Complexity: O(k)
 */
function countCharacters(str) {
  const charMap = new Map();
  
  for (const char of str) {
    charMap.set(char, (charMap.get(char) || 0) + 1);
  }
  
  return charMap;
}

// Test
const result = countCharacters('hello');
console.log(result);
// Map(4) { 'h' => 1, 'e' => 1, 'l' => 2, 'o' => 1 }
```

### **Approach 3: Using Reduce (Functional)**
```javascript
/**
 * Count characters using reduce
 * Time Complexity: O(n)
 * Space Complexity: O(k)
 */
function countCharacters(str) {
  return str.split('').reduce((acc, char) => {
    acc[char] = (acc[char] || 0) + 1;
    return acc;
  }, {});
}
```

**Interview Tips:**
- Object/Map approach is standard: O(n) time, O(k) space
- Map is better for non-string keys and maintains insertion order
- Clarify requirements: case sensitivity, special characters, spaces

</details>

10. Flatten a nested array by one level

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using flat() Method (Modern)**
```javascript
/**
 * Flatten array by one level using built-in flat()
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * ES2019+ feature - simplest approach
 * 
 * @param {Array} arr - Nested array
 * @returns {Array} - Flattened array
 */
function flattenOneLevel(arr) {
  return arr.flat(1); // 1 is default depth
}

// Test cases
console.log(flattenOneLevel([1, [2, 3], 4, [5, 6]]));
// [1, 2, 3, 4, 5, 6]

console.log(flattenOneLevel([1, [2, [3, 4]], 5]));
// [1, 2, [3, 4], 5] - only one level flattened
```

### **Approach 2: Using concat() with Spread**
```javascript
/**
 * Flatten using concat and spread operator
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Works in ES6+
 */
function flattenOneLevel(arr) {
  return [].concat(...arr);
}

// Test cases
console.log(flattenOneLevel([1, [2, 3], 4, [5, 6]]));
// [1, 2, 3, 4, 5, 6]
```

### **Approach 3: Using reduce() (Functional)**
```javascript
/**
 * Flatten using reduce
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function flattenOneLevel(arr) {
  return arr.reduce((flattened, item) => {
    return Array.isArray(item)
      ? [...flattened, ...item]
      : [...flattened, item];
  }, []);
}

// More efficient: using concat
function flattenOneLevel(arr) {
  return arr.reduce((flattened, item) => {
    return flattened.concat(item);
  }, []);
}
```

### **Approach 4: Manual Loop**
```javascript
/**
 * Flatten using traditional loop
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function flattenOneLevel(arr) {
  const result = [];
  
  for (const item of arr) {
    if (Array.isArray(item)) {
      for (const subItem of item) {
        result.push(subItem);
      }
    } else {
      result.push(item);
    }
  }
  
  return result;
}
```

### **Bonus: Flatten Completely (Any Depth)**
```javascript
/**
 * Flatten array to any depth recursively
 */
function flattenDeep(arr) {
  return arr.reduce((flattened, item) => {
    return Array.isArray(item)
      ? [...flattened, ...flattenDeep(item)]
      : [...flattened, item];
  }, []);
}

// Using flat with Infinity
function flattenDeep(arr) {
  return arr.flat(Infinity);
}

// Test
console.log(flattenDeep([1, [2, [3, [4, [5]]]]]));
// [1, 2, 3, 4, 5]
```

**Interview Tips:**
- `arr.flat()` is the modern, simplest approach (ES2019+)
- `[].concat(...arr)` works in ES6+ for one level
- For multiple levels, can use `arr.flat(depth)` or recursion
- Manual loop gives most control and works everywhere

</details>

### **Basic Algorithms**

11. Implement FizzBuzz (print numbers 1-100, "Fizz" for multiples of 3, "Buzz" for multiples of 5, "FizzBuzz" for both)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Classic If-Else Chain**
```javascript
/**
 * FizzBuzz using if-else statements
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * @param {number} n - Upper limit (default: 100)
 * @returns {void} - Prints results to console
 */
function fizzBuzz(n = 100) {
  for (let i = 1; i <= n; i++) {
    if (i % 15 === 0) {
      console.log('FizzBuzz');
    } else if (i % 3 === 0) {
      console.log('Fizz');
    } else if (i % 5 === 0) {
      console.log('Buzz');
    } else {
      console.log(i);
    }
  }
}

// Test
fizzBuzz(15);
// 1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, 11, Fizz, 13, 14, FizzBuzz
```

### **Approach 2: String Concatenation (Cleaner)**
```javascript
/**
 * FizzBuzz using string building
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * More maintainable for additional rules
 */
function fizzBuzz(n = 100) {
  for (let i = 1; i <= n; i++) {
    let output = '';
    
    if (i % 3 === 0) output += 'Fizz';
    if (i % 5 === 0) output += 'Buzz';
    
    console.log(output || i);
  }
}

// Test
fizzBuzz(15);
```

### **Approach 3: Array Map (Functional)**
```javascript
/**
 * FizzBuzz using functional approach
 * Time Complexity: O(n)
 * Space Complexity: O(n) - stores results
 */
function fizzBuzz(n = 100) {
  return Array.from({ length: n }, (_, i) => {
    const num = i + 1;
    let result = '';
    
    if (num % 3 === 0) result += 'Fizz';
    if (num % 5 === 0) result += 'Buzz';
    
    return result || num;
  });
}

// Test
console.log(fizzBuzz(15));
// [1, 2, 'Fizz', 4, 'Buzz', 'Fizz', 7, 8, 'Fizz', 'Buzz', 11, 'Fizz', 13, 14, 'FizzBuzz']
```

### **Approach 4: Lookup Object (Scalable)**
```javascript
/**
 * FizzBuzz using object lookup
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Easy to extend with more rules
 */
function fizzBuzz(n = 100, rules = { 3: 'Fizz', 5: 'Buzz' }) {
  for (let i = 1; i <= n; i++) {
    let output = '';
    
    for (const [divisor, word] of Object.entries(rules)) {
      if (i % divisor === 0) output += word;
    }
    
    console.log(output || i);
  }
}

// Test with custom rules
fizzBuzz(20, { 3: 'Fizz', 5: 'Buzz', 7: 'Bang' });
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready FizzBuzz with validation and options
 * 
 * @param {number} start - Starting number (default: 1)
 * @param {number} end - Ending number (default: 100)
 * @param {Object} options - Custom rules { divisor: word }
 * @param {boolean} returnArray - Return array instead of printing
 * @returns {Array|void}
 */
function fizzBuzz(start = 1, end = 100, options = {}) {
  const {
    rules = { 3: 'Fizz', 5: 'Buzz' },
    returnArray = false
  } = options;
  
  // Validation
  if (typeof start !== 'number' || typeof end !== 'number') {
    throw new TypeError('Start and end must be numbers');
  }
  
  if (start > end) {
    throw new RangeError('Start must be less than or equal to end');
  }
  
  // Convert rules to sorted array for consistent order
  const sortedRules = Object.entries(rules)
    .map(([divisor, word]) => [Number(divisor), word])
    .sort((a, b) => a[0] - b[0]);
  
  const results = [];
  
  for (let i = start; i <= end; i++) {
    let output = '';
    
    for (const [divisor, word] of sortedRules) {
      if (i % divisor === 0) {
        output += word;
      }
    }
    
    const result = output || i;
    
    if (returnArray) {
      results.push(result);
    } else {
      console.log(result);
    }
  }
  
  return returnArray ? results : undefined;
}

// Comprehensive test suite
console.log('=== Standard FizzBuzz ===');
fizzBuzz(1, 15);

console.log('\n=== Custom Range ===');
fizzBuzz(10, 20);

console.log('\n=== Custom Rules ===');
fizzBuzz(1, 20, { rules: { 2: 'Even', 3: 'Three', 7: 'Lucky' } });

console.log('\n=== Return Array ===');
const result = fizzBuzz(1, 15, { returnArray: true });
console.log(result);

// Edge cases
console.log('\n=== Single Number ===');
fizzBuzz(15, 15);

console.log('\n=== Large Numbers ===');
fizzBuzz(1000, 1015);
```

### **Bonus: FizzBuzz Variations**
```javascript
/**
 * FizzBuzzBazz with three rules
 */
function fizzBuzzBazz(n) {
  for (let i = 1; i <= n; i++) {
    let output = '';
    if (i % 3 === 0) output += 'Fizz';
    if (i % 5 === 0) output += 'Buzz';
    if (i % 7 === 0) output += 'Bazz';
    console.log(output || i);
  }
}

/**
 * Reverse FizzBuzz (countdown)
 */
function reverseFizzBuzz(n = 100) {
  for (let i = n; i >= 1; i--) {
    let output = '';
    if (i % 3 === 0) output += 'Fizz';
    if (i % 5 === 0) output += 'Buzz';
    console.log(output || i);
  }
}

/**
 * FizzBuzz as a generator
 */
function* fizzBuzzGenerator(start = 1, end = 100) {
  for (let i = start; i <= end; i++) {
    let output = '';
    if (i % 3 === 0) output += 'Fizz';
    if (i % 5 === 0) output += 'Buzz';
    yield output || i;
  }
}

// Use generator
for (const value of fizzBuzzGenerator(1, 15)) {
  console.log(value);
}
```

**Interview Tips:**
- Check divisibility by 15 first (or build string) to avoid wrong "Fizz" or "Buzz" only
- String concatenation approach is more maintainable
- Mention that % operator checks divisibility
- Discuss how to extend with more rules (object lookup)
- Can optimize by checking smaller divisor first

</details>

12. Check if a number is prime

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Trial Division**
```javascript
/**
 * Check if number is prime using basic trial division
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * @param {number} num - Number to check
 * @returns {boolean} - True if prime
 */
function isPrime(num) {
  // Handle edge cases
  if (num <= 1) return false;
  if (num <= 3) return true;
  
  // Check divisibility from 2 to num-1
  for (let i = 2; i < num; i++) {
    if (num % i === 0) {
      return false;
    }
  }
  
  return true;
}

// Test cases
console.log(isPrime(2));    // true
console.log(isPrime(17));   // true
console.log(isPrime(4));    // false
console.log(isPrime(1));    // false
console.log(isPrime(0));    // false
```

### **Approach 2: Optimized to √n (Efficient)**
```javascript
/**
 * Check prime by testing up to square root
 * Time Complexity: O(√n)
 * Space Complexity: O(1)
 */
function isPrime(num) {
  if (num <= 1) return false;
  if (num <= 3) return true;
  
  if (num % 2 === 0 || num % 3 === 0) return false;
  
  for (let i = 5; i * i <= num; i = i + 6) {
    if (num % i === 0 || num % (i + 2) === 0) {
      return false;
    }
  }
  
  return true;
}
```

**Interview Tips:**
- `O(√n)` is the optimal time complexity for trial division
- Handle edge cases: 0, 1, 2, 3
- Explain why checking up to the square root is sufficient

</details>
