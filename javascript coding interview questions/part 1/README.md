# JavaScript Coding Interview Questions

A comprehensive collection of frequently asked JavaScript and frontend coding questions in technical interviews, organized by difficulty level.

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
    
    if (!count) {
      return false; // Character missing or count exhausted
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
 * @param {Object} rules - Custom rules { divisor: word }
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
 * 
 * Most efficient basic approach
 */
function isPrime(num) {
  if (num <= 1) return false;
  if (num <= 3) return true;
  
  // Eliminate even numbers and multiples of 3
  if (num % 2 === 0 || num % 3 === 0) return false;
  
  // Check odd divisors up to √n
  for (let i = 2; i * i <= num; i++) {
    if (num % i === 0) {
      return false;
    }
  }
  
  return true;
}

// Test cases
console.log(isPrime(29));   // true
console.log(isPrime(100));  // false
console.log(isPrime(97));   // true
```

### **Approach 3: 6k ± 1 Optimization**
```javascript
/**
 * Optimized prime check using 6k ± 1 pattern
 * Time Complexity: O(√n)
 * Space Complexity: O(1)
 * 
 * All primes > 3 are of form 6k ± 1
 */
function isPrime(num) {
  if (num <= 1) return false;
  if (num <= 3) return true;
  if (num % 2 === 0 || num % 3 === 0) return false;
  
  // Check numbers of form 6k ± 1 up to √n
  let i = 5;
  while (i * i <= num) {
    if (num % i === 0 || num % (i + 2) === 0) {
      return false;
    }
    i += 6;
  }
  
  return true;
}

// Test cases
console.log(isPrime(101));  // true
console.log(isPrime(103));  // true
console.log(isPrime(104));  // false
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready prime checker with validation
 * 
 * @param {number} num - Number to check
 * @returns {boolean} - True if prime
 * @throws {TypeError} - If input is not a number
 */
function isPrime(num) {
  // Validation
  if (typeof num !== 'number' || !Number.isFinite(num)) {
    throw new TypeError('Input must be a finite number');
  }
  
  // Handle non-integers
  if (!Number.isInteger(num)) {
    return false;
  }
  
  // Handle negative numbers and 0, 1
  if (num <= 1) return false;
  if (num === 2) return true;
  if (num === 3) return true;
  
  // Eliminate even numbers
  if (num % 2 === 0) return false;
  
  // Eliminate multiples of 3
  if (num % 3 === 0) return false;
  
  // Check using 6k ± 1 optimization
  let i = 5;
  while (i * i <= num) {
    if (num % i === 0 || num % (i + 2) === 0) {
      return false;
    }
    i += 6;
  }
  
  return true;
}

// Comprehensive test suite
console.log(isPrime(2));      // true - smallest prime
console.log(isPrime(3));      // true
console.log(isPrime(17));     // true
console.log(isPrime(29));     // true
console.log(isPrime(97));     // true
console.log(isPrime(4));      // false
console.log(isPrime(1));      // false
console.log(isPrime(0));      // false
console.log(isPrime(-5));     // false
console.log(isPrime(100));    // false
console.log(isPrime(1009));   // true - large prime

// Edge cases
console.log(isPrime(2.5));    // false - not integer
console.log(isPrime(Infinity)); // TypeError
```

### **Bonus: Find All Primes Up to N (Sieve of Eratosthenes)**
```javascript
/**
 * Find all prime numbers up to n
 * Time Complexity: O(n log log n)
 * Space Complexity: O(n)
 * 
 * Most efficient for finding multiple primes
 */
function sieveOfEratosthenes(n) {
  if (n < 2) return [];
  
  // Create boolean array, initially all true
  const isPrime = new Array(n + 1).fill(true);
  isPrime[0] = isPrime[1] = false;
  
  // Sieve process
  for (let i = 2; i * i <= n; i++) {
    if (isPrime[i]) {
      // Mark all multiples as not prime
      for (let j = i * i; j <= n; j += i) {
        isPrime[j] = false;
      }
    }
  }
  
  // Collect all primes
  const primes = [];
  for (let i = 2; i <= n; i++) {
    if (isPrime[i]) primes.push(i);
  }
  
  return primes;
}

// Test
console.log(sieveOfEratosthenes(30));
// [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]

console.log(sieveOfEratosthenes(100).length);
// 25 primes under 100
```

**Interview Tips:**
- Basic approach is O(n), optimized is O(√n)
- Only need to check up to √n because factors come in pairs
- Eliminate even numbers and multiples of 3 first
- 6k ± 1 optimization is fastest for single number check
- For finding many primes, use Sieve of Eratosthenes
- Handle edge cases: 0, 1, negative numbers, non-integers

</details>

13. Calculate factorial of a number (iterative and recursive)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Iterative (Space Efficient)**
```javascript
/**
 * Calculate factorial using iteration
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * @param {number} n - Non-negative integer
 * @returns {number} - Factorial of n
 */
function factorial(n) {
  if (n < 0) {
    throw new Error('Factorial not defined for negative numbers');
  }
  
  if (n === 0 || n === 1) return 1;
  
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  
  return result;
}

// Test cases
console.log(factorial(0));   // 1
console.log(factorial(1));   // 1
console.log(factorial(5));   // 120
console.log(factorial(10));  // 3628800
```

### **Approach 2: Recursive (Classic)**
```javascript
/**
 * Calculate factorial using recursion
 * Time Complexity: O(n)
 * Space Complexity: O(n) - call stack
 * 
 * Classic recursive approach
 */
function factorial(n) {
  // Base cases
  if (n < 0) {
    throw new Error('Factorial not defined for negative numbers');
  }
  
  if (n === 0 || n === 1) return 1;
  
  // Recursive case
  return n * factorial(n - 1);
}

// Test cases
console.log(factorial(5));   // 120
console.log(factorial(6));   // 720
```

### **Approach 3: Tail Recursion (Optimized Recursive)**
```javascript
/**
 * Calculate factorial using tail recursion
 * Time Complexity: O(n)
 * Space Complexity: O(1) in languages with TCO, O(n) in JavaScript
 * 
 * Can be optimized by compilers with tail call optimization
 */
function factorial(n, accumulator = 1) {
  if (n < 0) {
    throw new Error('Factorial not defined for negative numbers');
  }
  
  if (n === 0 || n === 1) return accumulator;
  
  return factorial(n - 1, n * accumulator);
}

// Test cases
console.log(factorial(5));   // 120
console.log(factorial(7));   // 5040
```

### **Approach 4: Using Reduce (Functional)**
```javascript
/**
 * Calculate factorial using reduce
 * Time Complexity: O(n)
 * Space Complexity: O(n) - array creation
 */
function factorial(n) {
  if (n < 0) {
    throw new Error('Factorial not defined for negative numbers');
  }
  
  if (n === 0 || n === 1) return 1;
  
  return Array.from({ length: n }, (_, i) => i + 1)
    .reduce((product, num) => product * num, 1);
}

// Test cases
console.log(factorial(5));   // 120
console.log(factorial(4));   // 24
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready factorial with caching and validation
 * 
 * @param {number} n - Non-negative integer
 * @returns {number|bigint} - Factorial of n
 */
function factorial(n) {
  // Validation
  if (typeof n !== 'number' || !Number.isInteger(n)) {
    throw new TypeError('Input must be an integer');
  }
  
  if (n < 0) {
    throw new RangeError('Factorial not defined for negative numbers');
  }
  
  // Handle base cases
  if (n === 0 || n === 1) return 1;
  
  // For large numbers, use BigInt to avoid overflow
  if (n > 20) {
    return factorialBigInt(n);
  }
  
  // Iterative calculation
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  
  return result;
}

/**
 * Factorial using BigInt for large numbers
 */
function factorialBigInt(n) {
  let result = 1n;
  for (let i = 2n; i <= BigInt(n); i++) {
    result *= i;
  }
  return result;
}

// Comprehensive test suite
console.log(factorial(0));      // 1
console.log(factorial(1));      // 1
console.log(factorial(5));      // 120
console.log(factorial(10));     // 3628800
console.log(factorial(20));     // 2432902008176640000
console.log(factorial(25));     // BigInt (prevents overflow)

// Edge cases
try {
  console.log(factorial(-5));   // Error
} catch (e) {
  console.error(e.message);
}

try {
  console.log(factorial(3.5));  // Error
} catch (e) {
  console.error(e.message);
}
```

### **Bonus: Memoized Factorial (Performance)**
```javascript
/**
 * Factorial with memoization for repeated calls
 */
const factorialMemoized = (() => {
  const cache = { 0: 1, 1: 1 };
  
  return function factorial(n) {
    if (n < 0) {
      throw new Error('Factorial not defined for negative numbers');
    }
    
    if (cache[n] !== undefined) {
      return cache[n];
    }
    
    cache[n] = n * factorial(n - 1);
    return cache[n];
  };
})();

// Test - second call uses cache
console.log(factorialMemoized(10));  // Calculates
console.log(factorialMemoized(10));  // Returns cached
console.log(factorialMemoized(11));  // Only calculates 11, reuses 10!
```

### **Bonus: Calculate nPr and nCr**
```javascript
/**
 * Permutation: nPr = n! / (n-r)!
 */
function permutation(n, r) {
  if (r > n) return 0;
  
  let result = 1;
  for (let i = n; i > n - r; i--) {
    result *= i;
  }
  return result;
}

/**
 * Combination: nCr = n! / (r! * (n-r)!)
 */
function combination(n, r) {
  if (r > n) return 0;
  if (r === 0 || r === n) return 1;
  
  // Optimize by using smaller r
  r = Math.min(r, n - r);
  
  let result = 1;
  for (let i = 0; i < r; i++) {
    result *= (n - i);
    result /= (i + 1);
  }
  
  return Math.round(result);
}

// Test
console.log(permutation(5, 3));  // 60 (5P3)
console.log(combination(5, 3));  // 10 (5C3)
```

**Interview Tips:**
- Iterative is more space-efficient: O(1) vs O(n)
- Recursive is more intuitive but uses call stack
- JavaScript numbers overflow after 20! (use BigInt)
- Tail recursion optimization not guaranteed in JavaScript
- Memoization improves performance for repeated calls
- Always validate input: negative numbers, non-integers

</details>

14. Find the sum of all numbers in an array

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Loop (Basic)**
```javascript
/**
 * Sum array elements using for loop
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * @param {number[]} arr - Array of numbers
 * @returns {number} - Sum of all elements
 */
function sumArray(arr) {
  let sum = 0;
  
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i];
  }
  
  return sum;
}

// Test cases
console.log(sumArray([1, 2, 3, 4, 5]));        // 15
console.log(sumArray([10, 20, 30]));           // 60
console.log(sumArray([]));                     // 0
console.log(sumArray([-1, -2, -3]));           // -6
```

### **Approach 2: Using reduce() (Functional)**
```javascript
/**
 * Sum array elements using reduce
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Most concise and idiomatic
 */
function sumArray(arr) {
  return arr.reduce((sum, num) => sum + num, 0);
}

// Test cases
console.log(sumArray([1, 2, 3, 4, 5]));        // 15
console.log(sumArray([100, 200, 300]));        // 600
```

### **Approach 3: Using for...of (Modern)**
```javascript
/**
 * Sum array elements using for...of
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Clean and readable
 */
function sumArray(arr) {
  let sum = 0;
  
  for (const num of arr) {
    sum += num;
  }
  
  return sum;
}

// Test cases
console.log(sumArray([1, 2, 3, 4, 5]));        // 15
console.log(sumArray([5, 10, 15]));            // 30
```

### **Approach 4: Recursive**
```javascript
/**
 * Sum array elements recursively
 * Time Complexity: O(n)
 * Space Complexity: O(n) - call stack
 */
function sumArray(arr) {
  // Base case
  if (arr.length === 0) return 0;
  
  // Recursive case
  return arr[0] + sumArray(arr.slice(1));
}

// More efficient with index
function sumArray(arr, index = 0) {
  if (index >= arr.length) return 0;
  return arr[index] + sumArray(arr, index + 1);
}

// Test cases
console.log(sumArray([1, 2, 3, 4, 5]));        // 15
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready sum with validation and options
 * 
 * @param {number[]} arr - Array of numbers
 * @param {Object} options - Configuration options
 * @param {boolean} options.ignoreNaN - Skip NaN values (default: true)
 * @param {boolean} options.absolute - Sum absolute values (default: false)
 * @returns {number} - Sum of array elements
 */
function sumArray(arr, options = {}) {
  const { ignoreNaN = true, absolute = false } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('Input must be an array');
  }
  
  let sum = 0;
  
  for (const value of arr) {
    // Type check
    if (typeof value !== 'number') {
      continue; // Skip non-numbers
    }
    
    // Handle NaN
    if (Number.isNaN(value)) {
      if (ignoreNaN) continue;
      return NaN;
    }
    
    // Handle Infinity
    if (!Number.isFinite(value)) {
      return value; // Return Infinity/-Infinity
    }
    
    // Add to sum
    sum += absolute ? Math.abs(value) : value;
  }
  
  return sum;
}

// Comprehensive test suite
console.log(sumArray([1, 2, 3, 4, 5]));                    // 15
console.log(sumArray([10, 20, 30, 40]));                   // 100
console.log(sumArray([]));                                 // 0
console.log(sumArray([-1, -2, -3]));                       // -6
console.log(sumArray([-1, -2, -3], { absolute: true }));   // 6
console.log(sumArray([1, NaN, 3]));                        // 4 (ignores NaN)
console.log(sumArray([1, NaN, 3], { ignoreNaN: false }));  // NaN
console.log(sumArray([1, 'a', 3]));                        // 4 (skips non-numbers)
console.log(sumArray([1, 2, Infinity]));                   // Infinity

// Edge cases
console.log(sumArray([0, 0, 0]));                          // 0
console.log(sumArray([1.5, 2.5, 3.5]));                    // 7.5
console.log(sumArray([-10, 10]));                          // 0
```

### **Bonus: Sum with Condition**
```javascript
/**
 * Sum array elements that meet a condition
 */
function sumWhere(arr, predicate) {
  return arr.reduce((sum, num) => {
    return predicate(num) ? sum + num : sum;
  }, 0);
}

// Examples
console.log(sumWhere([1, 2, 3, 4, 5], num => num % 2 === 0));
// 6 (sum of even numbers: 2 + 4)

console.log(sumWhere([1, 2, 3, 4, 5], num => num > 2));
// 12 (sum of numbers > 2: 3 + 4 + 5)

console.log(sumWhere([-5, -2, 0, 3, 7], num => num > 0));
// 10 (sum of positive numbers: 3 + 7)
```

### **Bonus: Cumulative Sum**
```javascript
/**
 * Calculate cumulative sum array
 */
function cumulativeSum(arr) {
  const result = [];
  let sum = 0;
  
  for (const num of arr) {
    sum += num;
    result.push(sum);
  }
  
  return result;
}

// Using reduce
function cumulativeSum(arr) {
  return arr.reduce((acc, num, i) => {
    const sum = i === 0 ? num : acc[i - 1] + num;
    return [...acc, sum];
  }, []);
}

// Test
console.log(cumulativeSum([1, 2, 3, 4, 5]));
// [1, 3, 6, 10, 15]

console.log(cumulativeSum([10, 20, 30]));
// [10, 30, 60]
```

**Interview Tips:**
- reduce() is the most idiomatic JavaScript solution
- Always handle empty array case (return 0)
- Consider edge cases: NaN, Infinity, mixed types
- For large arrays, simple loop is most performant
- Mention floating point precision issues for decimals
- Can optimize with parallel processing for very large arrays

</details>

15. Check if a number is even or odd

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Modulo Operator (Standard)**
```javascript
/**
 * Check if number is even using modulo
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * @param {number} num - Number to check
 * @returns {boolean} - True if even
 */
function isEven(num) {
  return num % 2 === 0;
}

function isOdd(num) {
  return num % 2 !== 0;
}

// Test cases
console.log(isEven(4));     // true
console.log(isEven(5));     // false
console.log(isOdd(7));      // true
console.log(isOdd(10));     // false
```

### **Approach 2: Using Bitwise AND (Performance)**
```javascript
/**
 * Check if even using bitwise operation
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * Faster than modulo for integers
 */
function isEven(num) {
  return (num & 1) === 0;
}

function isOdd(num) {
  return (num & 1) === 1;
}

// Test cases
console.log(isEven(8));     // true
console.log(isEven(9));     // false
console.log(isOdd(11));     // true
```

### **Approach 3: Using Math.abs for Negative Numbers**
```javascript
/**
 * Handle negative numbers properly
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 */
function isEven(num) {
  return Math.abs(num) % 2 === 0;
}

function isOdd(num) {
  return Math.abs(num) % 2 === 1;
}

// Test cases
console.log(isEven(-4));    // true
console.log(isEven(-5));    // false
console.log(isOdd(-7));     // true
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready even/odd checker with validation
 * 
 * @param {number} num - Number to check
 * @returns {string} - 'even', 'odd', or 'neither' for non-integers
 */
function checkEvenOdd(num) {
  // Validation
  if (typeof num !== 'number' || !Number.isFinite(num)) {
    throw new TypeError('Input must be a finite number');
  }
  
  // Handle non-integers
  if (!Number.isInteger(num)) {
    return 'neither';
  }
  
  // Check even/odd
  return num % 2 === 0 ? 'even' : 'odd';
}

// Alternative: return boolean
function isEven(num) {
  if (typeof num !== 'number' || !Number.isFinite(num)) {
    throw new TypeError('Input must be a finite number');
  }
  
  if (!Number.isInteger(num)) {
    throw new Error('Input must be an integer');
  }
  
  return num % 2 === 0;
}

// Comprehensive test suite
console.log(checkEvenOdd(4));      // 'even'
console.log(checkEvenOdd(7));      // 'odd'
console.log(checkEvenOdd(-6));     // 'even'
console.log(checkEvenOdd(-3));     // 'odd'
console.log(checkEvenOdd(0));      // 'even'
console.log(checkEvenOdd(3.5));    // 'neither'

// Edge cases
console.log(checkEvenOdd(1000000)); // 'even'
console.log(checkEvenOdd(-999999)); // 'odd'
```

### **Bonus: Filter Even/Odd from Array**
```javascript
/**
 * Filter even or odd numbers from array
 */
function filterEven(arr) {
  return arr.filter(num => num % 2 === 0);
}

function filterOdd(arr) {
  return arr.filter(num => num % 2 !== 0);
}

// Test
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
console.log(filterEven(numbers));  // [2, 4, 6, 8, 10]
console.log(filterOdd(numbers));   // [1, 3, 5, 7, 9]

/**
 * Partition array into even and odd
 */
function partitionEvenOdd(arr) {
  const even = [];
  const odd = [];
  
  for (const num of arr) {
    if (num % 2 === 0) {
      even.push(num);
    } else {
      odd.push(num);
    }
  }
  
  return { even, odd };
}

// Test
console.log(partitionEvenOdd([1, 2, 3, 4, 5, 6]));
// { even: [2, 4, 6], odd: [1, 3, 5] }
```

**Interview Tips:**
- Modulo (%) is standard and most readable
- Bitwise AND (&1) is faster but less clear
- Remember: 0 is even
- Negative numbers: -4 is even, -5 is odd
- For non-integers, clarify requirements
- Mention that % can give negative results with negative numbers

</details>

16. Convert Celsius to Fahrenheit

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Conversion Formula**
```javascript
/**
 * Convert Celsius to Fahrenheit
 * Formula: F = (C × 9/5) + 32
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * @param {number} celsius - Temperature in Celsius
 * @returns {number} - Temperature in Fahrenheit
 */
function celsiusToFahrenheit(celsius) {
  return (celsius * 9/5) + 32;
}

// Test cases
console.log(celsiusToFahrenheit(0));      // 32
console.log(celsiusToFahrenheit(100));    // 212
console.log(celsiusToFahrenheit(-40));    // -40
console.log(celsiusToFahrenheit(37));     // 98.6
```

### **Approach 2: Alternative Formula (1.8 multiplier)**
```javascript
/**
 * Using 1.8 instead of 9/5 for clarity
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 */
function celsiusToFahrenheit(celsius) {
  return (celsius * 1.8) + 32;
}

// Test cases
console.log(celsiusToFahrenheit(25));     // 77
console.log(celsiusToFahrenheit(-10));    // 14
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready temperature converter with validation
 * 
 * @param {number} celsius - Temperature in Celsius
 * @param {Object} options - Configuration options
 * @param {number} options.decimals - Decimal places (default: 2)
 * @returns {number} - Temperature in Fahrenheit
 */
function celsiusToFahrenheit(celsius, options = {}) {
  const { decimals = 2 } = options;
  
  // Validation
  if (typeof celsius !== 'number' || !Number.isFinite(celsius)) {
    throw new TypeError('Input must be a finite number');
  }
  
  // Check absolute zero
  if (celsius < -273.15) {
    throw new RangeError('Temperature cannot be below absolute zero (-273.15°C)');
  }
  
  // Convert
  const fahrenheit = (celsius * 9/5) + 32;
  
  // Round to specified decimal places
  return Number(fahrenheit.toFixed(decimals));
}

// Reverse conversion
function fahrenheitToCelsius(fahrenheit, options = {}) {
  const { decimals = 2 } = options;
  
  if (typeof fahrenheit !== 'number' || !Number.isFinite(fahrenheit)) {
    throw new TypeError('Input must be a finite number');
  }
  
  if (fahrenheit < -459.67) {
    throw new RangeError('Temperature cannot be below absolute zero (-459.67°F)');
  }
  
  const celsius = (fahrenheit - 32) * 5/9;
  return Number(celsius.toFixed(decimals));
}

// Comprehensive test suite
console.log(celsiusToFahrenheit(0));          // 32
console.log(celsiusToFahrenheit(100));        // 212
console.log(celsiusToFahrenheit(-40));        // -40 (same in both)
console.log(celsiusToFahrenheit(37));         // 98.6
console.log(celsiusToFahrenheit(25));         // 77

console.log(fahrenheitToCelsius(32));         // 0
console.log(fahrenheitToCelsius(212));        // 100
console.log(fahrenheitToCelsius(98.6));       // 37

// With custom decimals
console.log(celsiusToFahrenheit(25.5555, { decimals: 4 }));  // 77.9999

// Edge cases
console.log(celsiusToFahrenheit(-273.15));    // -459.67 (absolute zero)
```

### **Bonus: Multi-Unit Temperature Converter**
```javascript
/**
 * Convert between Celsius, Fahrenheit, and Kelvin
 */
class TemperatureConverter {
  static celsiusToFahrenheit(c) {
    return (c * 9/5) + 32;
  }
  
  static celsiusToKelvin(c) {
    return c + 273.15;
  }
  
  static fahrenheitToCelsius(f) {
    return (f - 32) * 5/9;
  }
  
  static fahrenheitToKelvin(f) {
    return this.celsiusToKelvin(this.fahrenheitToCelsius(f));
  }
  
  static kelvinToCelsius(k) {
    return k - 273.15;
  }
  
  static kelvinToFahrenheit(k) {
    return this.celsiusToFahrenheit(this.kelvinToCelsius(k));
  }
  
  static convert(value, from, to) {
    const converters = {
      'C-F': this.celsiusToFahrenheit,
      'C-K': this.celsiusToKelvin,
      'F-C': this.fahrenheitToCelsius,
      'F-K': this.fahrenheitToKelvin,
      'K-C': this.kelvinToCelsius,
      'K-F': this.kelvinToFahrenheit
    };
    
    const key = `${from}-${to}`;
    
    if (from === to) return value;
    
    const converter = converters[key];
    if (!converter) {
      throw new Error(`Invalid conversion: ${from} to ${to}`);
    }
    
    return Number(converter.call(this, value).toFixed(2));
  }
}

// Test
console.log(TemperatureConverter.convert(0, 'C', 'F'));    // 32
console.log(TemperatureConverter.convert(100, 'C', 'K'));  // 373.15
console.log(TemperatureConverter.convert(32, 'F', 'C'));   // 0
console.log(TemperatureConverter.convert(300, 'K', 'C'));  // 26.85
```

**Interview Tips:**
- Formula: F = (C × 9/5) + 32 or F = (C × 1.8) + 32
- Reverse: C = (F - 32) × 5/9
- -40°C = -40°F (same in both scales)
- Absolute zero: -273.15°C = -459.67°F = 0K
- Consider rounding for display purposes

</details>

17. Generate Fibonacci sequence up to n terms

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Iterative (Most Efficient)**
```javascript
/**
 * Generate Fibonacci sequence using iteration
 * Time Complexity: O(n)
 * Space Complexity: O(n) - for result array
 * 
 * @param {number} n - Number of terms
 * @returns {number[]} - Fibonacci sequence
 */
function fibonacci(n) {
  if (n <= 0) return [];
  if (n === 1) return [0];
  if (n === 2) return [0, 1];
  
  const fib = [0, 1];
  
  for (let i = 2; i < n; i++) {
    fib[i] = fib[i - 1] + fib[i - 2];
  }
  
  return fib;
}

// Test cases
console.log(fibonacci(10));
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

console.log(fibonacci(5));
// [0, 1, 1, 2, 3]
```

### **Approach 2: Recursive (Classic but Inefficient)**
```javascript
/**
 * Generate Fibonacci using recursion
 * Time Complexity: O(2^n) - exponential, very slow
 * Space Complexity: O(n) - call stack
 * 
 * Not recommended for production
 */
function fibonacci(n) {
  // Helper function to calculate nth Fibonacci number
  function fib(num) {
    if (num <= 1) return num;
    return fib(num - 1) + fib(num - 2);
  }
  
  const result = [];
  for (let i = 0; i < n; i++) {
    result.push(fib(i));
  }
  
  return result;
}

// Test - slow for large n
console.log(fibonacci(10));
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### **Approach 3: Memoized Recursion (Optimized)**
```javascript
/**
 * Fibonacci with memoization
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function fibonacci(n) {
  const memo = {};
  
  function fib(num) {
    if (num <= 1) return num;
    
    if (memo[num]) return memo[num];
    
    memo[num] = fib(num - 1) + fib(num - 2);
    return memo[num];
  }
  
  const result = [];
  for (let i = 0; i < n; i++) {
    result.push(fib(i));
  }
  
  return result;
}

// Test
console.log(fibonacci(15));
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377]
```

### **Approach 4: Using Generator**
```javascript
/**
 * Fibonacci generator for lazy evaluation
 * Time Complexity: O(1) per next()
 * Space Complexity: O(1)
 * 
 * Memory efficient for large sequences
 */
function* fibonacciGenerator() {
  let [prev, curr] = [0, 1];
  
  yield prev;
  yield curr;
  
  while (true) {
    [prev, curr] = [curr, prev + curr];
    yield curr;
  }
}

// Usage
function fibonacci(n) {
  const result = [];
  const gen = fibonacciGenerator();
  
  for (let i = 0; i < n; i++) {
    result.push(gen.next().value);
  }
  
  return result;
}

// Test
console.log(fibonacci(10));
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready Fibonacci generator
 * 
 * @param {number} n - Number of terms
 * @param {Object} options - Configuration options
 * @param {boolean} options.useBigInt - Use BigInt for large numbers (default: false)
 * @returns {Array} - Fibonacci sequence
 */
function fibonacci(n, options = {}) {
  const { useBigInt = false } = options;
  
  // Validation
  if (typeof n !== 'number' || !Number.isInteger(n)) {
    throw new TypeError('Input must be an integer');
  }
  
  if (n < 0) {
    throw new RangeError('Input must be non-negative');
  }
  
  if (n === 0) return [];
  if (n === 1) return useBigInt ? [0n] : [0];
  if (n === 2) return useBigInt ? [0n, 1n] : [0, 1];
  
  if (useBigInt) {
    const fib = [0n, 1n];
    for (let i = 2; i < n; i++) {
      fib[i] = fib[i - 1] + fib[i - 2];
    }
    return fib;
  } else {
    const fib = [0, 1];
    for (let i = 2; i < n; i++) {
      fib[i] = fib[i - 1] + fib[i - 2];
    }
    return fib;
  }
}

// Get nth Fibonacci number
function getNthFibonacci(n, useBigInt = false) {
  if (n < 0) throw new RangeError('Input must be non-negative');
  if (n === 0) return useBigInt ? 0n : 0;
  if (n === 1) return useBigInt ? 1n : 1;
  
  if (useBigInt) {
    let [prev, curr] = [0n, 1n];
    for (let i = 2; i <= n; i++) {
      [prev, curr] = [curr, prev + curr];
    }
    return curr;
  } else {
    let [prev, curr] = [0, 1];
    for (let i = 2; i <= n; i++) {
      [prev, curr] = [curr, prev + curr];
    }
    return curr;
  }
}

// Comprehensive test suite
console.log(fibonacci(10));
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

console.log(fibonacci(0));   // []
console.log(fibonacci(1));   // [0]
console.log(fibonacci(2));   // [0, 1]

// Large numbers with BigInt
console.log(fibonacci(100, { useBigInt: true }));

console.log(getNthFibonacci(10));      // 55
console.log(getNthFibonacci(50, true)); // BigInt for large numbers
```

### **Bonus: Fibonacci Up to Maximum Value**
```javascript
/**
 * Generate Fibonacci numbers up to a maximum value
 */
function fibonacciUpTo(max) {
  if (max < 0) return [];
  
  const fib = [0];
  if (max === 0) return fib;
  
  fib.push(1);
  
  while (true) {
    const next = fib[fib.length - 1] + fib[fib.length - 2];
    if (next > max) break;
    fib.push(next);
  }
  
  return fib;
}

// Test
console.log(fibonacciUpTo(100));
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]

console.log(fibonacciUpTo(1000));
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987]
```

**Interview Tips:**
- Iterative is most efficient: O(n) time and space
- Recursive without memoization is O(2^n) - exponential
- Memoization reduces recursive to O(n)
- Generator is memory-efficient for large sequences
- For n > 78, use BigInt to avoid overflow
- Sequence starts: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34...

</details>

18. Find the missing number in an array of 1 to n

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Sum Formula (Optimal)**
```javascript
/**
 * Find missing number using mathematical formula
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Formula: Sum of 1 to n = n(n+1)/2
 * Missing = Expected Sum - Actual Sum
 * 
 * @param {number[]} arr - Array of numbers from 1 to n with one missing
 * @returns {number} - Missing number
 */
function findMissingNumber(arr) {
  const n = arr.length + 1; // Original length
  const expectedSum = (n * (n + 1)) / 2;
  const actualSum = arr.reduce((sum, num) => sum + num, 0);
  
  return expectedSum - actualSum;
}

// Test cases
console.log(findMissingNumber([1, 2, 3, 5]));        // 4
console.log(findMissingNumber([1, 2, 4, 5, 6]));     // 3
console.log(findMissingNumber([2, 3, 4, 5]));        // 1
```

### **Approach 2: Using XOR (Bit Manipulation)**
```javascript
/**
 * Find missing number using XOR
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * XOR properties: a ^ a = 0, a ^ 0 = a
 * XOR all numbers 1 to n and all array elements
 * Everything cancels except missing number
 */
function findMissingNumber(arr) {
  const n = arr.length + 1;
  let xor = 0;
  
  // XOR with all numbers from 1 to n
  for (let i = 1; i <= n; i++) {
    xor ^= i;
  }
  
  // XOR with all array elements
  for (const num of arr) {
    xor ^= num;
  }
  
  return xor;
}

// Test cases
console.log(findMissingNumber([1, 2, 3, 5]));        // 4
console.log(findMissingNumber([1, 2, 4, 5, 6]));     // 3
```

### **Approach 3: Using Set (Simple)**
```javascript
/**
 * Find missing number using Set
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Create set, check which number is missing
 */
function findMissingNumber(arr) {
  const n = arr.length + 1;
  const numSet = new Set(arr);
  
  for (let i = 1; i <= n; i++) {
    if (!numSet.has(i)) {
      return i;
    }
  }
  
  return -1; // Should never reach here
}

// Test cases
console.log(findMissingNumber([1, 2, 3, 5]));        // 4
console.log(findMissingNumber([3, 4, 5, 6, 7]));     // 1 or 2 (depends on range)
```

### **Approach 4: Sort and Find Gap**
```javascript
/**
 * Find missing by sorting and looking for gap
 * Time Complexity: O(n log n) - due to sorting
 * Space Complexity: O(1) or O(n) depending on sort
 * 
 * Less efficient but intuitive
 */
function findMissingNumber(arr) {
  arr.sort((a, b) => a - b);
  
  for (let i = 0; i < arr.length; i++) {
    // Expected number at position i (assuming starts at 1)
    const expected = arr[0] + i;
    
    if (arr[i] !== expected) {
      return expected;
    }
  }
  
  // Missing is at the end
  return arr[arr.length - 1] + 1;
}

// Test cases
console.log(findMissingNumber([1, 2, 3, 5]));        // 4
console.log(findMissingNumber([2, 3, 4, 5]));        // 1 (if starts at 1)
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready missing number finder
 * 
 * @param {number[]} arr - Array with one missing number
 * @param {Object} options - Configuration options
 * @param {number} options.start - Starting number (default: 1)
 * @returns {number} - Missing number
 */
function findMissingNumber(arr, options = {}) {
  const { start = 1 } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('Input must be an array');
  }
  
  if (arr.length === 0) {
    return start;
  }
  
  // Calculate range
  const n = arr.length + 1;
  const end = start + n - 1;
  
  // Expected sum: sum of arithmetic sequence
  const expectedSum = (n * (start + end)) / 2;
  
  // Actual sum
  const actualSum = arr.reduce((sum, num) => sum + num, 0);
  
  return expectedSum - actualSum;
}

// Comprehensive test suite
console.log(findMissingNumber([1, 2, 3, 5]));              // 4
console.log(findMissingNumber([1, 2, 4, 5, 6]));           // 3
console.log(findMissingNumber([2, 3, 4, 5]));              // 1 or 6
console.log(findMissingNumber([10, 11, 13, 14], { start: 10 })); // 12

// Edge cases
console.log(findMissingNumber([2]));                       // 1
console.log(findMissingNumber([1]));                       // 2
console.log(findMissingNumber([1, 3]));                    // 2
```

### **Bonus: Find All Missing Numbers**
```javascript
/**
 * Find all missing numbers in array
 */
function findAllMissingNumbers(arr, n) {
  const numSet = new Set(arr);
  const missing = [];
  
  for (let i = 1; i <= n; i++) {
    if (!numSet.has(i)) {
      missing.push(i);
    }
  }
  
  return missing;
}

// Using cyclic sort approach (modify array in place)
function findAllMissingNumbers(arr) {
  const n = arr.length;
  
  // Place each number at its correct index
  let i = 0;
  while (i < n) {
    const correctIndex = arr[i] - 1;
    
    if (arr[i] > 0 && arr[i] <= n && arr[i] !== arr[correctIndex]) {
      // Swap
      [arr[i], arr[correctIndex]] = [arr[correctIndex], arr[i]];
    } else {
      i++;
    }
  }
  
  // Find missing numbers
  const missing = [];
  for (let i = 0; i < n; i++) {
    if (arr[i] !== i + 1) {
      missing.push(i + 1);
    }
  }
  
  return missing;
}

// Test
console.log(findAllMissingNumbers([1, 2, 4, 6], 6));
// [3, 5]

console.log(findAllMissingNumbers([4, 3, 2, 7, 8, 2, 3, 1]));
// [5, 6] - using cyclic sort
```

**Interview Tips:**
- Sum formula is optimal: O(n) time, O(1) space
- XOR is clever and doesn't risk integer overflow
- Set approach is simple but uses O(n) space
- Clarify: does sequence start at 0 or 1?
- Clarify: is only one number missing?
- For large numbers, sum formula might overflow (use XOR or BigInt)
- Cyclic sort is elegant for finding multiple missing numbers

</details>

19. Swap two variables without using a third variable

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Destructuring (Modern & Recommended)**
```javascript
/**
 * Swap variables using ES6 destructuring
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * Cleanest and most readable approach
 */
function swap() {
  let a = 5;
  let b = 10;
  
  console.log('Before:', a, b);  // 5, 10
  
  [a, b] = [b, a];
  
  console.log('After:', a, b);   // 10, 5
  
  return [a, b];
}

swap();
```

### **Approach 2: Arithmetic (Addition/Subtraction)**
```javascript
/**
 * Swap using arithmetic operations
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * Works for numbers only
 * Risk: potential overflow for very large numbers
 */
function swap(a, b) {
  console.log('Before:', a, b);
  
  a = a + b;  // a now contains sum
  b = a - b;  // b now contains original a
  a = a - b;  // a now contains original b
  
  console.log('After:', a, b);
  return [a, b];
}

swap(5, 10);   // 10, 5
swap(100, 200); // 200, 100
```

### **Approach 3: Arithmetic (Multiplication/Division)**
```javascript
/**
 * Swap using multiplication and division
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * Risk: doesn't work if either number is 0
 * Risk: potential overflow/precision loss
 */
function swap(a, b) {
  if (a === 0 || b === 0) {
    throw new Error('Cannot use with zero values');
  }
  
  console.log('Before:', a, b);
  
  a = a * b;  // a now contains product
  b = a / b;  // b now contains original a
  a = a / b;  // a now contains original b
  
  console.log('After:', a, b);
  return [a, b];
}

swap(5, 10);    // 10, 5
swap(3, 7);     // 7, 3
```

### **Approach 4: XOR (Bit Manipulation)**
```javascript
/**
 * Swap using XOR bitwise operation
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * Works for integers without overflow risk
 * XOR properties: a ^ a = 0, a ^ 0 = a
 */
function swap(a, b) {
  console.log('Before:', a, b);
  
  a = a ^ b;  // a now contains XOR of both
  b = a ^ b;  // b = (a ^ b) ^ b = a
  a = a ^ b;  // a = (a ^ b) ^ a = b
  
  console.log('After:', a, b);
  return [a, b];
}

swap(5, 10);    // 10, 5
swap(15, 30);   // 30, 15
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready variable swap with multiple methods
 * 
 * @param {*} a - First value
 * @param {*} b - Second value
 * @param {string} method - Swap method ('destructuring', 'arithmetic', 'xor')
 * @returns {Array} - [b, a]
 */
function swap(a, b, method = 'destructuring') {
  switch (method) {
    case 'destructuring':
      // Modern, works with any type
      [a, b] = [b, a];
      break;
      
    case 'arithmetic':
      // Only for numbers
      if (typeof a !== 'number' || typeof b !== 'number') {
        throw new TypeError('Arithmetic method requires numbers');
      }
      a = a + b;
      b = a - b;
      a = a - b;
      break;
      
    case 'xor':
      // Only for integers
      if (!Number.isInteger(a) || !Number.isInteger(b)) {
        throw new TypeError('XOR method requires integers');
      }
      a = a ^ b;
      b = a ^ b;
      a = a ^ b;
      break;
      
    default:
      throw new Error(`Unknown method: ${method}`);
  }
  
  return [a, b];
}

// Comprehensive test suite
console.log('=== Destructuring (Default) ===');
console.log(swap(5, 10));                    // [10, 5]
console.log(swap('hello', 'world'));         // ['world', 'hello']
console.log(swap([1, 2], [3, 4]));          // [[3, 4], [1, 2]]

console.log('\n=== Arithmetic ===');
console.log(swap(5, 10, 'arithmetic'));      // [10, 5]
console.log(swap(-5, 15, 'arithmetic'));     // [15, -5]

console.log('\n=== XOR ===');
console.log(swap(5, 10, 'xor'));            // [10, 5]
console.log(swap(100, 200, 'xor'));         // [200, 100]

// Edge cases
console.log(swap(0, 5));                     // [5, 0]
console.log(swap(-10, -20));                 // [-20, -10]
```

### **Bonus: Swap Array Elements**
```javascript
/**
 * Swap two elements in an array
 */
function swapArrayElements(arr, i, j) {
  // Validation
  if (i < 0 || i >= arr.length || j < 0 || j >= arr.length) {
    throw new RangeError('Index out of bounds');
  }
  
  // Using destructuring
  [arr[i], arr[j]] = [arr[j], arr[i]];
  
  return arr;
}

// Test
const numbers = [1, 2, 3, 4, 5];
swapArrayElements(numbers, 0, 4);
console.log(numbers);  // [5, 2, 3, 4, 1]

/**
 * Swap object properties
 */
function swapObjectProps(obj, key1, key2) {
  [obj[key1], obj[key2]] = [obj[key2], obj[key1]];
  return obj;
}

// Test
const person = { name: 'John', age: 30 };
swapObjectProps(person, 'name', 'age');
console.log(person);  // { name: 30, age: 'John' }
```

### **Performance Comparison**
```javascript
/**
 * Compare performance of different swap methods
 */
function benchmarkSwap() {
  const iterations = 1000000;
  
  console.time('Destructuring');
  for (let i = 0; i < iterations; i++) {
    let a = 5, b = 10;
    [a, b] = [b, a];
  }
  console.timeEnd('Destructuring');
  
  console.time('Arithmetic');
  for (let i = 0; i < iterations; i++) {
    let a = 5, b = 10;
    a = a + b;
    b = a - b;
    a = a - b;
  }
  console.timeEnd('Arithmetic');
  
  console.time('XOR');
  for (let i = 0; i < iterations; i++) {
    let a = 5, b = 10;
    a = a ^ b;
    b = a ^ b;
    a = a ^ b;
  }
  console.timeEnd('XOR');
}

benchmarkSwap();
```

**Interview Tips:**
- Destructuring is the modern, recommended approach
- Works with any data type
- Arithmetic methods only work with numbers
- XOR is clever but only for integers
- Arithmetic risks overflow with large numbers
- XOR is fastest for integers
- Always prefer readability over cleverness in production

</details>

20. Check if a year is a leap year

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Modulo (Standard Algorithm)**
```javascript
/**
 * Check if year is a leap year
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * Rules:
 * 1. Divisible by 4 AND
 * 2. (Not divisible by 100 OR divisible by 400)
 * 
 * @param {number} year - Year to check
 * @returns {boolean} - True if leap year
 */
function isLeapYear(year) {
  return (year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0);
}

// Test cases
console.log(isLeapYear(2000));   // true (divisible by 400)
console.log(isLeapYear(2020));   // true (divisible by 4, not by 100)
console.log(isLeapYear(1900));   // false (divisible by 100, not by 400)
console.log(isLeapYear(2021));   // false (not divisible by 4)
```

### **Approach 2: Step-by-Step Logic**
```javascript
/**
 * Leap year check with explicit steps
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * More readable version
 */
function isLeapYear(year) {
  // Step 1: If divisible by 400, it's a leap year
  if (year % 400 === 0) {
    return true;
  }
  
  // Step 2: If divisible by 100 (but not 400), it's not a leap year
  if (year % 100 === 0) {
    return false;
  }
  
  // Step 3: If divisible by 4 (but not 100), it's a leap year
  if (year % 4 === 0) {
    return true;
  }
  
  // Step 4: Otherwise, it's not a leap year
  return false;
}

// Test cases
console.log(isLeapYear(2024));   // true
console.log(isLeapYear(2023));   // false
console.log(isLeapYear(2100));   // false
console.log(isLeapYear(2400));   // true
```

### **Approach 3: Using Date Object**
```javascript
/**
 * Check leap year using JavaScript Date
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * Leverages built-in calendar logic
 * February has 29 days in leap years
 */
function isLeapYear(year) {
  // Create date for Feb 29 of given year
  const feb29 = new Date(year, 1, 29);
  
  // If it's still Feb (month 1), then Feb 29 exists
  return feb29.getMonth() === 1;
}

// Alternative: check days in February
function isLeapYear(year) {
  // Last day of February (month 1)
  const lastDayOfFeb = new Date(year, 2, 0);
  return lastDayOfFeb.getDate() === 29;
}

// Test cases
console.log(isLeapYear(2020));   // true
console.log(isLeapYear(2021));   // false
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready leap year checker with validation
 * 
 * @param {number} year - Year to check
 * @returns {boolean} - True if leap year
 * @throws {TypeError} - If year is not a number
 * @throws {RangeError} - If year is out of valid range
 */
function isLeapYear(year) {
  // Validation
  if (typeof year !== 'number' || !Number.isInteger(year)) {
    throw new TypeError('Year must be an integer');
  }
  
  // Reasonable year range (Gregorian calendar started in 1582)
  if (year < 1582) {
    throw new RangeError('Year must be 1582 or later (Gregorian calendar)');
  }
  
  // Leap year algorithm
  return (year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0);
}

// Get leap years in a range
function getLeapYears(startYear, endYear) {
  if (startYear > endYear) {
    throw new RangeError('Start year must be <= end year');
  }
  
  const leapYears = [];
  
  for (let year = startYear; year <= endYear; year++) {
    if (isLeapYear(year)) {
      leapYears.push(year);
    }
  }
  
  return leapYears;
}

// Comprehensive test suite
console.log('=== Leap Years ===');
console.log(isLeapYear(2000));   // true
console.log(isLeapYear(2004));   // true
console.log(isLeapYear(2020));   // true
console.log(isLeapYear(2024));   // true

console.log('\n=== Not Leap Years ===');
console.log(isLeapYear(1900));   // false
console.log(isLeapYear(2001));   // false
console.log(isLeapYear(2021));   // false
console.log(isLeapYear(2100));   // false

console.log('\n=== Leap Years 2020-2040 ===');
console.log(getLeapYears(2020, 2040));
// [2020, 2024, 2028, 2032, 2036, 2040]

// Edge cases
console.log('\n=== Edge Cases ===');
console.log(isLeapYear(2400));   // true
console.log(isLeapYear(1600));   // true
```

### **Bonus: Days in Month**
```javascript
/**
 * Get number of days in a month
 */
function daysInMonth(year, month) {
  // month is 1-12
  const daysInMonths = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
  
  if (month < 1 || month > 12) {
    throw new RangeError('Month must be between 1 and 12');
  }
  
  // Check if February and leap year
  if (month === 2 && isLeapYear(year)) {
    return 29;
  }
  
  return daysInMonths[month - 1];
}

// Using Date object (more reliable)
function daysInMonth(year, month) {
  // Last day of the month
  return new Date(year, month, 0).getDate();
}

// Test
console.log(daysInMonth(2020, 2));   // 29 (Feb in leap year)
console.log(daysInMonth(2021, 2));   // 28 (Feb in non-leap year)
console.log(daysInMonth(2020, 1));   // 31 (January)
console.log(daysInMonth(2020, 4));   // 30 (April)
```

### **Bonus: Next/Previous Leap Year**
```javascript
/**
 * Find next leap year
 */
function nextLeapYear(year) {
  let next = year + 1;
  while (!isLeapYear(next)) {
    next++;
  }
  return next;
}

/**
 * Find previous leap year
 */
function previousLeapYear(year) {
  let prev = year - 1;
  while (!isLeapYear(prev) && prev >= 1582) {
    prev--;
  }
  return prev >= 1582 ? prev : null;
}

// Test
console.log(nextLeapYear(2021));      // 2024
console.log(nextLeapYear(2020));      // 2024
console.log(previousLeapYear(2021));  // 2020
console.log(previousLeapYear(2020));  // 2016
```

**Interview Tips:**
- Standard algorithm: divisible by 4, except centuries unless divisible by 400
- Remember: year 2000 was leap, 1900 was not, 2100 will not be
- Gregorian calendar rule: most simple and correct approach
- Date object method works but less explicit
- Leap year occurs every 4 years, except every 100 years, except every 400 years
- Examples: 2000, 2004, 2008, 2012, 2016, 2020, 2024 are leap years
- 1900, 2100, 2200, 2300 are NOT leap years (divisible by 100 but not 400)

</details>

### **Objects & Basic Operations**

21. Count properties in an object

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Object.keys()**
```javascript
/**
 * Count properties in an object
 * Time Complexity: O(n) where n is number of properties
 * Space Complexity: O(n) for keys array
 * 
 * Only counts own enumerable properties
 * 
 * @param {Object} obj - Object to count
 * @returns {number} - Number of properties
 */
function countProperties(obj) {
  return Object.keys(obj).length;
}

// Test cases
const person = { name: 'John', age: 30, city: 'NYC' };
console.log(countProperties(person));  // 3

const empty = {};
console.log(countProperties(empty));   // 0
```

### **Approach 2: Using for...in Loop**
```javascript
/**
 * Count properties using for...in
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Counts own enumerable properties
 */
function countProperties(obj) {
  let count = 0;
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      count++;
    }
  }
  
  return count;
}

// Test
const car = { brand: 'Toyota', model: 'Camry', year: 2020 };
console.log(countProperties(car));  // 3
```

### **Approach 3: Using Object.getOwnPropertyNames()**
```javascript
/**
 * Count all own properties (including non-enumerable)
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function countAllProperties(obj) {
  return Object.getOwnPropertyNames(obj).length;
}

// Test with non-enumerable property
const obj = { a: 1, b: 2 };
Object.defineProperty(obj, 'c', {
  value: 3,
  enumerable: false
});

console.log(Object.keys(obj).length);              // 2 (enumerable only)
console.log(countAllProperties(obj));              // 3 (includes non-enumerable)
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready property counter with options
 * 
 * @param {Object} obj - Object to count
 * @param {Object} options - Configuration options
 * @param {boolean} options.includeInherited - Include inherited properties (default: false)
 * @param {boolean} options.includeNonEnumerable - Include non-enumerable (default: false)
 * @param {boolean} options.includeSymbols - Include symbol properties (default: false)
 * @returns {number} - Number of properties
 */
function countProperties(obj, options = {}) {
  const {
    includeInherited = false,
    includeNonEnumerable = false,
    includeSymbols = false
  } = options;
  
  // Validation
  if (obj === null || obj === undefined) {
    return 0;
  }
  
  if (typeof obj !== 'object' && typeof obj !== 'function') {
    return 0;
  }
  
  let count = 0;
  
  if (includeNonEnumerable) {
    // Count all own properties
    count = Object.getOwnPropertyNames(obj).length;
  } else {
    // Count only enumerable own properties
    count = Object.keys(obj).length;
  }
  
  // Add symbol properties if requested
  if (includeSymbols) {
    count += Object.getOwnPropertySymbols(obj).length;
  }
  
  // Add inherited properties if requested
  if (includeInherited) {
    let proto = Object.getPrototypeOf(obj);
    
    while (proto && proto !== Object.prototype) {
      if (includeNonEnumerable) {
        count += Object.getOwnPropertyNames(proto).length;
      } else {
        count += Object.keys(proto).length;
      }
      
      if (includeSymbols) {
        count += Object.getOwnPropertySymbols(proto).length;
      }
      
      proto = Object.getPrototypeOf(proto);
    }
  }
  
  return count;
}

// Comprehensive test suite
const person = {
  name: 'John',
  age: 30,
  city: 'NYC'
};

console.log('=== Basic Count ===');
console.log(countProperties(person));  // 3

// With non-enumerable property
Object.defineProperty(person, 'id', {
  value: 123,
  enumerable: false
});

console.log('\n=== With Non-Enumerable ===');
console.log(countProperties(person));  // 3 (default)
console.log(countProperties(person, { includeNonEnumerable: true }));  // 4

// With symbols
const sym = Symbol('secret');
person[sym] = 'hidden';

console.log('\n=== With Symbols ===');
console.log(countProperties(person));  // 3
console.log(countProperties(person, { includeSymbols: true }));  // 4
console.log(countProperties(person, { 
  includeNonEnumerable: true, 
  includeSymbols: true 
}));  // 5

// With inheritance
class Animal {
  constructor() {
    this.type = 'animal';
  }
}

class Dog extends Animal {
  constructor() {
    super();
    this.breed = 'Labrador';
  }
}

const dog = new Dog();

console.log('\n=== With Inheritance ===');
console.log(countProperties(dog));  // 2 (own properties only)
console.log(countProperties(dog, { includeInherited: true }));  // More

// Edge cases
console.log('\n=== Edge Cases ===');
console.log(countProperties(null));       // 0
console.log(countProperties(undefined));  // 0
console.log(countProperties({}));         // 0
console.log(countProperties([]));         // 0
```

### **Bonus: Property Statistics**
```javascript
/**
 * Get detailed property statistics
 */
function getPropertyStats(obj) {
  if (obj === null || obj === undefined) {
    return {
      total: 0,
      enumerable: 0,
      nonEnumerable: 0,
      symbols: 0,
      inherited: 0
    };
  }
  
  const stats = {
    total: 0,
    enumerable: Object.keys(obj).length,
    nonEnumerable: 0,
    symbols: Object.getOwnPropertySymbols(obj).length,
    inherited: 0
  };
  
  stats.nonEnumerable = Object.getOwnPropertyNames(obj).length - stats.enumerable;
  stats.total = stats.enumerable + stats.nonEnumerable + stats.symbols;
  
  // Count inherited
  let proto = Object.getPrototypeOf(obj);
  while (proto && proto !== Object.prototype) {
    stats.inherited += Object.getOwnPropertyNames(proto).length;
    stats.inherited += Object.getOwnPropertySymbols(proto).length;
    proto = Object.getPrototypeOf(proto);
  }
  
  return stats;
}

// Test
const testObj = { a: 1, b: 2 };
Object.defineProperty(testObj, 'c', { value: 3, enumerable: false });
testObj[Symbol('test')] = 'symbol';

console.log(getPropertyStats(testObj));
// { total: 4, enumerable: 2, nonEnumerable: 1, symbols: 1, inherited: ... }
```

**Interview Tips:**
- Object.keys() is the most common and returns enumerable own properties
- Object.getOwnPropertyNames() includes non-enumerable properties
- Object.getOwnPropertySymbols() for symbol properties
- for...in loops through enumerable properties (including inherited)
- Always use hasOwnProperty() when using for...in to check own properties
- Clarify: do you need inherited properties? Non-enumerable? Symbols?

</details>

22. Deep clone an object

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using JSON Methods (Simple)**
```javascript
/**
 * Deep clone using JSON
 * Time Complexity: O(n) where n is number of properties
 * Space Complexity: O(n)
 * 
 * Limitations:
 * - Doesn't handle functions, undefined, symbols, dates
 * - Loses prototypes
 * - Circular references cause error
 * 
 * @param {Object} obj - Object to clone
 * @returns {Object} - Deep cloned object
 */
function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj));
}

// Test cases
const original = {
  name: 'John',
  age: 30,
  address: {
    city: 'NYC',
    country: 'USA'
  },
  hobbies: ['reading', 'coding']
};

const cloned = deepClone(original);
cloned.address.city = 'LA';

console.log(original.address.city);  // 'NYC' (unchanged)
console.log(cloned.address.city);    // 'LA'
```

### **Approach 2: Recursive Deep Clone**
```javascript
/**
 * Recursive deep clone
 * Time Complexity: O(n)
 * Space Complexity: O(d) where d is depth (call stack)
 * 
 * Handles most data types
 */
function deepClone(obj) {
  // Handle null and primitives
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // Handle Date
  if (obj instanceof Date) {
    return new Date(obj.getTime());
  }
  
  // Handle Array
  if (Array.isArray(obj)) {
    return obj.map(item => deepClone(item));
  }
  
  // Handle Object
  const clonedObj = {};
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      clonedObj[key] = deepClone(obj[key]);
    }
  }
  
  return clonedObj;
}

// Test
const data = {
  name: 'John',
  date: new Date(),
  nested: {
    arr: [1, 2, { deep: 'value' }]
  }
};

const copy = deepClone(data);
copy.nested.arr[2].deep = 'modified';

console.log(data.nested.arr[2].deep);    // 'value'
console.log(copy.nested.arr[2].deep);    // 'modified'
```

### **Approach 3: Using structuredClone() (Modern)**
```javascript
/**
 * Deep clone using built-in structuredClone (Node 17+, modern browsers)
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Handles: Date, RegExp, Map, Set, ArrayBuffer, etc.
 * Handles circular references
 * Cannot clone functions
 */
function deepClone(obj) {
  return structuredClone(obj);
}

// Test
const obj = {
  date: new Date(),
  regex: /test/gi,
  map: new Map([['key', 'value']]),
  set: new Set([1, 2, 3])
};

// Circular reference
obj.self = obj;

const cloned = deepClone(obj);
console.log(cloned.date instanceof Date);    // true
console.log(cloned.regex instanceof RegExp); // true
console.log(cloned.map instanceof Map);      // true
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready deep clone with circular reference handling
 * 
 * @param {*} obj - Value to clone
 * @param {WeakMap} hash - Track circular references
 * @returns {*} - Deep cloned value
 */
function deepClone(obj, hash = new WeakMap()) {
  // Primitives and null
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // Check for circular reference
  if (hash.has(obj)) {
    return hash.get(obj);
  }
  
  // Handle Date
  if (obj instanceof Date) {
    return new Date(obj.getTime());
  }
  
  // Handle RegExp
  if (obj instanceof RegExp) {
    return new RegExp(obj.source, obj.flags);
  }
  
  // Handle Map
  if (obj instanceof Map) {
    const clonedMap = new Map();
    hash.set(obj, clonedMap);
    
    obj.forEach((value, key) => {
      clonedMap.set(key, deepClone(value, hash));
    });
    
    return clonedMap;
  }
  
  // Handle Set
  if (obj instanceof Set) {
    const clonedSet = new Set();
    hash.set(obj, clonedSet);
    
    obj.forEach(value => {
      clonedSet.add(deepClone(value, hash));
    });
    
    return clonedSet;
  }
  
  // Handle Array
  if (Array.isArray(obj)) {
    const clonedArr = [];
    hash.set(obj, clonedArr);
    
    obj.forEach((item, index) => {
      clonedArr[index] = deepClone(item, hash);
    });
    
    return clonedArr;
  }
  
  // Handle Object
  // Preserve prototype
  const clonedObj = Object.create(Object.getPrototypeOf(obj));
  hash.set(obj, clonedObj);
  
  // Clone all own properties (including non-enumerable)
  Object.getOwnPropertyNames(obj).forEach(key => {
    const descriptor = Object.getOwnPropertyDescriptor(obj, key);
    
    if (descriptor.value !== undefined) {
      descriptor.value = deepClone(descriptor.value, hash);
    }
    
    Object.defineProperty(clonedObj, key, descriptor);
  });
  
  // Clone symbol properties
  Object.getOwnPropertySymbols(obj).forEach(sym => {
    const descriptor = Object.getOwnPropertyDescriptor(obj, sym);
    
    if (descriptor.value !== undefined) {
      descriptor.value = deepClone(descriptor.value, hash);
    }
    
    Object.defineProperty(clonedObj, sym, descriptor);
  });
  
  return clonedObj;
}

// Comprehensive test suite
console.log('=== Basic Objects ===');
const person = {
  name: 'John',
  age: 30,
  address: {
    city: 'NYC',
    zip: 10001
  }
};

const clonedPerson = deepClone(person);
clonedPerson.address.city = 'LA';

console.log(person.address.city);        // 'NYC'
console.log(clonedPerson.address.city);  // 'LA'

console.log('\n=== Arrays ===');
const arr = [1, [2, 3], { a: 4 }];
const clonedArr = deepClone(arr);
clonedArr[1][0] = 999;
clonedArr[2].a = 888;

console.log(arr[1][0]);      // 2
console.log(arr[2].a);       // 4
console.log(clonedArr[1][0]); // 999
console.log(clonedArr[2].a);  // 888

console.log('\n=== Special Objects ===');
const special = {
  date: new Date('2024-01-01'),
  regex: /test/gi,
  map: new Map([['key', 'value']]),
  set: new Set([1, 2, 3])
};

const clonedSpecial = deepClone(special);
console.log(clonedSpecial.date instanceof Date);    // true
console.log(clonedSpecial.regex instanceof RegExp); // true
console.log(clonedSpecial.map instanceof Map);      // true
console.log(clonedSpecial.set instanceof Set);      // true

console.log('\n=== Circular Reference ===');
const circular = { name: 'test' };
circular.self = circular;

const clonedCircular = deepClone(circular);
console.log(clonedCircular.self === clonedCircular); // true

console.log('\n=== Class Instance ===');
class Person {
  constructor(name) {
    this.name = name;
  }
  
  greet() {
    return `Hello, ${this.name}`;
  }
}

const john = new Person('John');
const clonedJohn = deepClone(john);

console.log(clonedJohn instanceof Person);  // true
console.log(clonedJohn.greet());           // 'Hello, John'
```

### **Bonus: Shallow Clone Comparison**
```javascript
/**
 * Shallow clone (only top level)
 */
function shallowClone(obj) {
  // Using spread operator
  return { ...obj };
  
  // Or using Object.assign
  // return Object.assign({}, obj);
}

// Test shallow vs deep
const original = {
  name: 'John',
  address: { city: 'NYC' }
};

const shallow = shallowClone(original);
const deep = deepClone(original);

shallow.name = 'Jane';           // Only affects shallow
shallow.address.city = 'LA';     // Affects BOTH original and shallow

deep.address.city = 'Boston';    // Only affects deep

console.log(original.name);          // 'John'
console.log(original.address.city);  // 'LA' (shallow modified it!)
console.log(shallow.address.city);   // 'LA'
console.log(deep.address.city);      // 'Boston'
```

**Interview Tips:**
- JSON.stringify/parse is simple but has limitations (no functions, dates as strings)
- Recursive approach is most flexible and commonly used
- structuredClone() is the modern standard (Node 17+, recent browsers)
- Always handle circular references in production
- Shallow clone: spread operator, Object.assign()
- Deep clone: nested objects/arrays are fully copied
- Consider lodash.cloneDeep() for production if available

</details>

23. Merge two objects

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Spread Operator (Shallow)**
```javascript
/**
 * Merge objects using spread operator
 * Time Complexity: O(n + m)
 * Space Complexity: O(n + m)
 * 
 * Shallow merge - later values override earlier ones
 * 
 * @param {Object} obj1 - First object
 * @param {Object} obj2 - Second object
 * @returns {Object} - Merged object
 */
function mergeObjects(obj1, obj2) {
  return { ...obj1, ...obj2 };
}

// Test cases
const obj1 = { a: 1, b: 2, c: 3 };
const obj2 = { b: 4, d: 5 };

const merged = mergeObjects(obj1, obj2);
console.log(merged);  // { a: 1, b: 4, c: 3, d: 5 }
```

### **Approach 2: Using Object.assign()**
```javascript
/**
 * Merge using Object.assign()
 * Time Complexity: O(n + m)
 * Space Complexity: O(n + m)
 * 
 * Shallow merge
 */
function mergeObjects(obj1, obj2) {
  return Object.assign({}, obj1, obj2);
}

// Test
const a = { x: 1, y: 2 };
const b = { y: 3, z: 4 };

console.log(mergeObjects(a, b));  // { x: 1, y: 3, z: 4 }
```

### **Approach 3: Deep Merge (Recursive)**
```javascript
/**
 * Deep merge - recursively merge nested objects
 * Time Complexity: O(n * d) where d is depth
 * Space Complexity: O(d) for recursion
 * 
 * Handles nested objects and arrays
 */
function deepMerge(obj1, obj2) {
  // If either is not an object, return obj2 (override)
  if (typeof obj1 !== 'object' || obj1 === null) return obj2;
  if (typeof obj2 !== 'object' || obj2 === null) return obj2;
  
  // Create result starting with obj1
  const result = { ...obj1 };
  
  // Merge each property from obj2
  for (const key in obj2) {
    if (obj2.hasOwnProperty(key)) {
      // If both are objects, merge recursively
      if (
        typeof obj2[key] === 'object' &&
        obj2[key] !== null &&
        !Array.isArray(obj2[key]) &&
        typeof result[key] === 'object' &&
        result[key] !== null &&
        !Array.isArray(result[key])
      ) {
        result[key] = deepMerge(result[key], obj2[key]);
      } else {
        // Otherwise, override
        result[key] = obj2[key];
      }
    }
  }
  
  return result;
}

// Test deep merge
const user1 = {
  name: 'John',
  age: 30,
  address: {
    city: 'NYC',
    zip: 10001
  },
  hobbies: ['reading']
};

const user2 = {
  age: 31,
  address: {
    country: 'USA'
  },
  hobbies: ['coding']
};

const merged = deepMerge(user1, user2);
console.log(merged);
/*
{
  name: 'John',
  age: 31,
  address: {
    city: 'NYC',
    zip: 10001,
    country: 'USA'
  },
  hobbies: ['coding']
}
*/
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready object merger with options
 * 
 * @param {Object} target - Target object
 * @param {Object} source - Source object
 * @param {Object} options - Merge options
 * @param {boolean} options.deep - Deep merge (default: true)
 * @param {boolean} options.arrayMerge - Merge arrays instead of replace (default: false)
 * @param {boolean} options.clone - Clone instead of mutate target (default: true)
 * @returns {Object} - Merged object
 */
function mergeObjects(target, source, options = {}) {
  const {
    deep = true,
    arrayMerge = false,
    clone = true
  } = options;
  
  // Validation
  if (typeof target !== 'object' || target === null) {
    throw new TypeError('Target must be an object');
  }
  
  if (typeof source !== 'object' || source === null) {
    return clone ? { ...target } : target;
  }
  
  // Start with target (clone if requested)
  const result = clone ? { ...target } : target;
  
  // Helper for deep merge
  function merge(tgt, src) {
    for (const key in src) {
      if (src.hasOwnProperty(key)) {
        const sourceValue = src[key];
        const targetValue = tgt[key];
        
        // Handle arrays
        if (Array.isArray(sourceValue)) {
          if (arrayMerge && Array.isArray(targetValue)) {
            tgt[key] = [...targetValue, ...sourceValue];
          } else {
            tgt[key] = [...sourceValue]; // Clone array
          }
        }
        // Handle nested objects
        else if (
          deep &&
          sourceValue !== null &&
          typeof sourceValue === 'object' &&
          targetValue !== null &&
          typeof targetValue === 'object' &&
          !Array.isArray(targetValue)
        ) {
          tgt[key] = merge({ ...targetValue }, sourceValue);
        }
        // Override primitive or null values
        else {
          tgt[key] = sourceValue;
        }
      }
    }
    
    return tgt;
  }
  
  return merge(result, source);
}

// Merge multiple objects
function mergeMultiple(...objects) {
  return objects.reduce((acc, obj) => mergeObjects(acc, obj), {});
}

// Comprehensive test suite
console.log('=== Shallow Merge ===');
const obj1 = { a: 1, b: { x: 10 } };
const obj2 = { b: { y: 20 }, c: 3 };

const shallow = mergeObjects(obj1, obj2, { deep: false });
console.log(shallow);
// { a: 1, b: { y: 20 }, c: 3 } - b is replaced, not merged

console.log('\n=== Deep Merge ===');
const deep = mergeObjects(obj1, obj2, { deep: true });
console.log(deep);
// { a: 1, b: { x: 10, y: 20 }, c: 3 } - b is merged

console.log('\n=== Array Merge ===');
const arr1 = { items: [1, 2], name: 'test' };
const arr2 = { items: [3, 4], age: 30 };

const arrMerged = mergeObjects(arr1, arr2, { arrayMerge: true });
console.log(arrMerged);
// { items: [1, 2, 3, 4], name: 'test', age: 30 }

const arrReplaced = mergeObjects(arr1, arr2, { arrayMerge: false });
console.log(arrReplaced);
// { items: [3, 4], name: 'test', age: 30 }

console.log('\n=== Multiple Objects ===');
const result = mergeMultiple(
  { a: 1, b: 2 },
  { b: 3, c: 4 },
  { c: 5, d: 6 }
);
console.log(result);  // { a: 1, b: 3, c: 5, d: 6 }

console.log('\n=== Mutation Test ===');
const original = { x: 1, nested: { y: 2 } };
const addition = { nested: { z: 3 } };

const mutated = mergeObjects(original, addition, { clone: false });
console.log(original === mutated);  // true (mutated in place)

const cloned = mergeObjects(original, addition, { clone: true });
console.log(original === cloned);   // false (new object)
```

### **Bonus: Smart Merge Strategies**
```javascript
/**
 * Custom merge with conflict resolution
 */
function mergeWithStrategy(obj1, obj2, strategy = 'override') {
  const result = { ...obj1 };
  
  for (const key in obj2) {
    if (obj2.hasOwnProperty(key)) {
      // If key exists in both
      if (key in result) {
        switch (strategy) {
          case 'override':
            result[key] = obj2[key];
            break;
            
          case 'keep':
            // Keep obj1 value
            break;
            
          case 'concat': // For arrays/strings
            if (Array.isArray(result[key]) && Array.isArray(obj2[key])) {
              result[key] = [...result[key], ...obj2[key]];
            } else if (typeof result[key] === 'string' && typeof obj2[key] === 'string') {
              result[key] = result[key] + obj2[key];
            } else {
              result[key] = obj2[key];
            }
            break;
            
          case 'sum': // For numbers
            if (typeof result[key] === 'number' && typeof obj2[key] === 'number') {
              result[key] = result[key] + obj2[key];
            } else {
              result[key] = obj2[key];
            }
            break;
        }
      } else {
        result[key] = obj2[key];
      }
    }
  }
  
  return result;
}

// Test strategies
console.log(mergeWithStrategy(
  { a: 1, b: [1, 2] },
  { a: 2, b: [3, 4] },
  'concat'
));  // { a: 2, b: [1, 2, 3, 4] }

console.log(mergeWithStrategy(
  { a: 10, b: 20 },
  { a: 5, b: 15 },
  'sum'
));  // { a: 15, b: 35 }
```

**Interview Tips:**
- Spread operator is simplest for shallow merge
- Object.assign() modifies first argument (use empty object as target)
- Deep merge requires recursion for nested objects
- Clarify: should arrays be merged or replaced?
- Clarify: should it mutate original or return new object?
- Consider using lodash.merge() for production
- Order matters: later objects override earlier ones

</details>

24. Check if an object is empty

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Object.keys()**
```javascript
/**
 * Check if object is empty using Object.keys()
 * Time Complexity: O(1) - only checks if keys exist
 * Space Complexity: O(1)
 * 
 * Checks enumerable own properties only
 * 
 * @param {Object} obj - Object to check
 * @returns {boolean} - True if empty
 */
function isEmpty(obj) {
  return Object.keys(obj).length === 0;
}

// Test cases
console.log(isEmpty({}));                    // true
console.log(isEmpty({ name: 'John' }));      // false
console.log(isEmpty({ a: undefined }));      // false (has property)
```

### **Approach 2: Using for...in Loop**
```javascript
/**
 * Check if empty using for...in
 * Time Complexity: O(1) - returns on first property
 * Space Complexity: O(1)
 * 
 * Also checks enumerable properties
 */
function isEmpty(obj) {
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      return false;
    }
  }
  return true;
}

// Test
console.log(isEmpty({}));                    // true
console.log(isEmpty({ x: 1 }));              // false
```

### **Approach 3: Using JSON.stringify()**
```javascript
/**
 * Check if empty using JSON
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Simple but less efficient
 */
function isEmpty(obj) {
  return JSON.stringify(obj) === '{}';
}

// Test
console.log(isEmpty({}));                    // true
console.log(isEmpty({ a: 1 }));              // false
```

### **Approach 4: Using Object.getOwnPropertyNames()**
```javascript
/**
 * Check including non-enumerable properties
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 */
function isEmpty(obj) {
  return Object.getOwnPropertyNames(obj).length === 0 &&
         Object.getOwnPropertySymbols(obj).length === 0;
}

// Test with non-enumerable
const obj = {};
Object.defineProperty(obj, 'hidden', {
  value: 'secret',
  enumerable: false
});

console.log(Object.keys(obj).length === 0);  // true (enumerable only)
console.log(isEmpty(obj));                    // false (includes non-enumerable)
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready empty check with validation
 * 
 * @param {*} value - Value to check
 * @param {Object} options - Check options
 * @param {boolean} options.includeNonEnumerable - Check non-enumerable (default: false)
 * @param {boolean} options.includeSymbols - Check symbol properties (default: false)
 * @param {boolean} options.includeInherited - Check inherited properties (default: false)
 * @returns {boolean} - True if empty
 */
function isEmpty(value, options = {}) {
  const {
    includeNonEnumerable = false,
    includeSymbols = false,
    includeInherited = false
  } = options;
  
  // Handle null and undefined
  if (value === null || value === undefined) {
    return true;
  }
  
  // Handle primitives
  if (typeof value !== 'object' && typeof value !== 'function') {
    return false; // Primitives are not "empty"
  }
  
  // Handle arrays
  if (Array.isArray(value)) {
    return value.length === 0;
  }
  
  // Handle Map
  if (value instanceof Map) {
    return value.size === 0;
  }
  
  // Handle Set
  if (value instanceof Set) {
    return value.size === 0;
  }
  
  // Check own properties
  if (includeNonEnumerable) {
    if (Object.getOwnPropertyNames(value).length > 0) {
      return false;
    }
  } else {
    if (Object.keys(value).length > 0) {
      return false;
    }
  }
  
  // Check symbols
  if (includeSymbols) {
    if (Object.getOwnPropertySymbols(value).length > 0) {
      return false;
    }
  }
  
  // Check inherited properties
  if (includeInherited) {
    for (const key in value) {
      return false; // Has at least one inherited property
    }
  }
  
  return true;
}

// Comprehensive test suite
console.log('=== Basic Objects ===');
console.log(isEmpty({}));                          // true
console.log(isEmpty({ a: 1 }));                    // false
console.log(isEmpty({ a: undefined }));            // false (has property)

console.log('\n=== Arrays ===');
console.log(isEmpty([]));                          // true
console.log(isEmpty([1, 2, 3]));                   // false
console.log(isEmpty(['']));                        // false (has element)

console.log('\n=== Map and Set ===');
console.log(isEmpty(new Map()));                   // true
console.log(isEmpty(new Map([['key', 'value']]))); // false
console.log(isEmpty(new Set()));                   // true
console.log(isEmpty(new Set([1, 2])));             // false

console.log('\n=== Primitives ===');
console.log(isEmpty(null));                        // true
console.log(isEmpty(undefined));                   // true
console.log(isEmpty(''));                          // false
console.log(isEmpty(0));                           // false

console.log('\n=== Non-Enumerable ===');
const obj = {};
Object.defineProperty(obj, 'hidden', {
  value: 'secret',
  enumerable: false
});

console.log(isEmpty(obj));                         // true (default)
console.log(isEmpty(obj, { includeNonEnumerable: true })); // false

console.log('\n=== Symbols ===');
const symObj = {};
symObj[Symbol('test')] = 'value';

console.log(isEmpty(symObj));                      // true (default)
console.log(isEmpty(symObj, { includeSymbols: true })); // false

console.log('\n=== Inherited Properties ===');
const parent = { inherited: 'value' };
const child = Object.create(parent);

console.log(isEmpty(child));                       // true (own properties)
console.log(isEmpty(child, { includeInherited: true })); // false
```

### **Bonus: Type-Specific Empty Checks**
```javascript
/**
 * Comprehensive emptiness checker for different types
 */
function isEmptyValue(value) {
  // null or undefined
  if (value === null || value === undefined) return true;
  
  // Boolean
  if (typeof value === 'boolean') return false;
  
  // Number
  if (typeof value === 'number') return value === 0 || isNaN(value);
  
  // String
  if (typeof value === 'string') return value.trim().length === 0;
  
  // Array
  if (Array.isArray(value)) return value.length === 0;
  
  // Map or Set
  if (value instanceof Map || value instanceof Set) return value.size === 0;
  
  // Object
  if (typeof value === 'object') return Object.keys(value).length === 0;
  
  return false;
}

// Test
console.log(isEmptyValue(null));         // true
console.log(isEmptyValue(undefined));    // true
console.log(isEmptyValue(''));           // true
console.log(isEmptyValue('  '));         // true (whitespace)
console.log(isEmptyValue(0));            // true
console.log(isEmptyValue([]));           // true
console.log(isEmptyValue({}));           // true
console.log(isEmptyValue(false));        // false (boolean is valid)
console.log(isEmptyValue('test'));       // false
console.log(isEmptyValue([1]));          // false
```

**Interview Tips:**
- Object.keys() is most common and efficient for objects
- Returns array of enumerable own property names
- Empty object: {} has length 0
- Object with undefined values is NOT empty: { a: undefined }
- Consider different "empty" meanings: no properties vs no enumerable properties
- for...in is more efficient (stops at first property)
- JSON.stringify is simple but inefficient (serializes entire object)
- Clarify: should check inherited properties? Non-enumerable? Symbols?

</details>

25. Convert object to array of key-value pairs

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Object.entries()**
```javascript
/**
 * Convert object to array of [key, value] pairs
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Returns enumerable own properties
 * 
 * @param {Object} obj - Object to convert
 * @returns {Array} - Array of [key, value] pairs
 */
function objectToArray(obj) {
  return Object.entries(obj);
}

// Test cases
const person = { name: 'John', age: 30, city: 'NYC' };
console.log(objectToArray(person));
// [['name', 'John'], ['age', 30], ['city', 'NYC']]
```

### **Approach 2: Using Object.keys() and map()**
```javascript
/**
 * Convert using Object.keys() and map
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function objectToArray(obj) {
  return Object.keys(obj).map(key => [key, obj[key]]);
}

// Test
const data = { a: 1, b: 2, c: 3 };
console.log(objectToArray(data));
// [['a', 1], ['b', 2], ['c', 3]]
```

### **Approach 3: Using for...in Loop**
```javascript
/**
 * Convert using for...in
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function objectToArray(obj) {
  const result = [];
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      result.push([key, obj[key]]);
    }
  }
  
  return result;
}

// Test
const obj = { x: 10, y: 20, z: 30 };
console.log(objectToArray(obj));
// [['x', 10], ['y', 20], ['z', 30]]
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready object to array converter
 * 
 * @param {Object} obj - Object to convert
 * @param {Object} options - Conversion options
 * @param {string} options.format - Output format: 'entries', 'keys', 'values', 'objects'
 * @param {boolean} options.includeNonEnumerable - Include non-enumerable (default: false)
 * @param {boolean} options.includeSymbols - Include symbol properties (default: false)
 * @returns {Array} - Converted array
 */
function objectToArray(obj, options = {}) {
  const {
    format = 'entries',
    includeNonEnumerable = false,
    includeSymbols = false
  } = options;
  
  // Validation
  if (obj === null || obj === undefined) {
    return [];
  }
  
  if (typeof obj !== 'object' && typeof obj !== 'function') {
    throw new TypeError('Input must be an object');
  }
  
  // Get property keys
  let keys;
  
  if (includeNonEnumerable) {
    keys = Object.getOwnPropertyNames(obj);
  } else {
    keys = Object.keys(obj);
  }
  
  // Add symbol properties if requested
  if (includeSymbols) {
    keys = [...keys, ...Object.getOwnPropertySymbols(obj)];
  }
  
  // Convert based on format
  switch (format) {
    case 'entries':
      // [[key, value], ...]
      return keys.map(key => [key, obj[key]]);
      
    case 'keys':
      // [key1, key2, ...]
      return keys;
      
    case 'values':
      // [value1, value2, ...]
      return keys.map(key => obj[key]);
      
    case 'objects':
      // [{key: 'key1', value: 'value1'}, ...]
      return keys.map(key => ({ key, value: obj[key] }));
      
    default:
      throw new Error(`Unknown format: ${format}`);
  }
}

// Reverse: array to object
function arrayToObject(arr, format = 'entries') {
  const obj = {};
  
  switch (format) {
    case 'entries':
      // [[key, value], ...] => object
      arr.forEach(([key, value]) => {
        obj[key] = value;
      });
      break;
      
    case 'objects':
      // [{key: 'k', value: 'v'}, ...] => object
      arr.forEach(item => {
        obj[item.key] = item.value;
      });
      break;
      
    default:
      throw new Error(`Unknown format: ${format}`);
  }
  
  return obj;
}

// Comprehensive test suite
const person = {
  name: 'John',
  age: 30,
  city: 'NYC',
  country: 'USA'
};

console.log('=== Entries Format (Default) ===');
console.log(objectToArray(person));
// [['name', 'John'], ['age', 30], ['city', 'NYC'], ['country', 'USA']]

console.log('\n=== Keys Format ===');
console.log(objectToArray(person, { format: 'keys' }));
// ['name', 'age', 'city', 'country']

console.log('\n=== Values Format ===');
console.log(objectToArray(person, { format: 'values' }));
// ['John', 30, 'NYC', 'USA']

console.log('\n=== Objects Format ===');
console.log(objectToArray(person, { format: 'objects' }));
/*
[
  { key: 'name', value: 'John' },
  { key: 'age', value: 30 },
  { key: 'city', value: 'NYC' },
  { key: 'country', value: 'USA' }
]
*/

console.log('\n=== With Non-Enumerable ===');
const obj = { a: 1, b: 2 };
Object.defineProperty(obj, 'hidden', {
  value: 'secret',
  enumerable: false
});

console.log(objectToArray(obj));  // [['a', 1], ['b', 2]]
console.log(objectToArray(obj, { includeNonEnumerable: true }));
// [['a', 1], ['b', 2], ['hidden', 'secret']]

console.log('\n=== With Symbols ===');
const symObj = { x: 1 };
const sym = Symbol('test');
symObj[sym] = 'symbol value';

console.log(objectToArray(symObj));  // [['x', 1]]
console.log(objectToArray(symObj, { includeSymbols: true }));
// [['x', 1], [Symbol(test), 'symbol value']]

console.log('\n=== Array to Object ===');
const entries = [['name', 'Jane'], ['age', 25]];
console.log(arrayToObject(entries));  // { name: 'Jane', age: 25 }

const objects = [
  { key: 'x', value: 10 },
  { key: 'y', value: 20 }
];
console.log(arrayToObject(objects, 'objects'));  // { x: 10, y: 20 }
```

### **Bonus: Advanced Conversions**
```javascript
/**
 * Convert nested object to flat array
 */
function objectToFlatArray(obj, prefix = '') {
  const result = [];
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      const fullKey = prefix ? `${prefix}.${key}` : key;
      const value = obj[key];
      
      if (value !== null && typeof value === 'object' && !Array.isArray(value)) {
        // Recursively flatten nested objects
        result.push(...objectToFlatArray(value, fullKey));
      } else {
        result.push([fullKey, value]);
      }
    }
  }
  
  return result;
}

// Convert object to Map
function objectToMap(obj) {
  return new Map(Object.entries(obj));
}

// Convert Map to object
function mapToObject(map) {
  return Object.fromEntries(map);
}

// Test
const nested = {
  user: {
    name: 'John',
    address: {
      city: 'NYC',
      zip: 10001
    }
  },
  active: true
};

console.log(objectToFlatArray(nested));
/*
[
  ['user.name', 'John'],
  ['user.address.city', 'NYC'],
  ['user.address.zip', 10001],
  ['active', true]
]
*/

const data = { a: 1, b: 2, c: 3 };
const map = objectToMap(data);
console.log(map);  // Map(3) { 'a' => 1, 'b' => 2, 'c' => 3 }

const objFromMap = mapToObject(map);
console.log(objFromMap);  // { a: 1, b: 2, c: 3 }
```

**Interview Tips:**
- Object.entries() is the modern, recommended approach
- Returns array of [key, value] tuples
- Only includes enumerable own properties
- Object.keys() returns just keys, Object.values() returns just values
- Object.fromEntries() is the reverse (array to object)
- Can convert to Map using new Map(Object.entries(obj))
- For nested objects, consider flattening or recursive conversion

</details>

26. Group array of objects by a property

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using reduce()**
```javascript
/**
 * Group array of objects by a property
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {Array} arr - Array of objects
 * @param {string} key - Property to group by
 * @returns {Object} - Grouped object
 */
function groupBy(arr, key) {
  return arr.reduce((result, item) => {
    const groupKey = item[key];
    
    if (!result[groupKey]) {
      result[groupKey] = [];
    }
    
    result[groupKey].push(item);
    return result;
  }, {});
}

// Test cases
const users = [
  { name: 'John', age: 30, city: 'NYC' },
  { name: 'Jane', age: 25, city: 'LA' },
  { name: 'Bob', age: 30, city: 'NYC' },
  { name: 'Alice', age: 25, city: 'Chicago' }
];

console.log(groupBy(users, 'age'));
/*
{
  '25': [
    { name: 'Jane', age: 25, city: 'LA' },
    { name: 'Alice', age: 25, city: 'Chicago' }
  ],
  '30': [
    { name: 'John', age: 30, city: 'NYC' },
    { name: 'Bob', age: 30, city: 'NYC' }
  ]
}
*/

console.log(groupBy(users, 'city'));
/*
{
  'NYC': [
    { name: 'John', age: 30, city: 'NYC' },
    { name: 'Bob', age: 30, city: 'NYC' }
  ],
  'LA': [{ name: 'Jane', age: 25, city: 'LA' }],
  'Chicago': [{ name: 'Alice', age: 25, city: 'Chicago' }]
}
*/
```

### **Approach 2: Using forEach()**
```javascript
/**
 * Group using forEach
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function groupBy(arr, key) {
  const result = {};
  
  arr.forEach(item => {
    const groupKey = item[key];
    
    if (!result[groupKey]) {
      result[groupKey] = [];
    }
    
    result[groupKey].push(item);
  });
  
  return result;
}

// Test
const products = [
  { name: 'Laptop', category: 'Electronics', price: 1000 },
  { name: 'Phone', category: 'Electronics', price: 500 },
  { name: 'Shirt', category: 'Clothing', price: 50 }
];

console.log(groupBy(products, 'category'));
/*
{
  'Electronics': [
    { name: 'Laptop', category: 'Electronics', price: 1000 },
    { name: 'Phone', category: 'Electronics', price: 500 }
  ],
  'Clothing': [
    { name: 'Shirt', category: 'Clothing', price: 50 }
  ]
}
*/
```

### **Approach 3: Using Map for Complex Keys**
```javascript
/**
 * Group using Map (better for non-string keys)
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function groupBy(arr, key) {
  const map = new Map();
  
  arr.forEach(item => {
    const groupKey = item[key];
    
    if (!map.has(groupKey)) {
      map.set(groupKey, []);
    }
    
    map.get(groupKey).push(item);
  });
  
  return map;
}

// Test with complex keys
const data = [
  { id: 1, tags: ['A', 'B'] },
  { id: 2, tags: ['A', 'B'] },
  { id: 3, tags: ['C'] }
];

// Group by array reference (Note: arrays are compared by reference)
const grouped = groupBy(data, 'tags');
console.log(grouped);
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready groupBy with advanced features
 * 
 * @param {Array} arr - Array of objects
 * @param {string|Function} key - Property name or function to extract key
 * @param {Object} options - Grouping options
 * @param {Function} options.transform - Transform each group's items
 * @param {Function} options.aggregate - Aggregate each group (instead of array)
 * @returns {Object|Map} - Grouped data
 */
function groupBy(arr, key, options = {}) {
  const { transform, aggregate } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('First argument must be an array');
  }
  
  const result = {};
  
  // Helper to get key from item
  const getKey = typeof key === 'function' 
    ? key 
    : (item) => item[key];
  
  arr.forEach(item => {
    const groupKey = getKey(item);
    
    // Skip undefined/null keys
    if (groupKey === undefined || groupKey === null) {
      return;
    }
    
    if (!result[groupKey]) {
      result[groupKey] = [];
    }
    
    result[groupKey].push(item);
  });
  
  // Apply transformations if provided
  if (transform) {
    Object.keys(result).forEach(key => {
      result[key] = transform(result[key]);
    });
  }
  
  // Apply aggregation if provided
  if (aggregate) {
    Object.keys(result).forEach(key => {
      result[key] = aggregate(result[key]);
    });
  }
  
  return result;
}

// Group by multiple properties
function groupByMultiple(arr, keys) {
  return arr.reduce((result, item) => {
    // Create composite key
    const compositeKey = keys.map(k => item[k]).join('|');
    
    if (!result[compositeKey]) {
      result[compositeKey] = [];
    }
    
    result[compositeKey].push(item);
    return result;
  }, {});
}

// Comprehensive test suite
const employees = [
  { name: 'John', department: 'IT', salary: 80000, level: 'Senior' },
  { name: 'Jane', department: 'IT', salary: 70000, level: 'Mid' },
  { name: 'Bob', department: 'HR', salary: 60000, level: 'Senior' },
  { name: 'Alice', department: 'IT', salary: 90000, level: 'Senior' },
  { name: 'Charlie', department: 'HR', salary: 55000, level: 'Junior' }
];

console.log('=== Basic Grouping ===');
console.log(groupBy(employees, 'department'));

console.log('\n=== Group by Function ===');
const byRange = groupBy(employees, emp => {
  if (emp.salary < 60000) return 'Low';
  if (emp.salary < 80000) return 'Medium';
  return 'High';
});
console.log(byRange);

console.log('\n=== Group with Transform (count) ===');
const counts = groupBy(employees, 'department', {
  transform: group => group.length
});
console.log(counts);  // { IT: 3, HR: 2 }

console.log('\n=== Group with Aggregation (average salary) ===');
const avgSalary = groupBy(employees, 'department', {
  aggregate: group => {
    const total = group.reduce((sum, emp) => sum + emp.salary, 0);
    return total / group.length;
  }
});
console.log(avgSalary);  // { IT: 80000, HR: 57500 }

console.log('\n=== Group by Multiple Properties ===');
const multiGroup = groupByMultiple(employees, ['department', 'level']);
console.log(multiGroup);
/*
{
  'IT|Senior': [...],
  'IT|Mid': [...],
  'HR|Senior': [...],
  'HR|Junior': [...]
}
*/

console.log('\n=== Nested Grouping ===');
function nestedGroupBy(arr, keys) {
  if (keys.length === 0) return arr;
  
  const [firstKey, ...remainingKeys] = keys;
  const grouped = groupBy(arr, firstKey);
  
  if (remainingKeys.length > 0) {
    Object.keys(grouped).forEach(key => {
      grouped[key] = nestedGroupBy(grouped[key], remainingKeys);
    });
  }
  
  return grouped;
}

const nested = nestedGroupBy(employees, ['department', 'level']);
console.log(nested);
/*
{
  IT: {
    Senior: [...],
    Mid: [...]
  },
  HR: {
    Senior: [...],
    Junior: [...]
  }
}
*/
```

### **Bonus: Using Object.groupBy() (ES2024)**
```javascript
/**
 * Modern groupBy using built-in method (ES2024)
 * Available in Node 21+ and modern browsers
 */
function groupByModern(arr, key) {
  return Object.groupBy(arr, item => item[key]);
}

// Or with Map
function groupByMap(arr, key) {
  return Map.groupBy(arr, item => item[key]);
}

// Test
const items = [
  { type: 'fruit', name: 'apple' },
  { type: 'fruit', name: 'banana' },
  { type: 'veggie', name: 'carrot' }
];

// Note: Object.groupBy might not be available in all environments yet
// console.log(Object.groupBy(items, item => item.type));
```

**Interview Tips:**
- reduce() is the most common and flexible approach
- forEach() is more readable for simple cases
- Use Map instead of Object for non-string keys
- Function as key parameter allows computed grouping (e.g., by ranges)
- Consider lodash.groupBy() for production use
- ES2024 introduces Object.groupBy() natively
- For multiple properties, can use composite keys or nested grouping
- Common use cases: analytics, data visualization, reporting

</details>

27. Remove falsy values from an array

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using filter() with Boolean**
```javascript
/**
 * Remove falsy values using filter
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Falsy values: false, 0, '', null, undefined, NaN
 * 
 * @param {Array} arr - Array with potential falsy values
 * @returns {Array} - Array with only truthy values
 */
function removeFalsy(arr) {
  return arr.filter(Boolean);
}

// Test cases
console.log(removeFalsy([0, 1, false, 2, '', 3, null, 'hello', undefined, NaN]));
// [1, 2, 3, 'hello']

console.log(removeFalsy([false, 0, '', null, undefined, NaN]));
// []

console.log(removeFalsy([1, 2, 3, 'test', true]));
// [1, 2, 3, 'test', true] (all truthy)
```

### **Approach 2: Using filter() with Explicit Check**
```javascript
/**
 * Remove falsy with explicit condition
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function removeFalsy(arr) {
  return arr.filter(item => item);
}

// Or more explicit
function removeFalsy(arr) {
  return arr.filter(item => !!item);
}

// Test
const mixed = [1, 0, 'text', false, null, 2, undefined, '', 3];
console.log(removeFalsy(mixed));  // [1, 'text', 2, 3]
```

### **Approach 3: Using reduce()**
```javascript
/**
 * Remove falsy using reduce
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function removeFalsy(arr) {
  return arr.reduce((result, item) => {
    if (item) {
      result.push(item);
    }
    return result;
  }, []);
}

// Test
console.log(removeFalsy([0, 1, false, 2, '', 3]));  // [1, 2, 3]
```

### **Approach 4: In-Place Removal (Modify Original)**
```javascript
/**
 * Remove falsy in-place (mutates array)
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
function removeFalsy(arr) {
  let writeIndex = 0;
  
  for (let i = 0; i < arr.length; i++) {
    if (arr[i]) {
      arr[writeIndex] = arr[i];
      writeIndex++;
    }
  }
  
  arr.length = writeIndex;
  return arr;
}

// Test
const data = [1, 0, 2, false, 3, null, 4];
removeFalsy(data);
console.log(data);  // [1, 2, 3, 4] (original array modified)
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready falsy remover with options
 * 
 * @param {Array} arr - Input array
 * @param {Object} options - Filter options
 * @param {boolean} options.keepZero - Keep 0 as valid (default: false)
 * @param {boolean} options.keepEmptyString - Keep '' as valid (default: false)
 * @param {boolean} options.keepFalse - Keep false as valid (default: false)
 * @param {boolean} options.inPlace - Modify array in place (default: false)
 * @returns {Array} - Filtered array
 */
function removeFalsy(arr, options = {}) {
  const {
    keepZero = false,
    keepEmptyString = false,
    keepFalse = false,
    inPlace = false
  } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('Input must be an array');
  }
  
  // Filter function
  const shouldKeep = (item) => {
    // null and undefined are always removed
    if (item === null || item === undefined) {
      return false;
    }
    
    // NaN is always removed
    if (typeof item === 'number' && isNaN(item)) {
      return false;
    }
    
    // Handle special cases
    if (item === 0 && keepZero) return true;
    if (item === '' && keepEmptyString) return true;
    if (item === false && keepFalse) return true;
    
    // General truthy check
    return !!item;
  };
  
  if (inPlace) {
    let writeIndex = 0;
    
    for (let i = 0; i < arr.length; i++) {
      if (shouldKeep(arr[i])) {
        arr[writeIndex] = arr[i];
        writeIndex++;
      }
    }
    
    arr.length = writeIndex;
    return arr;
  } else {
    return arr.filter(shouldKeep);
  }
}

// Remove specific falsy values
function removeValues(arr, values) {
  return arr.filter(item => !values.includes(item));
}

// Keep only specific types
function keepTypes(arr, types) {
  return arr.filter(item => {
    const itemType = Array.isArray(item) ? 'array' : typeof item;
    return types.includes(itemType);
  });
}

// Comprehensive test suite
const testArray = [
  0, 1, false, 2, '', 3, null, 'hello', undefined, NaN, true, [], {}
];

console.log('=== Default (Remove All Falsy) ===');
console.log(removeFalsy(testArray));
// [1, 2, 3, 'hello', true, [], {}]

console.log('\n=== Keep Zero ===');
console.log(removeFalsy(testArray, { keepZero: true }));
// [0, 1, 2, 3, 'hello', true, [], {}]

console.log('\n=== Keep Empty String ===');
console.log(removeFalsy(testArray, { keepEmptyString: true }));
// [1, 2, '', 3, 'hello', true, [], {}]

console.log('\n=== Keep False ===');
console.log(removeFalsy(testArray, { keepFalse: true }));
// [1, false, 2, 3, 'hello', true, [], {}]

console.log('\n=== Keep All Special Falsy ===');
console.log(removeFalsy(testArray, {
  keepZero: true,
  keepEmptyString: true,
  keepFalse: true
}));
// [0, 1, false, 2, '', 3, 'hello', true, [], {}]
// Only null, undefined, NaN removed

console.log('\n=== Remove Specific Values ===');
console.log(removeValues([1, 2, null, 3, null, 4], [null]));
// [1, 2, 3, 4]

console.log(removeValues([1, 2, 0, 3, 0, 4], [0]));
// [1, 2, 3, 4]

console.log('\n=== Keep Only Numbers ===');
console.log(keepTypes(testArray, ['number']));
// [0, 1, 2, 3] (NaN is excluded)

console.log('\n=== Keep Only Strings and Numbers ===');
console.log(keepTypes(testArray, ['string', 'number']));
// [0, 1, 2, '', 3, 'hello']

console.log('\n=== In-Place Modification ===');
const mutableArray = [0, 1, false, 2, null, 3];
removeFalsy(mutableArray, { inPlace: true });
console.log(mutableArray);  // [1, 2, 3]
```

### **Bonus: Remove Empty Values (More Strict)**
```javascript
/**
 * Remove empty/null/undefined values (stricter than falsy)
 */
function removeEmpty(arr) {
  return arr.filter(item => {
    // Remove null and undefined
    if (item === null || item === undefined) return false;
    
    // Remove NaN
    if (typeof item === 'number' && isNaN(item)) return false;
    
    // Remove empty strings
    if (typeof item === 'string' && item.trim() === '') return false;
    
    // Remove empty arrays
    if (Array.isArray(item) && item.length === 0) return false;
    
    // Remove empty objects
    if (
      typeof item === 'object' &&
      !Array.isArray(item) &&
      Object.keys(item).length === 0
    ) return false;
    
    return true;
  });
}

// Test
const data = [
  0,           // kept (valid number)
  '',          // removed (empty string)
  '  ',        // removed (whitespace only)
  false,       // kept (valid boolean)
  null,        // removed
  undefined,   // removed
  [],          // removed (empty array)
  [1],         // kept
  {},          // removed (empty object)
  { a: 1 },    // kept
  NaN          // removed
];

console.log(removeEmpty(data));
// [0, false, [1], { a: 1 }]
```

**Interview Tips:**
- filter(Boolean) is the simplest and most common
- Falsy values in JavaScript: false, 0, '', null, undefined, NaN
- filter() creates new array; original unchanged unless inPlace option
- Consider if 0 should be kept (e.g., valid count/index)
- Consider if empty string should be kept (e.g., optional field)
- For more control, use explicit filter conditions
- Array.prototype.filter() is O(n) time, O(n) space

</details>

28. Find the intersection of two arrays

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using filter() and includes()**
```javascript
/**
 * Find intersection of two arrays
 * Time Complexity: O(n * m) - includes is O(m) for each element
 * Space Complexity: O(k) where k is intersection size
 * 
 * @param {Array} arr1 - First array
 * @param {Array} arr2 - Second array
 * @returns {Array} - Common elements
 */
function intersection(arr1, arr2) {
  return arr1.filter(item => arr2.includes(item));
}

// Test cases
console.log(intersection([1, 2, 3, 4], [3, 4, 5, 6]));
// [3, 4]

console.log(intersection(['a', 'b', 'c'], ['b', 'c', 'd']));
// ['b', 'c']

console.log(intersection([1, 2, 3], [4, 5, 6]));
// []
```

### **Approach 2: Using Set (Optimal)**
```javascript
/**
 * Find intersection using Set
 * Time Complexity: O(n + m)
 * Space Complexity: O(min(n, m))
 * 
 * Much faster for large arrays
 */
function intersection(arr1, arr2) {
  const set2 = new Set(arr2);
  return arr1.filter(item => set2.has(item));
}

// Or create Set from smaller array for optimization
function intersection(arr1, arr2) {
  // Use smaller array for Set to save space
  if (arr2.length < arr1.length) {
    [arr1, arr2] = [arr2, arr1];
  }
  
  const set = new Set(arr2);
  return arr1.filter(item => set.has(item));
}

// Test
console.log(intersection([1, 2, 3, 4, 5], [3, 4, 5, 6, 7]));
// [3, 4, 5]
```

### **Approach 3: Remove Duplicates from Result**
```javascript
/**
 * Intersection with unique values only
 * Time Complexity: O(n + m)
 * Space Complexity: O(k)
 */
function intersection(arr1, arr2) {
  const set1 = new Set(arr1);
  const set2 = new Set(arr2);
  
  return [...set1].filter(item => set2.has(item));
}

// Test with duplicates
console.log(intersection([1, 1, 2, 2, 3], [2, 2, 3, 3, 4]));
// [2, 3] (no duplicates in result)
```

### **Approach 4: Using reduce()**
```javascript
/**
 * Intersection using reduce
 * Time Complexity: O(n + m)
 * Space Complexity: O(m + k)
 */
function intersection(arr1, arr2) {
  const set2 = new Set(arr2);
  
  return arr1.reduce((result, item) => {
    if (set2.has(item) && !result.includes(item)) {
      result.push(item);
    }
    return result;
  }, []);
}

// Test
console.log(intersection([1, 2, 3, 4], [2, 3, 5, 6]));
// [2, 3]
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready array intersection
 * 
 * @param {Array} arr1 - First array
 * @param {Array} arr2 - Second array
 * @param {Object} options - Intersection options
 * @param {boolean} options.unique - Return unique values only (default: true)
 * @param {Function} options.comparator - Custom comparison function
 * @returns {Array} - Intersection result
 */
function intersection(arr1, arr2, options = {}) {
  const { unique = true, comparator } = options;
  
  // Validation
  if (!Array.isArray(arr1) || !Array.isArray(arr2)) {
    throw new TypeError('Both arguments must be arrays');
  }
  
  if (arr1.length === 0 || arr2.length === 0) {
    return [];
  }
  
  // Use custom comparator if provided
  if (comparator) {
    return arr1.filter(item1 =>
      arr2.some(item2 => comparator(item1, item2))
    );
  }
  
  // Standard intersection
  const set2 = new Set(arr2);
  const result = arr1.filter(item => set2.has(item));
  
  // Return unique values if requested
  return unique ? [...new Set(result)] : result;
}

// Intersection of multiple arrays
function intersectionMultiple(...arrays) {
  if (arrays.length === 0) return [];
  if (arrays.length === 1) return arrays[0];
  
  return arrays.reduce((acc, arr) => intersection(acc, arr));
}

// Intersection of arrays of objects
function intersectionBy(arr1, arr2, key) {
  const map = new Map(arr2.map(item => [item[key], item]));
  
  return arr1.filter(item => map.has(item[key]));
}

// Comprehensive test suite
console.log('=== Basic Intersection ===');
console.log(intersection([1, 2, 3, 4, 5], [3, 4, 5, 6, 7]));
// [3, 4, 5]

console.log('\n=== With Duplicates (Unique: true) ===');
console.log(intersection([1, 1, 2, 2, 3], [2, 2, 3, 3, 4], { unique: true }));
// [2, 3]

console.log('\n=== With Duplicates (Unique: false) ===');
console.log(intersection([1, 1, 2, 2, 3], [2, 2, 3, 3, 4], { unique: false }));
// [2, 2, 3] (duplicates preserved from arr1)

console.log('\n=== With Custom Comparator ===');
const arr1 = [{ id: 1, name: 'A' }, { id: 2, name: 'B' }];
const arr2 = [{ id: 2, name: 'B' }, { id: 3, name: 'C' }];

const result = intersection(arr1, arr2, {
  comparator: (a, b) => a.id === b.id
});
console.log(result);  // [{ id: 2, name: 'B' }]

console.log('\n=== Multiple Arrays ===');
console.log(intersectionMultiple(
  [1, 2, 3, 4, 5],
  [2, 3, 4, 5, 6],
  [3, 4, 5, 6, 7]
));  // [3, 4, 5]

console.log('\n=== Arrays of Objects (By Key) ===');
const users1 = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 3, name: 'Bob' }
];

const users2 = [
  { id: 2, name: 'Jane' },
  { id: 3, name: 'Bob' },
  { id: 4, name: 'Alice' }
];

console.log(intersectionBy(users1, users2, 'id'));
// [{ id: 2, name: 'Jane' }, { id: 3, name: 'Bob' }]

console.log('\n=== Empty Arrays ===');
console.log(intersection([], [1, 2, 3]));        // []
console.log(intersection([1, 2, 3], []));        // []
console.log(intersection([], []));               // []

console.log('\n=== No Common Elements ===');
console.log(intersection([1, 2, 3], [4, 5, 6])); // []
```

### **Bonus: Symmetric Difference (XOR)**
```javascript
/**
 * Get elements that are in either array but not in both
 */
function symmetricDifference(arr1, arr2) {
  const set1 = new Set(arr1);
  const set2 = new Set(arr2);
  
  const onlyInArr1 = arr1.filter(item => !set2.has(item));
  const onlyInArr2 = arr2.filter(item => !set1.has(item));
  
  return [...new Set([...onlyInArr1, ...onlyInArr2])];
}

// Or using filter on both arrays
function symmetricDifference(arr1, arr2) {
  return [
    ...arr1.filter(item => !arr2.includes(item)),
    ...arr2.filter(item => !arr1.includes(item))
  ];
}

// Test
console.log(symmetricDifference([1, 2, 3, 4], [3, 4, 5, 6]));
// [1, 2, 5, 6] (elements unique to each array)
```

**Interview Tips:**
- Set.has() is O(1) vs array.includes() is O(n)
- For large arrays, always use Set for better performance
- Clarify: should result contain duplicates?
- Clarify: should maintain order from first array?
- Consider lodash.intersection() for production
- Can extend to multiple arrays by reducing pairwise
- For objects, need custom comparator or key-based comparison

</details>

29. Check if an array contains a specific value

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using includes() (Modern)**
```javascript
/**
 * Check if array contains value
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Best for primitives
 * 
 * @param {Array} arr - Array to search
 * @param {*} value - Value to find
 * @returns {boolean} - True if found
 */
function contains(arr, value) {
  return arr.includes(value);
}

// Test cases
console.log(contains([1, 2, 3, 4, 5], 3));        // true
console.log(contains([1, 2, 3, 4, 5], 6));        // false
console.log(contains(['a', 'b', 'c'], 'b'));      // true
console.log(contains([1, 2, NaN, 4], NaN));       // true (includes handles NaN)
```

### **Approach 2: Using indexOf()**
```javascript
/**
 * Check using indexOf
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Returns -1 if not found
 */
function contains(arr, value) {
  return arr.indexOf(value) !== -1;
}

// Or using findIndex for objects
function contains(arr, predicate) {
  return arr.findIndex(predicate) !== -1;
}

// Test
console.log(contains([10, 20, 30], 20));          // true
console.log(contains([10, 20, 30], 25));          // false
console.log(contains([1, 2, NaN, 4], NaN));       // false (indexOf can't find NaN)
```

### **Approach 3: Using some()**
```javascript
/**
 * Check using some() with predicate
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * More flexible for complex conditions
 */
function contains(arr, value) {
  return arr.some(item => item === value);
}

// Or with custom predicate
function containsWhere(arr, predicate) {
  return arr.some(predicate);
}

// Test
console.log(contains([1, 2, 3], 2));              // true

const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' }
];

console.log(containsWhere(users, user => user.id === 2));     // true
console.log(containsWhere(users, user => user.name === 'Bob')); // false
```

### **Approach 4: Using Set**
```javascript
/**
 * Check using Set (efficient for multiple lookups)
 * Time Complexity: O(1) per lookup after O(n) construction
 * Space Complexity: O(n)
 */
function createContainsChecker(arr) {
  const set = new Set(arr);
  return (value) => set.has(value);
}

// Test
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const containsNumber = createContainsChecker(numbers);

console.log(containsNumber(5));    // true
console.log(containsNumber(15));   // false
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready contains check
 * 
 * @param {Array} arr - Array to search
 * @param {*} value - Value to find
 * @param {Object} options - Search options
 * @param {Function} options.comparator - Custom comparison function
 * @param {number} options.fromIndex - Start search from index
 * @param {boolean} options.strict - Use strict equality (default: true)
 * @returns {boolean} - True if found
 */
function contains(arr, value, options = {}) {
  const { comparator, fromIndex = 0, strict = true } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('First argument must be an array');
  }
  
  // Handle custom comparator
  if (comparator) {
    return arr.some(comparator);
  }
  
  // Handle fromIndex
  if (fromIndex !== 0) {
    return arr.slice(fromIndex).includes(value);
  }
  
  // Standard check
  if (strict) {
    return arr.includes(value);
  } else {
    // Loose equality
    return arr.some(item => item == value);
  }
}

// Check if array contains any of the values
function containsAny(arr, values) {
  return values.some(value => arr.includes(value));
}

// Check if array contains all of the values
function containsAll(arr, values) {
  return values.every(value => arr.includes(value));
}

// Check if array contains object with property
function containsObjectWith(arr, key, value) {
  return arr.some(item => item && item[key] === value);
}

// Comprehensive test suite
console.log('=== Basic Contains ===');
const numbers = [1, 2, 3, 4, 5];
console.log(contains(numbers, 3));               // true
console.log(contains(numbers, 6));               // false

console.log('\n=== With Custom Comparator ===');
const users = [
  { id: 1, name: 'John', age: 30 },
  { id: 2, name: 'Jane', age: 25 }
];

console.log(contains(users, null, {
  comparator: user => user.name === 'Jane'
}));  // true

console.log('\n=== From Index ===');
console.log(contains([1, 2, 3, 4, 5], 2, { fromIndex: 0 }));  // true
console.log(contains([1, 2, 3, 4, 5], 2, { fromIndex: 2 }));  // false

console.log('\n=== Strict vs Loose ===');
console.log(contains([1, 2, 3], '2', { strict: true }));   // false
console.log(contains([1, 2, 3], '2', { strict: false }));  // true

console.log('\n=== Contains Any ===');
console.log(containsAny([1, 2, 3, 4], [5, 6, 7]));        // false
console.log(containsAny([1, 2, 3, 4], [3, 5, 7]));        // true

console.log('\n=== Contains All ===');
console.log(containsAll([1, 2, 3, 4, 5], [2, 4]));        // true
console.log(containsAll([1, 2, 3, 4, 5], [2, 6]));        // false

console.log('\n=== Contains Object With Property ===');
const products = [
  { id: 1, name: 'Laptop', price: 1000 },
  { id: 2, name: 'Phone', price: 500 }
];

console.log(containsObjectWith(products, 'name', 'Laptop'));  // true
console.log(containsObjectWith(products, 'name', 'Tablet'));  // false
console.log(containsObjectWith(products, 'price', 500));      // true

console.log('\n=== Special Values ===');
console.log(contains([1, 2, NaN, 4], NaN));           // true (includes handles NaN)
console.log(contains([1, 2, undefined, 4], undefined)); // true
console.log(contains([1, 2, null, 4], null));         // true
```

### **Bonus: Deep Contains for Objects**
```javascript
/**
 * Check if array contains object with deep equality
 */
function deepContains(arr, obj) {
  return arr.some(item => JSON.stringify(item) === JSON.stringify(obj));
}

// Or using lodash-style deep equal
function deepEqual(a, b) {
  if (a === b) return true;
  
  if (typeof a !== 'object' || typeof b !== 'object') return false;
  if (a === null || b === null) return false;
  
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  
  if (keysA.length !== keysB.length) return false;
  
  return keysA.every(key => deepEqual(a[key], b[key]));
}

function deepContains(arr, obj) {
  return arr.some(item => deepEqual(item, obj));
}

// Test
const objects = [
  { a: 1, b: { c: 2 } },
  { a: 2, b: { c: 3 } }
];

console.log(deepContains(objects, { a: 1, b: { c: 2 } }));  // true
console.log(deepContains(objects, { a: 1, b: { c: 3 } }));  // false
```

**Interview Tips:**
- includes() is modern and recommended (ES2016+)
- indexOf() is older but widely supported
- includes() handles NaN correctly, indexOf() doesn't
- some() is more flexible for complex conditions
- For objects, need custom comparison (can't use === by reference)
- Set.has() is O(1) but requires O(n) setup - good for multiple lookups
- Consider using find() if you need the actual element
- includes() supports fromIndex parameter: arr.includes(value, fromIndex)

</details>

30. Capitalize the first letter of each word in a string

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using split(), map(), and join()**
```javascript
/**
 * Capitalize first letter of each word
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {string} str - Input string
 * @returns {string} - Title cased string
 */
function capitalizeWords(str) {
  return str
    .split(' ')
    .map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
    .join(' ');
}

// Test cases
console.log(capitalizeWords('hello world'));
// 'Hello World'

console.log(capitalizeWords('javascript is awesome'));
// 'Javascript Is Awesome'

console.log(capitalizeWords('the quick brown fox'));
// 'The Quick Brown Fox'
```

### **Approach 2: Using Regular Expression**
```javascript
/**
 * Capitalize using regex
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * More concise
 */
function capitalizeWords(str) {
  return str.replace(/\b\w/g, char => char.toUpperCase());
}

// Or lowercase the rest
function capitalizeWords(str) {
  return str.toLowerCase().replace(/\b\w/g, char => char.toUpperCase());
}

// Test
console.log(capitalizeWords('hello world'));              // 'Hello World'
console.log(capitalizeWords('HELLO WORLD'));              // 'Hello World'
console.log(capitalizeWords('this is a test'));           // 'This Is A Test'
```

### **Approach 3: Using Loop (Manual)**
```javascript
/**
 * Capitalize using manual loop
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function capitalizeWords(str) {
  let result = '';
  let capitalizeNext = true;
  
  for (let i = 0; i < str.length; i++) {
    const char = str[i];
    
    if (char === ' ') {
      result += char;
      capitalizeNext = true;
    } else {
      if (capitalizeNext) {
        result += char.toUpperCase();
        capitalizeNext = false;
      } else {
        result += char.toLowerCase();
      }
    }
  }
  
  return result;
}

// Test
console.log(capitalizeWords('hello world'));  // 'Hello World'
console.log(capitalizeWords('a b c d'));      // 'A B C D'
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready title case converter
 * 
 * @param {string} str - Input string
 * @param {Object} options - Capitalization options
 * @param {Array} options.exceptions - Words to keep lowercase
 * @param {boolean} options.allWords - Capitalize all words including exceptions (default: false)
 * @param {boolean} options.firstWordAlways - Always capitalize first word (default: true)
 * @returns {string} - Title cased string
 */
function capitalizeWords(str, options = {}) {
  const {
    exceptions = ['a', 'an', 'the', 'and', 'but', 'or', 'for', 'nor', 'on', 'at', 'to', 'by', 'in', 'of'],
    allWords = false,
    firstWordAlways = true
  } = options;
  
  // Validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  if (str.trim() === '') {
    return str;
  }
  
  const words = str.trim().split(/\s+/);
  
  return words
    .map((word, index) => {
      const lowerWord = word.toLowerCase();
      
      // Always capitalize first word if option is set
      if (index === 0 && firstWordAlways) {
        return word.charAt(0).toUpperCase() + word.slice(1).toLowerCase();
      }
      
      // Check if word is in exceptions list
      if (!allWords && exceptions.includes(lowerWord)) {
        return lowerWord;
      }
      
      // Capitalize first letter
      return word.charAt(0).toUpperCase() + word.slice(1).toLowerCase();
    })
    .join(' ');
}

// Capitalize only first letter of entire string
function capitalizeFirst(str) {
  if (!str) return str;
  return str.charAt(0).toUpperCase() + str.slice(1);
}

// Capitalize using proper title case rules
function toTitleCase(str) {
  // Articles, conjunctions, and prepositions to keep lowercase (unless first/last)
  const minorWords = new Set([
    'a', 'an', 'the', 'and', 'but', 'or', 'for', 'nor', 'as', 'at',
    'by', 'for', 'from', 'in', 'into', 'near', 'of', 'on', 'onto',
    'to', 'with'
  ]);
  
  const words = str.toLowerCase().split(/\s+/);
  
  return words
    .map((word, index) => {
      // Always capitalize first and last word
      if (index === 0 || index === words.length - 1) {
        return word.charAt(0).toUpperCase() + word.slice(1);
      }
      
      // Keep minor words lowercase
      if (minorWords.has(word)) {
        return word;
      }
      
      // Capitalize everything else
      return word.charAt(0).toUpperCase() + word.slice(1);
    })
    .join(' ');
}

// Comprehensive test suite
console.log('=== Basic Capitalization ===');
console.log(capitalizeWords('hello world'));
// 'Hello World'

console.log(capitalizeWords('the quick brown fox jumps over the lazy dog'));
// 'The Quick Brown Fox Jumps Over The Lazy Dog'

console.log('\n=== With Exceptions (Title Case) ===');
console.log(capitalizeWords('the lord of the rings', { allWords: false }));
// 'The Lord of the Rings'

console.log(capitalizeWords('a tale of two cities', { allWords: false }));
// 'A Tale of Two Cities'

console.log('\n=== All Words (No Exceptions) ===');
console.log(capitalizeWords('the lord of the rings', { allWords: true }));
// 'The Lord Of The Rings'

console.log('\n=== Proper Title Case ===');
console.log(toTitleCase('the lord of the rings'));
// 'The Lord of the Rings'

console.log(toTitleCase('gone with the wind'));
// 'Gone with the Wind'

console.log(toTitleCase('harry potter and the chamber of secrets'));
// 'Harry Potter and the Chamber of Secrets'

console.log('\n=== Capitalize First Only ===');
console.log(capitalizeFirst('hello world'));
// 'Hello world'

console.log(capitalizeFirst('the quick brown fox'));
// 'The quick brown fox'

console.log('\n=== Edge Cases ===');
console.log(capitalizeWords(''));                    // ''
console.log(capitalizeWords('a'));                   // 'A'
console.log(capitalizeWords('   hello   world   '));  // 'Hello World'
console.log(capitalizeWords('HELLO WORLD'));         // 'Hello World'
console.log(capitalizeWords('hello-world'));         // 'Hello-world' (hyphen not handled)
```

### **Bonus: Advanced Capitalization**
```javascript
/**
 * Capitalize with punctuation handling
 */
function capitalizeWithPunctuation(str) {
  return str.replace(/(?:^|\s|[.!?])\w/g, match => match.toUpperCase());
}

/**
 * Capitalize hyphenated words
 */
function capitalizeHyphenated(str) {
  return str
    .split(/(\s+|-)/)
    .map(part => {
      if (part === ' ' || part === '-') return part;
      return part.charAt(0).toUpperCase() + part.slice(1).toLowerCase();
    })
    .join('');
}

/**
 * Convert to camelCase
 */
function toCamelCase(str) {
  return str
    .toLowerCase()
    .replace(/[^a-zA-Z0-9]+(.)/g, (match, char) => char.toUpperCase());
}

/**
 * Convert to PascalCase
 */
function toPascalCase(str) {
  const camel = toCamelCase(str);
  return camel.charAt(0).toUpperCase() + camel.slice(1);
}

// Test advanced
console.log(capitalizeWithPunctuation('hello. how are you? i am fine!'));
// 'Hello. How Are You? I Am Fine!'

console.log(capitalizeHyphenated('hello-world-example'));
// 'Hello-World-Example'

console.log(toCamelCase('hello world test'));
// 'helloWorldTest'

console.log(toPascalCase('hello world test'));
// 'HelloWorldTest'
```

**Interview Tips:**
- split/map/join is most common and readable
- Regex /\b\w/g matches word boundaries and first character
- \b is word boundary, \w is word character (alphanumeric + underscore)
- Title case rules: lowercase articles, conjunctions, prepositions
- Always capitalize first and last word in title case
- Consider handling hyphens, apostrophes, punctuation
- CSS text-transform: capitalize does similar in styling
- Different style guides (APA, Chicago, MLA) have different rules

</details>

---

## **MEDIUM LEVEL (31-65)**

### **Array Manipulations**

31. Implement array.map() from scratch

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Implementation**
```javascript
/**
 * Implement Array.prototype.map()
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {Function} callback - Function to execute on each element
 * @param {*} thisArg - Value to use as 'this' when executing callback
 * @returns {Array} - New array with results
 */
Array.prototype.myMap = function(callback, thisArg) {
  const result = [];
  
  for (let i = 0; i < this.length; i++) {
    // Check if index exists (handle sparse arrays)
    if (i in this) {
      result[i] = callback.call(thisArg, this[i], i, this);
    }
  }
  
  return result;
};

// Test cases
const numbers = [1, 2, 3, 4, 5];
console.log(numbers.myMap(x => x * 2));
// [2, 4, 6, 8, 10]

console.log(numbers.myMap((x, index) => x + index));
// [1, 3, 5, 7, 9]

const words = ['hello', 'world'];
console.log(words.myMap(word => word.toUpperCase()));
// ['HELLO', 'WORLD']
```

### **Approach 2: With Validation**
```javascript
/**
 * Enhanced map with validation
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
Array.prototype.myMap = function(callback, thisArg) {
  // Validation
  if (this == null) {
    throw new TypeError('Array.prototype.myMap called on null or undefined');
  }
  
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  const O = Object(this);
  const len = O.length >>> 0; // Convert to unsigned 32-bit integer
  const result = new Array(len);
  
  for (let i = 0; i < len; i++) {
    if (i in O) {
      result[i] = callback.call(thisArg, O[i], i, O);
    }
  }
  
  return result;
};

// Test
const arr = [1, 2, 3];
console.log(arr.myMap(x => x * 3));  // [3, 6, 9]
```

### **Approach 3: Handle Sparse Arrays**
```javascript
/**
 * Map that properly handles sparse arrays
 * Sparse arrays have "holes" - indices without values
 */
Array.prototype.myMap = function(callback, thisArg) {
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  const result = [];
  
  for (let i = 0; i < this.length; i++) {
    // Only process indices that exist
    if (i in this) {
      result[i] = callback.call(thisArg, this[i], i, this);
    }
  }
  
  // Preserve array length (including holes)
  result.length = this.length;
  
  return result;
};

// Test with sparse array
const sparse = [1, , 3, , 5]; // Has holes at index 1 and 3
console.log(sparse.myMap(x => x * 2));
// [2, empty, 6, empty, 10]
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready map implementation following ES specification
 * 
 * @param {Function} callback - Function that produces element of new array
 * @param {*} thisArg - Value to use as this when executing callback
 * @returns {Array} - New array with each element being the result of callback
 */
Array.prototype.myMap = function(callback, thisArg) {
  // 1. Let O be ? ToObject(this value)
  if (this == null) {
    throw new TypeError('Array.prototype.myMap called on null or undefined');
  }
  
  const O = Object(this);
  
  // 2. Let len be ? LengthOfArrayLike(O)
  const len = O.length >>> 0;
  
  // 3. If IsCallable(callback) is false, throw a TypeError
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  // 4. Let A be ? ArraySpeciesCreate(O, len)
  const A = new Array(len);
  
  // 5. Let k be 0
  let k = 0;
  
  // 6. Repeat, while k < len
  while (k < len) {
    // a. Let Pk be ! ToString(k)
    const Pk = String(k);
    
    // b. Let kPresent be ? HasProperty(O, Pk)
    const kPresent = Pk in O;
    
    // c. If kPresent is true, then
    if (kPresent) {
      // i. Let kValue be ? Get(O, Pk)
      const kValue = O[Pk];
      
      // ii. Let mappedValue be ? Call(callback, thisArg, « kValue, k, O »)
      const mappedValue = callback.call(thisArg, kValue, k, O);
      
      // iii. Perform ? CreateDataPropertyOrThrow(A, Pk, mappedValue)
      A[Pk] = mappedValue;
    }
    
    // d. Set k to k + 1
    k++;
  }
  
  // 7. Return A
  return A;
};

// Alternative: Standalone function (not modifying prototype)
function map(array, callback, thisArg) {
  if (!Array.isArray(array)) {
    throw new TypeError('First argument must be an array');
  }
  
  if (typeof callback !== 'function') {
    throw new TypeError('Second argument must be a function');
  }
  
  const result = new Array(array.length);
  
  for (let i = 0; i < array.length; i++) {
    if (i in array) {
      result[i] = callback.call(thisArg, array[i], i, array);
    }
  }
  
  return result;
}

// Comprehensive test suite
console.log('=== Basic Mapping ===');
const nums = [1, 2, 3, 4, 5];
console.log(nums.myMap(x => x * 2));
// [2, 4, 6, 8, 10]

console.log('\n=== With Index ===');
console.log(nums.myMap((x, i) => x + i));
// [1, 3, 5, 7, 9]

console.log('\n=== With Array Context ===');
console.log(nums.myMap((x, i, arr) => x + arr.length));
// [6, 7, 8, 9, 10]

console.log('\n=== With thisArg ===');
const multiplier = { factor: 10 };
console.log(nums.myMap(function(x) {
  return x * this.factor;
}, multiplier));
// [10, 20, 30, 40, 50]

console.log('\n=== Sparse Arrays ===');
const sparse = [1, , 3, , 5];
console.log(sparse.myMap(x => x * 2));
// [2, empty, 6, empty, 10]

console.log('\n=== Empty Array ===');
console.log([].myMap(x => x * 2));
// []

console.log('\n=== Type Conversion ===');
const strings = ['1', '2', '3'];
console.log(strings.myMap(Number));
// [1, 2, 3]

console.log('\n=== Object Properties ===');
const users = [
  { name: 'John', age: 30 },
  { name: 'Jane', age: 25 }
];
console.log(users.myMap(user => user.name));
// ['John', 'Jane']

console.log('\n=== Standalone Function ===');
console.log(map([1, 2, 3], x => x * 3));
// [3, 6, 9]

// Error cases
console.log('\n=== Error Handling ===');
try {
  [1, 2, 3].myMap('not a function');
} catch (e) {
  console.log(e.message);  // "not a function is not a function"
}

try {
  Array.prototype.myMap.call(null, x => x);
} catch (e) {
  console.log(e.message);  // "Array.prototype.myMap called on null or undefined"
}
```

### **Bonus: Performance Comparison**
```javascript
/**
 * Compare native map vs custom implementation
 */
function benchmarkMap() {
  const largeArray = Array.from({ length: 100000 }, (_, i) => i);
  
  console.time('Native map');
  const native = largeArray.map(x => x * 2);
  console.timeEnd('Native map');
  
  console.time('Custom myMap');
  const custom = largeArray.myMap(x => x * 2);
  console.timeEnd('Custom myMap');
  
  console.log('Results match:', JSON.stringify(native) === JSON.stringify(custom));
}

benchmarkMap();
```

**Interview Tips:**
- map() creates a new array; doesn't modify original
- Callback receives three arguments: element, index, array
- Always returns array of same length as original
- Handles sparse arrays (skips holes)
- thisArg parameter sets 'this' context in callback
- Return value: new array with transformed elements
- Doesn't execute callback for missing indices in sparse arrays
- Common use: transform each element, extract properties

</details>

32. Implement array.filter() from scratch

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Implementation**
```javascript
/**
 * Implement Array.prototype.filter()
 * Time Complexity: O(n)
 * Space Complexity: O(n) worst case
 * 
 * @param {Function} callback - Function to test each element
 * @param {*} thisArg - Value to use as 'this' when executing callback
 * @returns {Array} - New array with elements that pass the test
 */
Array.prototype.myFilter = function(callback, thisArg) {
  const result = [];
  
  for (let i = 0; i < this.length; i++) {
    if (i in this) {
      if (callback.call(thisArg, this[i], i, this)) {
        result.push(this[i]);
      }
    }
  }
  
  return result;
};

// Test cases
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
console.log(numbers.myFilter(x => x % 2 === 0));
// [2, 4, 6, 8, 10]

console.log(numbers.myFilter(x => x > 5));
// [6, 7, 8, 9, 10]

const words = ['hello', 'hi', 'hey', 'greetings'];
console.log(words.myFilter(word => word.length > 3));
// ['hello', 'greetings']
```

### **Approach 2: With Validation**
```javascript
/**
 * Enhanced filter with validation
 * Time Complexity: O(n)
 * Space Complexity: O(k) where k is number of passing elements
 */
Array.prototype.myFilter = function(callback, thisArg) {
  // Validation
  if (this == null) {
    throw new TypeError('Array.prototype.myFilter called on null or undefined');
  }
  
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  const O = Object(this);
  const len = O.length >>> 0;
  const result = [];
  
  for (let i = 0; i < len; i++) {
    if (i in O) {
      const value = O[i];
      if (callback.call(thisArg, value, i, O)) {
        result.push(value);
      }
    }
  }
  
  return result;
};

// Test
const arr = [1, 2, 3, 4, 5];
console.log(arr.myFilter(x => x >= 3));  // [3, 4, 5]
```

### **Approach 3: Handle Edge Cases**
```javascript
/**
 * Filter with comprehensive edge case handling
 */
Array.prototype.myFilter = function(callback, thisArg) {
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  const result = [];
  
  for (let i = 0; i < this.length; i++) {
    // Skip holes in sparse arrays
    if (!(i in this)) continue;
    
    const element = this[i];
    
    // Only add if callback returns truthy value
    if (callback.call(thisArg, element, i, this)) {
      result.push(element);
    }
  }
  
  return result;
};

// Test with sparse array
const sparse = [1, , 3, , 5, , 7];
console.log(sparse.myFilter(x => x > 2));
// [3, 5, 7]
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready filter implementation following ES specification
 * 
 * @param {Function} callback - Predicate function to test each element
 * @param {*} thisArg - Value to use as this when executing callback
 * @returns {Array} - New array with elements that pass the test
 */
Array.prototype.myFilter = function(callback, thisArg) {
  // 1. Let O be ? ToObject(this value)
  if (this == null) {
    throw new TypeError('Array.prototype.myFilter called on null or undefined');
  }
  
  const O = Object(this);
  
  // 2. Let len be ? LengthOfArrayLike(O)
  const len = O.length >>> 0;
  
  // 3. If IsCallable(callback) is false, throw a TypeError
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  // 4. Let A be a new empty List
  const A = [];
  
  // 5. Let k be 0
  let k = 0;
  
  // 6. Repeat, while k < len
  while (k < len) {
    // a. Let Pk be ! ToString(k)
    const Pk = String(k);
    
    // b. Let kPresent be ? HasProperty(O, Pk)
    const kPresent = Pk in O;
    
    // c. If kPresent is true, then
    if (kPresent) {
      // i. Let kValue be ? Get(O, Pk)
      const kValue = O[Pk];
      
      // ii. Let selected be ToBoolean(? Call(callback, thisArg, « kValue, k, O »))
      const selected = callback.call(thisArg, kValue, k, O);
      
      // iii. If selected is true, then
      if (selected) {
        // 1. Append kValue to A
        A.push(kValue);
      }
    }
    
    // d. Set k to k + 1
    k++;
  }
  
  // 7. Return A
  return A;
};

// Alternative: Standalone function (not modifying prototype)
function filter(array, callback, thisArg) {
  if (!Array.isArray(array)) {
    throw new TypeError('First argument must be an array');
  }
  
  if (typeof callback !== 'function') {
    throw new TypeError('Second argument must be a function');
  }
  
  const result = [];
  
  for (let i = 0; i < array.length; i++) {
    if (i in array) {
      const value = array[i];
      if (callback.call(thisArg, value, i, array)) {
        result.push(value);
      }
    }
  }
  
  return result;
}

// Comprehensive test suite
console.log('=== Basic Filtering ===');
const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
console.log(nums.myFilter(x => x % 2 === 0));
// [2, 4, 6, 8, 10]

console.log('\n=== With Index ===');
console.log(nums.myFilter((x, i) => i % 2 === 0));
// [1, 3, 5, 7, 9] (elements at even indices)

console.log('\n=== With Array Context ===');
console.log(nums.myFilter((x, i, arr) => x > arr.length / 2));
// [6, 7, 8, 9, 10]

console.log('\n=== With thisArg ===');
const config = { threshold: 5 };
console.log(nums.myFilter(function(x) {
  return x > this.threshold;
}, config));
// [6, 7, 8, 9, 10]

console.log('\n=== Filter Truthy Values ===');
console.log([0, 1, false, 2, '', 3, null, 4].myFilter(Boolean));
// [1, 2, 3, 4]

console.log('\n=== Filter Objects ===');
const users = [
  { name: 'John', age: 30, active: true },
  { name: 'Jane', age: 25, active: false },
  { name: 'Bob', age: 35, active: true }
];
console.log(users.myFilter(user => user.active));
// [{ name: 'John', age: 30, active: true }, { name: 'Bob', age: 35, active: true }]

console.log('\n=== No Matches ===');
console.log([1, 2, 3].myFilter(x => x > 10));
// []

console.log('\n=== All Match ===');
console.log([1, 2, 3].myFilter(x => x > 0));
// [1, 2, 3]

console.log('\n=== Empty Array ===');
console.log([].myFilter(x => true));
// []

console.log('\n=== Sparse Arrays ===');
const sparse = [1, , 3, , 5];
console.log(sparse.myFilter(x => x >= 3));
// [3, 5] (holes are skipped)

console.log('\n=== Standalone Function ===');
console.log(filter([1, 2, 3, 4, 5], x => x % 2 !== 0));
// [1, 3, 5]

// Error cases
console.log('\n=== Error Handling ===');
try {
  [1, 2, 3].myFilter('not a function');
} catch (e) {
  console.log(e.message);  // "not a function is not a function"
}
```

### **Bonus: Filter Variants**
```javascript
/**
 * Find first element that passes test
 */
Array.prototype.myFind = function(callback, thisArg) {
  for (let i = 0; i < this.length; i++) {
    if (i in this) {
      if (callback.call(thisArg, this[i], i, this)) {
        return this[i];
      }
    }
  }
  return undefined;
};

/**
 * Find index of first element that passes test
 */
Array.prototype.myFindIndex = function(callback, thisArg) {
  for (let i = 0; i < this.length; i++) {
    if (i in this) {
      if (callback.call(thisArg, this[i], i, this)) {
        return i;
      }
    }
  }
  return -1;
};

// Test
const numbers = [1, 2, 3, 4, 5];
console.log(numbers.myFind(x => x > 3));       // 4
console.log(numbers.myFindIndex(x => x > 3));  // 3
```

**Interview Tips:**
- filter() creates a new array; doesn't modify original
- Only includes elements where callback returns truthy value
- Callback receives three arguments: element, index, array
- Result array can be shorter, same length, or empty
- Skips holes in sparse arrays
- Returns empty array if no elements pass
- Common use: remove unwanted elements, select subset
- filter(Boolean) removes all falsy values

</details>

33. Implement array.reduce() from scratch

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Implementation**
```javascript
/**
 * Implement Array.prototype.reduce()
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * @param {Function} callback - Reducer function
 * @param {*} initialValue - Initial accumulator value (optional)
 * @returns {*} - Final accumulated value
 */
Array.prototype.myReduce = function(callback, initialValue) {
  let accumulator;
  let startIndex = 0;
  
  // If no initial value, use first element
  if (arguments.length < 2) {
    if (this.length === 0) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
    accumulator = this[0];
    startIndex = 1;
  } else {
    accumulator = initialValue;
  }
  
  for (let i = startIndex; i < this.length; i++) {
    if (i in this) {
      accumulator = callback(accumulator, this[i], i, this);
    }
  }
  
  return accumulator;
};

// Test cases
const numbers = [1, 2, 3, 4, 5];
console.log(numbers.myReduce((sum, x) => sum + x, 0));
// 15

console.log(numbers.myReduce((product, x) => product * x, 1));
// 120

console.log(numbers.myReduce((max, x) => Math.max(max, x)));
// 5
```

### **Approach 2: With Validation**
```javascript
/**
 * Enhanced reduce with validation
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
Array.prototype.myReduce = function(callback, initialValue) {
  // Validation
  if (this == null) {
    throw new TypeError('Array.prototype.myReduce called on null or undefined');
  }
  
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  const O = Object(this);
  const len = O.length >>> 0;
  
  let k = 0;
  let accumulator;
  
  if (arguments.length >= 2) {
    accumulator = initialValue;
  } else {
    // Find first element in array
    let kPresent = false;
    while (!kPresent && k < len) {
      kPresent = k in O;
      if (kPresent) {
        accumulator = O[k];
      }
      k++;
    }
    
    if (!kPresent) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
  }
  
  while (k < len) {
    if (k in O) {
      accumulator = callback(accumulator, O[k], k, O);
    }
    k++;
  }
  
  return accumulator;
};

// Test
const arr = [1, 2, 3, 4];
console.log(arr.myReduce((sum, x) => sum + x));  // 10
```

### **Approach 3: Handle Sparse Arrays**
```javascript
/**
 * Reduce that properly handles sparse arrays
 */
Array.prototype.myReduce = function(callback, initialValue) {
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  const hasInitialValue = arguments.length >= 2;
  let accumulator = initialValue;
  let foundFirst = hasInitialValue;
  
  for (let i = 0; i < this.length; i++) {
    // Skip holes
    if (!(i in this)) continue;
    
    if (!foundFirst) {
      // First element becomes accumulator
      accumulator = this[i];
      foundFirst = true;
    } else {
      accumulator = callback(accumulator, this[i], i, this);
    }
  }
  
  if (!foundFirst) {
    throw new TypeError('Reduce of empty array with no initial value');
  }
  
  return accumulator;
};

// Test with sparse array
const sparse = [, 1, , 2, , 3];
console.log(sparse.myReduce((sum, x) => sum + x, 0));
// 6 (skips holes)
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready reduce implementation following ES specification
 * 
 * @param {Function} callback - Reducer function (accumulator, currentValue, index, array)
 * @param {*} initialValue - Initial value for accumulator
 * @returns {*} - Final accumulated result
 */
Array.prototype.myReduce = function(callback, initialValue) {
  // 1. Let O be ? ToObject(this value)
  if (this == null) {
    throw new TypeError('Array.prototype.myReduce called on null or undefined');
  }
  
  const O = Object(this);
  
  // 2. Let len be ? LengthOfArrayLike(O)
  const len = O.length >>> 0;
  
  // 3. If IsCallable(callback) is false, throw a TypeError
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  // 4. If len = 0 and initialValue is not present, throw a TypeError
  if (len === 0 && arguments.length < 2) {
    throw new TypeError('Reduce of empty array with no initial value');
  }
  
  // 5. Let k be 0
  let k = 0;
  let accumulator;
  
  // 6. If initialValue is present, then
  if (arguments.length >= 2) {
    // a. Set accumulator to initialValue
    accumulator = initialValue;
  } else {
    // 7. Else,
    // a. Let kPresent be false
    let kPresent = false;
    
    // b. Repeat, while kPresent is false and k < len
    while (!kPresent && k < len) {
      // i. Let Pk be ! ToString(k)
      const Pk = String(k);
      
      // ii. Set kPresent to ? HasProperty(O, Pk)
      kPresent = Pk in O;
      
      // iii. If kPresent is true, then
      if (kPresent) {
        // 1. Set accumulator to ? Get(O, Pk)
        accumulator = O[Pk];
      }
      
      // iv. Set k to k + 1
      k++;
    }
    
    // c. If kPresent is false, throw a TypeError
    if (!kPresent) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
  }
  
  // 8. Repeat, while k < len
  while (k < len) {
    // a. Let Pk be ! ToString(k)
    const Pk = String(k);
    
    // b. Let kPresent be ? HasProperty(O, Pk)
    const kPresent = Pk in O;
    
    // c. If kPresent is true, then
    if (kPresent) {
      // i. Let kValue be ? Get(O, Pk)
      const kValue = O[Pk];
      
      // ii. Set accumulator to ? Call(callback, undefined, « accumulator, kValue, k, O »)
      accumulator = callback(accumulator, kValue, k, O);
    }
    
    // d. Set k to k + 1
    k++;
  }
  
  // 9. Return accumulator
  return accumulator;
};

// Alternative: Standalone function
function reduce(array, callback, initialValue) {
  if (!Array.isArray(array)) {
    throw new TypeError('First argument must be an array');
  }
  
  if (typeof callback !== 'function') {
    throw new TypeError('Second argument must be a function');
  }
  
  const hasInitial = arguments.length >= 3;
  let accumulator = initialValue;
  let startIndex = 0;
  
  if (!hasInitial) {
    if (array.length === 0) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
    
    // Find first element
    while (startIndex < array.length && !(startIndex in array)) {
      startIndex++;
    }
    
    if (startIndex >= array.length) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
    
    accumulator = array[startIndex];
    startIndex++;
  }
  
  for (let i = startIndex; i < array.length; i++) {
    if (i in array) {
      accumulator = callback(accumulator, array[i], i, array);
    }
  }
  
  return accumulator;
}

// Comprehensive test suite
console.log('=== Sum ===');
const nums = [1, 2, 3, 4, 5];
console.log(nums.myReduce((sum, x) => sum + x, 0));
// 15

console.log('\n=== Product ===');
console.log(nums.myReduce((product, x) => product * x, 1));
// 120

console.log('\n=== Max Value ===');
console.log(nums.myReduce((max, x) => Math.max(max, x)));
// 5

console.log('\n=== Build Object ===');
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' }
];
const userMap = users.myReduce((acc, user) => {
  acc[user.id] = user.name;
  return acc;
}, {});
console.log(userMap);
// { '1': 'John', '2': 'Jane' }

console.log('\n=== Flatten Array ===');
const nested = [[1, 2], [3, 4], [5, 6]];
console.log(nested.myReduce((flat, arr) => flat.concat(arr), []));
// [1, 2, 3, 4, 5, 6]

console.log('\n=== Count Occurrences ===');
const items = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const counts = items.myReduce((acc, item) => {
  acc[item] = (acc[item] || 0) + 1;
  return acc;
}, {});
console.log(counts);
// { apple: 3, banana: 2, orange: 1 }

console.log('\n=== No Initial Value ===');
console.log([1, 2, 3, 4].myReduce((sum, x) => sum + x));
// 10 (starts with 1)

console.log('\n=== Single Element ===');
console.log([5].myReduce((sum, x) => sum + x, 10));
// 15

console.log([5].myReduce((sum, x) => sum + x));
// 5 (no reduction happens)

console.log('\n=== Empty Array with Initial ===');
console.log([].myReduce((sum, x) => sum + x, 0));
// 0

console.log('\n=== Sparse Array ===');
const sparse = [1, , 3, , 5];
console.log(sparse.myReduce((sum, x) => sum + x, 0));
// 9 (skips holes)

console.log('\n=== Standalone Function ===');
console.log(reduce([1, 2, 3, 4], (sum, x) => sum + x, 0));
// 10

// Error cases
console.log('\n=== Error Handling ===');
try {
  [].myReduce((sum, x) => sum + x);
} catch (e) {
  console.log(e.message);  // "Reduce of empty array with no initial value"
}

try {
  [1, 2, 3].myReduce('not a function');
} catch (e) {
  console.log(e.message);  // "not a function is not a function"
}
```

### **Bonus: reduceRight() Implementation**
```javascript
/**
 * Implement reduceRight (right-to-left reduction)
 */
Array.prototype.myReduceRight = function(callback, initialValue) {
  if (typeof callback !== 'function') {
    throw new TypeError(callback + ' is not a function');
  }
  
  const hasInitial = arguments.length >= 2;
  let accumulator = initialValue;
  let k = this.length - 1;
  
  if (!hasInitial) {
    if (this.length === 0) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
    
    // Find last element
    while (k >= 0 && !(k in this)) {
      k--;
    }
    
    if (k < 0) {
      throw new TypeError('Reduce of empty array with no initial value');
    }
    
    accumulator = this[k];
    k--;
  }
  
  while (k >= 0) {
    if (k in this) {
      accumulator = callback(accumulator, this[k], k, this);
    }
    k--;
  }
  
  return accumulator;
};

// Test
const arr = [1, 2, 3, 4];
console.log(arr.myReduceRight((result, x) => result + x, ''));
// '4321' (processes right to left)
```

**Interview Tips:**
- reduce() processes array left-to-right, accumulating a single value
- Callback receives four arguments: accumulator, currentValue, index, array
- If no initial value, first element is used as accumulator
- Empty array with no initial value throws TypeError
- Skips holes in sparse arrays
- Versatile: can build objects, arrays, count, sum, etc.
- reduceRight() processes right-to-left
- Common uses: sum, product, flatten, grouping, transforming

</details>

34. Find all pairs in an array that sum to a target value

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Brute Force (Nested Loops)**
```javascript
/**
 * Find all pairs that sum to target using brute force
 * Time Complexity: O(n²)
 * Space Complexity: O(k) where k is number of pairs
 * 
 * @param {number[]} arr - Input array
 * @param {number} target - Target sum
 * @returns {Array} - Array of pairs
 */
function findPairs(arr, target) {
  const pairs = [];
  
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] + arr[j] === target) {
        pairs.push([arr[i], arr[j]]);
      }
    }
  }
  
  return pairs;
}

// Test cases
console.log(findPairs([1, 2, 3, 4, 5], 5));
// [[1, 4], [2, 3]]

console.log(findPairs([1, 5, 7, -1, 5], 6));
// [[1, 5], [1, 5], [7, -1]]

console.log(findPairs([1, 2, 3], 10));
// []
```

### **Approach 2: Using Set (Optimal)**
```javascript
/**
 * Find pairs using Set for O(n) time
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * More efficient for large arrays
 */
function findPairs(arr, target) {
  const pairs = [];
  const seen = new Set();
  
  for (const num of arr) {
    const complement = target - num;
    
    if (seen.has(complement)) {
      pairs.push([complement, num]);
    }
    
    seen.add(num);
  }
  
  return pairs;
}

// Test
console.log(findPairs([1, 2, 3, 4, 5], 5));
// [[1, 4], [2, 3]]

console.log(findPairs([3, 5, 2, -4, 8, 11], 7));
// [[5, 2], [11, -4]]
```

### **Approach 3: Two Pointer (For Sorted Array)**
```javascript
/**
 * Find pairs using two pointers (requires sorted array)
 * Time Complexity: O(n log n) for sorting + O(n) for finding
 * Space Complexity: O(1) excluding output
 */
function findPairs(arr, target) {
  const sorted = [...arr].sort((a, b) => a - b);
  const pairs = [];
  let left = 0;
  let right = sorted.length - 1;
  
  while (left < right) {
    const sum = sorted[left] + sorted[right];
    
    if (sum === target) {
      pairs.push([sorted[left], sorted[right]]);
      left++;
      right--;
    } else if (sum < target) {
      left++;
    } else {
      right--;
    }
  }
  
  return pairs;
}

// Test
console.log(findPairs([1, 5, 3, 7, 2], 8));
// [[1, 7], [3, 5]]
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready pair finder with options
 * 
 * @param {number[]} arr - Input array
 * @param {number} target - Target sum
 * @param {Object} options - Configuration options
 * @param {boolean} options.unique - Return unique pairs only (default: false)
 * @param {boolean} options.indices - Return indices instead of values (default: false)
 * @param {boolean} options.allowDuplicates - Allow same element twice (default: false)
 * @returns {Array} - Array of pairs
 */
function findPairs(arr, target, options = {}) {
  const {
    unique = false,
    indices = false,
    allowDuplicates = false
  } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('First argument must be an array');
  }
  
  if (typeof target !== 'number') {
    throw new TypeError('Target must be a number');
  }
  
  const pairs = [];
  const seen = new Map(); // value -> array of indices
  const usedPairs = new Set(); // for unique pairs
  
  for (let i = 0; i < arr.length; i++) {
    const num = arr[i];
    const complement = target - num;
    
    if (seen.has(complement)) {
      const complementIndices = seen.get(complement);
      
      for (const j of complementIndices) {
        // Skip if not allowing duplicates and it's the same index
        if (!allowDuplicates && i === j) continue;
        
        // Create pair
        let pair;
        if (indices) {
          pair = j < i ? [j, i] : [i, j];
        } else {
          pair = [complement, num];
        }
        
        // Check uniqueness if needed
        if (unique) {
          const pairKey = indices 
            ? `${pair[0]},${pair[1]}`
            : `${Math.min(pair[0], pair[1])},${Math.max(pair[0], pair[1])}`;
          
          if (usedPairs.has(pairKey)) continue;
          usedPairs.add(pairKey);
        }
        
        pairs.push(pair);
        
        // Only add first complementIndex if not allowing multiple pairs per element
        if (!allowDuplicates) break;
      }
    }
    
    // Add current number to seen map
    if (!seen.has(num)) {
      seen.set(num, []);
    }
    seen.get(num).push(i);
  }
  
  return pairs;
}

// Find count of pairs instead of actual pairs
function countPairs(arr, target) {
  const seen = new Map(); // value -> count
  let count = 0;
  
  for (const num of arr) {
    const complement = target - num;
    
    if (seen.has(complement)) {
      count += seen.get(complement);
    }
    
    seen.set(num, (seen.get(num) || 0) + 1);
  }
  
  return count;
}

// Find unique pairs only
function findUniquePairs(arr, target) {
  const sorted = [...arr].sort((a, b) => a - b);
  const pairs = [];
  let left = 0;
  let right = sorted.length - 1;
  
  while (left < right) {
    const sum = sorted[left] + sorted[right];
    
    if (sum === target) {
      pairs.push([sorted[left], sorted[right]]);
      
      // Skip duplicates
      const leftVal = sorted[left];
      const rightVal = sorted[right];
      
      while (left < right && sorted[left] === leftVal) left++;
      while (left < right && sorted[right] === rightVal) right--;
    } else if (sum < target) {
      left++;
    } else {
      right--;
    }
  }
  
  return pairs;
}

// Comprehensive test suite
console.log('=== Basic Pairs ===');
const nums = [1, 2, 3, 4, 5, 6];
console.log(findPairs(nums, 7));
// [[1, 6], [2, 5], [3, 4]]

console.log('\n=== With Duplicates ===');
const dups = [1, 5, 7, -1, 5];
console.log(findPairs(dups, 6));
// [[1, 5], [1, 5], [7, -1]]

console.log('\n=== Unique Pairs ===');
console.log(findPairs(dups, 6, { unique: true }));
// [[1, 5], [7, -1]]

console.log('\n=== Return Indices ===');
console.log(findPairs([2, 7, 11, 15], 9, { indices: true }));
// [[0, 1]] (indices of 2 and 7)

console.log('\n=== No Pairs Found ===');
console.log(findPairs([1, 2, 3], 10));
// []

console.log('\n=== Negative Numbers ===');
console.log(findPairs([-5, -2, 0, 3, 5, 8], 3));
// [[-5, 8], [-2, 5], [0, 3]]

console.log('\n=== Count Pairs ===');
console.log(countPairs([1, 5, 3, 3, 3], 6));
// 4 (pairs: [3,3], [3,3], [3,3], [1,5])

console.log('\n=== Unique Pairs (Sorted Method) ===');
console.log(findUniquePairs([1, 1, 2, 3, 3, 4, 5], 5));
// [[1, 4], [2, 3]]

console.log('\n=== Same Number Twice ===');
const arr = [3, 3];
console.log(findPairs(arr, 6));
// [[3, 3]]

console.log(findPairs([3], 6, { allowDuplicates: true }));
// [] (can't pair with itself unless array has duplicate)
```

### **Bonus: K-Sum Problem (Generalized)**
```javascript
/**
 * Find all pairs/triplets/quadruplets that sum to target
 */
function kSum(arr, target, k) {
  const result = [];
  
  function helper(start, k, currentSum, path) {
    if (k === 0) {
      if (currentSum === target) {
        result.push([...path]);
      }
      return;
    }
    
    for (let i = start; i < arr.length; i++) {
      helper(i + 1, k - 1, currentSum + arr[i], [...path, arr[i]]);
    }
  }
  
  helper(0, k, 0, []);
  return result;
}

// Test
console.log(kSum([1, 2, 3, 4, 5], 9, 2));  // 2-sum (pairs)
// [[1, 8], [2, 7], [3, 6], [4, 5]]

console.log(kSum([1, 2, 3, 4], 6, 3));     // 3-sum (triplets)
// [[1, 2, 3]]
```

**Interview Tips:**
- Hash Set approach is optimal: O(n) time, O(n) space
- Brute force is O(n²) but uses O(1) space (excluding output)
- Two pointer works for sorted arrays: O(n log n)
- Clarify: should handle duplicates? Return indices or values?
- Clarify: can use same element twice? (e.g., [3, 3] for target 6)
- For unique pairs, use Set or two-pointer on sorted array
- Can extend to 3-sum, 4-sum with similar approaches
- Common interview variation: return just one pair vs all pairs

</details>

35. Rotate an array by k positions

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using slice() and concat()**
```javascript
/**
 * Rotate array to the right by k positions
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {Array} arr - Input array
 * @param {number} k - Number of positions to rotate
 * @returns {Array} - Rotated array
 */
function rotateArray(arr, k) {
  const n = arr.length;
  if (n === 0) return arr;
  
  // Normalize k (handle k > n)
  k = k % n;
  
  // Split and recombine
  return arr.slice(-k).concat(arr.slice(0, -k));
}

// Test cases
console.log(rotateArray([1, 2, 3, 4, 5], 2));
// [4, 5, 1, 2, 3]

console.log(rotateArray([1, 2, 3, 4, 5, 6, 7], 3));
// [5, 6, 7, 1, 2, 3, 4]

console.log(rotateArray([1, 2], 5));
// [2, 1] (5 % 2 = 1)
```

### **Approach 2: Using Array Destructuring**
```javascript
/**
 * Rotate using spread operator
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function rotateArray(arr, k) {
  const n = arr.length;
  if (n === 0) return arr;
  
  k = k % n;
  
  return [...arr.slice(-k), ...arr.slice(0, -k)];
}

// Test
console.log(rotateArray([1, 2, 3, 4, 5], 2));
// [4, 5, 1, 2, 3]
```

### **Approach 3: In-Place Rotation (Reversal Algorithm)**
```javascript
/**
 * Rotate in-place using reversal algorithm
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Algorithm:
 * 1. Reverse entire array
 * 2. Reverse first k elements
 * 3. Reverse remaining n-k elements
 */
function rotateArray(arr, k) {
  const n = arr.length;
  if (n === 0) return arr;
  
  k = k % n;
  
  // Helper to reverse a portion of array
  function reverse(start, end) {
    while (start < end) {
      [arr[start], arr[end]] = [arr[end], arr[start]];
      start++;
      end--;
    }
  }
  
  // Reverse entire array
  reverse(0, n - 1);
  
  // Reverse first k elements
  reverse(0, k - 1);
  
  // Reverse remaining elements
  reverse(k, n - 1);
  
  return arr;
}

// Test
const arr = [1, 2, 3, 4, 5, 6, 7];
console.log(rotateArray(arr, 3));
// [5, 6, 7, 1, 2, 3, 4]
```

### **Approach 4: Using unshift() and pop()**
```javascript
/**
 * Rotate by moving elements one at a time
 * Time Complexity: O(n * k)
 * Space Complexity: O(1)
 * 
 * Less efficient but simple
 */
function rotateArray(arr, k) {
  k = k % arr.length;
  
  for (let i = 0; i < k; i++) {
    arr.unshift(arr.pop());
  }
  
  return arr;
}

// Test
console.log(rotateArray([1, 2, 3, 4, 5], 2));
// [4, 5, 1, 2, 3]
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready array rotation with options
 * 
 * @param {Array} arr - Input array
 * @param {number} k - Number of positions to rotate
 * @param {Object} options - Rotation options
 * @param {string} options.direction - 'right' or 'left' (default: 'right')
 * @param {boolean} options.inPlace - Modify array in place (default: false)
 * @returns {Array} - Rotated array
 */
function rotateArray(arr, k, options = {}) {
  const { direction = 'right', inPlace = false } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('First argument must be an array');
  }
  
  if (typeof k !== 'number' || !Number.isInteger(k)) {
    throw new TypeError('k must be an integer');
  }
  
  const n = arr.length;
  
  if (n === 0 || k === 0) {
    return inPlace ? arr : [...arr];
  }
  
  // Normalize k
  k = ((k % n) + n) % n; // Handle negative k
  
  // Adjust for direction
  if (direction === 'left') {
    k = n - k;
  }
  
  if (inPlace) {
    // In-place rotation using reversal
    function reverse(start, end) {
      while (start < end) {
        [arr[start], arr[end]] = [arr[end], arr[start]];
        start++;
        end--;
      }
    }
    
    reverse(0, n - 1);
    reverse(0, k - 1);
    reverse(k, n - 1);
    
    return arr;
  } else {
    // Create new array
    return [...arr.slice(-k), ...arr.slice(0, -k)];
  }
}

// Rotate left
function rotateLeft(arr, k) {
  return rotateArray(arr, k, { direction: 'left' });
}

// Rotate right
function rotateRight(arr, k) {
  return rotateArray(arr, k, { direction: 'right' });
}

// Comprehensive test suite
console.log('=== Rotate Right ===');
console.log(rotateArray([1, 2, 3, 4, 5], 2));
// [4, 5, 1, 2, 3]

console.log('\n=== Rotate Left ===');
console.log(rotateArray([1, 2, 3, 4, 5], 2, { direction: 'left' }));
// [3, 4, 5, 1, 2]

console.log('\n=== In-Place ===');
const arr1 = [1, 2, 3, 4, 5];
rotateArray(arr1, 2, { inPlace: true });
console.log(arr1);
// [4, 5, 1, 2, 3]

console.log('\n=== k > length ===');
console.log(rotateArray([1, 2, 3], 5));
// [2, 3, 1] (5 % 3 = 2)

console.log('\n=== Negative k (Rotate Left) ===');
console.log(rotateArray([1, 2, 3, 4, 5], -2));
// [3, 4, 5, 1, 2]

console.log('\n=== k = 0 ===');
console.log(rotateArray([1, 2, 3], 0));
// [1, 2, 3]

console.log('\n=== Empty Array ===');
console.log(rotateArray([], 5));
// []

console.log('\n=== Single Element ===');
console.log(rotateArray([1], 10));
// [1]

console.log('\n=== Helper Functions ===');
console.log(rotateLeft([1, 2, 3, 4, 5], 2));
// [3, 4, 5, 1, 2]

console.log(rotateRight([1, 2, 3, 4, 5], 2));
// [4, 5, 1, 2, 3]
```

### **Bonus: Rotate 2D Matrix**
```javascript
/**
 * Rotate 2D matrix 90 degrees clockwise
 */
function rotateMatrix(matrix) {
  const n = matrix.length;
  
  // Transpose
  for (let i = 0; i < n; i++) {
    for (let j = i + 1; j < n; j++) {
      [matrix[i][j], matrix[j][i]] = [matrix[j][i], matrix[i][j]];
    }
  }
  
  // Reverse each row
  for (let i = 0; i < n; i++) {
    matrix[i].reverse();
  }
  
  return matrix;
}

// Test
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

console.log(rotateMatrix(matrix));
/*
[
  [7, 4, 1],
  [8, 5, 2],
  [9, 6, 3]
]
*/
```

**Interview Tips:**
- Clarify: rotate right or left? In-place or new array?
- Handle k > array.length by using modulo: k % n
- Reversal algorithm is optimal for in-place: O(n) time, O(1) space
- slice/concat creates new array: O(n) time, O(n) space
- Negative k can mean rotate in opposite direction
- For very large k, always normalize with modulo
- Common variation: rotate 2D matrix 90 degrees

</details>

36. Find the longest consecutive sequence in an array

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Sort First (Simple)**
```javascript
/**
 * Find longest consecutive sequence by sorting
 * Time Complexity: O(n log n) due to sorting
 * Space Complexity: O(1) or O(n) depending on sort
 * 
 * @param {number[]} arr - Input array
 * @returns {number} - Length of longest sequence
 */
function longestConsecutive(arr) {
  if (arr.length === 0) return 0;
  
  // Sort array
  const sorted = [...arr].sort((a, b) => a - b);
  
  let longest = 1;
  let current = 1;
  
  for (let i = 1; i < sorted.length; i++) {
    // Skip duplicates
    if (sorted[i] === sorted[i - 1]) continue;
    
    // Check if consecutive
    if (sorted[i] === sorted[i - 1] + 1) {
      current++;
      longest = Math.max(longest, current);
    } else {
      current = 1;
    }
  }
  
  return longest;
}

// Test cases
console.log(longestConsecutive([100, 4, 200, 1, 3, 2]));
// 4 (sequence: 1, 2, 3, 4)

console.log(longestConsecutive([0, 3, 7, 2, 5, 8, 4, 6, 0, 1]));
// 9 (sequence: 0, 1, 2, 3, 4, 5, 6, 7, 8)
```

### **Approach 2: Using Set (Optimal)**
```javascript
/**
 * Find longest consecutive using Set
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Only start counting from numbers that are sequence starts
 */
function longestConsecutive(arr) {
  if (arr.length === 0) return 0;
  
  const numSet = new Set(arr);
  let longest = 0;
  
  for (const num of numSet) {
    // Only start sequence if this is the beginning
    // (if num-1 doesn't exist, this is a sequence start)
    if (!numSet.has(num - 1)) {
      let currentNum = num;
      let currentLength = 1;
      
      // Count consecutive numbers
      while (numSet.has(currentNum + 1)) {
        currentNum++;
        currentLength++;
      }
      
      longest = Math.max(longest, currentLength);
    }
  }
  
  return longest;
}

// Test
console.log(longestConsecutive([100, 4, 200, 1, 3, 2]));
// 4

console.log(longestConsecutive([0, 3, 7, 2, 5, 8, 4, 6, 0, 1]));
// 9
```

### **Approach 3: Using Map (Track Ranges)**
```javascript
/**
 * Find consecutive using Map to track ranges
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function longestConsecutive(arr) {
  if (arr.length === 0) return 0;
  
  const map = new Map(); // number -> length of sequence containing it
  let longest = 0;
  
  for (const num of arr) {
    if (map.has(num)) continue; // Skip duplicates
    
    // Check if num-1 and num+1 exist
    const left = map.get(num - 1) || 0;
    const right = map.get(num + 1) || 0;
    
    // Current sequence length
    const length = left + right + 1;
    longest = Math.max(longest, length);
    
    // Update endpoints of the sequence
    map.set(num, length);
    map.set(num - left, length);
    map.set(num + right, length);
  }
  
  return longest;
}

// Test
console.log(longestConsecutive([100, 4, 200, 1, 3, 2]));
// 4
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready longest consecutive sequence finder
 * 
 * @param {number[]} arr - Input array
 * @param {Object} options - Configuration options
 * @param {boolean} options.returnSequence - Return actual sequence (default: false)
 * @param {boolean} options.allowGaps - Allow gaps of size n (default: 0)
 * @returns {number|Array} - Length or actual sequence
 */
function longestConsecutive(arr, options = {}) {
  const { returnSequence = false, allowGaps = 0 } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('Input must be an array');
  }
  
  if (arr.length === 0) {
    return returnSequence ? [] : 0;
  }
  
  const numSet = new Set(arr);
  let longest = 0;
  let longestSequence = [];
  
  for (const num of numSet) {
    // Only start from sequence beginnings
    if (!numSet.has(num - 1)) {
      let currentNum = num;
      let currentLength = 1;
      let sequence = [num];
      
      // Build sequence
      while (numSet.has(currentNum + 1)) {
        currentNum++;
        currentLength++;
        sequence.push(currentNum);
      }
      
      if (currentLength > longest) {
        longest = currentLength;
        longestSequence = sequence;
      }
    }
  }
  
  return returnSequence ? longestSequence : longest;
}

// Find all consecutive sequences
function findAllSequences(arr) {
  if (arr.length === 0) return [];
  
  const numSet = new Set(arr);
  const sequences = [];
  
  for (const num of numSet) {
    if (!numSet.has(num - 1)) {
      const sequence = [num];
      let currentNum = num;
      
      while (numSet.has(currentNum + 1)) {
        currentNum++;
        sequence.push(currentNum);
      }
      
      if (sequence.length > 1) {
        sequences.push(sequence);
      }
    }
  }
  
  return sequences.sort((a, b) => b.length - a.length);
}

// Get starting number of longest sequence
function longestConsecutiveStart(arr) {
  if (arr.length === 0) return null;
  
  const numSet = new Set(arr);
  let longest = 0;
  let startNum = null;
  
  for (const num of numSet) {
    if (!numSet.has(num - 1)) {
      let currentNum = num;
      let currentLength = 1;
      
      while (numSet.has(currentNum + 1)) {
        currentNum++;
        currentLength++;
      }
      
      if (currentLength > longest) {
        longest = currentLength;
        startNum = num;
      }
    }
  }
  
  return startNum;
}

// Comprehensive test suite
console.log('=== Basic Test ===');
console.log(longestConsecutive([100, 4, 200, 1, 3, 2]));
// 4

console.log('\n=== Long Sequence ===');
console.log(longestConsecutive([0, 3, 7, 2, 5, 8, 4, 6, 0, 1]));
// 9

console.log('\n=== Return Sequence ===');
console.log(longestConsecutive([100, 4, 200, 1, 3, 2], { returnSequence: true }));
// [1, 2, 3, 4]

console.log('\n=== With Duplicates ===');
console.log(longestConsecutive([1, 2, 0, 1, 1, 2, 3, 4]));
// 5 (0, 1, 2, 3, 4)

console.log('\n=== No Sequence ===');
console.log(longestConsecutive([10, 20, 30]));
// 1

console.log('\n=== All Consecutive ===');
console.log(longestConsecutive([5, 6, 7, 8, 9]));
// 5

console.log('\n=== Negative Numbers ===');
console.log(longestConsecutive([-1, -2, 0, 1, 2, -3]));
// 6 (-3, -2, -1, 0, 1, 2)

console.log('\n=== Empty Array ===');
console.log(longestConsecutive([]));
// 0

console.log('\n=== Find All Sequences ===');
console.log(findAllSequences([100, 4, 200, 1, 3, 2, 201, 202]));
// [[1, 2, 3, 4], [200, 201, 202]]

console.log('\n=== Get Start Number ===');
console.log(longestConsecutiveStart([100, 4, 200, 1, 3, 2]));
// 1
```

### **Bonus: Consecutive Subarray Sum**
```javascript
/**
 * Find longest subarray with consecutive numbers (order doesn't matter)
 */
function longestConsecutiveSubarray(arr) {
  let maxLength = 0;
  
  for (let i = 0; i < arr.length; i++) {
    const seen = new Set();
    let min = arr[i];
    let max = arr[i];
    seen.add(arr[i]);
    
    for (let j = i + 1; j < arr.length; j++) {
      if (seen.has(arr[j])) break; // Duplicate
      
      seen.add(arr[j]);
      min = Math.min(min, arr[j]);
      max = Math.max(max, arr[j]);
      
      // Check if all numbers in range [min, max] are present
      if (max - min === j - i) {
        maxLength = Math.max(maxLength, j - i + 1);
      }
    }
  }
  
  return maxLength;
}

// Test
console.log(longestConsecutiveSubarray([1, 4, 2, 3, 6, 5]));
// 6 (entire array is consecutive: 1, 2, 3, 4, 5, 6)
```

**Interview Tips:**
- Set-based solution is optimal: O(n) time and space
- Key insight: only check sequences starting from their minimum
- Sorting approach is O(n log n) but simpler
- Handle duplicates by using Set
- Can return length, sequence, or starting position
- Similar to "Union Find" problem
- Common follow-up: find all sequences, not just longest

</details>

37. Implement chunk array (split array into chunks of size n)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using slice() in Loop**
```javascript
/**
 * Split array into chunks of size n
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {Array} arr - Input array
 * @param {number} size - Chunk size
 * @returns {Array} - Array of chunks
 */
function chunk(arr, size) {
  const result = [];
  
  for (let i = 0; i < arr.length; i += size) {
    result.push(arr.slice(i, i + size));
  }
  
  return result;
}

// Test cases
console.log(chunk([1, 2, 3, 4, 5, 6, 7, 8], 3));
// [[1, 2, 3], [4, 5, 6], [7, 8]]

console.log(chunk([1, 2, 3, 4, 5], 2));
// [[1, 2], [3, 4], [5]]

console.log(chunk([1, 2, 3], 5));
// [[1, 2, 3]]
```

### **Approach 2: Using Array.from()**
```javascript
/**
 * Chunk using Array.from with mapping function
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function chunk(arr, size) {
  return Array.from(
    { length: Math.ceil(arr.length / size) },
    (_, i) => arr.slice(i * size, i * size + size)
  );
}

// Test
console.log(chunk([1, 2, 3, 4, 5, 6, 7], 2));
// [[1, 2], [3, 4], [5, 6], [7]]
```

### **Approach 3: Using reduce()**
```javascript
/**
 * Chunk using reduce
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function chunk(arr, size) {
  return arr.reduce((chunks, item, index) => {
    const chunkIndex = Math.floor(index / size);
    
    if (!chunks[chunkIndex]) {
      chunks[chunkIndex] = [];
    }
    
    chunks[chunkIndex].push(item);
    return chunks;
  }, []);
}

// Test
console.log(chunk([1, 2, 3, 4, 5, 6], 2));
// [[1, 2], [3, 4], [5, 6]]
```

### **Approach 4: Using splice() (Mutating)**
```javascript
/**
 * Chunk by mutating original array
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * Warning: modifies original array
 */
function chunk(arr, size) {
  const result = [];
  
  while (arr.length > 0) {
    result.push(arr.splice(0, size));
  }
  
  return result;
}

// Test
const data = [1, 2, 3, 4, 5, 6, 7];
console.log(chunk(data, 3));
// [[1, 2, 3], [4, 5, 6], [7]]
console.log(data);  // [] (original is now empty)
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready array chunking with options
 * 
 * @param {Array} arr - Input array
 * @param {number} size - Chunk size
 * @param {Object} options - Chunking options
 * @param {boolean} options.balanced - Balance chunk sizes (default: false)
 * @param {boolean} options.fillLast - Fill last chunk to size (default: false)
 * @param {*} options.fillValue - Value to use for filling (default: undefined)
 * @returns {Array} - Array of chunks
 */
function chunk(arr, size, options = {}) {
  const { balanced = false, fillLast = false, fillValue } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('First argument must be an array');
  }
  
  if (!Number.isInteger(size) || size <= 0) {
    throw new TypeError('Chunk size must be a positive integer');
  }
  
  if (arr.length === 0) {
    return [];
  }
  
  // Balanced chunking (distribute elements evenly)
  if (balanced) {
    const numChunks = Math.ceil(arr.length / size);
    const baseSize = Math.floor(arr.length / numChunks);
    const remainder = arr.length % numChunks;
    
    const result = [];
    let index = 0;
    
    for (let i = 0; i < numChunks; i++) {
      // First 'remainder' chunks get one extra element
      const chunkSize = i < remainder ? baseSize + 1 : baseSize;
      result.push(arr.slice(index, index + chunkSize));
      index += chunkSize;
    }
    
    return result;
  }
  
  // Standard chunking
  const result = [];
  
  for (let i = 0; i < arr.length; i += size) {
    const chunk = arr.slice(i, i + size);
    
    // Fill last chunk if requested
    if (fillLast && i + size > arr.length) {
      while (chunk.length < size) {
        chunk.push(fillValue);
      }
    }
    
    result.push(chunk);
  }
  
  return result;
}

// Chunk with overlap
function chunkWithOverlap(arr, size, overlap) {
  if (overlap >= size) {
    throw new Error('Overlap must be less than chunk size');
  }
  
  const result = [];
  const step = size - overlap;
  
  for (let i = 0; i < arr.length; i += step) {
    const chunk = arr.slice(i, i + size);
    if (chunk.length === size) {
      result.push(chunk);
    }
  }
  
  return result;
}

// Chunk into n groups
function chunkIntoN(arr, n) {
  const result = [];
  const chunkSize = Math.ceil(arr.length / n);
  
  for (let i = 0; i < arr.length; i += chunkSize) {
    result.push(arr.slice(i, i + chunkSize));
  }
  
  return result;
}

// Comprehensive test suite
console.log('=== Basic Chunking ===');
console.log(chunk([1, 2, 3, 4, 5, 6, 7, 8], 3));
// [[1, 2, 3], [4, 5, 6], [7, 8]]

console.log('\n=== Exact Division ===');
console.log(chunk([1, 2, 3, 4, 5, 6], 2));
// [[1, 2], [3, 4], [5, 6]]

console.log('\n=== Size Larger Than Array ===');
console.log(chunk([1, 2, 3], 5));
// [[1, 2, 3]]

console.log('\n=== Empty Array ===');
console.log(chunk([], 3));
// []

console.log('\n=== Balanced Chunking ===');
console.log(chunk([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], 3, { balanced: true }));
// [[1, 2, 3, 4], [5, 6, 7], [8, 9, 10]]
// More balanced than [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]

console.log('\n=== Fill Last Chunk ===');
console.log(chunk([1, 2, 3, 4, 5], 2, { fillLast: true, fillValue: null }));
// [[1, 2], [3, 4], [5, null]]

console.log('\n=== With Overlap ===');
console.log(chunkWithOverlap([1, 2, 3, 4, 5, 6, 7], 3, 1));
// [[1, 2, 3], [3, 4, 5], [5, 6, 7]]

console.log('\n=== Chunk Into N Groups ===');
console.log(chunkIntoN([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], 3));
// [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10]]

console.log('\n=== Single Element Per Chunk ===');
console.log(chunk([1, 2, 3, 4], 1));
// [[1], [2], [3], [4]]
```

### **Bonus: Chunk String**
```javascript
/**
 * Chunk a string into array of substrings
 */
function chunkString(str, size) {
  const chunks = [];
  
  for (let i = 0; i < str.length; i += size) {
    chunks.push(str.slice(i, i + size));
  }
  
  return chunks;
}

// Or using regex
function chunkString(str, size) {
  const regex = new RegExp(`.{1,${size}}`, 'g');
  return str.match(regex) || [];
}

// Test
console.log(chunkString('abcdefghij', 3));
// ['abc', 'def', 'ghi', 'j']

console.log(chunkString('hello world', 5));
// ['hello', ' worl', 'd']
```

**Interview Tips:**
- slice() is non-mutating and safest approach
- splice() mutates original array - avoid unless needed
- Handle edge cases: empty array, size > length, size = 1
- Balanced chunking distributes elements more evenly
- Can add overlap for sliding window applications
- Common use: pagination, batch processing, parallel processing
- Consider lodash.chunk() for production

</details>

38. Find all subarrays that sum to zero

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Brute Force (Check All Subarrays)**
```javascript
/**
 * Find all subarrays with sum = 0 using brute force
 * Time Complexity: O(n²)
 * Space Complexity: O(k) where k is result size
 * 
 * @param {number[]} arr - Input array
 * @returns {Array} - Array of subarrays that sum to 0
 */
function findZeroSumSubarrays(arr) {
  const result = [];
  
  for (let i = 0; i < arr.length; i++) {
    let sum = 0;
    
    for (let j = i; j < arr.length; j++) {
      sum += arr[j];
      
      if (sum === 0) {
        result.push(arr.slice(i, j + 1));
      }
    }
  }
  
  return result;
}

// Test cases
console.log(findZeroSumSubarrays([4, 2, -3, 1, 6]));
// [[2, -3, 1]]

console.log(findZeroSumSubarrays([3, 4, -7, 3, 1, 3, 1, -4, -2, -2]));
// [[3, 4, -7], [3, 1, -4], [3, 4, -7, 3, 1, 3, 1, -4, -2, -2]]

console.log(findZeroSumSubarrays([0, 1, -1, 0]));
// [[0], [1, -1], [0]]
```

### **Approach 2: Using Prefix Sum with HashMap**
```javascript
/**
 * Find zero-sum subarrays using prefix sums
 * Time Complexity: O(n²) worst case, but more efficient in practice
 * Space Complexity: O(n)
 * 
 * If sum[i] == sum[j], then subarray (i, j] has sum 0
 */
function findZeroSumSubarrays(arr) {
  const result = [];
  const sumMap = new Map();
  sumMap.set(0, [-1]); // Handle subarrays starting from index 0
  
  let sum = 0;
  
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i];
    
    if (sumMap.has(sum)) {
      // All indices where this sum occurred before
      const indices = sumMap.get(sum);
      
      for (const startIndex of indices) {
        result.push(arr.slice(startIndex + 1, i + 1));
      }
    }
    
    // Store current index for this sum
    if (!sumMap.has(sum)) {
      sumMap.set(sum, []);
    }
    sumMap.get(sum).push(i);
  }
  
  return result;
}

// Test
console.log(findZeroSumSubarrays([4, 2, -3, 1, 6]));
// [[2, -3, 1]]

console.log(findZeroSumSubarrays([1, -1, 2, -2, 3, -3]));
// [[1, -1], [2, -2], [3, -3], [1, -1, 2, -2], [2, -2, 3, -3], [1, -1, 2, -2, 3, -3]]
```

### **Approach 3: Count Zero-Sum Subarrays**
```javascript
/**
 * Just count how many subarrays sum to zero
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function countZeroSumSubarrays(arr) {
  const sumMap = new Map();
  sumMap.set(0, 1); // Empty subarray
  
  let sum = 0;
  let count = 0;
  
  for (const num of arr) {
    sum += num;
    
    if (sumMap.has(sum)) {
      count += sumMap.get(sum);
    }
    
    sumMap.set(sum, (sumMap.get(sum) || 0) + 1);
  }
  
  return count;
}

// Test
console.log(countZeroSumSubarrays([4, 2, -3, 1, 6]));
// 1

console.log(countZeroSumSubarrays([1, -1, 2, -2]));
// 3 ([1, -1], [2, -2], [1, -1, 2, -2])
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready zero-sum subarray finder
 * 
 * @param {number[]} arr - Input array
 * @param {Object} options - Find options
 * @param {number} options.target - Target sum (default: 0)
 * @param {boolean} options.returnIndices - Return indices instead of subarrays
 * @param {boolean} options.countOnly - Return count only
 * @returns {Array|number} - Subarrays, indices, or count
 */
function findZeroSumSubarrays(arr, options = {}) {
  const {
    target = 0,
    returnIndices = false,
    countOnly = false
  } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('Input must be an array');
  }
  
  if (arr.length === 0) {
    return countOnly ? 0 : [];
  }
  
  const sumMap = new Map();
  sumMap.set(0, [-1]);
  
  let sum = 0;
  let count = 0;
  const result = [];
  
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i];
    const lookingFor = sum - target;
    
    if (sumMap.has(lookingFor)) {
      const indices = sumMap.get(lookingFor);
      
      for (const startIndex of indices) {
        count++;
        
        if (!countOnly) {
          if (returnIndices) {
            result.push([startIndex + 1, i]);
          } else {
            result.push(arr.slice(startIndex + 1, i + 1));
          }
        }
      }
    }
    
    if (!sumMap.has(sum)) {
      sumMap.set(sum, []);
    }
    sumMap.get(sum).push(i);
  }
  
  return countOnly ? count : result;
}

// Check if array has any zero-sum subarray
function hasZeroSumSubarray(arr) {
  const seen = new Set([0]);
  let sum = 0;
  
  for (const num of arr) {
    sum += num;
    
    if (seen.has(sum)) {
      return true;
    }
    
    seen.add(sum);
  }
  
  return false;
}

// Find longest zero-sum subarray
function longestZeroSumSubarray(arr) {
  const sumMap = new Map();
  sumMap.set(0, -1);
  
  let sum = 0;
  let maxLength = 0;
  let result = [];
  
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i];
    
    if (sumMap.has(sum)) {
      const length = i - sumMap.get(sum);
      if (length > maxLength) {
        maxLength = length;
        result = arr.slice(sumMap.get(sum) + 1, i + 1);
      }
    } else {
      sumMap.set(sum, i);
    }
  }
  
  return result;
}

// Comprehensive test suite
console.log('=== Find All Subarrays ===');
console.log(findZeroSumSubarrays([4, 2, -3, 1, 6]));
// [[2, -3, 1]]

console.log('\n=== With Single Zeros ===');
console.log(findZeroSumSubarrays([0, 1, -1, 0]));
// [[0], [1, -1], [0, 1, -1, 0], [0]]

console.log('\n=== Count Only ===');
console.log(findZeroSumSubarrays([1, -1, 2, -2], { countOnly: true }));
// 3

console.log('\n=== Return Indices ===');
console.log(findZeroSumSubarrays([4, 2, -3, 1, 6], { returnIndices: true }));
// [[1, 3]] (indices of subarray [2, -3, 1])

console.log('\n=== Target Sum (Not Zero) ===');
console.log(findZeroSumSubarrays([1, 2, 3, 4, 5], { target: 9 }));
// [[2, 3, 4], [4, 5]]

console.log('\n=== Has Zero-Sum Subarray ===');
console.log(hasZeroSumSubarray([4, 2, -3, 1, 6]));
// true

console.log(hasZeroSumSubarray([1, 2, 3, 4]));
// false

console.log('\n=== Longest Zero-Sum Subarray ===');
console.log(longestZeroSumSubarray([3, 4, -7, 3, 1, 3, 1, -4, -2, -2]));
// [3, 4, -7, 3, 1, 3, 1, -4, -2, -2] (length 10)

console.log(longestZeroSumSubarray([1, -1, 3, 2, -2, -3, 3]));
// [1, -1, 3, 2, -2, -3, 3] (length 7, entire array)

console.log('\n=== No Zero-Sum Subarray ===');
console.log(findZeroSumSubarrays([1, 2, 3, 4]));
// []

console.log('\n=== All Negative ===');
console.log(findZeroSumSubarrays([-1, -2, -3]));
// []

console.log('\n=== Complex Case ===');
console.log(findZeroSumSubarrays([1, 2, -3, 4, -4, 5, -5]));
// [[1, 2, -3], [4, -4], [5, -5], [4, -4, 5, -5]]
```

### **Bonus: Maximum Length Subarray with Sum K**
```javascript
/**
 * Find maximum length subarray with sum = k
 */
function maxLengthSumK(arr, k) {
  const sumMap = new Map();
  sumMap.set(0, -1);
  
  let sum = 0;
  let maxLength = 0;
  
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i];
    
    if (sumMap.has(sum - k)) {
      maxLength = Math.max(maxLength, i - sumMap.get(sum - k));
    }
    
    if (!sumMap.has(sum)) {
      sumMap.set(sum, i);
    }
  }
  
  return maxLength;
}

// Test
console.log(maxLengthSumK([10, 5, 2, 7, 1, 9], 15));
// 4 (subarray [5, 2, 7, 1])
```

**Interview Tips:**
- Prefix sum approach is key insight
- If sum[i] == sum[j], then subarray (i, j] sums to 0
- Use HashMap to store sum -> indices mapping
- O(n) time for counting, O(n²) worst case for finding all
- Can generalize to find subarrays with sum = k
- Single zero counts as zero-sum subarray
- Common follow-ups: longest zero-sum, count, check existence

</details>

39. Sort an array of objects by multiple properties

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using sort() with Multiple Comparisons**
```javascript
/**
 * Sort array of objects by multiple properties
 * Time Complexity: O(n log n)
 * Space Complexity: O(1) or O(log n) depending on sort implementation
 * 
 * @param {Array} arr - Array of objects
 * @param {Array} keys - Array of property names to sort by
 * @returns {Array} - Sorted array
 */
function sortByMultiple(arr, keys) {
  return arr.sort((a, b) => {
    for (const key of keys) {
      if (a[key] < b[key]) return -1;
      if (a[key] > b[key]) return 1;
    }
    return 0;
  });
}

// Test cases
const users = [
  { name: 'John', age: 30, score: 85 },
  { name: 'Jane', age: 25, score: 90 },
  { name: 'Bob', age: 30, score: 75 },
  { name: 'Alice', age: 25, score: 90 }
];

console.log(sortByMultiple(users, ['age', 'score']));
/*
[
  { name: 'Jane', age: 25, score: 90 },
  { name: 'Alice', age: 25, score: 90 },
  { name: 'Bob', age: 30, score: 75 },
  { name: 'John', age: 30, score: 85 }
]
*/
```

### **Approach 2: With Ascending/Descending Order**
```javascript
/**
 * Sort with order specification for each property
 * Time Complexity: O(n log n)
 * Space Complexity: O(1)
 * 
 * @param {Array} arr - Array of objects
 * @param {Array} sortBy - Array of {key, order} objects
 */
function sortByMultiple(arr, sortBy) {
  return arr.sort((a, b) => {
    for (const { key, order = 'asc' } of sortBy) {
      const aVal = a[key];
      const bVal = b[key];
      
      let comparison = 0;
      
      if (aVal < bVal) comparison = -1;
      else if (aVal > bVal) comparison = 1;
      
      if (comparison !== 0) {
        return order === 'asc' ? comparison : -comparison;
      }
    }
    return 0;
  });
}

// Test
const data = [
  { name: 'John', age: 30, score: 85 },
  { name: 'Jane', age: 25, score: 90 },
  { name: 'Bob', age: 30, score: 75 }
];

console.log(sortByMultiple(data, [
  { key: 'age', order: 'asc' },
  { key: 'score', order: 'desc' }
]));
/*
[
  { name: 'Jane', age: 25, score: 90 },
  { name: 'John', age: 30, score: 85 },
  { name: 'Bob', age: 30, score: 75 }
]
*/
```

### **Approach 3: Using localeCompare for Strings**
```javascript
/**
 * Sort with proper string comparison
 * Time Complexity: O(n log n)
 * Space Complexity: O(1)
 */
function sortByMultiple(arr, keys) {
  return arr.sort((a, b) => {
    for (const key of keys) {
      const aVal = a[key];
      const bVal = b[key];
      
      // String comparison
      if (typeof aVal === 'string' && typeof bVal === 'string') {
        const comparison = aVal.localeCompare(bVal);
        if (comparison !== 0) return comparison;
      }
      // Number comparison
      else {
        if (aVal < bVal) return -1;
        if (aVal > bVal) return 1;
      }
    }
    return 0;
  });
}

// Test
const people = [
  { firstName: 'John', lastName: 'Doe', age: 30 },
  { firstName: 'Jane', lastName: 'Doe', age: 25 },
  { firstName: 'Bob', lastName: 'Smith', age: 30 }
];

console.log(sortByMultiple(people, ['lastName', 'firstName']));
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready multi-property sorter
 * 
 * @param {Array} arr - Array of objects to sort
 * @param {Array|Object} criteria - Sort criteria
 * @param {Object} options - Sort options
 * @param {boolean} options.clone - Sort a copy instead of in-place
 * @param {boolean} options.stable - Maintain stable sort (preserve original order for equal elements)
 * @returns {Array} - Sorted array
 */
function sortByMultiple(arr, criteria, options = {}) {
  const { clone = false, stable = false } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('First argument must be an array');
  }
  
  // Normalize criteria to array format
  let sortCriteria;
  if (typeof criteria === 'string') {
    sortCriteria = [{ key: criteria, order: 'asc' }];
  } else if (Array.isArray(criteria)) {
    sortCriteria = criteria.map(item => {
      if (typeof item === 'string') {
        return { key: item, order: 'asc' };
      }
      return { key: item.key, order: item.order || 'asc', type: item.type };
    });
  } else {
    throw new TypeError('Criteria must be a string or array');
  }
  
  // Clone if requested
  const arrayToSort = clone ? [...arr] : arr;
  
  // Add original index for stable sort
  if (stable) {
    arrayToSort.forEach((item, index) => {
      item.__originalIndex = index;
    });
  }
  
  arrayToSort.sort((a, b) => {
    for (const criterion of sortCriteria) {
      const { key, order, type } = criterion;
      
      // Get values (support nested properties like 'user.name')
      const aVal = getNestedValue(a, key);
      const bVal = getNestedValue(b, key);
      
      // Handle null/undefined
      if (aVal == null && bVal == null) continue;
      if (aVal == null) return 1;
      if (bVal == null) return -1;
      
      let comparison = 0;
      
      // Type-specific comparison
      if (type === 'string' || (typeof aVal === 'string' && typeof bVal === 'string')) {
        comparison = aVal.localeCompare(bVal, undefined, { numeric: true, sensitivity: 'base' });
      } else if (type === 'number' || (typeof aVal === 'number' && typeof bVal === 'number')) {
        comparison = aVal - bVal;
      } else if (type === 'date' || (aVal instanceof Date && bVal instanceof Date)) {
        comparison = aVal.getTime() - bVal.getTime();
      } else {
        // Generic comparison
        if (aVal < bVal) comparison = -1;
        else if (aVal > bVal) comparison = 1;
      }
      
      if (comparison !== 0) {
        return order === 'desc' ? -comparison : comparison;
      }
    }
    
    // Stable sort: use original index as tiebreaker
    if (stable) {
      return a.__originalIndex - b.__originalIndex;
    }
    
    return 0;
  });
  
  // Clean up stable sort markers
  if (stable) {
    arrayToSort.forEach(item => {
      delete item.__originalIndex;
    });
  }
  
  return arrayToSort;
}

// Helper to get nested property value
function getNestedValue(obj, path) {
  return path.split('.').reduce((value, key) => value?.[key], obj);
}

// Sort by function instead of property name
function sortByFunction(arr, compareFn) {
  return arr.sort(compareFn);
}

// Create reusable sorter
function createSorter(...criteria) {
  return (arr) => sortByMultiple(arr, criteria, { clone: true });
}

// Comprehensive test suite
console.log('=== Basic Multi-Property Sort ===');
const employees = [
  { name: 'John', department: 'IT', salary: 80000 },
  { name: 'Jane', department: 'HR', salary: 70000 },
  { name: 'Bob', department: 'IT', salary: 90000 },
  { name: 'Alice', department: 'HR', salary: 75000 }
];

console.log(sortByMultiple(employees, ['department', 'salary']));
/*
Sorted by department first, then salary within each department
*/

console.log('\n=== With Descending Order ===');
console.log(sortByMultiple(employees, [
  { key: 'department', order: 'asc' },
  { key: 'salary', order: 'desc' }
], { clone: true }));
/*
HR: Alice (75000), Jane (70000)
IT: Bob (90000), John (80000)
*/

console.log('\n=== Nested Properties ===');
const users = [
  { name: 'John', address: { city: 'NYC', zip: 10001 } },
  { name: 'Jane', address: { city: 'LA', zip: 90001 } },
  { name: 'Bob', address: { city: 'NYC', zip: 10002 } }
];

console.log(sortByMultiple(users, ['address.city', 'address.zip']));

console.log('\n=== Date Sorting ===');
const events = [
  { name: 'Event A', date: new Date('2024-03-15'), priority: 2 },
  { name: 'Event B', date: new Date('2024-03-10'), priority: 1 },
  { name: 'Event C', date: new Date('2024-03-10'), priority: 3 }
];

console.log(sortByMultiple(events, [
  { key: 'date', type: 'date' },
  { key: 'priority', order: 'desc' }
]));

console.log('\n=== String with Numbers (Natural Sort) ===');
const items = [
  { id: 'item10' },
  { id: 'item2' },
  { id: 'item1' },
  { id: 'item20' }
];

console.log(sortByMultiple(items, [{ key: 'id', type: 'string' }]));
// Natural sort: item1, item2, item10, item20

console.log('\n=== Null/Undefined Handling ===');
const data = [
  { name: 'John', age: 30 },
  { name: 'Jane', age: null },
  { name: 'Bob', age: 25 },
  { name: 'Alice', age: undefined }
];

console.log(sortByMultiple(data, ['age', 'name']));
// Nulls/undefined sorted to end

console.log('\n=== Reusable Sorter ===');
const sortByDeptAndSalary = createSorter(
  { key: 'department' },
  { key: 'salary', order: 'desc' }
);

console.log(sortByDeptAndSalary(employees));

console.log('\n=== Clone vs In-Place ===');
const original = [{ id: 3 }, { id: 1 }, { id: 2 }];
const sorted = sortByMultiple(original, ['id'], { clone: true });

console.log('Original:', original.map(x => x.id));  // [3, 1, 2]
console.log('Sorted:', sorted.map(x => x.id));      // [1, 2, 3]
```

### **Bonus: Advanced Sorting Utilities**
```javascript
/**
 * Sort with custom comparator for specific property
 */
function sortWithComparators(arr, comparators) {
  return arr.sort((a, b) => {
    for (const { key, compare } of comparators) {
      const result = compare(a[key], b[key]);
      if (result !== 0) return result;
    }
    return 0;
  });
}

/**
 * Multi-level grouping and sorting
 */
function groupAndSort(arr, groupBy, sortBy) {
  const grouped = {};
  
  // Group
  arr.forEach(item => {
    const key = item[groupBy];
    if (!grouped[key]) grouped[key] = [];
    grouped[key].push(item);
  });
  
  // Sort each group
  Object.keys(grouped).forEach(key => {
    grouped[key] = sortByMultiple(grouped[key], sortBy);
  });
  
  return grouped;
}

/**
 * Sort by multiple properties with priority weights
 */
function sortWithWeights(arr, criteria) {
  return arr.sort((a, b) => {
    let totalScore = 0;
    
    for (const { key, weight = 1, order = 'asc' } of criteria) {
      const aVal = a[key];
      const bVal = b[key];
      
      let comparison = 0;
      if (aVal < bVal) comparison = -1;
      else if (aVal > bVal) comparison = 1;
      
      totalScore += comparison * weight * (order === 'desc' ? -1 : 1);
    }
    
    return totalScore;
  });
}

// Test
const products = [
  { name: 'Laptop', price: 1000, rating: 4.5 },
  { name: 'Mouse', price: 25, rating: 4.8 },
  { name: 'Keyboard', price: 75, rating: 4.7 }
];

console.log(sortWithWeights(products, [
  { key: 'rating', weight: 2, order: 'desc' },
  { key: 'price', weight: 1, order: 'asc' }
]));
// Prioritize rating more than price
```

**Interview Tips:**
- sort() is in-place by default; clone array if needed
- localeCompare() for proper string comparison (handles accents, case)
- Handle null/undefined values explicitly
- Can sort by nested properties using dot notation
- Order matters: primary sort first, then secondary, etc.
- Consider stable vs unstable sort implementations
- Common use: tables, data grids, reports, search results
- lodash.orderBy() and lodash.sortBy() for production use

</details>

40. Find the majority element in an array (appears more than n/2 times)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using HashMap (Count Frequencies)**
```javascript
/**
 * Find majority element using frequency map
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {Array} arr - Input array
 * @returns {*} - Majority element or null
 */
function findMajorityElement(arr) {
  const n = arr.length;
  const countMap = new Map();
  
  for (const num of arr) {
    countMap.set(num, (countMap.get(num) || 0) + 1);
    
    if (countMap.get(num) > n / 2) {
      return num;
    }
  }
  
  return null;
}

// Test cases
console.log(findMajorityElement([3, 2, 3]));
// 3

console.log(findMajorityElement([2, 2, 1, 1, 1, 2, 2]));
// 2

console.log(findMajorityElement([1, 2, 3]));
// null (no majority)
```

### **Approach 2: Boyer-Moore Voting Algorithm (Optimal)**
```javascript
/**
 * Find majority element using Boyer-Moore algorithm
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Works only if majority element is guaranteed to exist
 */
function findMajorityElement(arr) {
  if (arr.length === 0) return null;
  
  // Phase 1: Find candidate
  let candidate = arr[0];
  let count = 1;
  
  for (let i = 1; i < arr.length; i++) {
    if (count === 0) {
      candidate = arr[i];
      count = 1;
    } else if (arr[i] === candidate) {
      count++;
    } else {
      count--;
    }
  }
  
  // Phase 2: Verify candidate (if majority not guaranteed)
  count = 0;
  for (const num of arr) {
    if (num === candidate) {
      count++;
    }
  }
  
  return count > arr.length / 2 ? candidate : null;
}

// Test
console.log(findMajorityElement([3, 2, 3]));
// 3

console.log(findMajorityElement([2, 2, 1, 1, 1, 2, 2]));
// 2
```

### **Approach 3: Using Sorting**
```javascript
/**
 * Find majority element by sorting
 * Time Complexity: O(n log n)
 * Space Complexity: O(1) or O(n) depending on sort
 * 
 * If majority exists, it will be at index n/2 after sorting
 */
function findMajorityElement(arr) {
  if (arr.length === 0) return null;
  
  const sorted = [...arr].sort();
  const candidate = sorted[Math.floor(arr.length / 2)];
  
  // Verify
  let count = 0;
  for (const num of arr) {
    if (num === candidate) count++;
  }
  
  return count > arr.length / 2 ? candidate : null;
}

// Test
console.log(findMajorityElement([2, 2, 1, 1, 1, 2, 2]));
// 2
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready majority element finder
 * 
 * @param {Array} arr - Input array
 * @param {Object} options - Find options
 * @param {number} options.threshold - Threshold fraction (default: 0.5 for >n/2)
 * @param {boolean} options.returnCount - Return {element, count} instead of just element
 * @param {Function} options.comparator - Custom equality comparator
 * @returns {*} - Majority element(s) or null
 */
function findMajorityElement(arr, options = {}) {
  const {
    threshold = 0.5,
    returnCount = false,
    comparator = (a, b) => a === b
  } = options;
  
  // Validation
  if (!Array.isArray(arr)) {
    throw new TypeError('Input must be an array');
  }
  
  if (arr.length === 0) {
    return null;
  }
  
  const n = arr.length;
  const requiredCount = Math.floor(n * threshold) + 1;
  
  // Boyer-Moore for >n/2 majority
  if (threshold === 0.5) {
    // Phase 1: Find candidate
    let candidate = arr[0];
    let count = 1;
    
    for (let i = 1; i < n; i++) {
      if (count === 0) {
        candidate = arr[i];
        count = 1;
      } else if (comparator(arr[i], candidate)) {
        count++;
      } else {
        count--;
      }
    }
    
    // Phase 2: Verify
    count = 0;
    for (const item of arr) {
      if (comparator(item, candidate)) {
        count++;
      }
    }
    
    if (count >= requiredCount) {
      return returnCount ? { element: candidate, count } : candidate;
    }
    
    return null;
  }
  
  // For other thresholds, use frequency map
  const countMap = new Map();
  
  for (const item of arr) {
    let found = false;
    
    // Check if item already in map (using comparator)
    for (const [key, value] of countMap) {
      if (comparator(key, item)) {
        countMap.set(key, value + 1);
        found = true;
        
        if (countMap.get(key) >= requiredCount) {
          return returnCount ? { element: key, count: countMap.get(key) } : key;
        }
        break;
      }
    }
    
    if (!found) {
      countMap.set(item, 1);
    }
  }
  
  return null;
}

// Find all elements appearing more than n/k times
function findMajorityElements(arr, k) {
  const n = arr.length;
  const threshold = Math.floor(n / k);
  const countMap = new Map();
  const result = [];
  
  for (const num of arr) {
    countMap.set(num, (countMap.get(num) || 0) + 1);
  }
  
  for (const [num, count] of countMap) {
    if (count > threshold) {
      result.push(num);
    }
  }
  
  return result;
}

// Get frequency of most common element
function getMostFrequent(arr) {
  if (arr.length === 0) return null;
  
  const countMap = new Map();
  let maxCount = 0;
  let mostFrequent = null;
  
  for (const num of arr) {
    const count = (countMap.get(num) || 0) + 1;
    countMap.set(num, count);
    
    if (count > maxCount) {
      maxCount = count;
      mostFrequent = num;
    }
  }
  
  return { element: mostFrequent, count: maxCount };
}

// Comprehensive test suite
console.log('=== Basic Majority ===');
console.log(findMajorityElement([3, 2, 3]));
// 3

console.log(findMajorityElement([2, 2, 1, 1, 1, 2, 2]));
// 2

console.log('\n=== No Majority ===');
console.log(findMajorityElement([1, 2, 3]));
// null

console.log('\n=== Return with Count ===');
console.log(findMajorityElement([2, 2, 1, 1, 1, 2, 2], { returnCount: true }));
// { element: 2, count: 4 }

console.log('\n=== Custom Threshold (>n/3) ===');
console.log(findMajorityElement([1, 1, 2, 2, 3, 3, 3], { threshold: 1/3 }));
// 3 (appears 3 times > 7/3)

console.log('\n=== Object Comparison ===');
const objects = [
  { id: 1, name: 'A' },
  { id: 1, name: 'A' },
  { id: 2, name: 'B' },
  { id: 1, name: 'A' }
];

console.log(findMajorityElement(objects, {
  comparator: (a, b) => a.id === b.id
}));
// { id: 1, name: 'A' }

console.log('\n=== Find All Elements > n/3 ===');
console.log(findMajorityElements([3, 2, 3, 1, 1, 2, 3], 3));
// [3] (appears 3 times > 7/3)

console.log(findMajorityElements([1, 1, 2, 2, 3], 3));
// [1, 2] (both appear 2 times > 5/3)

console.log('\n=== Most Frequent (Not Necessarily Majority) ===');
console.log(getMostFrequent([1, 2, 2, 3, 3, 3]));
// { element: 3, count: 3 }

console.log('\n=== Single Element ===');
console.log(findMajorityElement([5]));
// 5

console.log('\n=== All Same ===');
console.log(findMajorityElement([7, 7, 7, 7]));
// 7

console.log('\n=== Empty Array ===');
console.log(findMajorityElement([]));
// null
```

### **Bonus: Majority Element in Sliding Window**
```javascript
/**
 * Find majority element in each window of size k
 */
function majorityInWindows(arr, k) {
  const result = [];
  
  for (let i = 0; i <= arr.length - k; i++) {
    const window = arr.slice(i, i + k);
    const majority = findMajorityElement(window);
    result.push(majority);
  }
  
  return result;
}

// Test
console.log(majorityInWindows([1, 2, 2, 2, 3, 3, 3, 3, 4], 3));
// [2, 2, null, 3, 3, 3, 3]
```

**Interview Tips:**
- Boyer-Moore algorithm is optimal: O(n) time, O(1) space
- Works for majority element appearing >n/2 times
- HashMap approach is more general but uses O(n) space
- Always verify candidate in phase 2 if not guaranteed
- Can extend to find elements appearing >n/k times
- Sorting approach is O(n log n) but intuitive
- Common follow-up: find all elements >n/3, >n/k
- Clarify: is majority guaranteed to exist?

</details>

### **String Manipulations**

41. Implement a function to compress a string (e.g., "aaabbc" → "a3b2c1")

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Iterate and Count**
```javascript
/**
 * Compress string using run-length encoding
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {string} str - Input string
 * @returns {string} - Compressed string
 */
function compressString(str) {
  if (str.length === 0) return str;
  
  let compressed = '';
  let count = 1;
  
  for (let i = 1; i <= str.length; i++) {
    // Check if current char is different or end of string
    if (i === str.length || str[i] !== str[i - 1]) {
      compressed += str[i - 1] + count;
      count = 1;
    } else {
      count++;
    }
  }
  
  // Return original if compressed is not shorter
  return compressed.length < str.length ? compressed : str;
}

// Test cases
console.log(compressString('aaabbc'));
// 'a3b2c1'

console.log(compressString('aabcccccaaa'));
// 'a2b1c5a3'

console.log(compressString('abc'));
// 'abc' (compressed would be 'a1b1c1', longer)

console.log(compressString('aaa'));
// 'a3'
```

### **Approach 2: Using Array (More Efficient)**
```javascript
/**
 * Compress using array for better performance
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * String concatenation is slow; array push + join is faster
 */
function compressString(str) {
  if (str.length === 0) return str;
  
  const result = [];
  let count = 1;
  
  for (let i = 1; i <= str.length; i++) {
    if (i === str.length || str[i] !== str[i - 1]) {
      result.push(str[i - 1], count);
      count = 1;
    } else {
      count++;
    }
  }
  
  const compressed = result.join('');
  return compressed.length < str.length ? compressed : str;
}

// Test
console.log(compressString('aabbccc'));
// 'a2b2c3'
```

### **Approach 3: Without Count if 1**
```javascript
/**
 * Compress but omit count if it's 1
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function compressString(str) {
  if (str.length === 0) return str;
  
  let compressed = '';
  let count = 1;
  
  for (let i = 1; i <= str.length; i++) {
    if (i === str.length || str[i] !== str[i - 1]) {
      compressed += str[i - 1];
      if (count > 1) {
        compressed += count;
      }
      count = 1;
    } else {
      count++;
    }
  }
  
  return compressed.length < str.length ? compressed : str;
}

// Test
console.log(compressString('aaabbc'));
// 'a3b2c' (instead of 'a3b2c1')
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready string compression
 * 
 * @param {string} str - Input string
 * @param {Object} options - Compression options
 * @param {boolean} options.omitSingleCounts - Don't show count if 1 (default: false)
 * @param {boolean} options.alwaysCompress - Return compressed even if longer (default: false)
 * @param {boolean} options.caseInsensitive - Treat a and A as same (default: false)
 * @returns {string} - Compressed string
 */
function compressString(str, options = {}) {
  const {
    omitSingleCounts = false,
    alwaysCompress = false,
    caseInsensitive = false
  } = options;
  
  // Validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  if (str.length === 0) {
    return str;
  }
  
  // Normalize case if requested
  const workStr = caseInsensitive ? str.toLowerCase() : str;
  
  const result = [];
  let count = 1;
  let currentChar = workStr[0];
  
  for (let i = 1; i <= workStr.length; i++) {
    if (i === workStr.length || workStr[i] !== currentChar) {
      // Add character
      result.push(currentChar);
      
      // Add count if needed
      if (!omitSingleCounts || count > 1) {
        result.push(count);
      }
      
      if (i < workStr.length) {
        currentChar = workStr[i];
        count = 1;
      }
    } else {
      count++;
    }
  }
  
  const compressed = result.join('');
  
  // Return original if compressed is longer (unless alwaysCompress)
  return alwaysCompress || compressed.length < str.length ? compressed : str;
}

// Decompress a compressed string
function decompressString(str) {
  let result = '';
  let i = 0;
  
  while (i < str.length) {
    const char = str[i];
    i++;
    
    // Get count (may be multiple digits)
    let countStr = '';
    while (i < str.length && /\d/.test(str[i])) {
      countStr += str[i];
      i++;
    }
    
    const count = countStr ? parseInt(countStr) : 1;
    result += char.repeat(count);
  }
  
  return result;
}

// Compress with custom encoding
function compressWithEncoding(str, encoding = 'rle') {
  switch (encoding) {
    case 'rle': // Run-length encoding
      return compressString(str);
      
    case 'count': // Just show unique chars with counts
      const counts = {};
      for (const char of str) {
        counts[char] = (counts[char] || 0) + 1;
      }
      return Object.entries(counts)
        .map(([char, count]) => `${char}${count}`)
        .join('');
      
    default:
      throw new Error(`Unknown encoding: ${encoding}`);
  }
}

// Comprehensive test suite
console.log('=== Basic Compression ===');
console.log(compressString('aaabbc'));
// 'a3b2c1'

console.log(compressString('aabcccccaaa'));
// 'a2b1c5a3'

console.log('\n=== Not Worth Compressing ===');
console.log(compressString('abc'));
// 'abc' (original returned)

console.log(compressString('abcd'));
// 'abcd'

console.log('\n=== Omit Single Counts ===');
console.log(compressString('aaabbc', { omitSingleCounts: true }));
// 'a3b2c' (no '1' after 'c')

console.log('\n=== Always Compress ===');
console.log(compressString('abc', { alwaysCompress: true }));
// 'a1b1c1'

console.log('\n=== Case Insensitive ===');
console.log(compressString('AAAaaaBBB', { caseInsensitive: true }));
// 'a6b3'

console.log('\n=== Empty and Single Character ===');
console.log(compressString(''));
// ''

console.log(compressString('a'));
// 'a'

console.log(compressString('aaaaa'));
// 'a5'

console.log('\n=== Decompression ===');
console.log(decompressString('a3b2c1'));
// 'aaabbc'

console.log(decompressString('a10b2'));
// 'aaaaaaaaaabb'

console.log('\n=== Long Runs ===');
console.log(compressString('a'.repeat(100)));
// 'a100'

console.log('\n=== Mixed Case ===');
console.log(compressString('AAAaaBBBbb'));
// 'A3a2B3b2'

console.log('\n=== Numbers and Special Chars ===');
console.log(compressString('111222!!!'));
// '132231!3'
```

### **Bonus: Advanced Compression**
```javascript
/**
 * Two-pass compression (chars then numbers)
 */
function advancedCompress(str) {
  const charCounts = {};
  
  for (const char of str) {
    charCounts[char] = (charCounts[char] || 0) + 1;
  }
  
  // Sort by frequency (optional optimization)
  const sorted = Object.entries(charCounts)
    .sort((a, b) => b[1] - a[1]);
  
  return sorted.map(([char, count]) => `${char}:${count}`).join(',');
}

// Test
console.log(advancedCompress('aaabbcccccc'));
// 'c:6,a:3,b:2'
```

**Interview Tips:**
- Run-length encoding (RLE) is the standard approach
- Use array.push() + join() instead of string concatenation for performance
- Check if compressed is actually shorter; return original if not
- Handle edge cases: empty string, single char, all unique chars
- Can omit count of 1 for better compression
- Common variation: case-insensitive compression
- Decompression should reverse the process exactly
- Used in: image compression, data transmission, file compression

</details>

42. Find the longest substring without repeating characters

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Sliding Window with Set**
```javascript
/**
 * Find longest substring without repeating characters
 * Time Complexity: O(n)
 * Space Complexity: O(min(n, m)) where m is charset size
 * 
 * @param {string} str - Input string
 * @returns {number} - Length of longest substring
 */
function lengthOfLongestSubstring(str) {
  const seen = new Set();
  let maxLength = 0;
  let left = 0;
  
  for (let right = 0; right < str.length; right++) {
    // Shrink window while we have duplicates
    while (seen.has(str[right])) {
      seen.delete(str[left]);
      left++;
    }
    
    seen.add(str[right]);
    maxLength = Math.max(maxLength, right - left + 1);
  }
  
  return maxLength;
}

// Test cases
console.log(lengthOfLongestSubstring('abcabcbb'));
// 3 ('abc')

console.log(lengthOfLongestSubstring('bbbbb'));
// 1 ('b')

console.log(lengthOfLongestSubstring('pwwkew'));
// 3 ('wke' or 'kew')

console.log(lengthOfLongestSubstring(''));
// 0
```

### **Approach 2: Sliding Window with HashMap (Optimized)**
```javascript
/**
 * Optimized sliding window using HashMap
 * Time Complexity: O(n)
 * Space Complexity: O(min(n, m))
 * 
 * Skip to character position + 1 when duplicate found
 */
function lengthOfLongestSubstring(str) {
  const charIndex = new Map(); // char -> last seen index
  let maxLength = 0;
  let left = 0;
  
  for (let right = 0; right < str.length; right++) {
    const char = str[right];
    
    // If char seen and within current window, move left pointer
    if (charIndex.has(char) && charIndex.get(char) >= left) {
      left = charIndex.get(char) + 1;
    }
    
    charIndex.set(char, right);
    maxLength = Math.max(maxLength, right - left + 1);
  }
  
  return maxLength;
}

// Test
console.log(lengthOfLongestSubstring('abcabcbb'));
// 3

console.log(lengthOfLongestSubstring('tmmzuxt'));
// 5 ('mzuxt')
```

### **Approach 3: Return Actual Substring**
```javascript
/**
 * Return the actual longest substring
 * Time Complexity: O(n)
 * Space Complexity: O(min(n, m))
 */
function longestSubstringWithoutRepeating(str) {
  const charIndex = new Map();
  let maxLength = 0;
  let maxStart = 0;
  let left = 0;
  
  for (let right = 0; right < str.length; right++) {
    const char = str[right];
    
    if (charIndex.has(char) && charIndex.get(char) >= left) {
      left = charIndex.get(char) + 1;
    }
    
    const currentLength = right - left + 1;
    if (currentLength > maxLength) {
      maxLength = currentLength;
      maxStart = left;
    }
    
    charIndex.set(char, right);
  }
  
  return str.substring(maxStart, maxStart + maxLength);
}

// Test
console.log(longestSubstringWithoutRepeating('abcabcbb'));
// 'abc'

console.log(longestSubstringWithoutRepeating('pwwkew'));
// 'wke' or 'kew'
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready longest substring finder
 * 
 * @param {string} str - Input string
 * @param {Object} options - Find options
 * @param {boolean} options.returnString - Return substring instead of length
 * @param {boolean} options.caseInsensitive - Ignore case for duplicates
 * @param {boolean} options.ignoreSpaces - Don't count spaces as characters
 * @param {Array} options.allowedRepeats - Characters that can repeat
 * @returns {number|string} - Length or actual substring
 */
function lengthOfLongestSubstring(str, options = {}) {
  const {
    returnString = false,
    caseInsensitive = false,
    ignoreSpaces = false,
    allowedRepeats = []
  } = options;
  
  // Validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  if (str.length === 0) {
    return returnString ? '' : 0;
  }
  
  // Normalize
  const workStr = caseInsensitive ? str.toLowerCase() : str;
  const allowedSet = new Set(allowedRepeats);
  
  const charIndex = new Map();
  let maxLength = 0;
  let maxStart = 0;
  let left = 0;
  
  for (let right = 0; right < workStr.length; right++) {
    const char = workStr[right];
    
    // Skip spaces if requested
    if (ignoreSpaces && char === ' ') {
      continue;
    }
    
    // Skip if character is allowed to repeat
    if (allowedSet.has(char)) {
      continue;
    }
    
    // Move left pointer if duplicate found
    if (charIndex.has(char) && charIndex.get(char) >= left) {
      left = charIndex.get(char) + 1;
    }
    
    const currentLength = right - left + 1;
    if (currentLength > maxLength) {
      maxLength = currentLength;
      maxStart = left;
    }
    
    charIndex.set(char, right);
  }
  
  if (returnString) {
    return str.substring(maxStart, maxStart + maxLength);
  }
  
  return maxLength;
}

// Find all substrings without repeating characters
function findAllUniqueSubstrings(str) {
  const result = [];
  const seen = new Set();
  
  for (let i = 0; i < str.length; i++) {
    const charSet = new Set();
    
    for (let j = i; j < str.length; j++) {
      if (charSet.has(str[j])) break;
      
      charSet.add(str[j]);
      const substring = str.substring(i, j + 1);
      
      if (!seen.has(substring)) {
        result.push(substring);
        seen.add(substring);
      }
    }
  }
  
  return result.sort((a, b) => b.length - a.length);
}

// Count of substrings with k distinct characters
function countSubstringsWithKDistinct(str, k) {
  function atMostK(k) {
    const count = new Map();
    let result = 0;
    let left = 0;
    
    for (let right = 0; right < str.length; right++) {
      count.set(str[right], (count.get(str[right]) || 0) + 1);
      
      while (count.size > k) {
        count.set(str[left], count.get(str[left]) - 1);
        if (count.get(str[left]) === 0) {
          count.delete(str[left]);
        }
        left++;
      }
      
      result += right - left + 1;
    }
    
    return result;
  }
  
  return atMostK(k) - atMostK(k - 1);
}

// Comprehensive test suite
console.log('=== Basic Tests ===');
console.log(lengthOfLongestSubstring('abcabcbb'));
// 3

console.log(lengthOfLongestSubstring('bbbbb'));
// 1

console.log(lengthOfLongestSubstring('pwwkew'));
// 3

console.log('\n=== Return Substring ===');
console.log(lengthOfLongestSubstring('abcabcbb', { returnString: true }));
// 'abc'

console.log(lengthOfLongestSubstring('dvdf', { returnString: true }));
// 'vdf'

console.log('\n=== Case Insensitive ===');
console.log(lengthOfLongestSubstring('AaBbCc', { caseInsensitive: true }));
// 2 (treats A and a as same)

console.log('\n=== Ignore Spaces ===');
console.log(lengthOfLongestSubstring('ab cd ef', { ignoreSpaces: true }));
// Counts only non-space characters

console.log('\n=== Empty and Single Character ===');
console.log(lengthOfLongestSubstring(''));
// 0

console.log(lengthOfLongestSubstring('a'));
// 1

console.log('\n=== All Unique ===');
console.log(lengthOfLongestSubstring('abcdef'));
// 6

console.log('\n=== Find All Unique Substrings ===');
console.log(findAllUniqueSubstrings('abc').slice(0, 5));
// ['abc', 'ab', 'bc', 'a', 'b']

console.log('\n=== Long String ===');
console.log(lengthOfLongestSubstring('abcdefghijklmnopqrstuvwxyz'));
// 26 (all lowercase letters)

console.log('\n=== With Numbers and Special Chars ===');
console.log(lengthOfLongestSubstring('a1b2c3d4'));
// 8 (all unique)

console.log(lengthOfLongestSubstring('a1b2a3'));
// 4 ('b2a3')
```

### **Bonus: K-Distinct Characters**
```javascript
/**
 * Longest substring with at most K distinct characters
 */
function longestSubstringKDistinct(str, k) {
  const count = new Map();
  let maxLength = 0;
  let left = 0;
  
  for (let right = 0; right < str.length; right++) {
    count.set(str[right], (count.get(str[right]) || 0) + 1);
    
    while (count.size > k) {
      count.set(str[left], count.get(str[left]) - 1);
      if (count.get(str[left]) === 0) {
        count.delete(str[left]);
      }
      left++;
    }
    
    maxLength = Math.max(maxLength, right - left + 1);
  }
  
  return maxLength;
}

// Test
console.log(longestSubstringKDistinct('eceba', 2));
// 3 ('ece')

console.log(longestSubstringKDistinct('aa', 1));
// 2 ('aa')
```

**Interview Tips:**
- Sliding window with Set/Map is optimal: O(n) time
- Two pointers: right expands, left contracts
- Use HashMap to track last seen index for optimization
- Can extend to: k distinct chars, with conditions, return all
- Common follow-ups: return substring, case-insensitive, k-distinct
- Classic sliding window problem pattern
- Alternative: brute force O(n²) checking all substrings

</details>

43. Check if string has balanced parentheses

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Stack**
```javascript
/**
 * Check if parentheses are balanced using stack
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {string} str - Input string
 * @returns {boolean} - True if balanced
 */
function isBalanced(str) {
  const stack = [];
  const pairs = {
    ')': '(',
    '}': '{',
    ']': '['
  };
  
  for (const char of str) {
    // Opening brackets
    if (char === '(' || char === '{' || char === '[') {
      stack.push(char);
    }
    // Closing brackets
    else if (char === ')' || char === '}' || char === ']') {
      if (stack.length === 0 || stack.pop() !== pairs[char]) {
        return false;
      }
    }
    // Ignore other characters
  }
  
  return stack.length === 0;
}

// Test cases
console.log(isBalanced('()'));
// true

console.log(isBalanced('()[]{}'));
// true

console.log(isBalanced('(]'));
// false

console.log(isBalanced('([)]'));
// false

console.log(isBalanced('{[]}'));
// true

console.log(isBalanced('((()))'));
// true

console.log(isBalanced('(()'));
// false
```

### **Approach 2: Counter Method (Single Bracket Type)**
```javascript
/**
 * Check balance for single bracket type using counter
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * Only works for one type of bracket
 */
function isBalanced(str) {
  let count = 0;
  
  for (const char of str) {
    if (char === '(') {
      count++;
    } else if (char === ')') {
      count--;
      if (count < 0) return false;
    }
  }
  
  return count === 0;
}

// Test
console.log(isBalanced('(())'));
// true

console.log(isBalanced('(()'));
// false
```

### **Approach 3: Only Check Parentheses in String**
```javascript
/**
 * Check balance ignoring non-bracket characters
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function isBalanced(str) {
  const stack = [];
  const opening = new Set(['(', '{', '[']);
  const closing = new Set([')', '}', ']']);
  const pairs = { ')': '(', '}': '{', ']': '[' };
  
  for (const char of str) {
    if (opening.has(char)) {
      stack.push(char);
    } else if (closing.has(char)) {
      if (stack.length === 0 || stack.pop() !== pairs[char]) {
        return false;
      }
    }
    // Ignore other characters
  }
  
  return stack.length === 0;
}

// Test
console.log(isBalanced('function test() { return [1, 2]; }'));
// true (ignores letters, numbers, etc.)

console.log(isBalanced('let x = (a + b) * [c + d];'));
// true
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready balanced bracket checker
 * 
 * @param {string} str - Input string
 * @param {Object} options - Check options
 * @param {Object} options.brackets - Custom bracket pairs (default: (){}[])
 * @param {boolean} options.strict - Only allow bracket characters (default: false)
 * @param {boolean} options.details - Return detailed error info (default: false)
 * @returns {boolean|Object} - Result or detailed info
 */
function isBalanced(str, options = {}) {
  const {
    brackets = { ')': '(', '}': '{', ']': '[' },
    strict = false,
    details = false
  } = options;
  
  // Validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  const stack = [];
  const opening = new Set(Object.values(brackets));
  const closing = new Set(Object.keys(brackets));
  
  for (let i = 0; i < str.length; i++) {
    const char = str[i];
    
    if (opening.has(char)) {
      stack.push({ char, index: i });
    } else if (closing.has(char)) {
      if (stack.length === 0) {
        if (details) {
          return {
            balanced: false,
            error: 'Unexpected closing bracket',
            char,
            index: i
          };
        }
        return false;
      }
      
      const last = stack.pop();
      if (last.char !== brackets[char]) {
        if (details) {
          return {
            balanced: false,
            error: 'Mismatched brackets',
            expected: brackets[char],
            found: last.char,
            openIndex: last.index,
            closeIndex: i
          };
        }
        return false;
      }
    } else if (strict && char !== ' ') {
      if (details) {
        return {
          balanced: false,
          error: 'Invalid character in strict mode',
          char,
          index: i
        };
      }
      return false;
    }
  }
  
  if (stack.length > 0) {
    if (details) {
      return {
        balanced: false,
        error: 'Unclosed brackets',
        unclosed: stack.map(item => ({ char: item.char, index: item.index }))
      };
    }
    return false;
  }
  
  return details ? { balanced: true } : true;
}

// Check and get matching pairs
function getMatchingPairs(str) {
  const stack = [];
  const pairs = [];
  const bracketPairs = { ')': '(', '}': '{', ']': '[' };
  const opening = new Set(Object.values(bracketPairs));
  const closing = new Set(Object.keys(bracketPairs));
  
  for (let i = 0; i < str.length; i++) {
    const char = str[i];
    
    if (opening.has(char)) {
      stack.push({ char, index: i });
    } else if (closing.has(char)) {
      if (stack.length > 0 && stack[stack.length - 1].char === bracketPairs[char]) {
        const openBracket = stack.pop();
        pairs.push({
          open: openBracket.index,
          close: i,
          type: char
        });
      }
    }
  }
  
  return pairs;
}

// Get all unbalanced brackets
function getUnbalancedBrackets(str) {
  const stack = [];
  const unbalanced = [];
  const bracketPairs = { ')': '(', '}': '{', ']': '[' };
  const opening = new Set(Object.values(bracketPairs));
  const closing = new Set(Object.keys(bracketPairs));
  
  for (let i = 0; i < str.length; i++) {
    const char = str[i];
    
    if (opening.has(char)) {
      stack.push({ char, index: i });
    } else if (closing.has(char)) {
      if (stack.length === 0 || stack[stack.length - 1].char !== bracketPairs[char]) {
        unbalanced.push({ char, index: i, type: 'unmatched-close' });
      } else {
        stack.pop();
      }
    }
  }
  
  // Remaining in stack are unclosed
  stack.forEach(item => {
    unbalanced.push({ char: item.char, index: item.index, type: 'unclosed' });
  });
  
  return unbalanced;
}

// Comprehensive test suite
console.log('=== Basic Tests ===');
console.log(isBalanced('()'));
// true

console.log(isBalanced('()[]{}'));
// true

console.log(isBalanced('(]'));
// false

console.log('\n=== Nested Brackets ===');
console.log(isBalanced('([{}])'));
// true

console.log(isBalanced('({[]})'));
// true

console.log(isBalanced('({[}])'));
// false (mismatched)

console.log('\n=== With Text ===');
console.log(isBalanced('function test() { return [1, 2, 3]; }'));
// true

console.log(isBalanced('let x = (a + b) * {c + d];'));
// false

console.log('\n=== Edge Cases ===');
console.log(isBalanced(''));
// true (empty is balanced)

console.log(isBalanced('abc'));
// true (no brackets)

console.log(isBalanced('('));
// false (unclosed)

console.log(isBalanced(')'));
// false (unexpected closing)

console.log('\n=== Detailed Error Info ===');
console.log(isBalanced('({[}])', { details: true }));
/*
{
  balanced: false,
  error: 'Mismatched brackets',
  expected: '[',
  found: '{',
  openIndex: 1,
  closeIndex: 3
}
*/

console.log(isBalanced('((())', { details: true }));
/*
{
  balanced: false,
  error: 'Unclosed brackets',
  unclosed: [{ char: '(', index: 0 }]
}
*/

console.log('\n=== Custom Brackets ===');
console.log(isBalanced('<html><body></body></html>', {
  brackets: { '>': '<' }
}));
// true

console.log('\n=== Strict Mode ===');
console.log(isBalanced('(())', { strict: true }));
// true

console.log(isBalanced('(a)', { strict: true }));
// false (non-bracket character)

console.log('\n=== Get Matching Pairs ===');
console.log(getMatchingPairs('(a [b] {c})'));
/*
[
  { open: 3, close: 5, type: ']' },
  { open: 7, close: 9, type: '}' },
  { open: 0, close: 10, type: ')' }
]
*/

console.log('\n=== Get Unbalanced Brackets ===');
console.log(getUnbalancedBrackets('((a)[b}'));
/*
[
  { char: '}', index: 6, type: 'unmatched-close' },
  { char: '(', index: 0, type: 'unclosed' }
]
*/

console.log('\n=== Multiple Same Type ===');
console.log(isBalanced('(((())))'));
// true

console.log(isBalanced('(((()))'));
// false

console.log('\n=== Complex Expressions ===');
console.log(isBalanced('if (x > 0) { arr[i] = {a: 1}; }'));
// true

console.log(isBalanced('let obj = { arr: [1, 2, (3 + 4)] };'));
// true
```

### **Bonus: Advanced Validations**
```javascript
/**
 * Validate bracket balance with maximum nesting depth
 */
function isBalancedWithDepth(str, maxDepth = Infinity) {
  const stack = [];
  const pairs = { ')': '(', '}': '{', ']': '[' };
  
  for (const char of str) {
    if (char === '(' || char === '{' || char === '[') {
      stack.push(char);
      if (stack.length > maxDepth) {
        return false;
      }
    } else if (char === ')' || char === '}' || char === ']') {
      if (stack.length === 0 || stack.pop() !== pairs[char]) {
        return false;
      }
    }
  }
  
  return stack.length === 0;
}

// Test
console.log(isBalancedWithDepth('((()))', 3));
// true (max depth is 3)

console.log(isBalancedWithDepth('((()))', 2));
// false (exceeds max depth)

/**
 * Fix unbalanced brackets by adding missing ones
 */
function fixBrackets(str) {
  const stack = [];
  const pairs = { ')': '(', '}': '{', ']': '[' };
  const reverse = { '(': ')', '{': '}', '[': ']' };
  let result = str;
  
  for (const char of str) {
    if (char === '(' || char === '{' || char === '[') {
      stack.push(char);
    } else if (char === ')' || char === '}' || char === ']') {
      if (stack.length > 0 && stack[stack.length - 1] === pairs[char]) {
        stack.pop();
      }
    }
  }
  
  // Add missing closing brackets
  while (stack.length > 0) {
    result += reverse[stack.pop()];
  }
  
  return result;
}

// Test
console.log(fixBrackets('((a)'));
// '((a))'

console.log(fixBrackets('{[}'));
// '{[}]' (partial fix)
```

**Interview Tips:**
- Stack is the canonical solution: O(n) time, O(n) space
- Counter works only for single bracket type
- Remember to check stack is empty at the end (unclosed brackets)
- Handle edge cases: empty string, no brackets, only opening/closing
- Can extend to: custom brackets, maximum depth, error details
- Common follow-ups: return positions, fix unbalanced, check specific types
- Used in: code editors, compilers, expression validators
- Alternative: recursion (less efficient), regex (limited)

</details>

44. Reverse words in a sentence

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Built-in Methods**
```javascript
/**
 * Reverse words in a sentence
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {string} str - Input sentence
 * @returns {string} - Reversed sentence
 */
function reverseWords(str) {
  return str.split(' ').reverse().join(' ');
}

// Test cases
console.log(reverseWords('Hello World'));
// 'World Hello'

console.log(reverseWords('The quick brown fox'));
// 'fox brown quick The'

console.log(reverseWords('JavaScript is awesome'));
// 'awesome is JavaScript'
```

### **Approach 2: Without Built-in Reverse**
```javascript
/**
 * Reverse words manually
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function reverseWords(str) {
  const words = str.split(' ');
  const reversed = [];
  
  for (let i = words.length - 1; i >= 0; i--) {
    reversed.push(words[i]);
  }
  
  return reversed.join(' ');
}

// Test
console.log(reverseWords('Hello World'));
// 'World Hello'
```

### **Approach 3: Handle Multiple Spaces**
```javascript
/**
 * Reverse words handling multiple spaces
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function reverseWords(str) {
  // Filter out empty strings from multiple spaces
  const words = str.split(' ').filter(word => word.length > 0);
  return words.reverse().join(' ');
}

// Test
console.log(reverseWords('  Hello   World  '));
// 'World Hello'

console.log(reverseWords('a  b    c'));
// 'c b a'
```

### **Approach 4: Two-Pointer In-Place (Array of Characters)**
```javascript
/**
 * Reverse words in-place (if using array)
 * Time Complexity: O(n)
 * Space Complexity: O(1) excluding input conversion
 */
function reverseWords(str) {
  const chars = str.split('');
  
  // Helper to reverse a portion of array
  function reverse(arr, start, end) {
    while (start < end) {
      [arr[start], arr[end]] = [arr[end], arr[start]];
      start++;
      end--;
    }
  }
  
  // Reverse entire string
  reverse(chars, 0, chars.length - 1);
  
  // Reverse each word
  let start = 0;
  for (let i = 0; i <= chars.length; i++) {
    if (i === chars.length || chars[i] === ' ') {
      reverse(chars, start, i - 1);
      start = i + 1;
    }
  }
  
  return chars.join('');
}

// Test
console.log(reverseWords('Hello World'));
// 'World Hello'

console.log(reverseWords('The quick brown fox'));
// 'fox brown quick The'
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready word reverser
 * 
 * @param {string} str - Input string
 * @param {Object} options - Reverse options
 * @param {boolean} options.preserveSpaces - Keep original spacing (default: false)
 * @param {boolean} options.trim - Trim leading/trailing spaces (default: true)
 * @param {RegExp} options.delimiter - Word delimiter pattern (default: /\s+/)
 * @param {boolean} options.reverseChars - Also reverse characters in each word (default: false)
 * @returns {string} - Reversed string
 */
function reverseWords(str, options = {}) {
  const {
    preserveSpaces = false,
    trim = true,
    delimiter = /\s+/,
    reverseChars = false
  } = options;
  
  // Validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  if (str.length === 0) {
    return str;
  }
  
  let workStr = trim ? str.trim() : str;
  
  // Preserve original spacing pattern
  if (preserveSpaces) {
    const words = [];
    const spaces = [];
    let currentWord = '';
    let currentSpace = '';
    
    for (const char of workStr) {
      if (char === ' ') {
        if (currentWord) {
          words.push(currentWord);
          currentWord = '';
        }
        currentSpace += char;
      } else {
        if (currentSpace) {
          spaces.push(currentSpace);
          currentSpace = '';
        }
        currentWord += char;
      }
    }
    
    if (currentWord) words.push(currentWord);
    if (currentSpace) spaces.push(currentSpace);
    
    // Reverse words
    words.reverse();
    
    // Reconstruct with original spacing
    let result = '';
    for (let i = 0; i < words.length; i++) {
      result += words[i];
      if (i < spaces.length) {
        result += spaces[i];
      }
    }
    
    return result;
  }
  
  // Standard word reversal
  const words = workStr.split(delimiter).filter(word => word.length > 0);
  
  if (reverseChars) {
    const reversedWords = words.map(word => 
      word.split('').reverse().join('')
    );
    return reversedWords.reverse().join(' ');
  }
  
  return words.reverse().join(' ');
}

// Reverse only specific parts
function reverseWordsPartial(str, start, end) {
  const words = str.split(' ');
  const startIdx = Math.max(0, start);
  const endIdx = Math.min(words.length, end);
  
  const before = words.slice(0, startIdx);
  const toReverse = words.slice(startIdx, endIdx);
  const after = words.slice(endIdx);
  
  return [...before, ...toReverse.reverse(), ...after].join(' ');
}

// Reverse with punctuation handling
function reverseWordsPreservePunctuation(str) {
  // Extract words (letters only)
  const words = str.match(/\b\w+\b/g) || [];
  
  // Reverse words
  const reversed = words.reverse();
  
  // Replace words in original string
  let index = 0;
  return str.replace(/\b\w+\b/g, () => reversed[index++]);
}

// Comprehensive test suite
console.log('=== Basic Reversal ===');
console.log(reverseWords('Hello World'));
// 'World Hello'

console.log(reverseWords('The quick brown fox'));
// 'fox brown quick The'

console.log('\n=== Multiple Spaces ===');
console.log(reverseWords('  Hello   World  '));
// 'World Hello'

console.log(reverseWords('a  b    c', { trim: false }));
// 'c b a'

console.log('\n=== Preserve Spaces ===');
console.log(reverseWords('a  b    c', { preserveSpaces: true }));
// 'c  b    a' (keeps original spacing pattern)

console.log('\n=== No Trim ===');
console.log(reverseWords('  Hello World  ', { trim: false }));
// '  World Hello  '

console.log('\n=== Custom Delimiter ===');
console.log(reverseWords('apple,banana,cherry', { delimiter: /,/ }));
// 'cherry banana apple'

console.log(reverseWords('one-two-three', { delimiter: /-/ }));
// 'three two one'

console.log('\n=== Reverse Characters Too ===');
console.log(reverseWords('Hello World', { reverseChars: true }));
// 'dlroW olleH'

console.log('\n=== Edge Cases ===');
console.log(reverseWords(''));
// ''

console.log(reverseWords('SingleWord'));
// 'SingleWord'

console.log(reverseWords('   '));
// ''

console.log('\n=== Partial Reversal ===');
console.log(reverseWordsPartial('one two three four five', 1, 4));
// 'one four three two five'

console.log('\n=== With Punctuation ===');
console.log(reverseWordsPreservePunctuation('Hello, World!'));
// 'World, Hello!'

console.log(reverseWordsPreservePunctuation('Hi there, how are you?'));
// 'you are how there, Hi?'

console.log('\n=== Multiple Words Same ===');
console.log(reverseWords('test test test'));
// 'test test test'

console.log('\n=== Numbers ===');
console.log(reverseWords('1 2 3 4 5'));
// '5 4 3 2 1'

console.log('\n=== Mixed Content ===');
console.log(reverseWords('The price is $100.50 today'));
// 'today $100.50 is price The'
```

### **Bonus: Advanced Word Operations**
```javascript
/**
 * Reverse alternate words
 */
function reverseAlternateWords(str) {
  return str.split(' ').map((word, index) => {
    return index % 2 === 1 ? word.split('').reverse().join('') : word;
  }).join(' ');
}

// Test
console.log(reverseAlternateWords('Hello World From JavaScript'));
// 'Hello dlroW From tpircSavaJ'

/**
 * Reverse words maintaining case pattern
 */
function reverseWordsKeepCase(str) {
  const words = str.split(' ');
  const reversed = words.reverse();
  
  return words.map((original, idx) => {
    const reversed_word = reversed[idx];
    return reversed_word.split('').map((char, i) => {
      if (i < original.length) {
        return original[i] === original[i].toUpperCase() 
          ? char.toUpperCase() 
          : char.toLowerCase();
      }
      return char;
    }).join('');
  }).join(' ');
}

// Test
console.log(reverseWordsKeepCase('Hello World'));
// 'Dlrow Olleh' (W->O is capital, h->d is lowercase, etc.)
```

**Interview Tips:**
- split().reverse().join() is simplest: O(n) time
- Handle multiple spaces with filter or regex
- Two-pointer in-place approach for character arrays
- Common variations: preserve spacing, reverse chars too, partial reversal
- Watch for edge cases: empty, single word, all spaces
- Can use regex for complex delimiters
- Follow-up: reverse only alternate words, preserve punctuation
- Used in: text processing, natural language tasks

</details>

45. Implement string search (indexOf) without using built-in methods

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Naive String Matching**
```javascript
/**
 * Implement indexOf without built-in methods
 * Time Complexity: O(n * m) where n = text length, m = pattern length
 * Space Complexity: O(1)
 * 
 * @param {string} text - Text to search in
 * @param {string} pattern - Pattern to search for
 * @returns {number} - Index of first occurrence or -1
 */
function indexOf(text, pattern) {
  if (pattern.length === 0) return 0;
  if (pattern.length > text.length) return -1;
  
  for (let i = 0; i <= text.length - pattern.length; i++) {
    let found = true;
    
    for (let j = 0; j < pattern.length; j++) {
      if (text[i + j] !== pattern[j]) {
        found = false;
        break;
      }
    }
    
    if (found) return i;
  }
  
  return -1;
}

// Test cases
console.log(indexOf('hello world', 'world'));
// 6

console.log(indexOf('hello world', 'hello'));
// 0

console.log(indexOf('hello world', 'xyz'));
// -1

console.log(indexOf('aaaaa', 'aa'));
// 0

console.log(indexOf('', 'test'));
// -1

console.log(indexOf('test', ''));
// 0
```

### **Approach 2: Find All Occurrences**
```javascript
/**
 * Find all occurrences of pattern
 * Time Complexity: O(n * m)
 * Space Complexity: O(k) where k is number of matches
 */
function findAll(text, pattern) {
  const indices = [];
  
  if (pattern.length === 0) return indices;
  
  for (let i = 0; i <= text.length - pattern.length; i++) {
    let match = true;
    
    for (let j = 0; j < pattern.length; j++) {
      if (text[i + j] !== pattern[j]) {
        match = false;
        break;
      }
    }
    
    if (match) {
      indices.push(i);
    }
  }
  
  return indices;
}

// Test
console.log(findAll('ababa', 'aba'));
// [0, 2]

console.log(findAll('aaaa', 'aa'));
// [0, 1, 2]
```

### **Approach 3: KMP Algorithm (Optimal)**
```javascript
/**
 * Knuth-Morris-Pratt algorithm for string matching
 * Time Complexity: O(n + m)
 * Space Complexity: O(m)
 * 
 * More efficient for large texts and patterns
 */
function indexOf(text, pattern) {
  if (pattern.length === 0) return 0;
  if (pattern.length > text.length) return -1;
  
  // Build LPS (Longest Proper Prefix which is also Suffix) array
  const lps = buildLPS(pattern);
  
  let i = 0; // text pointer
  let j = 0; // pattern pointer
  
  while (i < text.length) {
    if (text[i] === pattern[j]) {
      i++;
      j++;
      
      if (j === pattern.length) {
        return i - j; // Found match
      }
    } else {
      if (j > 0) {
        j = lps[j - 1]; // Use LPS to skip characters
      } else {
        i++;
      }
    }
  }
  
  return -1;
}

function buildLPS(pattern) {
  const lps = new Array(pattern.length).fill(0);
  let length = 0;
  let i = 1;
  
  while (i < pattern.length) {
    if (pattern[i] === pattern[length]) {
      length++;
      lps[i] = length;
      i++;
    } else {
      if (length > 0) {
        length = lps[length - 1];
      } else {
        lps[i] = 0;
        i++;
      }
    }
  }
  
  return lps;
}

// Test
console.log(indexOf('abcabcabc', 'abcabc'));
// 0

console.log(indexOf('aaaaaab', 'aaab'));
// 3
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready string search implementation
 * 
 * @param {string} text - Text to search in
 * @param {string} pattern - Pattern to search for
 * @param {Object} options - Search options
 * @param {number} options.startIndex - Start search from this index (default: 0)
 * @param {boolean} options.caseInsensitive - Ignore case (default: false)
 * @param {boolean} options.findAll - Return all occurrences (default: false)
 * @param {boolean} options.useKMP - Use KMP algorithm (default: true for large inputs)
 * @returns {number|Array} - Index or array of indices
 */
function indexOf(text, pattern, options = {}) {
  const {
    startIndex = 0,
    caseInsensitive = false,
    findAll = false,
    useKMP = pattern.length > 10
  } = options;
  
  // Validation
  if (typeof text !== 'string' || typeof pattern !== 'string') {
    throw new TypeError('Both arguments must be strings');
  }
  
  if (pattern.length === 0) {
    return findAll ? [] : startIndex;
  }
  
  if (startIndex < 0 || startIndex >= text.length) {
    return findAll ? [] : -1;
  }
  
  // Normalize case if needed
  const searchText = caseInsensitive ? text.toLowerCase() : text;
  const searchPattern = caseInsensitive ? pattern.toLowerCase() : pattern;
  
  if (findAll) {
    return findAllOccurrences(searchText, searchPattern, startIndex);
  }
  
  if (useKMP) {
    return kmpSearch(searchText, searchPattern, startIndex);
  }
  
  return naiveSearch(searchText, searchPattern, startIndex);
}

function naiveSearch(text, pattern, start) {
  for (let i = start; i <= text.length - pattern.length; i++) {
    let match = true;
    
    for (let j = 0; j < pattern.length; j++) {
      if (text[i + j] !== pattern[j]) {
        match = false;
        break;
      }
    }
    
    if (match) return i;
  }
  
  return -1;
}

function findAllOccurrences(text, pattern, start) {
  const indices = [];
  
  for (let i = start; i <= text.length - pattern.length; i++) {
    let match = true;
    
    for (let j = 0; j < pattern.length; j++) {
      if (text[i + j] !== pattern[j]) {
        match = false;
        break;
      }
    }
    
    if (match) {
      indices.push(i);
    }
  }
  
  return indices;
}

function kmpSearch(text, pattern, start) {
  const lps = buildLPS(pattern);
  let i = start;
  let j = 0;
  
  while (i < text.length) {
    if (text[i] === pattern[j]) {
      i++;
      j++;
      
      if (j === pattern.length) {
        return i - j;
      }
    } else {
      if (j > 0) {
        j = lps[j - 1];
      } else {
        i++;
      }
    }
  }
  
  return -1;
}

function buildLPS(pattern) {
  const lps = new Array(pattern.length).fill(0);
  let length = 0;
  let i = 1;
  
  while (i < pattern.length) {
    if (pattern[i] === pattern[length]) {
      length++;
      lps[i] = length;
      i++;
    } else {
      if (length > 0) {
        length = lps[length - 1];
      } else {
        lps[i] = 0;
        i++;
      }
    }
  }
  
  return lps;
}

// Implement lastIndexOf (search from end)
function lastIndexOf(text, pattern) {
  if (pattern.length === 0) return text.length;
  if (pattern.length > text.length) return -1;
  
  for (let i = text.length - pattern.length; i >= 0; i--) {
    let match = true;
    
    for (let j = 0; j < pattern.length; j++) {
      if (text[i + j] !== pattern[j]) {
        match = false;
        break;
      }
    }
    
    if (match) return i;
  }
  
  return -1;
}

// Count occurrences
function countOccurrences(text, pattern) {
  const indices = indexOf(text, pattern, { findAll: true });
  return indices.length;
}

// Comprehensive test suite
console.log('=== Basic Search ===');
console.log(indexOf('hello world', 'world'));
// 6

console.log(indexOf('hello world', 'hello'));
// 0

console.log(indexOf('hello world', 'xyz'));
// -1

console.log('\n=== Find All ===');
console.log(indexOf('ababa', 'aba', { findAll: true }));
// [0, 2]

console.log(indexOf('aaaa', 'aa', { findAll: true }));
// [0, 1, 2]

console.log('\n=== Case Insensitive ===');
console.log(indexOf('Hello World', 'WORLD', { caseInsensitive: true }));
// 6

console.log(indexOf('JavaScript', 'script', { caseInsensitive: true }));
// 4

console.log('\n=== Start Index ===');
console.log(indexOf('abcabc', 'abc', { startIndex: 1 }));
// 3

console.log(indexOf('hello hello', 'hello', { startIndex: 1 }));
// 6

console.log('\n=== Edge Cases ===');
console.log(indexOf('', 'test'));
// -1

console.log(indexOf('test', ''));
// 0

console.log(indexOf('abc', 'abcd'));
// -1 (pattern longer than text)

console.log('\n=== Last Index Of ===');
console.log(lastIndexOf('hello hello', 'hello'));
// 6

console.log(lastIndexOf('abcabc', 'abc'));
// 3

console.log('\n=== Count Occurrences ===');
console.log(countOccurrences('ababa', 'aba'));
// 2

console.log(countOccurrences('aaaa', 'aa'));
// 3

console.log('\n=== KMP Performance ===');
const longText = 'a'.repeat(10000) + 'b';
const pattern = 'a'.repeat(100) + 'b';
console.log(indexOf(longText, pattern, { useKMP: true }));
// 9900

console.log('\n=== Overlapping Matches ===');
console.log(indexOf('aaa', 'aa', { findAll: true }));
// [0, 1]
```

### **Bonus: Other String Matching Algorithms**
```javascript
/**
 * Rabin-Karp algorithm (rolling hash)
 */
function rabinKarp(text, pattern) {
  const d = 256; // Number of characters in alphabet
  const q = 101; // A prime number
  
  const m = pattern.length;
  const n = text.length;
  let patternHash = 0;
  let textHash = 0;
  let h = 1;
  
  // h = d^(m-1) % q
  for (let i = 0; i < m - 1; i++) {
    h = (h * d) % q;
  }
  
  // Calculate initial hash values
  for (let i = 0; i < m; i++) {
    patternHash = (d * patternHash + pattern.charCodeAt(i)) % q;
    textHash = (d * textHash + text.charCodeAt(i)) % q;
  }
  
  // Slide pattern over text
  for (let i = 0; i <= n - m; i++) {
    if (patternHash === textHash) {
      // Check character by character
      let match = true;
      for (let j = 0; j < m; j++) {
        if (text[i + j] !== pattern[j]) {
          match = false;
          break;
        }
      }
      if (match) return i;
    }
    
    // Calculate hash for next window
    if (i < n - m) {
      textHash = (d * (textHash - text.charCodeAt(i) * h) + text.charCodeAt(i + m)) % q;
      
      if (textHash < 0) {
        textHash += q;
      }
    }
  }
  
  return -1;
}

// Test
console.log(rabinKarp('hello world', 'world'));
// 6
```

**Interview Tips:**
- Naive approach: O(n*m) - check every position
- KMP algorithm: O(n+m) - optimal, uses LPS array
- Rabin-Karp: O(n+m) average - uses rolling hash
- Handle edge cases: empty strings, pattern longer than text
- Common optimizations: early termination, skip obvious mismatches
- Follow-ups: find all, case-insensitive, lastIndexOf, count
- Used in: text editors, search engines, DNA sequencing
- Built-in indexOf uses optimized C++ Boyer-Moore-Horspool variant

</details>

46. Count vowels and consonants in a string

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Simple Loop**
```javascript
/**
 * Count vowels and consonants
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 * 
 * @param {string} str - Input string
 * @returns {Object} - {vowels, consonants}
 */
function countVowelsConsonants(str) {
  const vowels = new Set(['a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U']);
  let vowelCount = 0;
  let consonantCount = 0;
  
  for (const char of str) {
    if (vowels.has(char)) {
      vowelCount++;
    } else if ((char >= 'a' && char <= 'z') || (char >= 'A' && char <= 'Z')) {
      consonantCount++;
    }
  }
  
  return { vowels: vowelCount, consonants: consonantCount };
}

// Test cases
console.log(countVowelsConsonants('Hello World'));
// { vowels: 3, consonants: 7 }

console.log(countVowelsConsonants('JavaScript'));
// { vowels: 3, consonants: 7 }

console.log(countVowelsConsonants('aeiou'));
// { vowels: 5, consonants: 0 }

console.log(countVowelsConsonants('bcdfg'));
// { vowels: 0, consonants: 5 }
```

### **Approach 2: Using Regex**
```javascript
/**
 * Count using regular expressions
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
function countVowelsConsonants(str) {
  const vowels = str.match(/[aeiou]/gi) || [];
  const consonants = str.match(/[bcdfghjklmnpqrstvwxyz]/gi) || [];
  
  return {
    vowels: vowels.length,
    consonants: consonants.length
  };
}

// Test
console.log(countVowelsConsonants('Hello World'));
// { vowels: 3, consonants: 7 }
```

### **Approach 3: Detailed Count (Each Vowel/Consonant)**
```javascript
/**
 * Count frequency of each vowel and consonant
 * Time Complexity: O(n)
 * Space Complexity: O(1) - fixed size map
 */
function countVowelsConsonantsDetailed(str) {
  const vowels = new Set(['a', 'e', 'i', 'o', 'u']);
  const vowelCounts = {};
  const consonantCounts = {};
  
  for (const char of str.toLowerCase()) {
    if (vowels.has(char)) {
      vowelCounts[char] = (vowelCounts[char] || 0) + 1;
    } else if (char >= 'a' && char <= 'z') {
      consonantCounts[char] = (consonantCounts[char] || 0) + 1;
    }
  }
  
  return {
    vowels: vowelCounts,
    consonants: consonantCounts,
    vowelTotal: Object.values(vowelCounts).reduce((a, b) => a + b, 0),
    consonantTotal: Object.values(consonantCounts).reduce((a, b) => a + b, 0)
  };
}

// Test
console.log(countVowelsConsonantsDetailed('Hello World'));
/*
{
  vowels: { e: 1, o: 2 },
  consonants: { h: 1, l: 3, w: 1, r: 1, d: 1 },
  vowelTotal: 3,
  consonantTotal: 7
}
*/
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready vowel and consonant counter
 * 
 * @param {string} str - Input string
 * @param {Object} options - Count options
 * @param {boolean} options.includeY - Treat 'y' as vowel (default: false)
 * @param {boolean} options.detailed - Return frequency of each letter (default: false)
 * @param {boolean} options.includeNumbers - Count numbers separately (default: false)
 * @param {boolean} options.includeSpecial - Count special chars separately (default: false)
 * @returns {Object} - Count results
 */
function countVowelsConsonants(str, options = {}) {
  const {
    includeY = false,
    detailed = false,
    includeNumbers = false,
    includeSpecial = false
  } = options;
  
  // Validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  const vowelSet = includeY 
    ? new Set(['a', 'e', 'i', 'o', 'u', 'y'])
    : new Set(['a', 'e', 'i', 'o', 'u']);
  
  const counts = {
    vowels: 0,
    consonants: 0
  };
  
  if (detailed) {
    counts.vowelFreq = {};
    counts.consonantFreq = {};
  }
  
  if (includeNumbers) {
    counts.numbers = 0;
    if (detailed) counts.numberFreq = {};
  }
  
  if (includeSpecial) {
    counts.special = 0;
    if (detailed) counts.specialFreq = {};
  }
  
  for (const char of str) {
    const lower = char.toLowerCase();
    
    if (vowelSet.has(lower)) {
      counts.vowels++;
      if (detailed) {
        counts.vowelFreq[lower] = (counts.vowelFreq[lower] || 0) + 1;
      }
    } else if (lower >= 'a' && lower <= 'z') {
      counts.consonants++;
      if (detailed) {
        counts.consonantFreq[lower] = (counts.consonantFreq[lower] || 0) + 1;
      }
    } else if (includeNumbers && char >= '0' && char <= '9') {
      counts.numbers++;
      if (detailed) {
        counts.numberFreq[char] = (counts.numberFreq[char] || 0) + 1;
      }
    } else if (includeSpecial && char !== ' ') {
      counts.special++;
      if (detailed) {
        counts.specialFreq[char] = (counts.specialFreq[char] || 0) + 1;
      }
    }
  }
  
  return counts;
}

// Get vowel/consonant ratio
function getVowelConsonantRatio(str) {
  const { vowels, consonants } = countVowelsConsonants(str);
  
  if (consonants === 0) return Infinity;
  
  return (vowels / consonants).toFixed(2);
}

// Check if string is vowel-heavy or consonant-heavy
function classifyString(str) {
  const { vowels, consonants } = countVowelsConsonants(str);
  const ratio = vowels / consonants;
  
  if (ratio > 1) return 'vowel-heavy';
  if (ratio < 0.5) return 'consonant-heavy';
  return 'balanced';
}

// Find longest vowel/consonant sequence
function longestSequence(str, type = 'vowel') {
  const vowels = new Set(['a', 'e', 'i', 'o', 'u', 'A', 'E', 'I', 'O', 'U']);
  let maxLength = 0;
  let currentLength = 0;
  let maxStart = 0;
  let currentStart = 0;
  
  for (let i = 0; i < str.length; i++) {
    const isVowel = vowels.has(str[i]);
    const isTarget = (type === 'vowel' && isVowel) || (type === 'consonant' && !isVowel && /[a-zA-Z]/.test(str[i]));
    
    if (isTarget) {
      if (currentLength === 0) {
        currentStart = i;
      }
      currentLength++;
      
      if (currentLength > maxLength) {
        maxLength = currentLength;
        maxStart = currentStart;
      }
    } else {
      currentLength = 0;
    }
  }
  
  return {
    length: maxLength,
    sequence: str.substring(maxStart, maxStart + maxLength)
  };
}

// Comprehensive test suite
console.log('=== Basic Count ===');
console.log(countVowelsConsonants('Hello World'));
// { vowels: 3, consonants: 7 }

console.log(countVowelsConsonants('JavaScript'));
// { vowels: 3, consonants: 7 }

console.log('\n=== Include Y as Vowel ===');
console.log(countVowelsConsonants('Happy Birthday', { includeY: true }));
// Y counted as vowel

console.log('\n=== Detailed Frequency ===');
console.log(countVowelsConsonants('Hello', { detailed: true }));
/*
{
  vowels: 2,
  consonants: 3,
  vowelFreq: { e: 1, o: 1 },
  consonantFreq: { h: 1, l: 2 }
}
*/

console.log('\n=== Include Numbers ===');
console.log(countVowelsConsonants('Hello123', { includeNumbers: true }));
// { vowels: 2, consonants: 3, numbers: 3 }

console.log('\n=== Include Special Characters ===');
console.log(countVowelsConsonants('Hello!@#', {
  includeSpecial: true,
  detailed: true
}));

console.log('\n=== Edge Cases ===');
console.log(countVowelsConsonants(''));
// { vowels: 0, consonants: 0 }

console.log(countVowelsConsonants('aeiou'));
// { vowels: 5, consonants: 0 }

console.log(countVowelsConsonants('bcdfg'));
// { vowels: 0, consonants: 5 }

console.log(countVowelsConsonants('123!@#'));
// { vowels: 0, consonants: 0 }

console.log('\n=== Vowel/Consonant Ratio ===');
console.log(getVowelConsonantRatio('Hello World'));
// '0.43'

console.log(getVowelConsonantRatio('aeiou'));
// 'Infinity'

console.log('\n=== Classify String ===');
console.log(classifyString('aeiou bcdfg'));
// 'balanced'

console.log(classifyString('programming'));
// 'consonant-heavy'

console.log(classifyString('beautiful'));
// 'vowel-heavy'

console.log('\n=== Longest Sequence ===');
console.log(longestSequence('programming', 'consonant'));
// { length: 5, sequence: 'prgrmm' }

console.log(longestSequence('beautiful', 'vowel'));
// { length: 3, sequence: 'eau' }

console.log('\n=== Case Insensitive ===');
console.log(countVowelsConsonants('HELLO world'));
// { vowels: 3, consonants: 7 }

console.log('\n=== Mixed Content ===');
console.log(countVowelsConsonants('The price is $100.50', {
  includeNumbers: true,
  includeSpecial: true
}));
```

### **Bonus: Advanced Analysis**
```javascript
/**
 * Calculate readability score based on vowel/consonant distribution
 */
function calculateReadability(str) {
  const { vowels, consonants } = countVowelsConsonants(str);
  const total = vowels + consonants;
  
  if (total === 0) return 0;
  
  const vowelRatio = vowels / total;
  
  // Optimal vowel ratio is around 0.38-0.42 for English
  const deviation = Math.abs(vowelRatio - 0.40);
  const score = Math.max(0, 100 - deviation * 200);
  
  return {
    score: Math.round(score),
    vowelRatio: vowelRatio.toFixed(2),
    rating: score > 80 ? 'excellent' : score > 60 ? 'good' : score > 40 ? 'fair' : 'poor'
  };
}

// Test
console.log(calculateReadability('The quick brown fox jumps over the lazy dog'));
// Analyzes vowel distribution quality
```

**Interview Tips:**
- Simple Set/loop approach is O(n) time, O(1) space
- Regex approach is concise but slightly slower
- Include 'y' as vowel is language-dependent (ask interviewer)
- Can extend to: frequency analysis, longest sequences, ratios
- Handle edge cases: empty string, no vowels/consonants, special chars
- Common follow-ups: most/least common, alternating pattern, balanced
- Used in: linguistics, readability analysis, text classification
- Can optimize with lookup table or bitmask for character classification

</details>

47. Find all permutations of a string

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Recursive Backtracking**
```javascript
/**
 * Find all permutations using recursion
 * Time Complexity: O(n! * n)
 * Space Complexity: O(n! * n) for result + O(n) recursion stack
 * 
 * @param {string} str - Input string
 * @returns {Array} - Array of all permutations
 */
function permutations(str) {
  const result = [];
  
  function backtrack(current, remaining) {
    if (remaining.length === 0) {
      result.push(current);
      return;
    }
    
    for (let i = 0; i < remaining.length; i++) {
      const char = remaining[i];
      const newRemaining = remaining.slice(0, i) + remaining.slice(i + 1);
      backtrack(current + char, newRemaining);
    }
  }
  
  backtrack('', str);
  return result;
}

// Test cases
console.log(permutations('abc'));
// ['abc', 'acb', 'bac', 'bca', 'cab', 'cba']

console.log(permutations('ab'));
// ['ab', 'ba']

console.log(permutations('a'));
// ['a']

console.log(permutations(''));
// ['']
```

### **Approach 2: Using Array Swap**
```javascript
/**
 * Generate permutations using in-place swapping
 * Time Complexity: O(n!)
 * Space Complexity: O(n) recursion stack
 */
function permutations(str) {
  const result = [];
  const chars = str.split('');
  
  function backtrack(index) {
    if (index === chars.length) {
      result.push(chars.join(''));
      return;
    }
    
    for (let i = index; i < chars.length; i++) {
      // Swap
      [chars[index], chars[i]] = [chars[i], chars[index]];
      
      // Recurse
      backtrack(index + 1);
      
      // Backtrack (swap back)
      [chars[index], chars[i]] = [chars[i], chars[index]];
    }
  }
  
  backtrack(0);
  return result;
}

// Test
console.log(permutations('abc'));
// ['abc', 'acb', 'bac', 'bca', 'cba', 'cab']
```

### **Approach 3: Handle Duplicate Characters**
```javascript
/**
 * Generate unique permutations (no duplicates)
 * Time Complexity: O(n!)
 * Space Complexity: O(n!)
 */
function uniquePermutations(str) {
  const result = [];
  const chars = str.split('').sort(); // Sort for easier duplicate handling
  const used = new Array(chars.length).fill(false);
  
  function backtrack(current) {
    if (current.length === chars.length) {
      result.push(current);
      return;
    }
    
    for (let i = 0; i < chars.length; i++) {
      // Skip if used or duplicate
      if (used[i]) continue;
      if (i > 0 && chars[i] === chars[i - 1] && !used[i - 1]) continue;
      
      used[i] = true;
      backtrack(current + chars[i]);
      used[i] = false;
    }
  }
  
  backtrack('');
  return result;
}

// Test
console.log(uniquePermutations('aab'));
// ['aab', 'aba', 'baa'] (no duplicates like 'aab' appearing twice)

console.log(uniquePermutations('aaa'));
// ['aaa'] (only one unique permutation)
```

### **Approach 4: Iterative (Heap's Algorithm)**
```javascript
/**
 * Heap's algorithm for generating permutations iteratively
 * Time Complexity: O(n!)
 * Space Complexity: O(n)
 */
function permutations(str) {
  const result = [];
  const arr = str.split('');
  const n = arr.length;
  const c = new Array(n).fill(0);
  
  result.push(arr.join(''));
  
  let i = 0;
  while (i < n) {
    if (c[i] < i) {
      if (i % 2 === 0) {
        [arr[0], arr[i]] = [arr[i], arr[0]];
      } else {
        [arr[c[i]], arr[i]] = [arr[i], arr[c[i]]];
      }
      
      result.push(arr.join(''));
      c[i]++;
      i = 0;
    } else {
      c[i] = 0;
      i++;
    }
  }
  
  return result;
}

// Test
console.log(permutations('abc'));
// All 6 permutations
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready permutation generator
 * 
 * @param {string} str - Input string
 * @param {Object} options - Generation options
 * @param {boolean} options.unique - Remove duplicate permutations (default: true)
 * @param {number} options.length - Generate permutations of specific length (default: str.length)
 * @param {boolean} options.caseInsensitive - Treat a and A as same (default: false)
 * @param {Function} options.filter - Filter function for permutations
 * @returns {Array} - Array of permutations
 */
function permutations(str, options = {}) {
  const {
    unique = true,
    length = str.length,
    caseInsensitive = false,
    filter = null
  } = options;
  
  // Validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  if (str.length === 0) {
    return [''];
  }
  
  const workStr = caseInsensitive ? str.toLowerCase() : str;
  const chars = workStr.split('');
  
  if (unique) {
    chars.sort();
  }
  
  const result = [];
  const used = new Array(chars.length).fill(false);
  
  function backtrack(current) {
    if (current.length === length) {
      if (!filter || filter(current)) {
        result.push(current);
      }
      return;
    }
    
    for (let i = 0; i < chars.length; i++) {
      if (used[i]) continue;
      
      // Skip duplicates if unique mode
      if (unique && i > 0 && chars[i] === chars[i - 1] && !used[i - 1]) {
        continue;
      }
      
      used[i] = true;
      backtrack(current + chars[i]);
      used[i] = false;
    }
  }
  
  backtrack('');
  return result;
}

// Generate k-length permutations (nPk)
function permutationsOfLength(str, k) {
  return permutations(str, { length: k });
}

// Count permutations without generating them
function countPermutations(str, unique = true) {
  if (!unique) {
    return factorial(str.length);
  }
  
  // Count with duplicates: n! / (n1! * n2! * ... * nk!)
  const freq = {};
  for (const char of str) {
    freq[char] = (freq[char] || 0) + 1;
  }
  
  let count = factorial(str.length);
  for (const f of Object.values(freq)) {
    count /= factorial(f);
  }
  
  return count;
}

function factorial(n) {
  if (n <= 1) return 1;
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}

// Generator function for memory efficiency
function* permutationGenerator(str) {
  const chars = str.split('');
  const n = chars.length;
  const c = new Array(n).fill(0);
  
  yield chars.join('');
  
  let i = 0;
  while (i < n) {
    if (c[i] < i) {
      if (i % 2 === 0) {
        [chars[0], chars[i]] = [chars[i], chars[0]];
      } else {
        [chars[c[i]], chars[i]] = [chars[i], chars[c[i]]];
      }
      
      yield chars.join('');
      c[i]++;
      i = 0;
    } else {
      c[i] = 0;
      i++;
    }
  }
}

// Get nth permutation without generating all
function nthPermutation(str, n) {
  const chars = str.split('').sort();
  const result = [];
  const available = [...chars];
  const len = chars.length;
  
  // Adjust for 0-based index
  n = n - 1;
  
  for (let i = 0; i < len; i++) {
    const fact = factorial(len - 1 - i);
    const index = Math.floor(n / fact);
    
    result.push(available[index]);
    available.splice(index, 1);
    
    n %= fact;
  }
  
  return result.join('');
}

// Comprehensive test suite
console.log('=== Basic Permutations ===');
console.log(permutations('abc'));
// ['abc', 'acb', 'bac', 'bca', 'cab', 'cba']

console.log(permutations('ab'));
// ['ab', 'ba']

console.log('\n=== Unique Permutations (With Duplicates) ===');
console.log(permutations('aab'));
// ['aab', 'aba', 'baa']

console.log(permutations('aaa'));
// ['aaa']

console.log('\n=== Non-Unique Mode ===');
console.log(permutations('aa', { unique: false }));
// ['aa', 'aa'] (duplicates allowed)

console.log('\n=== Specific Length (k-permutations) ===');
console.log(permutations('abc', { length: 2 }));
// ['ab', 'ac', 'ba', 'bc', 'ca', 'cb']

console.log(permutations('abcd', { length: 2 }));
// 12 permutations (4P2 = 12)

console.log('\n=== With Filter ===');
console.log(permutations('abc', {
  filter: perm => perm.startsWith('a')
}));
// ['abc', 'acb'] (only permutations starting with 'a')

console.log('\n=== Case Insensitive ===');
console.log(permutations('Abc', { caseInsensitive: true }));
// All lowercase permutations

console.log('\n=== Edge Cases ===');
console.log(permutations(''));
// ['']

console.log(permutations('a'));
// ['a']

console.log('\n=== Count Permutations ===');
console.log(countPermutations('abc'));
// 6 (3! = 6)

console.log(countPermutations('aab'));
// 3 (3! / 2! = 3)

console.log(countPermutations('aabb'));
// 6 (4! / (2! * 2!) = 6)

console.log('\n=== Nth Permutation ===');
console.log(nthPermutation('abc', 1));
// 'abc' (1st permutation)

console.log(nthPermutation('abc', 3));
// 'bac' (3rd permutation)

console.log(nthPermutation('abc', 6));
// 'cba' (6th/last permutation)

console.log('\n=== Generator (Memory Efficient) ===');
const gen = permutationGenerator('ab');
console.log(gen.next().value); // 'ab'
console.log(gen.next().value); // 'ba'
console.log(gen.next().done);  // true

console.log('\n=== Large String Count ===');
console.log(countPermutations('abcd'));
// 24 (4! = 24)

console.log(countPermutations('aabcd'));
// 60 (5! / 2! = 60)

console.log('\n=== Longer Strings ===');
console.log(permutations('abcd').length);
// 24

console.log(permutations('abc', { length: 1 }));
// ['a', 'b', 'c']
```

### **Bonus: Related Problems**
```javascript
/**
 * Check if one string is a permutation of another
 */
function isPermutation(str1, str2) {
  if (str1.length !== str2.length) return false;
  
  const freq = {};
  
  for (const char of str1) {
    freq[char] = (freq[char] || 0) + 1;
  }
  
  for (const char of str2) {
    if (!freq[char]) return false;
    freq[char]--;
  }
  
  return true;
}

// Test
console.log(isPermutation('abc', 'bca'));
// true

console.log(isPermutation('abc', 'def'));
// false

/**
 * Next lexicographic permutation
 */
function nextPermutation(str) {
  const chars = str.split('');
  let i = chars.length - 2;
  
  // Find first decreasing element from right
  while (i >= 0 && chars[i] >= chars[i + 1]) {
    i--;
  }
  
  if (i < 0) {
    return null; // Already last permutation
  }
  
  // Find smallest element greater than chars[i] to the right
  let j = chars.length - 1;
  while (chars[j] <= chars[i]) {
    j--;
  }
  
  // Swap
  [chars[i], chars[j]] = [chars[j], chars[i]];
  
  // Reverse the suffix
  let left = i + 1;
  let right = chars.length - 1;
  while (left < right) {
    [chars[left], chars[right]] = [chars[right], chars[left]];
    left++;
    right--;
  }
  
  return chars.join('');
}

// Test
console.log(nextPermutation('abc'));
// 'acb'

console.log(nextPermutation('acb'));
// 'bac'

console.log(nextPermutation('cba'));
// null (last permutation)

/**
 * Find all anagrams of a pattern in a string
 */
function findAnagrams(text, pattern) {
  const result = [];
  const patternFreq = {};
  const windowFreq = {};
  
  for (const char of pattern) {
    patternFreq[char] = (patternFreq[char] || 0) + 1;
  }
  
  let left = 0;
  let matched = 0;
  
  for (let right = 0; right < text.length; right++) {
    const char = text[right];
    
    if (char in patternFreq) {
      windowFreq[char] = (windowFreq[char] || 0) + 1;
      if (windowFreq[char] === patternFreq[char]) {
        matched++;
      }
    }
    
    if (right - left + 1 > pattern.length) {
      const leftChar = text[left];
      if (leftChar in patternFreq) {
        if (windowFreq[leftChar] === patternFreq[leftChar]) {
          matched--;
        }
        windowFreq[leftChar]--;
      }
      left++;
    }
    
    if (matched === Object.keys(patternFreq).length) {
      result.push(left);
    }
  }
  
  return result;
}

// Test
console.log(findAnagrams('cbaebabacd', 'abc'));
// [0, 6] (indices where anagrams start)
```

**Interview Tips:**
- Recursive backtracking is most intuitive: O(n!)
- Heap's algorithm is efficient for iterative generation
- For duplicates, sort first and skip identical adjacent elements
- n! grows very fast: 10! = 3.6M, be careful with large strings
- Can optimize with generator functions for memory efficiency
- Common variations: k-length permutations, nth permutation, next permutation
- Follow-ups: count without generating, check if permutation, find anagrams
- Used in: combinatorial problems, cryptography, puzzles

</details>

48. Implement a function to truncate a string with ellipsis

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Simple Truncation**
```javascript
/**
 * Truncate string with ellipsis
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 * 
 * @param {string} str - Input string
 * @param {number} maxLength - Maximum length including ellipsis
 * @returns {string} - Truncated string
 */
function truncate(str, maxLength) {
  if (str.length <= maxLength) {
    return str;
  }
  
  return str.slice(0, maxLength - 3) + '...';
}

// Test cases
console.log(truncate('Hello World', 8));
// 'Hello...'

console.log(truncate('JavaScript', 20));
// 'JavaScript' (no truncation needed)

console.log(truncate('This is a long string', 10));
// 'This is...'
```

### **Approach 2: Truncate at Word Boundary**
```javascript
/**
 * Truncate at word boundary to avoid cutting words
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
function truncate(str, maxLength) {
  if (str.length <= maxLength) {
    return str;
  }
  
  // Find last space before maxLength
  let truncated = str.slice(0, maxLength - 3);
  const lastSpace = truncated.lastIndexOf(' ');
  
  if (lastSpace > 0) {
    truncated = truncated.slice(0, lastSpace);
  }
  
  return truncated + '...';
}

// Test
console.log(truncate('The quick brown fox', 15));
// 'The quick...' (doesn't cut 'brown')

console.log(truncate('Hello World', 8));
// 'Hello...'
```

### **Approach 3: Custom Ellipsis**
```javascript
/**
 * Truncate with custom ellipsis character(s)
 * Time Complexity: O(1)
 * Space Complexity: O(1)
 */
function truncate(str, maxLength, ellipsis = '...') {
  if (str.length <= maxLength) {
    return str;
  }
  
  return str.slice(0, maxLength - ellipsis.length) + ellipsis;
}

// Test
console.log(truncate('Hello World', 8, '…'));
// 'Hello W…'

console.log(truncate('Hello World', 10, ' [...]'));
// 'Hell [...]'
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready string truncation
 * 
 * @param {string} str - Input string
 * @param {number} maxLength - Maximum length
 * @param {Object} options - Truncation options
 * @param {string} options.ellipsis - Ellipsis string (default: '...')
 * @param {boolean} options.wordBoundary - Break at word boundary (default: true)
 * @param {boolean} options.html - Preserve HTML tags (default: false)
 * @param {string} options.position - Where to truncate: 'end', 'middle', 'start' (default: 'end')
 * @returns {string} - Truncated string
 */
function truncate(str, maxLength, options = {}) {
  const {
    ellipsis = '...',
    wordBoundary = true,
    html = false,
    position = 'end'
  } = options;
  
  // Validation
  if (typeof str !== 'string') {
    throw new TypeError('First argument must be a string');
  }
  
  if (typeof maxLength !== 'number' || maxLength < 0) {
    throw new TypeError('maxLength must be a non-negative number');
  }
  
  // Remove HTML tags if not preserving
  let workStr = str;
  if (html) {
    const textOnly = str.replace(/<[^>]*>/g, '');
    if (textOnly.length <= maxLength) {
      return str;
    }
  } else {
    workStr = str;
  }
  
  if (workStr.length <= maxLength) {
    return str;
  }
  
  // Different truncation positions
  switch (position) {
    case 'start':
      return truncateStart(workStr, maxLength, ellipsis, wordBoundary);
    
    case 'middle':
      return truncateMiddle(workStr, maxLength, ellipsis);
    
    case 'end':
    default:
      return truncateEnd(workStr, maxLength, ellipsis, wordBoundary);
  }
}

function truncateEnd(str, maxLength, ellipsis, wordBoundary) {
  let truncated = str.slice(0, maxLength - ellipsis.length);
  
  if (wordBoundary) {
    const lastSpace = truncated.lastIndexOf(' ');
    if (lastSpace > 0) {
      truncated = truncated.slice(0, lastSpace);
    }
  }
  
  return truncated.trim() + ellipsis;
}

function truncateStart(str, maxLength, ellipsis, wordBoundary) {
  let truncated = str.slice(str.length - (maxLength - ellipsis.length));
  
  if (wordBoundary) {
    const firstSpace = truncated.indexOf(' ');
    if (firstSpace >= 0 && firstSpace < truncated.length - 1) {
      truncated = truncated.slice(firstSpace + 1);
    }
  }
  
  return ellipsis + truncated.trim();
}

function truncateMiddle(str, maxLength, ellipsis) {
  const charsToShow = maxLength - ellipsis.length;
  const frontChars = Math.ceil(charsToShow / 2);
  const backChars = Math.floor(charsToShow / 2);
  
  return str.slice(0, frontChars) + ellipsis + str.slice(str.length - backChars);
}

// Truncate with character count display
function truncateWithCount(str, maxLength, ellipsis = '...') {
  if (str.length <= maxLength) {
    return str;
  }
  
  const remaining = str.length - maxLength;
  return str.slice(0, maxLength - ellipsis.length) + ellipsis + ` (+${remaining})`;
}

// Truncate multiline text
function truncateLines(str, maxLines, ellipsis = '...') {
  const lines = str.split('\n');
  
  if (lines.length <= maxLines) {
    return str;
  }
  
  return lines.slice(0, maxLines).join('\n') + ellipsis;
}

// Smart truncate (tries to keep sentence structure)
function smartTruncate(str, maxLength) {
  if (str.length <= maxLength) {
    return str;
  }
  
  // Try to truncate at sentence boundary
  const truncated = str.slice(0, maxLength - 3);
  const lastPeriod = truncated.lastIndexOf('.');
  const lastQuestion = truncated.lastIndexOf('?');
  const lastExclamation = truncated.lastIndexOf('!');
  
  const sentenceEnd = Math.max(lastPeriod, lastQuestion, lastExclamation);
  
  if (sentenceEnd > maxLength * 0.5) {
    return str.slice(0, sentenceEnd + 1);
  }
  
  // Fall back to word boundary
  const lastSpace = truncated.lastIndexOf(' ');
  if (lastSpace > 0) {
    return truncated.slice(0, lastSpace) + '...';
  }
  
  return truncated + '...';
}

// Comprehensive test suite
console.log('=== Basic Truncation ===');
console.log(truncate('Hello World', 8));
// 'Hello...'

console.log(truncate('JavaScript Programming', 15));
// 'JavaScript...'

console.log('\n=== No Truncation Needed ===');
console.log(truncate('Hello', 10));
// 'Hello'

console.log('\n=== Word Boundary ===');
console.log(truncate('The quick brown fox', 15, { wordBoundary: true }));
// 'The quick...'

console.log(truncate('The quick brown fox', 15, { wordBoundary: false }));
// 'The quick b...'

console.log('\n=== Custom Ellipsis ===');
console.log(truncate('Hello World', 8, { ellipsis: '…' }));
// 'Hello W…'

console.log(truncate('Hello World', 10, { ellipsis: ' [more]' }));
// 'Hel [more]'

console.log('\n=== Start Truncation ===');
console.log(truncate('This is a long file path', 15, { position: 'start' }));
// '...file path'

console.log('\n=== Middle Truncation ===');
console.log(truncate('VeryLongFileName.txt', 15, { position: 'middle' }));
// 'VeryLo...me.txt'

console.log('\n=== With Count ===');
console.log(truncateWithCount('This is a very long string', 10));
// 'This is... (+16)'

console.log('\n=== Multiline Truncation ===');
const multiline = 'Line 1\nLine 2\nLine 3\nLine 4\nLine 5';
console.log(truncateLines(multiline, 3));
// 'Line 1\nLine 2\nLine 3...'

console.log('\n=== Smart Truncation ===');
console.log(smartTruncate('This is sentence one. This is sentence two.', 25));
// 'This is sentence one.'

console.log(smartTruncate('No punctuation here at all', 15));
// 'No punctuation...'

console.log('\n=== Edge Cases ===');
console.log(truncate('', 10));
// ''

console.log(truncate('Hi', 10));
// 'Hi'

console.log(truncate('Test', 3));
// 'Test' (can't truncate to less than ellipsis length)

console.log('\n=== HTML Content ===');
console.log(truncate('<p>Hello World</p>', 10, { html: true }));
// '<p>Hello World</p>' (preserves if text content fits)

console.log('\n=== Long Words ===');
console.log(truncate('Supercalifragilisticexpialidocious', 15));
// 'Supercalif...'

console.log('\n=== Unicode Characters ===');
console.log(truncate('Hello 👋 World 🌍', 10));
// Handles emoji properly
```

### **Bonus: Advanced Truncation**
```javascript
/**
 * Truncate with tooltip showing full text
 */
function truncateWithTooltip(str, maxLength) {
  if (str.length <= maxLength) {
    return str;
  }
  
  const truncated = str.slice(0, maxLength - 3) + '...';
  return {
    display: truncated,
    full: str,
    html: `<span title="${str}">${truncated}</span>`
  };
}

// Test
console.log(truncateWithTooltip('This is a long string', 10));
/*
{
  display: 'This is...',
  full: 'This is a long string',
  html: '<span title="This is a long string">This is...</span>'
}
*/

/**
 * Truncate preserving code formatting
 */
function truncateCode(code, maxLength) {
  if (code.length <= maxLength) {
    return code;
  }
  
  const lines = code.split('\n');
  let truncated = '';
  
  for (const line of lines) {
    if (truncated.length + line.length + 4 <= maxLength) {
      truncated += line + '\n';
    } else {
      truncated += '...';
      break;
    }
  }
  
  return truncated.trim();
}
```

**Interview Tips:**
- Simple slice + '...' is O(1) and usually sufficient
- Word boundary prevents cutting words in half
- Consider position: end (default), middle (file names), start (paths)
- Custom ellipsis for different contexts: '…', ' [more]', ' (more)'
- For HTML, strip tags before measuring or preserve them
- Smart truncation at sentence/paragraph boundaries
- Common use cases: previews, lists, mobile displays, tooltips
- Be careful with Unicode/emoji when counting characters

</details>

49. Check if one string is a rotation of another

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Concatenation Method**
```javascript
/**
 * Check if str2 is rotation of str1
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {string} str1 - First string
 * @param {string} str2 - Second string
 * @returns {boolean} - True if str2 is rotation of str1
 */
function isRotation(str1, str2) {
  // Must be same length and not empty
  if (str1.length !== str2.length || str1.length === 0) {
    return false;
  }
  
  // If str2 is rotation, it will be substring of str1+str1
  return (str1 + str1).includes(str2);
}

// Test cases
console.log(isRotation('waterbottle', 'erbottlewat'));
// true

console.log(isRotation('hello', 'lohel'));
// true

console.log(isRotation('hello', 'world'));
// false

console.log(isRotation('abc', 'cab'));
// true

console.log(isRotation('abc', 'acb'));
// false (permutation, not rotation)
```

### **Approach 2: Find Rotation Point**
```javascript
/**
 * Check rotation by finding the rotation point
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
function isRotation(str1, str2) {
  if (str1.length !== str2.length || str1.length === 0) {
    return false;
  }
  
  if (str1 === str2) {
    return true;
  }
  
  // Try each possible rotation point
  for (let i = 0; i < str1.length; i++) {
    let match = true;
    
    for (let j = 0; j < str1.length; j++) {
      if (str1[j] !== str2[(i + j) % str2.length]) {
        match = false;
        break;
      }
    }
    
    if (match) return true;
  }
  
  return false;
}

// Test
console.log(isRotation('abcde', 'cdeab'));
// true
```

### **Approach 3: Using KMP Pattern Matching**
```javascript
/**
 * Check rotation using KMP algorithm
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function isRotation(str1, str2) {
  if (str1.length !== str2.length || str1.length === 0) {
    return false;
  }
  
  return kmpSearch(str1 + str1, str2) !== -1;
}

function kmpSearch(text, pattern) {
  const lps = buildLPS(pattern);
  let i = 0, j = 0;
  
  while (i < text.length) {
    if (text[i] === pattern[j]) {
      i++;
      j++;
      if (j === pattern.length) {
        return i - j;
      }
    } else {
      if (j > 0) {
        j = lps[j - 1];
      } else {
        i++;
      }
    }
  }
  
  return -1;
}

function buildLPS(pattern) {
  const lps = new Array(pattern.length).fill(0);
  let len = 0;
  let i = 1;
  
  while (i < pattern.length) {
    if (pattern[i] === pattern[len]) {
      len++;
      lps[i] = len;
      i++;
    } else {
      if (len > 0) {
        len = lps[len - 1];
      } else {
        lps[i] = 0;
        i++;
      }
    }
  }
  
  return lps;
}

// Test
console.log(isRotation('abcde', 'cdeab'));
// true
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready rotation checker
 * 
 * @param {string} str1 - First string
 * @param {string} str2 - Second string
 * @param {Object} options - Check options
 * @param {boolean} options.caseInsensitive - Ignore case (default: false)
 * @param {boolean} options.findRotationPoint - Return rotation point if found (default: false)
 * @param {boolean} options.allowEmpty - Allow empty strings (default: false)
 * @returns {boolean|number} - True/false or rotation point index
 */
function isRotation(str1, str2, options = {}) {
  const {
    caseInsensitive = false,
    findRotationPoint = false,
    allowEmpty = false
  } = options;
  
  // Validation
  if (typeof str1 !== 'string' || typeof str2 !== 'string') {
    throw new TypeError('Both arguments must be strings');
  }
  
  if (!allowEmpty && (str1.length === 0 || str2.length === 0)) {
    return findRotationPoint ? -1 : false;
  }
  
  if (str1.length !== str2.length) {
    return findRotationPoint ? -1 : false;
  }
  
  // Normalize case if needed
  const s1 = caseInsensitive ? str1.toLowerCase() : str1;
  const s2 = caseInsensitive ? str2.toLowerCase() : str2;
  
  if (s1 === s2) {
    return findRotationPoint ? 0 : true;
  }
  
  // Find rotation point if requested
  if (findRotationPoint) {
    for (let i = 0; i < s1.length; i++) {
      let match = true;
      
      for (let j = 0; j < s1.length; j++) {
        if (s1[j] !== s2[(i + j) % s2.length]) {
          match = false;
          break;
        }
      }
      
      if (match) return i;
    }
    
    return -1;
  }
  
  // Standard concatenation check
  return (s1 + s1).includes(s2);
}

// Get the rotation point
function getRotationPoint(str1, str2) {
  return isRotation(str1, str2, { findRotationPoint: true });
}

// Get all possible rotations
function getAllRotations(str) {
  const rotations = [];
  
  for (let i = 0; i < str.length; i++) {
    rotations.push(str.slice(i) + str.slice(0, i));
  }
  
  return rotations;
}

// Rotate string by k positions
function rotateString(str, k) {
  if (str.length === 0) return str;
  
  k = k % str.length;
  if (k < 0) k += str.length;
  
  return str.slice(k) + str.slice(0, k);
}

// Find minimum rotation (lexicographically)
function minRotation(str) {
  let min = str;
  
  for (let i = 1; i < str.length; i++) {
    const rotation = str.slice(i) + str.slice(0, i);
    if (rotation < min) {
      min = rotation;
    }
  }
  
  return min;
}

// Comprehensive test suite
console.log('=== Basic Rotation Check ===');
console.log(isRotation('waterbottle', 'erbottlewat'));
// true

console.log(isRotation('hello', 'lohel'));
// true

console.log(isRotation('hello', 'world'));
// false

console.log('\n=== Same String ===');
console.log(isRotation('abc', 'abc'));
// true (0 rotation)

console.log('\n=== Not a Rotation ===');
console.log(isRotation('abc', 'acb'));
// false (this is a permutation, not rotation)

console.log('\n=== Different Lengths ===');
console.log(isRotation('abc', 'abcd'));
// false

console.log('\n=== Empty Strings ===');
console.log(isRotation('', ''));
// false (default behavior)

console.log(isRotation('', '', { allowEmpty: true }));
// true

console.log('\n=== Case Insensitive ===');
console.log(isRotation('Hello', 'LOHel', { caseInsensitive: true }));
// true

console.log('\n=== Find Rotation Point ===');
console.log(getRotationPoint('abcde', 'cdeab'));
// 2 (rotated by 2 positions to the left)

console.log(getRotationPoint('hello', 'lohel'));
// 3

console.log(getRotationPoint('abc', 'xyz'));
// -1 (not a rotation)

console.log('\n=== Get All Rotations ===');
console.log(getAllRotations('abc'));
// ['abc', 'bca', 'cab']

console.log(getAllRotations('abcd'));
// ['abcd', 'bcda', 'cdab', 'dabc']

console.log('\n=== Rotate String ===');
console.log(rotateString('abcde', 2));
// 'cdeab'

console.log(rotateString('abcde', -1));
// 'eabcd' (rotate right by 1)

console.log(rotateString('hello', 7));
// 'lohel' (wraps around: 7 % 5 = 2)

console.log('\n=== Minimum Rotation ===');
console.log(minRotation('cab'));
// 'abc'

console.log(minRotation('dbca'));
// 'acdb'

console.log('\n=== Single Character ===');
console.log(isRotation('a', 'a'));
// true

console.log('\n=== Repeating Characters ===');
console.log(isRotation('aaaa', 'aaaa'));
// true

console.log(isRotation('aabb', 'abba'));
// true

console.log(isRotation('aabb', 'baab'));
// true

console.log('\n=== Long Strings ===');
const long1 = 'a'.repeat(1000) + 'b' + 'a'.repeat(999);
const long2 = 'b' + 'a'.repeat(1999);
console.log(isRotation(long1, long2));
// true
```

### **Bonus: Related Problems**
```javascript
/**
 * Count distinct rotations
 */
function countDistinctRotations(str) {
  const seen = new Set();
  
  for (let i = 0; i < str.length; i++) {
    const rotation = str.slice(i) + str.slice(0, i);
    seen.add(rotation);
  }
  
  return seen.size;
}

// Test
console.log(countDistinctRotations('aaa'));
// 1

console.log(countDistinctRotations('aab'));
// 3

/**
 * Check if array is rotation of another array
 */
function isArrayRotation(arr1, arr2) {
  if (arr1.length !== arr2.length || arr1.length === 0) {
    return false;
  }
  
  const doubled = [...arr1, ...arr1];
  
  // Check if arr2 appears as subarray in doubled
  outer: for (let i = 0; i < arr1.length; i++) {
    for (let j = 0; j < arr2.length; j++) {
      if (doubled[i + j] !== arr2[j]) {
        continue outer;
      }
    }
    return true;
  }
  
  return false;
}

// Test
console.log(isArrayRotation([1, 2, 3, 4], [3, 4, 1, 2]));
// true

console.log(isArrayRotation([1, 2, 3], [2, 1, 3]));
// false
```

**Interview Tips:**
- Concatenation trick (str1+str1).includes(str2) is elegant: O(n)
- Must check: same length, not empty (unless allowed)
- Rotation != permutation (abc -> bca is rotation, abc -> acb is not)
- Works because rotation is just shifting: if rotated, will appear in doubled string
- Can extend to: find rotation point, rotate by k, minimum rotation
- Common follow-ups: arrays, circular lists, rotation count
- Used in: string matching, circular buffers, genome sequencing
- Alternative: manual rotation checking O(n²) but O(1) space

</details>

50. Remove all adjacent duplicates from a string

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Stack**
```javascript
/**
 * Remove adjacent duplicates using stack
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 * 
 * @param {string} str - Input string
 * @returns {string} - String with adjacent duplicates removed
 */
function removeAdjacentDuplicates(str) {
  const stack = [];
  
  for (const char of str) {
    if (stack.length > 0 && stack[stack.length - 1] === char) {
      stack.pop();
    } else {
      stack.push(char);
    }
  }
  
  return stack.join('');
}

// Test cases
console.log(removeAdjacentDuplicates('abbaca'));
// 'ca' (remove 'bb', then 'aa')

console.log(removeAdjacentDuplicates('azxxzy'));
// 'ay' (remove 'xx', then 'zz')

console.log(removeAdjacentDuplicates('aabbcc'));
// '' (all removed)

console.log(removeAdjacentDuplicates('abc'));
// 'abc' (no duplicates)
```

### **Approach 2: Iterative Removal**
```javascript
/**
 * Remove duplicates iteratively
 * Time Complexity: O(n²) worst case
 * Space Complexity: O(n)
 */
function removeAdjacentDuplicates(str) {
  let prev = '';
  let current = str;
  
  while (prev !== current) {
    prev = current;
    
    for (let i = 0; i < current.length - 1; i++) {
      if (current[i] === current[i + 1]) {
        current = current.slice(0, i) + current.slice(i + 2);
        break;
      }
    }
  }
  
  return current;
}

// Test
console.log(removeAdjacentDuplicates('abbaca'));
// 'ca'
```

### **Approach 3: Remove K Adjacent Duplicates**
```javascript
/**
 * Remove k adjacent duplicate characters
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function removeKAdjacentDuplicates(str, k) {
  const stack = []; // stores [char, count]
  
  for (const char of str) {
    if (stack.length > 0 && stack[stack.length - 1][0] === char) {
      stack[stack.length - 1][1]++;
      
      if (stack[stack.length - 1][1] === k) {
        stack.pop();
      }
    } else {
      stack.push([char, 1]);
    }
  }
  
  return stack.map(([char, count]) => char.repeat(count)).join('');
}

// Test
console.log(removeKAdjacentDuplicates('deeedbbcccbdaa', 3));
// 'aa' (removes 'eee', 'ccc', 'bbb' -> 'ddbdaa' -> 'aa')

console.log(removeKAdjacentDuplicates('pbbcggttciiippooaais', 2));
// 'ps'
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready adjacent duplicate remover
 * 
 * @param {string} str - Input string
 * @param {Object} options - Removal options
 * @param {number} options.k - Number of adjacent duplicates to remove (default: 2)
 * @param {boolean} options.caseInsensitive - Ignore case (default: false)
 * @param {boolean} options.recursive - Keep removing until no more (default: true)
 * @param {Function} options.shouldRemove - Custom function to determine removal
 * @returns {string} - String with duplicates removed
 */
function removeAdjacentDuplicates(str, options = {}) {
  const {
    k = 2,
    caseInsensitive = false,
    recursive = true,
    shouldRemove = null
  } = options;
  
  // Validation
  if (typeof str !== 'string') {
    throw new TypeError('Input must be a string');
  }
  
  if (k < 2) {
    throw new RangeError('k must be at least 2');
  }
  
  const workStr = caseInsensitive ? str.toLowerCase() : str;
  const stack = []; // [char, count, originalChars]
  
  for (let i = 0; i < workStr.length; i++) {
    const char = workStr[i];
    const originalChar = str[i];
    
    if (stack.length > 0 && stack[stack.length - 1][0] === char) {
      const top = stack[stack.length - 1];
      top[1]++;
      top[2].push(originalChar);
      
      // Check if should remove
      const remove = shouldRemove 
        ? shouldRemove(char, top[1])
        : top[1] === k;
      
      if (remove) {
        stack.pop();
      }
    } else {
      stack.push([char, 1, [originalChar]]);
    }
  }
  
  return stack.map(([, , originalChars]) => originalChars.join('')).join('');
}

// Count removed characters
function removeAndCount(str, k = 2) {
  const stack = [];
  let removed = 0;
  
  for (const char of str) {
    if (stack.length > 0 && stack[stack.length - 1][0] === char) {
      stack[stack.length - 1][1]++;
      
      if (stack[stack.length - 1][1] === k) {
        removed += k;
        stack.pop();
      }
    } else {
      stack.push([char, 1]);
    }
  }
  
  return {
    result: stack.map(([char, count]) => char.repeat(count)).join(''),
    removed
  };
}

// Get all removal steps
function getRemovalSteps(str, k = 2) {
  const steps = [str];
  const stack = [];
  
  for (const char of str) {
    if (stack.length > 0 && stack[stack.length - 1][0] === char) {
      stack[stack.length - 1][1]++;
      
      if (stack[stack.length - 1][1] === k) {
        stack.pop();
        const current = stack.map(([c, count]) => c.repeat(count)).join('');
        steps.push(current);
      }
    } else {
      stack.push([char, 1]);
    }
  }
  
  return steps;
}

// Remove all consecutive duplicates (reduce to single)
function reduceAdjacentDuplicates(str) {
  if (str.length === 0) return str;
  
  let result = str[0];
  
  for (let i = 1; i < str.length; i++) {
    if (str[i] !== str[i - 1]) {
      result += str[i];
    }
  }
  
  return result;
}

// Comprehensive test suite
console.log('=== Basic Removal (k=2) ===');
console.log(removeAdjacentDuplicates('abbaca'));
// 'ca'

console.log(removeAdjacentDuplicates('azxxzy'));
// 'ay'

console.log(removeAdjacentDuplicates('aabbcc'));
// ''

console.log('\n=== No Duplicates ===');
console.log(removeAdjacentDuplicates('abcdef'));
// 'abcdef'

console.log('\n=== K=3 Adjacent Duplicates ===');
console.log(removeAdjacentDuplicates('deeedbbcccbdaa', { k: 3 }));
// 'aa'

console.log(removeAdjacentDuplicates('aaabbbaaa', { k: 3 }));
// 'aaa' (removes first 'aaa', 'bbb', leaves last 'aaa')

console.log('\n=== Case Insensitive ===');
console.log(removeAdjacentDuplicates('aAbBcC', { caseInsensitive: true }));
// '' (treats A and a as same)

console.log('\n=== Custom Removal Function ===');
console.log(removeAdjacentDuplicates('aaabbbcc', {
  shouldRemove: (char, count) => count >= 3
}));
// 'cc' (removes runs of 3+)

console.log('\n=== Count Removed ===');
console.log(removeAndCount('abbaca'));
// { result: 'ca', removed: 4 }

console.log(removeAndCount('aabbcc'));
// { result: '', removed: 6 }

console.log('\n=== Removal Steps ===');
console.log(getRemovalSteps('abbaca'));
// ['abbaca', 'aaca', 'ca']

console.log('\n=== Reduce to Single ===');
console.log(reduceAdjacentDuplicates('aaabbbccc'));
// 'abc'

console.log(reduceAdjacentDuplicates('aabbbaac'));
// 'abac'

console.log('\n=== Edge Cases ===');
console.log(removeAdjacentDuplicates(''));
// ''

console.log(removeAdjacentDuplicates('a'));
// 'a'

console.log(removeAdjacentDuplicates('aa'));
// ''

console.log(removeAdjacentDuplicates('aaa'));
// 'a' (removes 'aa', leaves 'a')

console.log('\n=== Complex Cascading ===');
console.log(removeAdjacentDuplicates('abccba'));
// '' (removes 'cc' -> 'abba' -> removes 'bb' -> 'aa' -> removes 'aa' -> '')

console.log(removeAdjacentDuplicates('pbbcggttciiippooaais', { k: 2 }));
// 'ps'

console.log('\n=== Single Character Type ===');
console.log(removeAdjacentDuplicates('aaaa'));
// '' (removes all pairs)

console.log(removeAdjacentDuplicates('aaaaa'));
// 'a' (removes 2 pairs, leaves 1)

console.log('\n=== Long String ===');
console.log(removeAdjacentDuplicates('a'.repeat(100) + 'b'.repeat(100)));
// '' (all removed)
```

### **Bonus: Related Problems**
```javascript
/**
 * Make string palindrome by removing adjacent duplicates
 */
function canMakePalindrome(str) {
  const result = removeAdjacentDuplicates(str);
  return result === result.split('').reverse().join('');
}

// Test
console.log(canMakePalindrome('abccba'));
// true (becomes '')

/**
 * Minimum removals to make string without adjacent duplicates
 */
function minRemovalsToNoAdjacent(str) {
  let removals = 0;
  let prev = '';
  
  for (const char of str) {
    if (char === prev) {
      removals++;
    } else {
      prev = char;
    }
  }
  
  return removals;
}

// Test
console.log(minRemovalsToNoAdjacent('aaabbbccc'));
// 6 (remove 2 'a's, 2 'b's, 2 'c's)
```

**Interview Tips:**
- Stack is optimal solution: O(n) time, O(n) space
- Process character by character, pop if matches top of stack
- Can extend to k adjacent duplicates (store count with character)
- Removal is recursive/cascading: removing one pair may create another
- Common variations: k duplicates, case-insensitive, count removals
- Follow-ups: show steps, reduce to single, minimum removals
- Used in: text processing, game algorithms (candy crush), compression
- Similar to balanced parentheses but removes instead of validating

</details>

### **DOM & Browser**

51. Implement a debounce function

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Debounce**
```javascript
/**
 * Debounce function - delays execution until after wait time
 * Time Complexity: O(1) per call
 * Space Complexity: O(1)
 * 
 * @param {Function} func - Function to debounce
 * @param {number} wait - Wait time in milliseconds
 * @returns {Function} - Debounced function
 */
function debounce(func, wait) {
  let timeoutId;
  
  return function(...args) {
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, wait);
  };
}

// Test case
const log = debounce((msg) => console.log(msg), 1000);

log('First call');  // Cancelled
log('Second call'); // Cancelled
log('Third call');  // Executed after 1000ms

// Usage example: Search input
const searchInput = document.querySelector('#search');
const debouncedSearch = debounce((value) => {
  console.log('Searching for:', value);
  // API call here
}, 500);

searchInput?.addEventListener('input', (e) => {
  debouncedSearch(e.target.value);
});
```

### **Approach 2: With Immediate Execution**
```javascript
/**
 * Debounce with leading edge option
 * Execute immediately on first call, then wait
 */
function debounce(func, wait, immediate = false) {
  let timeoutId;
  
  return function(...args) {
    const callNow = immediate && !timeoutId;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (!immediate) {
        func.apply(this, args);
      }
    }, wait);
    
    if (callNow) {
      func.apply(this, args);
    }
  };
}

// Test
const immediateLog = debounce((msg) => console.log(msg), 1000, true);

immediateLog('First');  // Executes immediately
immediateLog('Second'); // Ignored
immediateLog('Third');  // Ignored
// After 1000ms, can execute again
```

### **Approach 3: With Cancel and Flush**
```javascript
/**
 * Advanced debounce with cancel and flush methods
 */
function debounce(func, wait) {
  let timeoutId;
  let lastArgs;
  let lastThis;
  
  const debounced = function(...args) {
    lastArgs = args;
    lastThis = this;
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      func.apply(lastThis, lastArgs);
      timeoutId = null;
    }, wait);
  };
  
  // Cancel pending execution
  debounced.cancel = function() {
    clearTimeout(timeoutId);
    timeoutId = null;
  };
  
  // Execute immediately with last arguments
  debounced.flush = function() {
    if (timeoutId) {
      clearTimeout(timeoutId);
      func.apply(lastThis, lastArgs);
      timeoutId = null;
    }
  };
  
  return debounced;
}

// Test
const saveDraft = debounce((text) => {
  console.log('Saving:', text);
}, 2000);

saveDraft('Hello');
saveDraft('Hello World');
saveDraft.flush(); // Save immediately
saveDraft('New text');
saveDraft.cancel(); // Cancel the save
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready debounce implementation
 * 
 * @param {Function} func - Function to debounce
 * @param {number} wait - Wait time in milliseconds
 * @param {Object} options - Debounce options
 * @param {boolean} options.leading - Execute on leading edge (default: false)
 * @param {boolean} options.trailing - Execute on trailing edge (default: true)
 * @param {number} options.maxWait - Maximum time to wait before execution
 * @returns {Function} - Debounced function with cancel and flush methods
 */
function debounce(func, wait, options = {}) {
  const {
    leading = false,
    trailing = true,
    maxWait = null
  } = options;
  
  // Validation
  if (typeof func !== 'function') {
    throw new TypeError('First argument must be a function');
  }
  
  if (typeof wait !== 'number' || wait < 0) {
    throw new TypeError('Wait time must be a non-negative number');
  }
  
  let timeoutId;
  let maxTimeoutId;
  let lastArgs;
  let lastThis;
  let result;
  let lastCallTime;
  let lastInvokeTime = 0;
  
  function invokeFunc(time) {
    const args = lastArgs;
    const thisArg = lastThis;
    
    lastArgs = lastThis = undefined;
    lastInvokeTime = time;
    result = func.apply(thisArg, args);
    return result;
  }
  
  function leadingEdge(time) {
    lastInvokeTime = time;
    
    // Start timer for trailing edge
    timeoutId = setTimeout(timerExpired, wait);
    
    // Execute if leading
    return leading ? invokeFunc(time) : result;
  }
  
  function remainingWait(time) {
    const timeSinceLastCall = time - lastCallTime;
    const timeSinceLastInvoke = time - lastInvokeTime;
    const timeWaiting = wait - timeSinceLastCall;
    
    return maxWait !== null
      ? Math.min(timeWaiting, maxWait - timeSinceLastInvoke)
      : timeWaiting;
  }
  
  function shouldInvoke(time) {
    const timeSinceLastCall = time - lastCallTime;
    const timeSinceLastInvoke = time - lastInvokeTime;
    
    return (
      lastCallTime === undefined ||
      timeSinceLastCall >= wait ||
      timeSinceLastCall < 0 ||
      (maxWait !== null && timeSinceLastInvoke >= maxWait)
    );
  }
  
  function timerExpired() {
    const time = Date.now();
    
    if (shouldInvoke(time)) {
      return trailingEdge(time);
    }
    
    // Restart timer
    timeoutId = setTimeout(timerExpired, remainingWait(time));
  }
  
  function trailingEdge(time) {
    timeoutId = undefined;
    
    // Only invoke if we have last args
    if (trailing && lastArgs) {
      return invokeFunc(time);
    }
    
    lastArgs = lastThis = undefined;
    return result;
  }
  
  function cancel() {
    if (timeoutId !== undefined) {
      clearTimeout(timeoutId);
    }
    if (maxTimeoutId !== undefined) {
      clearTimeout(maxTimeoutId);
    }
    
    lastInvokeTime = 0;
    lastArgs = lastCallTime = lastThis = timeoutId = maxTimeoutId = undefined;
  }
  
  function flush() {
    return timeoutId === undefined ? result : trailingEdge(Date.now());
  }
  
  function pending() {
    return timeoutId !== undefined;
  }
  
  function debounced(...args) {
    const time = Date.now();
    const isInvoking = shouldInvoke(time);
    
    lastArgs = args;
    lastThis = this;
    lastCallTime = time;
    
    if (isInvoking) {
      if (timeoutId === undefined) {
        return leadingEdge(lastCallTime);
      }
      
      if (maxWait !== null) {
        // Handle maxWait
        timeoutId = setTimeout(timerExpired, wait);
        return invokeFunc(lastCallTime);
      }
    }
    
    if (timeoutId === undefined) {
      timeoutId = setTimeout(timerExpired, wait);
    }
    
    return result;
  }
  
  debounced.cancel = cancel;
  debounced.flush = flush;
  debounced.pending = pending;
  
  return debounced;
}

// Real-world usage examples
console.log('=== Window Resize Handler ===');
let resizeCount = 0;
const handleResize = debounce(() => {
  resizeCount++;
  console.log('Window resized!', resizeCount);
}, 500);

// window.addEventListener('resize', handleResize);

console.log('\n=== Search Input ===');
const search = debounce((query) => {
  console.log('Searching for:', query);
  // fetch(`/api/search?q=${query}`)
}, 300);

console.log('\n=== Form Auto-Save ===');
const autoSave = debounce((formData) => {
  console.log('Auto-saving form...', formData);
  // API call to save draft
}, 2000, { maxWait: 5000 });

console.log('\n=== Scroll Handler with Leading Edge ===');
const trackScroll = debounce(() => {
  console.log('Scroll position:', window.scrollY);
}, 100, { leading: true, trailing: false });

console.log('\n=== Button Click Protection ===');
const submitForm = debounce((data) => {
  console.log('Submitting form...', data);
  // Prevent double submissions
}, 1000, { leading: true, trailing: false });

// Test suite
console.log('\n=== Test: Basic Debounce ===');
let callCount = 0;
const debounced = debounce(() => {
  callCount++;
}, 100);

debounced();
debounced();
debounced();
setTimeout(() => {
  console.log('Call count:', callCount); // Should be 1
}, 150);

console.log('\n=== Test: Cancel ===');
const cancellable = debounce(() => {
  console.log('This should not execute');
}, 100);

cancellable();
cancellable.cancel();

console.log('\n=== Test: Flush ===');
const flushable = debounce(() => {
  console.log('Flushed immediately');
}, 1000);

flushable();
flushable.flush(); // Executes immediately

console.log('\n=== Test: MaxWait ===');
const maxWaitTest = debounce(() => {
  console.log('MaxWait triggered');
}, 100, { maxWait: 500 });

// Even if called continuously, will execute at least every 500ms

console.log('\n=== Test: Leading Edge ===');
let leadingCount = 0;
const leadingDebounced = debounce(() => {
  leadingCount++;
  console.log('Leading edge:', leadingCount);
}, 100, { leading: true, trailing: false });

leadingDebounced(); // Executes immediately
leadingDebounced(); // Ignored
leadingDebounced(); // Ignored
```

### **Bonus: Advanced Variations**
```javascript
/**
 * Debounce with Promise support
 */
function debouncePromise(func, wait) {
  let timeoutId;
  let latestResolve;
  let latestReject;
  
  return function(...args) {
    return new Promise((resolve, reject) => {
      clearTimeout(timeoutId);
      
      latestResolve = resolve;
      latestReject = reject;
      
      timeoutId = setTimeout(async () => {
        try {
          const result = await func.apply(this, args);
          latestResolve(result);
        } catch (error) {
          latestReject(error);
        }
      }, wait);
    });
  };
}

// Test
const asyncSearch = debouncePromise(async (query) => {
  // Simulate API call
  return new Promise(resolve => {
    setTimeout(() => resolve(`Results for: ${query}`), 100);
  });
}, 500);

asyncSearch('test').then(console.log);

/**
 * Debounce with grouping by key
 */
function debounceByKey(func, wait) {
  const timeouts = new Map();
  
  return function(key, ...args) {
    if (timeouts.has(key)) {
      clearTimeout(timeouts.get(key));
    }
    
    const timeoutId = setTimeout(() => {
      func.call(this, key, ...args);
      timeouts.delete(key);
    }, wait);
    
    timeouts.set(key, timeoutId);
  };
}

// Test
const updateUser = debounceByKey((userId, data) => {
  console.log(`Updating user ${userId}:`, data);
}, 1000);

updateUser('user1', { name: 'Alice' });
updateUser('user2', { name: 'Bob' });
// Both will execute independently

/**
 * Adaptive debounce (adjusts delay based on call frequency)
 */
function adaptiveDebounce(func, minWait, maxWait) {
  let timeoutId;
  let callCount = 0;
  let lastCallTime;
  
  return function(...args) {
    const now = Date.now();
    
    if (lastCallTime && now - lastCallTime < 100) {
      callCount++;
    } else {
      callCount = 1;
    }
    
    lastCallTime = now;
    
    // Increase wait time based on call frequency
    const wait = Math.min(minWait + callCount * 50, maxWait);
    
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      func.apply(this, args);
      callCount = 0;
    }, wait);
  };
}
```

**Interview Tips:**
- Debounce delays execution until after calls have stopped for wait time
- Use for: search inputs, window resize, scroll events, form validation
- Key features: cancel (clear timeout), flush (execute immediately), pending (check status)
- Leading edge: execute on first call, then ignore until quiet period
- Trailing edge: execute after calls stop (default behavior)
- MaxWait: guarantee execution within maximum time
- Difference from throttle: throttle executes periodically, debounce waits for quiet
- Memory: store timeout IDs, last args, context (this)
- Common pitfall: losing `this` context (use arrow functions or apply)
- lodash.debounce is production standard but good to implement yourself

</details>

52. Implement a throttle function

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Throttle**
```javascript
/**
 * Throttle function - executes at most once per interval
 * Time Complexity: O(1) per call
 * Space Complexity: O(1)
 * 
 * @param {Function} func - Function to throttle
 * @param {number} limit - Time limit in milliseconds
 * @returns {Function} - Throttled function
 */
function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Test case
let scrollCount = 0;
const throttledScroll = throttle(() => {
  scrollCount++;
  console.log('Scroll event:', scrollCount);
}, 1000);

// Simulating rapid calls
for (let i = 0; i < 10; i++) {
  throttledScroll(); // Only first call executes
}
// After 1000ms, can execute again
```

### **Approach 2: With Leading and Trailing Options**
```javascript
/**
 * Throttle with leading and trailing edge control
 */
function throttle(func, limit, options = {}) {
  const { leading = true, trailing = true } = options;
  
  let lastRan;
  let lastFunc;
  
  return function(...args) {
    if (!lastRan) {
      if (leading) {
        func.apply(this, args);
      }
      lastRan = Date.now();
    } else {
      clearTimeout(lastFunc);
      
      lastFunc = setTimeout(() => {
        if (Date.now() - lastRan >= limit) {
          if (trailing) {
            func.apply(this, args);
          }
          lastRan = Date.now();
        }
      }, limit - (Date.now() - lastRan));
    }
  };
}

// Test
const log = throttle((msg) => console.log(msg), 1000, {
  leading: true,
  trailing: true
});
```

### **Approach 3: With Cancel Method**
```javascript
/**
 * Throttle with cancel functionality
 */
function throttle(func, limit) {
  let inThrottle;
  let lastFunc;
  let lastRan;
  
  const throttled = function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      lastRan = Date.now();
      inThrottle = true;
    } else {
      clearTimeout(lastFunc);
      lastFunc = setTimeout(() => {
        if (Date.now() - lastRan >= limit) {
          func.apply(this, args);
          lastRan = Date.now();
        }
      }, limit - (Date.now() - lastRan));
    }
    
    setTimeout(() => {
      inThrottle = false;
    }, limit);
  };
  
  throttled.cancel = function() {
    clearTimeout(lastFunc);
    inThrottle = false;
  };
  
  return throttled;
}

// Test
const save = throttle((data) => {
  console.log('Saving:', data);
}, 2000);

save('data1');
save('data2');
save.cancel(); // Cancel pending execution
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready throttle implementation
 * 
 * @param {Function} func - Function to throttle
 * @param {number} wait - Wait time in milliseconds
 * @param {Object} options - Throttle options
 * @param {boolean} options.leading - Execute on leading edge (default: true)
 * @param {boolean} options.trailing - Execute on trailing edge (default: true)
 * @returns {Function} - Throttled function with cancel method
 */
function throttle(func, wait, options = {}) {
  const { leading = true, trailing = true } = options;
  
  // Validation
  if (typeof func !== 'function') {
    throw new TypeError('First argument must be a function');
  }
  
  if (typeof wait !== 'number' || wait < 0) {
    throw new TypeError('Wait time must be a non-negative number');
  }
  
  let timeoutId;
  let lastArgs;
  let lastThis;
  let result;
  let lastCallTime;
  let lastInvokeTime = 0;
  
  function invokeFunc(time) {
    const args = lastArgs;
    const thisArg = lastThis;
    
    lastArgs = lastThis = undefined;
    lastInvokeTime = time;
    result = func.apply(thisArg, args);
    return result;
  }
  
  function leadingEdge(time) {
    lastInvokeTime = time;
    timeoutId = setTimeout(timerExpired, wait);
    return leading ? invokeFunc(time) : result;
  }
  
  function remainingWait(time) {
    const timeSinceLastCall = time - lastCallTime;
    const timeSinceLastInvoke = time - lastInvokeTime;
    const timeWaiting = wait - timeSinceLastInvoke;
    
    return timeWaiting;
  }
  
  function shouldInvoke(time) {
    const timeSinceLastCall = time - lastCallTime;
    const timeSinceLastInvoke = time - lastInvokeTime;
    
    return (
      lastCallTime === undefined ||
      timeSinceLastCall >= wait ||
      timeSinceLastCall < 0 ||
      timeSinceLastInvoke >= wait
    );
  }
  
  function timerExpired() {
    const time = Date.now();
    
    if (shouldInvoke(time)) {
      return trailingEdge(time);
    }
    
    timeoutId = setTimeout(timerExpired, remainingWait(time));
  }
  
  function trailingEdge(time) {
    timeoutId = undefined;
    
    if (trailing && lastArgs) {
      return invokeFunc(time);
    }
    
    lastArgs = lastThis = undefined;
    return result;
  }
  
  function cancel() {
    if (timeoutId !== undefined) {
      clearTimeout(timeoutId);
    }
    lastInvokeTime = 0;
    lastArgs = lastCallTime = lastThis = timeoutId = undefined;
  }
  
  function flush() {
    return timeoutId === undefined ? result : trailingEdge(Date.now());
  }
  
  function throttled(...args) {
    const time = Date.now();
    const isInvoking = shouldInvoke(time);
    
    lastArgs = args;
    lastThis = this;
    lastCallTime = time;
    
    if (isInvoking) {
      if (timeoutId === undefined) {
        return leadingEdge(lastCallTime);
      }
      if (trailing) {
        timeoutId = setTimeout(timerExpired, wait);
        return invokeFunc(lastCallTime);
      }
    }
    
    if (timeoutId === undefined) {
      timeoutId = setTimeout(timerExpired, wait);
    }
    
    return result;
  }
  
  throttled.cancel = cancel;
  throttled.flush = flush;
  
  return throttled;
}

// Real-world usage examples
console.log('=== Infinite Scroll ===');
const loadMore = throttle(() => {
  console.log('Loading more items...');
  // Fetch next page
}, 1000);

// window.addEventListener('scroll', () => {
//   if (window.innerHeight + window.scrollY >= document.body.offsetHeight) {
//     loadMore();
//   }
// });

console.log('\n=== Mouse Move Tracking ===');
const trackMouse = throttle((e) => {
  console.log('Mouse position:', e.clientX, e.clientY);
}, 100);

// document.addEventListener('mousemove', trackMouse);

console.log('\n=== API Rate Limiting ===');
const apiCall = throttle((endpoint) => {
  console.log('Making API call to:', endpoint);
  // fetch(endpoint)
}, 2000);

console.log('\n=== Game Loop Updates ===');
const updateGame = throttle(() => {
  console.log('Updating game state...');
  // Game logic here
}, 16); // ~60 FPS

console.log('\n=== Button Click Protection ===');
const handleSubmit = throttle((formData) => {
  console.log('Submitting form:', formData);
  // Prevent multiple rapid submissions
}, 3000, { leading: true, trailing: false });

// Test suite
console.log('\n=== Test: Basic Throttle ===');
let executeCount = 0;
const throttled = throttle(() => {
  executeCount++;
  console.log('Executed:', executeCount);
}, 100);

// Rapid calls
for (let i = 0; i < 5; i++) {
  throttled();
}
// Only first call executes immediately

console.log('\n=== Test: Leading = false ===');
const noLeading = throttle(() => {
  console.log('Trailing edge only');
}, 100, { leading: false });

noLeading();
noLeading();

console.log('\n=== Test: Trailing = false ===');
const noTrailing = throttle(() => {
  console.log('Leading edge only');
}, 100, { trailing: false });

noTrailing();
noTrailing();

console.log('\n=== Test: Cancel ===');
const cancellable = throttle(() => {
  console.log('This should not execute');
}, 100);

cancellable();
cancellable.cancel();
```

### **Bonus: Advanced Variations**
```javascript
/**
 * Throttle with Promise support
 */
function throttlePromise(func, wait) {
  let lastExecution = 0;
  let pendingPromise = null;
  
  return async function(...args) {
    const now = Date.now();
    const timeSinceLastExecution = now - lastExecution;
    
    if (timeSinceLastExecution >= wait) {
      lastExecution = now;
      return await func.apply(this, args);
    }
    
    if (!pendingPromise) {
      pendingPromise = new Promise((resolve) => {
        setTimeout(async () => {
          lastExecution = Date.now();
          const result = await func.apply(this, args);
          pendingPromise = null;
          resolve(result);
        }, wait - timeSinceLastExecution);
      });
    }
    
    return pendingPromise;
  };
}

/**
 * Throttle with queue (executes all calls, just throttled)
 */
function throttleWithQueue(func, wait) {
  const queue = [];
  let isThrottled = false;
  
  function processQueue() {
    if (queue.length === 0) {
      isThrottled = false;
      return;
    }
    
    const { args, context } = queue.shift();
    func.apply(context, args);
    
    setTimeout(processQueue, wait);
  }
  
  return function(...args) {
    queue.push({ args, context: this });
    
    if (!isThrottled) {
      isThrottled = true;
      processQueue();
    }
  };
}

/**
 * Dynamic throttle (adjusts based on system load)
 */
function dynamicThrottle(func, minWait, maxWait) {
  let lastExecution = 0;
  let performanceScore = 1;
  
  return function(...args) {
    const now = Date.now();
    const startTime = performance.now();
    
    // Calculate dynamic wait based on performance
    const wait = minWait + (maxWait - minWait) * (1 - performanceScore);
    
    if (now - lastExecution >= wait) {
      lastExecution = now;
      
      func.apply(this, args);
      
      // Measure execution time to adjust throttle
      const executionTime = performance.now() - startTime;
      performanceScore = Math.max(0, Math.min(1, 1 - executionTime / 100));
    }
  };
}

/**
 * Compare throttle vs debounce
 */
function compareThrottleDebounce() {
  let throttleCount = 0;
  let debounceCount = 0;
  
  const throttled = throttle(() => {
    throttleCount++;
    console.log('Throttle executed:', throttleCount);
  }, 100);
  
  const debounced = debounce(() => {
    debounceCount++;
    console.log('Debounce executed:', debounceCount);
  }, 100);
  
  // Simulate rapid calls
  const interval = setInterval(() => {
    throttled();
    debounced();
  }, 20);
  
  setTimeout(() => {
    clearInterval(interval);
    console.log('Final counts - Throttle:', throttleCount, 'Debounce:', debounceCount);
    // Throttle: ~5 (every 100ms), Debounce: 1 (after calls stop)
  }, 500);
}

// Helper for debounce comparison
function debounce(func, wait) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), wait);
  };
}
```

**Interview Tips:**
- Throttle executes at most once per time period
- Use for: scroll events, mouse movement, resize, API rate limiting
- Leading edge: execute immediately on first call
- Trailing edge: execute after interval even if still being called
- Difference from debounce: throttle executes periodically during activity, debounce waits for inactivity
- Performance: prevents excessive function calls during continuous events
- Common pattern: leading=true, trailing=true (execute both edges)
- Memory: track last execution time and timeout ID
- Use case examples: infinite scroll (throttle), search input (debounce)
- lodash.throttle is production standard with more features

</details>

53. Create a function to get all elements by class name recursively

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Using Built-in Method**
```javascript
/**
 * Get all elements by class name (simple wrapper)
 * Time Complexity: O(n) where n is DOM nodes
 * Space Complexity: O(k) where k is matching elements
 * 
 * @param {string} className - Class name to search for
 * @param {Element} root - Root element to search from
 * @returns {Array} - Array of matching elements
 */
function getElementsByClassName(className, root = document) {
  return Array.from(root.getElementsByClassName(className));
}

// Test
const buttons = getElementsByClassName('btn');
console.log('Found buttons:', buttons.length);
```

### **Approach 2: Recursive DOM Traversal**
```javascript
/**
 * Recursive implementation without built-in methods
 * Time Complexity: O(n)
 * Space Complexity: O(n) for recursion stack
 */
function getElementsByClassName(className, root = document.body) {
  const result = [];
  
  function traverse(element) {
    // Check if current element has the class
    if (element.classList && element.classList.contains(className)) {
      result.push(element);
    }
    
    // Recurse through children
    for (const child of element.children) {
      traverse(child);
    }
  }
  
  traverse(root);
  return result;
}

// Test
const elements = getElementsByClassName('highlight');
console.log('Found elements:', elements);
```

### **Approach 3: Iterative with Queue (BFS)**
```javascript
/**
 * Iterative breadth-first search implementation
 * Time Complexity: O(n)
 * Space Complexity: O(w) where w is max width of tree
 */
function getElementsByClassName(className, root = document.body) {
  const result = [];
  const queue = [root];
  
  while (queue.length > 0) {
    const element = queue.shift();
    
    if (element.classList && element.classList.contains(className)) {
      result.push(element);
    }
    
    queue.push(...element.children);
  }
  
  return result;
}

// Test
const items = getElementsByClassName('list-item');
console.log('Found items:', items);
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready element finder with multiple selectors
 * 
 * @param {string|Array} selector - Class name(s) or CSS selector
 * @param {Element} root - Root element to search from
 * @param {Object} options - Search options
 * @param {boolean} options.multiple - Match elements with all classes (default: false)
 * @param {boolean} options.exact - Exact class match only (default: false)
 * @param {number} options.maxDepth - Maximum depth to search (default: Infinity)
 * @param {Function} options.filter - Additional filter function
 * @returns {Array} - Array of matching elements
 */
function getElementsByClassName(selector, root = document.body, options = {}) {
  const {
    multiple = false,
    exact = false,
    maxDepth = Infinity,
    filter = null
  } = options;
  
  // Validation
  if (!root || !root.children) {
    throw new TypeError('Root must be a valid DOM element');
  }
  
  // Handle modern querySelector for complex selectors
  if (selector.includes('[') || selector.includes('>') || selector.includes('+')) {
    const elements = Array.from(root.querySelectorAll(selector));
    return filter ? elements.filter(filter) : elements;
  }
  
  // Parse class names
  const classNames = Array.isArray(selector)
    ? selector
    : selector.split(/\s+/).filter(Boolean);
  
  const result = [];
  
  function traverse(element, depth) {
    if (depth > maxDepth) return;
    
    if (element.classList) {
      let matches = false;
      
      if (multiple) {
        // Element must have all classes
        matches = classNames.every(cls => element.classList.contains(cls));
      } else if (exact) {
        // Element must have exactly these classes
        matches = element.classList.length === classNames.length &&
                 classNames.every(cls => element.classList.contains(cls));
      } else {
        // Element must have at least one class
        matches = classNames.some(cls => element.classList.contains(cls));
      }
      
      if (matches && (!filter || filter(element))) {
        result.push(element);
      }
    }
    
    // Recurse through children
    for (const child of element.children) {
      traverse(child, depth + 1);
    }
  }
  
  traverse(root, 0);
  return result;
}

// Get elements by multiple selectors
function getElementsBy(selectors, root = document.body) {
  const results = {};
  
  for (const [key, selector] of Object.entries(selectors)) {
    if (selector.startsWith('.')) {
      results[key] = getElementsByClassName(selector.slice(1), root);
    } else if (selector.startsWith('#')) {
      results[key] = [root.querySelector(selector)].filter(Boolean);
    } else {
      results[key] = Array.from(root.querySelectorAll(selector));
    }
  }
  
  return results;
}

// Get elements with custom attribute
function getElementsByAttribute(attr, value, root = document.body) {
  const result = [];
  
  function traverse(element) {
    if (element.hasAttribute(attr)) {
      if (value === undefined || element.getAttribute(attr) === value) {
        result.push(element);
      }
    }
    
    for (const child of element.children) {
      traverse(child);
    }
  }
  
  traverse(root);
  return result;
}

// Get elements by predicate function
function findElements(predicate, root = document.body) {
  const result = [];
  
  function traverse(element) {
    if (predicate(element)) {
      result.push(element);
    }
    
    for (const child of element.children) {
      traverse(child);
    }
  }
  
  traverse(root);
  return result;
}

// Get parent elements matching selector
function getParentsWithClass(element, className) {
  const parents = [];
  let current = element.parentElement;
  
  while (current) {
    if (current.classList && current.classList.contains(className)) {
      parents.push(current);
    }
    current = current.parentElement;
  }
  
  return parents;
}

// Comprehensive test suite
console.log('=== Basic Class Search ===');
// const buttons = getElementsByClassName('btn');
// console.log('Buttons:', buttons.length);

console.log('\n=== Multiple Classes (Any) ===');
// const elements = getElementsByClassName('btn primary');
// Elements with 'btn' OR 'primary'

console.log('\n=== Multiple Classes (All) ===');
// const elements = getElementsByClassName('btn primary', document.body, {
//   multiple: true
// });
// Elements with both 'btn' AND 'primary'

console.log('\n=== Exact Class Match ===');
// const elements = getElementsByClassName('btn', document.body, {
//   exact: true
// });
// Elements with only 'btn' class, no others

console.log('\n=== Limited Depth ===');
// const elements = getElementsByClassName('item', document.body, {
//   maxDepth: 3
// });
// Search only 3 levels deep

console.log('\n=== With Filter ===');
// const visibleButtons = getElementsByClassName('btn', document.body, {
//   filter: (el) => el.offsetParent !== null
// });
// Only visible buttons

console.log('\n=== By Attribute ===');
// const dataElements = getElementsByAttribute('data-id', '123');
// console.log('Elements with data-id="123":', dataElements.length);

console.log('\n=== By Predicate ===');
// const largeElements = findElements((el) => {
//   return el.offsetWidth > 500;
// });
// console.log('Large elements:', largeElements.length);

console.log('\n=== Multiple Selectors ===');
// const elements = getElementsBy({
//   buttons: '.btn',
//   headers: 'h1, h2, h3',
//   main: '#main'
// });
// console.log('Found:', elements);

console.log('\n=== Parent Search ===');
// const button = document.querySelector('.btn');
// const containers = getParentsWithClass(button, 'container');
// console.log('Parent containers:', containers);

// Example: Practical usage
function highlightElements() {
  const elements = getElementsByClassName('highlight');
  
  elements.forEach(el => {
    el.style.backgroundColor = 'yellow';
  });
}

function toggleClassRecursive(className, root = document.body) {
  const elements = getElementsByClassName(className, root);
  
  elements.forEach(el => {
    el.classList.toggle('active');
  });
}

// Performance comparison
function comparePerformance() {
  const className = 'test-class';
  const root = document.body;
  
  console.time('Native');
  const native = root.getElementsByClassName(className);
  console.timeEnd('Native');
  
  console.time('Recursive');
  const recursive = getElementsByClassName(className, root);
  console.timeEnd('Recursive');
  
  console.time('querySelector');
  const query = root.querySelectorAll(`.${className}`);
  console.timeEnd('querySelector');
}
```

### **Bonus: Advanced DOM Utilities**
```javascript
/**
 * Get elements matching complex criteria
 */
function getElementsAdvanced(criteria, root = document.body) {
  return findElements((el) => {
    // Check all criteria
    for (const [key, value] of Object.entries(criteria)) {
      switch (key) {
        case 'class':
          if (!el.classList.contains(value)) return false;
          break;
        case 'tag':
          if (el.tagName.toLowerCase() !== value.toLowerCase()) return false;
          break;
        case 'attr':
          for (const [attr, attrValue] of Object.entries(value)) {
            if (el.getAttribute(attr) !== attrValue) return false;
          }
          break;
        case 'text':
          if (!el.textContent.includes(value)) return false;
          break;
        case 'visible':
          if (value && el.offsetParent === null) return false;
          break;
      }
    }
    return true;
  }, root);
}

// Test
// const elements = getElementsAdvanced({
//   class: 'btn',
//   tag: 'button',
//   attr: { 'data-action': 'submit' },
//   visible: true
// });

/**
 * Get closest ancestor with class
 */
function closest(element, className) {
  let current = element;
  
  while (current && current !== document.body) {
    if (current.classList && current.classList.contains(className)) {
      return current;
    }
    current = current.parentElement;
  }
  
  return null;
}

/**
 * Get all siblings with class
 */
function getSiblingsWithClass(element, className) {
  const siblings = [];
  const parent = element.parentElement;
  
  if (!parent) return siblings;
  
  for (const child of parent.children) {
    if (child !== element && child.classList && child.classList.contains(className)) {
      siblings.push(child);
    }
  }
  
  return siblings;
}
```

**Interview Tips:**
- Native getElementsByClassName returns HTMLCollection (live), convert to array
- Recursive approach: DFS traversal of DOM tree
- Iterative approach: BFS using queue, avoids stack overflow
- Time complexity: O(n) where n is number of DOM nodes
- classList.contains() is efficient for class checking
- Can extend to: multiple classes, attributes, custom predicates
- querySelectorAll is more powerful but slightly slower for simple class search
- Consider max depth limit for very deep DOM trees
- Use filter option for additional criteria (visibility, size, etc.)
- Modern approach: use querySelectorAll for complex selectors

</details>

54. Implement event delegation for a list

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Event Delegation**
```javascript
/**
 * Basic event delegation for list items
 * Time Complexity: O(1) per event
 * Space Complexity: O(1)
 * 
 * @param {Element} listElement - Parent list element
 * @param {string} itemSelector - Selector for list items
 * @param {Function} handler - Event handler function
 */
function delegateEvent(listElement, itemSelector, handler) {
  listElement.addEventListener('click', (event) => {
    // Find the closest matching element
    const target = event.target.closest(itemSelector);
    
    if (target && listElement.contains(target)) {
      handler.call(target, event);
    }
  });
}

// Test usage
/*
<ul id="myList">
  <li data-id="1">Item 1</li>
  <li data-id="2">Item 2</li>
  <li data-id="3">Item 3</li>
</ul>
*/

// const list = document.getElementById('myList');
// delegateEvent(list, 'li', function(event) {
//   console.log('Clicked item:', this.dataset.id);
//   this.classList.toggle('selected');
// });
```

### **Approach 2: With Event Type Parameter**
```javascript
/**
 * Event delegation with configurable event type
 */
function delegateEvent(parent, selector, eventType, handler) {
  parent.addEventListener(eventType, (event) => {
    const target = event.target.closest(selector);
    
    if (target && parent.contains(target)) {
      handler.call(target, event);
    }
  });
}

// Test
// const list = document.getElementById('list');
// delegateEvent(list, '.item', 'click', function(event) {
//   console.log('Item clicked:', this.textContent);
// });

// delegateEvent(list, '.item', 'mouseenter', function(event) {
//   this.style.backgroundColor = 'lightblue';
// });
```

### **Approach 3: Multiple Event Types**
```javascript
/**
 * Delegate multiple event types at once
 */
function delegateEvents(parent, selector, events) {
  Object.entries(events).forEach(([eventType, handler]) => {
    parent.addEventListener(eventType, (event) => {
      const target = event.target.closest(selector);
      
      if (target && parent.contains(target)) {
        handler.call(target, event);
      }
    });
  });
}

// Test
// delegateEvents(list, '.item', {
//   click: function(event) {
//     console.log('Clicked:', this.textContent);
//   },
//   mouseenter: function(event) {
//     this.classList.add('hover');
//   },
//   mouseleave: function(event) {
//     this.classList.remove('hover');
//   }
// });
```

### **Production-Grade Solution**
```javascript
/**
 * Production-ready event delegation system
 * 
 * @param {Element} parent - Parent element
 * @param {string} selector - Child selector
 * @param {string|Object} eventType - Event type(s)
 * @param {Function} handler - Event handler
 * @param {Object} options - Delegation options
 * @returns {Function} - Cleanup function
 */
function delegate(parent, selector, eventType, handler, options = {}) {
  const {
    capture = false,
    once = false,
    passive = false,
    stopPropagation = false,
    preventDefault = false
  } = options;
  
  // Validation
  if (!parent || !parent.addEventListener) {
    throw new TypeError('Parent must be a valid DOM element');
  }
  
  if (typeof selector !== 'string') {
    throw new TypeError('Selector must be a string');
  }
  
  // Handle multiple event types
  const events = typeof eventType === 'string'
    ? [eventType]
    : eventType;
  
  const listeners = [];
  
  events.forEach(type => {
    const listener = (event) => {
      const target = event.target.closest(selector);
      
      if (target && parent.contains(target)) {
        if (stopPropagation) {
          event.stopPropagation();
        }
        
        if (preventDefault) {
          event.preventDefault();
        }
        
        // Call handler with target as context
        handler.call(target, event, target);
        
        if (once) {
          cleanup();
        }
      }
    };
    
    parent.addEventListener(type, listener, { capture, passive });
    listeners.push({ type, listener });
  });
  
  // Return cleanup function
  function cleanup() {
    listeners.forEach(({ type, listener }) => {
      parent.removeEventListener(type, listener, { capture });
    });
  }
  
  return cleanup;
}

// Advanced event delegation class
class EventDelegator {
  constructor(parent) {
    this.parent = parent;
    this.handlers = new Map();
  }
  
  on(selector, eventType, handler, options = {}) {
    const key = `${selector}:${eventType}`;
    
    if (this.handlers.has(key)) {
      console.warn(`Handler already exists for ${key}`);
      return this;
    }
    
    const cleanup = delegate(
      this.parent,
      selector,
      eventType,
      handler,
      options
    );
    
    this.handlers.set(key, cleanup);
    return this;
  }
  
  off(selector, eventType) {
    const key = `${selector}:${eventType}`;
    const cleanup = this.handlers.get(key);
    
    if (cleanup) {
      cleanup();
      this.handlers.delete(key);
    }
    
    return this;
  }
  
  offAll() {
    this.handlers.forEach(cleanup => cleanup());
    this.handlers.clear();
    return this;
  }
}

// Practical examples
console.log('=== Todo List Example ===');
/*
<ul id="todoList">
  <li>
    <span class="text">Task 1</span>
    <button class="delete">Delete</button>
  </li>
</ul>
*/

function setupTodoList() {
  const todoList = document.getElementById('todoList');
  
  if (!todoList) return;
  
  const delegator = new EventDelegator(todoList);
  
  // Click on list item text to edit
  delegator.on('.text', 'dblclick', function(event) {
    const currentText = this.textContent;
    const input = document.createElement('input');
    input.value = currentText;
    
    input.addEventListener('blur', () => {
      this.textContent = input.value;
      this.style.display = '';
    });
    
    this.style.display = 'none';
    this.parentElement.insertBefore(input, this);
    input.focus();
  });
  
  // Click delete button
  delegator.on('.delete', 'click', function(event) {
    event.stopPropagation();
    this.parentElement.remove();
  });
  
  // Hover effects
  delegator.on('li', 'mouseenter', function() {
    this.classList.add('hover');
  });
  
  delegator.on('li', 'mouseleave', function() {
    this.classList.remove('hover');
  });
  
  return delegator;
}

console.log('\n=== Sortable List Example ===');
function setupSortableList() {
  const list = document.getElementById('sortableList');
  
  if (!list) return;
  
  let draggedElement = null;
  
  delegate(list, '.item', 'dragstart', function(event) {
    draggedElement = this;
    this.classList.add('dragging');
    event.dataTransfer.effectAllowed = 'move';
  });
  
  delegate(list, '.item', 'dragend', function(event) {
    this.classList.remove('dragging');
    draggedElement = null;
  });
  
  delegate(list, '.item', 'dragover', function(event) {
    event.preventDefault();
    
    if (draggedElement && draggedElement !== this) {
      const rect = this.getBoundingClientRect();
      const middle = rect.top + rect.height / 2;
      
      if (event.clientY < middle) {
        this.parentNode.insertBefore(draggedElement, this);
      } else {
        this.parentNode.insertBefore(draggedElement, this.nextSibling);
      }
    }
  });
}

console.log('\n=== Dynamic List Example ===');
function setupDynamicList() {
  const container = document.getElementById('dynamicList');
  
  if (!container) return;
  
  // Add items dynamically
  const addButton = document.getElementById('addItem');
  let itemCount = 0;
  
  addButton?.addEventListener('click', () => {
    const item = document.createElement('li');
    item.className = 'list-item';
    item.dataset.id = ++itemCount;
    item.innerHTML = `
      <span>Item ${itemCount}</span>
      <button class="edit">Edit</button>
      <button class="remove">Remove</button>
    `;
    container.appendChild(item);
  });
  
  // Event delegation handles both existing and future items
  const delegator = new EventDelegator(container);
  
  delegator
    .on('.list-item', 'click', function(event) {
      console.log('Clicked item:', this.dataset.id);
    })
    .on('.edit', 'click', function(event) {
      event.stopPropagation();
      const span = this.parentElement.querySelector('span');
      const newText = prompt('Edit item:', span.textContent);
      if (newText) span.textContent = newText;
    })
    .on('.remove', 'click', function(event) {
      event.stopPropagation();
      this.parentElement.remove();
    });
  
  return delegator;
}

console.log('\n=== Context Menu Example ===');
function setupContextMenu() {
  const list = document.getElementById('list');
  const contextMenu = document.getElementById('contextMenu');
  
  if (!list || !contextMenu) return;
  
  let selectedItem = null;
  
  delegate(list, '.item', 'contextmenu', function(event) {
    event.preventDefault();
    selectedItem = this;
    
    contextMenu.style.display = 'block';
    contextMenu.style.left = event.pageX + 'px';
    contextMenu.style.top = event.pageY + 'px';
  });
  
  delegate(contextMenu, '.menu-item', 'click', function(event) {
    const action = this.dataset.action;
    
    if (selectedItem) {
      switch (action) {
        case 'edit':
          console.log('Edit:', selectedItem.textContent);
          break;
        case 'delete':
          selectedItem.remove();
          break;
        case 'duplicate':
          selectedItem.parentElement.insertBefore(
            selectedItem.cloneNode(true),
            selectedItem.nextSibling
          );
          break;
      }
    }
    
    contextMenu.style.display = 'none';
  });
  
  document.addEventListener('click', () => {
    contextMenu.style.display = 'none';
  });
}

// Test suite
console.log('\n=== Performance Comparison ===');
function comparePerformance() {
  const list = document.createElement('ul');
  
  // Add 1000 items
  for (let i = 0; i < 1000; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    li.className = 'item';
    list.appendChild(li);
  }
  
  document.body.appendChild(list);
  
  // Method 1: Individual listeners (BAD)
  console.time('Individual Listeners');
  list.querySelectorAll('.item').forEach(item => {
    item.addEventListener('click', () => {
      console.log('Clicked');
    });
  });
  console.timeEnd('Individual Listeners');
  
  // Method 2: Event delegation (GOOD)
  console.time('Event Delegation');
  delegate(list, '.item', 'click', () => {
    console.log('Clicked');
  });
  console.timeEnd('Event Delegation');
  
  document.body.removeChild(list);
}
```

### **Bonus: Advanced Patterns**
```javascript
/**
 * Delegate with data passing
 */
function delegateWithData(parent, selector, eventType, handler) {
  parent.addEventListener(eventType, (event) => {
    const target = event.target.closest(selector);
    
    if (target && parent.contains(target)) {
      const data = Object.assign({}, target.dataset);
      handler.call(target, event, data);
    }
  });
}

// Test
// delegateWithData(list, '.item', 'click', function(event, data) {
//   console.log('Item data:', data); // { id: '1', name: 'Item 1' }
// });

/**
 * Conditional delegation
 */
function delegateIf(parent, selector, eventType, condition, handler) {
  parent.addEventListener(eventType, (event) => {
    const target = event.target.closest(selector);
    
    if (target && parent.contains(target) && condition(target, event)) {
      handler.call(target, event);
    }
  });
}

// Test
// delegateIf(list, '.item', 'click', 
//   (target) => !target.classList.contains('disabled'),
//   function(event) {
//     console.log('Enabled item clicked');
//   }
// );
```

**Interview Tips:**
- Event delegation attaches one listener to parent instead of many to children
- Benefits: better performance, handles dynamic content, less memory
- Use event.target to find clicked element, closest() to match selector
- Works due to event bubbling (events propagate up DOM tree)
- Common use cases: lists, tables, dynamic content, single-page apps
- Always check if target matches selector and is within parent
- Can delegate multiple event types and selectors
- Return cleanup function to remove listeners when done
- Considerations: not all events bubble (focus, blur need focusin, focusout)
- Modern alternative: element.matches(selector) for checking

</details>

55. Create a simple carousel/slider

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Carousel**
```javascript
/**
 * Simple image carousel
 * 
 * HTML Structure:
 * <div class="carousel">
 *   <div class="carousel-track">
 *     <div class="carousel-item"><img src="img1.jpg"></div>
 *     <div class="carousel-item"><img src="img2.jpg"></div>
 *     <div class="carousel-item"><img src="img3.jpg"></div>
 *   </div>
 *   <button class="prev">❮</button>
 *   <button class="next">❯</button>
 * </div>
 */

class Carousel {
  constructor(element) {
    this.carousel = element;
    this.track = element.querySelector('.carousel-track');
    this.items = Array.from(element.querySelectorAll('.carousel-item'));
    this.prevButton = element.querySelector('.prev');
    this.nextButton = element.querySelector('.next');
    
    this.currentIndex = 0;
    this.itemWidth = this.items[0].offsetWidth;
    
    this.init();
  }
  
  init() {
    // Set up event listeners
    this.prevButton.addEventListener('click', () => this.prev());
    this.nextButton.addEventListener('click', () => this.next());
    
    // Update button states
    this.updateButtons();
  }
  
  next() {
    if (this.currentIndex < this.items.length - 1) {
      this.currentIndex++;
      this.updatePosition();
    }
  }
  
  prev() {
    if (this.currentIndex > 0) {
      this.currentIndex--;
      this.updatePosition();
    }
  }
  
  updatePosition() {
    const offset = -this.currentIndex * this.itemWidth;
    this.track.style.transform = `translateX(${offset}px)`;
    this.updateButtons();
  }
  
  updateButtons() {
    this.prevButton.disabled = this.currentIndex === 0;
    this.nextButton.disabled = this.currentIndex === this.items.length - 1;
  }
}

// CSS for basic carousel
const carouselCSS = `
.carousel {
  position: relative;
  overflow: hidden;
  width: 100%;
}

.carousel-track {
  display: flex;
  transition: transform 0.3s ease;
}

.carousel-item {
  flex: 0 0 100%;
  min-width: 100%;
}

.carousel-item img {
  width: 100%;
  display: block;
}

.prev, .next {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(0, 0, 0, 0.5);
  color: white;
  border: none;
  padding: 10px 15px;
  cursor: pointer;
}

.prev { left: 10px; }
.next { right: 10px; }

.prev:disabled, .next:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}
`;

// Usage
// const carousel = new Carousel(document.querySelector('.carousel'));
```

### **Approach 2: With Auto-Play**
```javascript
/**
 * Carousel with auto-play functionality
 */
class AutoCarousel {
  constructor(element, options = {}) {
    this.carousel = element;
    this.track = element.querySelector('.carousel-track');
    this.items = Array.from(element.querySelectorAll('.carousel-item'));
    this.prevButton = element.querySelector('.prev');
    this.nextButton = element.querySelector('.next');
    
    this.currentIndex = 0;
    this.autoPlayInterval = options.autoPlay ? options.interval || 3000 : null;
    this.loop = options.loop !== false;
    this.intervalId = null;
    
    this.init();
  }
  
  init() {
    this.prevButton.addEventListener('click', () => {
      this.prev();
      this.resetAutoPlay();
    });
    
    this.nextButton.addEventListener('click', () => {
      this.next();
      this.resetAutoPlay();
    });
    
    // Pause on hover
    this.carousel.addEventListener('mouseenter', () => this.pause());
    this.carousel.addEventListener('mouseleave', () => this.resume());
    
    if (this.autoPlayInterval) {
      this.startAutoPlay();
    }
    
    this.updatePosition();
  }
  
  next() {
    if (this.currentIndex < this.items.length - 1) {
      this.currentIndex++;
    } else if (this.loop) {
      this.currentIndex = 0;
    }
    this.updatePosition();
  }
  
  prev() {
    if (this.currentIndex > 0) {
      this.currentIndex--;
    } else if (this.loop) {
      this.currentIndex = this.items.length - 1;
    }
    this.updatePosition();
  }
  
  goTo(index) {
    this.currentIndex = Math.max(0, Math.min(index, this.items.length - 1));
    this.updatePosition();
  }
  
  updatePosition() {
    const offset = -this.currentIndex * 100;
    this.track.style.transform = `translateX(${offset}%)`;
    this.updateButtons();
    this.updateIndicators();
  }
  
  updateButtons() {
    if (this.loop) {
      this.prevButton.disabled = false;
      this.nextButton.disabled = false;
    } else {
      this.prevButton.disabled = this.currentIndex === 0;
      this.nextButton.disabled = this.currentIndex === this.items.length - 1;
    }
  }
  
  updateIndicators() {
    const indicators = this.carousel.querySelectorAll('.indicator');
    indicators.forEach((indicator, index) => {
      indicator.classList.toggle('active', index === this.currentIndex);
    });
  }
  
  startAutoPlay() {
    this.intervalId = setInterval(() => this.next(), this.autoPlayInterval);
  }
  
  pause() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
  }
  
  resume() {
    if (this.autoPlayInterval && !this.intervalId) {
      this.startAutoPlay();
    }
  }
  
  resetAutoPlay() {
    this.pause();
    this.resume();
  }
  
  destroy() {
    this.pause();
    // Remove event listeners
  }
}

// Usage
// const carousel = new AutoCarousel(document.querySelector('.carousel'), {
//   autoPlay: true,
//   interval: 3000,
//   loop: true
// });
```

### **Approach 3: With Indicators and Touch Support**
```javascript
/**
 * Full-featured carousel with touch support
 */
class AdvancedCarousel {
  constructor(element, options = {}) {
    this.carousel = element;
    this.track = element.querySelector('.carousel-track');
    this.items = Array.from(element.querySelectorAll('.carousel-item'));
    this.prevButton = element.querySelector('.prev');
    this.nextButton = element.querySelector('.next');
    
    // Options
    this.currentIndex = 0;
    this.autoPlayInterval = options.autoPlay ? options.interval || 3000 : null;
    this.loop = options.loop !== false;
    this.showIndicators = options.indicators !== false;
    this.swipeEnabled = options.swipe !== false;
    
    // Touch tracking
    this.touchStartX = 0;
    this.touchEndX = 0;
    this.isDragging = false;
    
    this.init();
  }
  
  init() {
    // Navigation buttons
    this.prevButton?.addEventListener('click', () => {
      this.prev();
      this.resetAutoPlay();
    });
    
    this.nextButton?.addEventListener('click', () => {
      this.next();
      this.resetAutoPlay();
    });
    
    // Create indicators
    if (this.showIndicators) {
      this.createIndicators();
    }
    
    // Touch/Swipe support
    if (this.swipeEnabled) {
      this.setupSwipe();
    }
    
    // Keyboard navigation
    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowLeft') this.prev();
      if (e.key === 'ArrowRight') this.next();
    });
    
    // Auto-play
    this.carousel.addEventListener('mouseenter', () => this.pause());
    this.carousel.addEventListener('mouseleave', () => this.resume());
    
    if (this.autoPlayInterval) {
      this.startAutoPlay();
    }
    
    this.updatePosition();
  }
  
  createIndicators() {
    const indicatorContainer = document.createElement('div');
    indicatorContainer.className = 'carousel-indicators';
    
    this.items.forEach((_, index) => {
      const indicator = document.createElement('button');
      indicator.className = 'indicator';
      indicator.setAttribute('aria-label', `Go to slide ${index + 1}`);
      indicator.addEventListener('click', () => {
        this.goTo(index);
        this.resetAutoPlay();
      });
      indicatorContainer.appendChild(indicator);
    });
    
    this.carousel.appendChild(indicatorContainer);
  }
  
  setupSwipe() {
    this.track.addEventListener('touchstart', (e) => {
      this.touchStartX = e.touches[0].clientX;
      this.isDragging = true;
      this.pause();
    }, { passive: true });
    
    this.track.addEventListener('touchmove', (e) => {
      if (!this.isDragging) return;
      this.touchEndX = e.touches[0].clientX;
    }, { passive: true });
    
    this.track.addEventListener('touchend', () => {
      if (!this.isDragging) return;
      
      const diff = this.touchStartX - this.touchEndX;
      const threshold = 50; // Minimum swipe distance
      
      if (Math.abs(diff) > threshold) {
        if (diff > 0) {
          this.next();
        } else {
          this.prev();
        }
      }
      
      this.isDragging = false;
      this.resume();
    });
    
    // Mouse drag support
    let mouseStartX = 0;
    let mouseDown = false;
    
    this.track.addEventListener('mousedown', (e) => {
      mouseStartX = e.clientX;
      mouseDown = true;
      this.track.style.cursor = 'grabbing';
    });
    
    this.track.addEventListener('mousemove', (e) => {
      if (!mouseDown) return;
      e.preventDefault();
    });
    
    this.track.addEventListener('mouseup', (e) => {
      if (!mouseDown) return;
      
      const diff = mouseStartX - e.clientX;
      const threshold = 50;
      
      if (Math.abs(diff) > threshold) {
        if (diff > 0) {
          this.next();
        } else {
          this.prev();
        }
      }
      
      mouseDown = false;
      this.track.style.cursor = 'grab';
    });
  }
  
  next() {
    if (this.currentIndex < this.items.length - 1) {
      this.currentIndex++;
    } else if (this.loop) {
      this.currentIndex = 0;
    }
    this.updatePosition();
  }
  
  prev() {
    if (this.currentIndex > 0) {
      this.currentIndex--;
    } else if (this.loop) {
      this.currentIndex = this.items.length - 1;
    }
    this.updatePosition();
  }
  
  goTo(index) {
    this.currentIndex = Math.max(0, Math.min(index, this.items.length - 1));
    this.updatePosition();
  }
  
  updatePosition() {
    const offset = -this.currentIndex * 100;
    this.track.style.transform = `translateX(${offset}%)`;
    this.updateButtons();
    this.updateIndicators();
  }
  
  updateButtons() {
    if (!this.prevButton || !this.nextButton) return;
    
    if (this.loop) {
      this.prevButton.disabled = false;
      this.nextButton.disabled = false;
    } else {
      this.prevButton.disabled = this.currentIndex === 0;
      this.nextButton.disabled = this.currentIndex === this.items.length - 1;
    }
  }
  
  updateIndicators() {
    const indicators = this.carousel.querySelectorAll('.indicator');
    indicators.forEach((indicator, index) => {
      indicator.classList.toggle('active', index === this.currentIndex);
    });
  }
  
  startAutoPlay() {
    this.intervalId = setInterval(() => this.next(), this.autoPlayInterval);
  }
  
  pause() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
  }
  
  resume() {
    if (this.autoPlayInterval && !this.intervalId) {
      this.startAutoPlay();
    }
  }
  
  resetAutoPlay() {
    this.pause();
    this.resume();
  }
  
  destroy() {
    this.pause();
    // Clean up event listeners
  }
}

// Complete CSS
const advancedCarouselCSS = `
.carousel {
  position: relative;
  overflow: hidden;
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

.carousel-track {
  display: flex;
  transition: transform 0.4s ease;
  cursor: grab;
}

.carousel-track:active {
  cursor: grabbing;
}

.carousel-item {
  flex: 0 0 100%;
  min-width: 100%;
}

.carousel-item img {
  width: 100%;
  height: auto;
  display: block;
}

.prev, .next {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(0, 0, 0, 0.5);
  color: white;
  border: none;
  padding: 12px 18px;
  cursor: pointer;
  font-size: 20px;
  z-index: 10;
  transition: background 0.3s;
}

.prev:hover, .next:hover {
  background: rgba(0, 0, 0, 0.8);
}

.prev:disabled, .next:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.prev { left: 10px; }
.next { right: 10px; }

.carousel-indicators {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 10px;
  z-index: 10;
}

.indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.5);
  border: none;
  cursor: pointer;
  transition: background 0.3s;
}

.indicator:hover {
  background: rgba(255, 255, 255, 0.8);
}

.indicator.active {
  background: white;
}
`;

// Usage examples
console.log('=== Basic Carousel ===');
// const carousel1 = new Carousel(document.querySelector('#carousel1'));

console.log('\n=== Auto-Play Carousel ===');
// const carousel2 = new AutoCarousel(document.querySelector('#carousel2'), {
//   autoPlay: true,
//   interval: 3000,
//   loop: true
// });

console.log('\n=== Advanced Carousel ===');
// const carousel3 = new AdvancedCarousel(document.querySelector('#carousel3'), {
//   autoPlay: true,
//   interval: 4000,
//   loop: true,
//   indicators: true,
//   swipe: true
// });

// HTML Template
const carouselHTML = `
<div class="carousel" id="myCarousel">
  <div class="carousel-track">
    <div class="carousel-item">
      <img src="image1.jpg" alt="Slide 1">
    </div>
    <div class="carousel-item">
      <img src="image2.jpg" alt="Slide 2">
    </div>
    <div class="carousel-item">
      <img src="image3.jpg" alt="Slide 3">
    </div>
  </div>
  <button class="prev" aria-label="Previous slide">❮</button>
  <button class="next" aria-label="Next slide">❯</button>
</div>
`;
```

### **Bonus: Fade Carousel**
```javascript
/**
 * Carousel with fade transition instead of slide
 */
class FadeCarousel {
  constructor(element, options = {}) {
    this.carousel = element;
    this.items = Array.from(element.querySelectorAll('.carousel-item'));
    this.currentIndex = 0;
    this.autoPlayInterval = options.interval || 3000;
    
    this.init();
  }
  
  init() {
    // Hide all items except first
    this.items.forEach((item, index) => {
      item.style.opacity = index === 0 ? '1' : '0';
      item.style.position = 'absolute';
      item.style.top = '0';
      item.style.left = '0';
      item.style.width = '100%';
      item.style.transition = 'opacity 0.5s ease';
    });
    
    // Start auto-play
    this.startAutoPlay();
    
    // Pause on hover
    this.carousel.addEventListener('mouseenter', () => this.pause());
    this.carousel.addEventListener('mouseleave', () => this.resume());
  }
  
  next() {
    this.items[this.currentIndex].style.opacity = '0';
    this.currentIndex = (this.currentIndex + 1) % this.items.length;
    this.items[this.currentIndex].style.opacity = '1';
  }
  
  startAutoPlay() {
    this.intervalId = setInterval(() => this.next(), this.autoPlayInterval);
  }
  
  pause() {
    clearInterval(this.intervalId);
  }
  
  resume() {
    this.startAutoPlay();
  }
}
```

**Interview Tips:**
- Basic carousel: translate track horizontally based on current index
- Use CSS transform for smooth animations (better than left/right)
- Key features: prev/next buttons, indicators, auto-play, loop
- Touch support: listen to touchstart/touchmove/touchend events
- Auto-play: use setInterval, pause on hover, reset on manual navigation
- Accessibility: ARIA labels, keyboard navigation, focus management
- Performance: use transform instead of margin/left for GPU acceleration
- Common variations: fade, vertical, multiple items visible, infinite loop
- Responsive: use percentages for positioning, handle window resize
- Libraries: Swiper.js, Slick, Glide.js for production

</details>

56. Implement infinite scroll

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Scroll Event Listener**
```javascript
/**
 * Basic infinite scroll using scroll event
 * 
 * @param {Function} loadMore - Function to load more content
 * @param {Object} options - Scroll options
 */
function infiniteScroll(loadMore, options = {}) {
  const {
    threshold = 200, // Distance from bottom to trigger load
    container = window
  } = options;
  
  let loading = false;
  
  const handleScroll = () => {
    if (loading) return;
    
    const scrollTop = container === window 
      ? window.pageYOffset 
      : container.scrollTop;
    
    const scrollHeight = container === window
      ? document.documentElement.scrollHeight
      : container.scrollHeight;
    
    const clientHeight = container === window
      ? window.innerHeight
      : container.clientHeight;
    
    const distanceFromBottom = scrollHeight - (scrollTop + clientHeight);
    
    if (distanceFromBottom < threshold) {
      loading = true;
      
      loadMore()
        .then(() => {
          loading = false;
        })
        .catch(error => {
          console.error('Error loading more:', error);
          loading = false;
        });
    }
  };
  
  container.addEventListener('scroll', handleScroll);
  
  // Return cleanup function
  return () => container.removeEventListener('scroll', handleScroll);
}

// Usage
// const cleanup = infiniteScroll(async () => {
//   const response = await fetch('/api/items?page=' + currentPage);
//   const items = await response.json();
//   renderItems(items);
//   currentPage++;
// }, { threshold: 300 });
```

### **Approach 2: Intersection Observer API**
```javascript
/**
 * Infinite scroll using Intersection Observer (modern approach)
 */
class InfiniteScroll {
  constructor(options = {}) {
    this.container = options.container || document.querySelector('.content');
    this.loadMore = options.loadMore;
    this.threshold = options.threshold || 0.8;
    this.rootMargin = options.rootMargin || '200px';
    
    this.loading = false;
    this.hasMore = true;
    
    this.init();
  }
  
  init() {
    // Create sentinel element at bottom
    this.sentinel = document.createElement('div');
    this.sentinel.className = 'infinite-scroll-sentinel';
    this.container.appendChild(this.sentinel);
    
    // Set up Intersection Observer
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersection(entries),
      {
        root: null, // viewport
        rootMargin: this.rootMargin,
        threshold: this.threshold
      }
    );
    
    this.observer.observe(this.sentinel);
  }
  
  async handleIntersection(entries) {
    const entry = entries[0];
    
    if (entry.isIntersecting && !this.loading && this.hasMore) {
      this.loading = true;
      
      try {
        const hasMore = await this.loadMore();
        
        if (hasMore === false) {
          this.hasMore = false;
          this.destroy();
        }
      } catch (error) {
        console.error('Error loading more:', error);
      } finally {
        this.loading = false;
      }
    }
  }
  
  destroy() {
    if (this.observer) {
      this.observer.disconnect();
      this.observer = null;
    }
    
    if (this.sentinel && this.sentinel.parentNode) {
      this.sentinel.parentNode.removeChild(this.sentinel);
    }
  }
}

// Usage
// const scroll = new InfiniteScroll({
//   container: document.querySelector('.feed'),
//   loadMore: async () => {
//     const response = await fetch(`/api/posts?page=${currentPage}`);
//     const data = await response.json();
//     
//     if (data.posts.length === 0) {
//       return false; // No more items
//     }
//     
//     renderPosts(data.posts);
//     currentPage++;
//     return true;
//   }
// });
```

### **Approach 3: With Throttle and Loading State**
```javascript
/**
 * Production-ready infinite scroll with throttle
 */
class AdvancedInfiniteScroll {
  constructor(options = {}) {
    this.container = options.container || window;
    this.contentContainer = options.contentContainer || document.querySelector('.content');
    this.loadMore = options.loadMore;
    this.threshold = options.threshold || 200;
    this.throttleDelay = options.throttleDelay || 200;
    
    this.loading = false;
    this.hasMore = true;
    this.currentPage = 1;
    
    this.createLoadingIndicator();
    this.init();
  }
  
  init() {
    // Throttle scroll handler
    this.handleScroll = this.throttle(() => {
      this.checkAndLoad();
    }, this.throttleDelay);
    
    this.container.addEventListener('scroll', this.handleScroll);
    
    // Initial check in case content is too short
    this.checkAndLoad();
  }
  
  throttle(func, delay) {
    let lastCall = 0;
    return function(...args) {
      const now = Date.now();
      if (now - lastCall >= delay) {
        lastCall = now;
        return func.apply(this, args);
      }
    };
  }
  
  checkAndLoad() {
    if (this.loading || !this.hasMore) return;
    
    const scrollTop = this.container === window 
      ? window.pageYOffset 
      : this.container.scrollTop;
    
    const scrollHeight = this.container === window
      ? document.documentElement.scrollHeight
      : this.container.scrollHeight;
    
    const clientHeight = this.container === window
      ? window.innerHeight
      : this.container.clientHeight;
    
    const distanceFromBottom = scrollHeight - (scrollTop + clientHeight);
    
    if (distanceFromBottom < this.threshold) {
      this.load();
    }
  }
  
  async load() {
    if (this.loading) return;
    
    this.loading = true;
    this.showLoading();
    
    try {
      const result = await this.loadMore(this.currentPage);
      
      if (!result || result.length === 0) {
        this.hasMore = false;
        this.showNoMoreContent();
      } else {
        this.currentPage++;
        // Check again in case content is still not enough to scroll
        setTimeout(() => this.checkAndLoad(), 100);
      }
    } catch (error) {
      console.error('Error loading content:', error);
      this.showError(error);
    } finally {
      this.loading = false;
      this.hideLoading();
    }
  }
  
  createLoadingIndicator() {
    this.loadingElement = document.createElement('div');
    this.loadingElement.className = 'infinite-scroll-loading';
    this.loadingElement.innerHTML = '<div class="spinner"></div>';
    this.loadingElement.style.display = 'none';
    
    if (this.contentContainer) {
      this.contentContainer.appendChild(this.loadingElement);
    }
  }
  
  showLoading() {
    if (this.loadingElement) {
      this.loadingElement.style.display = 'block';
    }
  }
  
  hideLoading() {
    if (this.loadingElement) {
      this.loadingElement.style.display = 'none';
    }
  }
  
  showNoMoreContent() {
    const message = document.createElement('div');
    message.className = 'no-more-content';
    message.textContent = 'No more items to load';
    
    if (this.contentContainer) {
      this.contentContainer.appendChild(message);
    }
  }
  
  showError(error) {
    const errorElement = document.createElement('div');
    errorElement.className = 'infinite-scroll-error';
    errorElement.innerHTML = `
      <p>Error loading content</p>
      <button class="retry-button">Retry</button>
    `;
    
    errorElement.querySelector('.retry-button').addEventListener('click', () => {
      errorElement.remove();
      this.load();
    });
    
    if (this.contentContainer) {
      this.contentContainer.appendChild(errorElement);
    }
  }
  
  destroy() {
    this.container.removeEventListener('scroll', this.handleScroll);
    
    if (this.loadingElement && this.loadingElement.parentNode) {
      this.loadingElement.parentNode.removeChild(this.loadingElement);
    }
  }
  
  reset() {
    this.currentPage = 1;
    this.hasMore = true;
    this.loading = false;
  }
}

// Complete example with API integration
class FeedWithInfiniteScroll {
  constructor(containerId) {
    this.container = document.getElementById(containerId);
    this.items = [];
    
    this.infiniteScroll = new AdvancedInfiniteScroll({
      contentContainer: this.container,
      loadMore: (page) => this.loadItems(page),
      threshold: 300
    });
  }
  
  async loadItems(page) {
    // Simulate API call
    const response = await fetch(`/api/items?page=${page}&limit=20`);
    const data = await response.json();
    
    if (data.items.length > 0) {
      this.renderItems(data.items);
      return data.items;
    }
    
    return null;
  }
  
  renderItems(items) {
    items.forEach(item => {
      const element = document.createElement('div');
      element.className = 'item';
      element.innerHTML = `
        <h3>${item.title}</h3>
        <p>${item.description}</p>
      `;
      this.container.appendChild(element);
    });
  }
}

// CSS for loading indicators
const infiniteScrollCSS = `
.infinite-scroll-loading {
  text-align: center;
  padding: 20px;
}

.spinner {
  width: 40px;
  height: 40px;
  margin: 0 auto;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.no-more-content {
  text-align: center;
  padding: 20px;
  color: #666;
}

.infinite-scroll-error {
  text-align: center;
  padding: 20px;
  color: #d32f2f;
}

.retry-button {
  margin-top: 10px;
  padding: 8px 16px;
  background: #3498db;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.retry-button:hover {
  background: #2980b9;
}
`;

// Usage examples
console.log('=== Basic Infinite Scroll ===');
// let currentPage = 1;
// infiniteScroll(async () => {
//   const items = await fetchItems(currentPage);
//   renderItems(items);
//   currentPage++;
// });

console.log('\n=== Intersection Observer ===');
// const scroll = new InfiniteScroll({
//   loadMore: async () => {
//     const items = await fetchItems();
//     return items.length > 0;
//   }
// });

console.log('\n=== Advanced with Features ===');
// const feed = new AdvancedInfiniteScroll({
//   container: window,
//   contentContainer: document.querySelector('.feed'),
//   loadMore: async (page) => {
//     const response = await fetch(`/api/posts?page=${page}`);
//     const data = await response.json();
//     renderPosts(data.posts);
//     return data.posts;
//   },
//   threshold: 300,
//   throttleDelay: 200
// });
```

### **Bonus: Virtual Scrolling**
```javascript
/**
 * Virtual scrolling for massive lists (only render visible items)
 */
class VirtualScroll {
  constructor(container, options = {}) {
    this.container = container;
    this.items = [];
    this.itemHeight = options.itemHeight || 50;
    this.buffer = options.buffer || 5;
    this.renderItem = options.renderItem;
    
    this.visibleStart = 0;
    this.visibleEnd = 0;
    
    this.init();
  }
  
  init() {
    this.viewport = document.createElement('div');
    this.viewport.style.overflow = 'auto';
    this.viewport.style.height = '100%';
    
    this.content = document.createElement('div');
    this.viewport.appendChild(this.content);
    this.container.appendChild(this.viewport);
    
    this.viewport.addEventListener('scroll', () => this.onScroll());
  }
  
  setItems(items) {
    this.items = items;
    this.content.style.height = (items.length * this.itemHeight) + 'px';
    this.onScroll();
  }
  
  onScroll() {
    const scrollTop = this.viewport.scrollTop;
    const viewportHeight = this.viewport.clientHeight;
    
    this.visibleStart = Math.max(0, Math.floor(scrollTop / this.itemHeight) - this.buffer);
    this.visibleEnd = Math.min(
      this.items.length,
      Math.ceil((scrollTop + viewportHeight) / this.itemHeight) + this.buffer
    );
    
    this.render();
  }
  
  render() {
    this.content.innerHTML = '';
    
    const offset = this.visibleStart * this.itemHeight;
    this.content.style.paddingTop = offset + 'px';
    
    for (let i = this.visibleStart; i < this.visibleEnd; i++) {
      const element = this.renderItem(this.items[i], i);
      this.content.appendChild(element);
    }
  }
}
```

**Interview Tips:**
- Infinite scroll loads content as user scrolls down
- Use Intersection Observer (modern) or scroll event (traditional)
- Throttle scroll events to improve performance (~200ms)
- Show loading indicator while fetching data
- Handle errors with retry button
- Track loading state to prevent duplicate requests
- Detect "no more content" condition
- Threshold: distance from bottom to trigger load (200-300px typical)
- Accessibility: ensure keyboard navigation works
- Performance: virtual scrolling for massive lists, lazy load images
- Common issues: too short content, API errors, race conditions
- Alternative: pagination for better UX in some cases

</details>

57. Create a star rating component

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Star Rating**
```javascript
/**
 * Basic star rating component
 * 
 * HTML:
 * <div class="star-rating" data-rating="0"></div>
 */
class StarRating {
  constructor(element, options = {}) {
    this.element = element;
    this.maxStars = options.maxStars || 5;
    this.currentRating = options.initialRating || 0;
    this.readonly = options.readonly || false;
    this.onChange = options.onChange || (() => {});
    
    this.init();
  }
  
  init() {
    this.element.innerHTML = '';
    this.stars = [];
    
    for (let i = 1; i <= this.maxStars; i++) {
      const star = document.createElement('span');
      star.className = 'star';
      star.innerHTML = '★';
      star.dataset.value = i;
      
      if (!this.readonly) {
        star.addEventListener('click', () => this.setRating(i));
        star.addEventListener('mouseenter', () => this.highlightStars(i));
      }
      
      this.element.appendChild(star);
      this.stars.push(star);
    }
    
    if (!this.readonly) {
      this.element.addEventListener('mouseleave', () => {
        this.highlightStars(this.currentRating);
      });
    }
    
    this.setRating(this.currentRating);
  }
  
  setRating(rating) {
    this.currentRating = rating;
    this.highlightStars(rating);
    this.onChange(rating);
  }
  
  highlightStars(rating) {
    this.stars.forEach((star, index) => {
      if (index < rating) {
        star.classList.add('active');
      } else {
        star.classList.remove('active');
      }
    });
  }
  
  getRating() {
    return this.currentRating;
  }
  
  reset() {
    this.setRating(0);
  }
}

// CSS
const starRatingCSS = `
.star-rating {
  display: inline-flex;
  gap: 5px;
  cursor: pointer;
}

.star {
  font-size: 30px;
  color: #ddd;
  transition: color 0.2s;
}

.star.active {
  color: #ffd700;
}

.star:hover {
  color: #ffed4e;
}

.star-rating.readonly {
  cursor: default;
}

.star-rating.readonly .star:hover {
  color: inherit;
}
`;

// Usage
// const rating = new StarRating(document.querySelector('.star-rating'), {
//   maxStars: 5,
//   initialRating: 3,
//   onChange: (rating) => {
//     console.log('New rating:', rating);
//   }
// });
```

### **Approach 2: With Half Stars**
```javascript
/**
 * Star rating with half-star support
 */
class HalfStarRating {
  constructor(element, options = {}) {
    this.element = element;
    this.maxStars = options.maxStars || 5;
    this.currentRating = options.initialRating || 0;
    this.allowHalf = options.allowHalf !== false;
    this.readonly = options.readonly || false;
    this.onChange = options.onChange || (() => {});
    
    this.init();
  }
  
  init() {
    this.element.innerHTML = '';
    this.element.className = 'star-rating-half';
    
    if (this.readonly) {
      this.element.classList.add('readonly');
    }
    
    this.stars = [];
    
    for (let i = 1; i <= this.maxStars; i++) {
      const starContainer = document.createElement('span');
      starContainer.className = 'star-container';
      starContainer.dataset.value = i;
      
      const starFull = document.createElement('span');
      starFull.className = 'star star-full';
      starFull.innerHTML = '★';
      
      const starEmpty = document.createElement('span');
      starEmpty.className = 'star star-empty';
      starEmpty.innerHTML = '★';
      
      starContainer.appendChild(starEmpty);
      starContainer.appendChild(starFull);
      
      if (!this.readonly) {
        // Click handlers for full and half stars
        starContainer.addEventListener('click', (e) => {
          const rect = starContainer.getBoundingClientRect();
          const x = e.clientX - rect.left;
          const isHalf = this.allowHalf && x < rect.width / 2;
          this.setRating(i - (isHalf ? 0.5 : 0));
        });
        
        starContainer.addEventListener('mousemove', (e) => {
          const rect = starContainer.getBoundingClientRect();
          const x = e.clientX - rect.left;
          const isHalf = this.allowHalf && x < rect.width / 2;
          this.highlightStars(i - (isHalf ? 0.5 : 0));
        });
      }
      
      this.element.appendChild(starContainer);
      this.stars.push(starContainer);
    }
    
    if (!this.readonly) {
      this.element.addEventListener('mouseleave', () => {
        this.highlightStars(this.currentRating);
      });
    }
    
    this.setRating(this.currentRating);
  }
  
  setRating(rating) {
    this.currentRating = Math.max(0, Math.min(this.maxStars, rating));
    this.highlightStars(this.currentRating);
    this.onChange(this.currentRating);
  }
  
  highlightStars(rating) {
    this.stars.forEach((star, index) => {
      const starValue = index + 1;
      const starFull = star.querySelector('.star-full');
      
      if (rating >= starValue) {
        starFull.style.width = '100%';
      } else if (rating > starValue - 1 && rating < starValue) {
        const percentage = ((rating - (starValue - 1)) * 100) + '%';
        starFull.style.width = percentage;
      } else {
        starFull.style.width = '0%';
      }
    });
  }
  
  getRating() {
    return this.currentRating;
  }
}

// CSS for half stars
const halfStarCSS = `
.star-rating-half {
  display: inline-flex;
  gap: 5px;
}

.star-container {
  position: relative;
  display: inline-block;
  font-size: 30px;
  cursor: pointer;
}

.star {
  transition: color 0.2s;
}

.star-empty {
  color: #ddd;
}

.star-full {
  position: absolute;
  top: 0;
  left: 0;
  color: #ffd700;
  overflow: hidden;
  width: 0%;
  transition: width 0.2s;
}

.star-rating-half.readonly {
  cursor: default;
}
`;
```

### **Approach 3: Production-Grade with Features**
```javascript
/**
 * Full-featured star rating component
 */
class AdvancedStarRating {
  constructor(element, options = {}) {
    this.element = element;
    this.maxStars = options.maxStars || 5;
    this.currentRating = options.initialRating || 0;
    this.allowHalf = options.allowHalf || false;
    this.allowClear = options.allowClear || false;
    this.readonly = options.readonly || false;
    this.showValue = options.showValue || false;
    this.starChar = options.starChar || '★';
    this.emptyStarChar = options.emptyStarChar || '☆';
    this.size = options.size || 'medium'; // small, medium, large
    this.color = options.color || '#ffd700';
    this.onChange = options.onChange || (() => {});
    this.onHover = options.onHover || (() => {});
    
    this.hoverRating = 0;
    this.init();
  }
  
  init() {
    this.element.innerHTML = '';
    this.element.className = `star-rating star-rating-${this.size}`;
    
    if (this.readonly) {
      this.element.classList.add('readonly');
    }
    
    // Create star container
    this.starsContainer = document.createElement('div');
    this.starsContainer.className = 'stars-container';
    this.element.appendChild(this.starsContainer);
    
    this.stars = [];
    this.createStars();
    
    // Create value display
    if (this.showValue) {
      this.valueDisplay = document.createElement('span');
      this.valueDisplay.className = 'rating-value';
      this.element.appendChild(this.valueDisplay);
      this.updateValueDisplay();
    }
    
    // Set initial rating
    this.setRating(this.currentRating, false);
  }
  
  createStars() {
    for (let i = 1; i <= this.maxStars; i++) {
      const starWrapper = document.createElement('span');
      starWrapper.className = 'star-wrapper';
      starWrapper.dataset.value = i;
      
      // Create star elements
      const starEmpty = document.createElement('span');
      starEmpty.className = 'star star-empty';
      starEmpty.innerHTML = this.emptyStarChar;
      
      const starFilled = document.createElement('span');
      starFilled.className = 'star star-filled';
      starFilled.innerHTML = this.starChar;
      starFilled.style.color = this.color;
      
      starWrapper.appendChild(starEmpty);
      starWrapper.appendChild(starFilled);
      
      if (!this.readonly) {
        starWrapper.addEventListener('click', (e) => this.handleClick(e, i));
        starWrapper.addEventListener('mousemove', (e) => this.handleMouseMove(e, i));
        starWrapper.addEventListener('mouseenter', () => {
          this.element.classList.add('hovering');
        });
      }
      
      this.starsContainer.appendChild(starWrapper);
      this.stars.push(starWrapper);
    }
    
    if (!this.readonly) {
      this.element.addEventListener('mouseleave', () => {
        this.element.classList.remove('hovering');
        this.highlightStars(this.currentRating);
        this.onHover(this.currentRating);
      });
    }
  }
  
  handleClick(event, starIndex) {
    if (this.readonly) return;
    
    let rating = starIndex;
    
    if (this.allowHalf) {
      const rect = event.currentTarget.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const isLeftHalf = x < rect.width / 2;
      
      if (isLeftHalf) {
        rating = starIndex - 0.5;
      }
    }
    
    // Allow clearing rating by clicking same star
    if (this.allowClear && this.currentRating === rating) {
      rating = 0;
    }
    
    this.setRating(rating, true);
  }
  
  handleMouseMove(event, starIndex) {
    if (this.readonly) return;
    
    let rating = starIndex;
    
    if (this.allowHalf) {
      const rect = event.currentTarget.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const isLeftHalf = x < rect.width / 2;
      
      if (isLeftHalf) {
        rating = starIndex - 0.5;
      }
    }
    
    this.hoverRating = rating;
    this.highlightStars(rating);
    this.onHover(rating);
  }
  
  setRating(rating, triggerChange = true) {
    this.currentRating = Math.max(0, Math.min(this.maxStars, rating));
    this.highlightStars(this.currentRating);
    this.updateValueDisplay();
    
    if (triggerChange) {
      this.onChange(this.currentRating);
    }
  }
  
  highlightStars(rating) {
    this.stars.forEach((star, index) => {
      const starValue = index + 1;
      const starFilled = star.querySelector('.star-filled');
      
      if (rating >= starValue) {
        starFilled.style.width = '100%';
        starFilled.style.opacity = '1';
      } else if (rating > starValue - 1 && rating < starValue) {
        const percentage = ((rating - (starValue - 1)) * 100);
        starFilled.style.width = percentage + '%';
        starFilled.style.opacity = '1';
      } else {
        starFilled.style.width = '0%';
        starFilled.style.opacity = '0';
      }
    });
  }
  
  updateValueDisplay() {
    if (this.valueDisplay) {
      this.valueDisplay.textContent = this.currentRating.toFixed(1);
    }
  }
  
  getRating() {
    return this.currentRating;
  }
  
  reset() {
    this.setRating(0);
  }
  
  destroy() {
    this.element.innerHTML = '';
  }
}

// Complete CSS
const advancedStarCSS = `
.star-rating {
  display: inline-flex;
  align-items: center;
  gap: 10px;
}

.stars-container {
  display: inline-flex;
  gap: 2px;
}

.star-wrapper {
  position: relative;
  display: inline-block;
  cursor: pointer;
}

.star {
  display: inline-block;
  transition: all 0.2s ease;
}

.star-empty {
  color: #ddd;
}

.star-filled {
  position: absolute;
  top: 0;
  left: 0;
  color: #ffd700;
  overflow: hidden;
  width: 0%;
  white-space: nowrap;
}

.star-rating-small .star {
  font-size: 18px;
}

.star-rating-medium .star {
  font-size: 24px;
}

.star-rating-large .star {
  font-size: 32px;
}

.star-rating.readonly .star-wrapper {
  cursor: default;
}

.star-rating.hovering .star-wrapper:hover .star-filled {
  transform: scale(1.2);
}

.rating-value {
  font-weight: bold;
  font-size: 18px;
  color: #333;
}
`;

// Usage examples
console.log('=== Basic Star Rating ===');
// const rating1 = new StarRating(document.querySelector('#rating1'), {
//   maxStars: 5,
//   initialRating: 3,
//   onChange: (rating) => console.log('Rating:', rating)
// });

console.log('\n=== Half Star Rating ===');
// const rating2 = new HalfStarRating(document.querySelector('#rating2'), {
//   maxStars: 5,
//   initialRating: 3.5,
//   allowHalf: true,
//   onChange: (rating) => console.log('Rating:', rating)
// });

console.log('\n=== Advanced Rating ===');
// const rating3 = new AdvancedStarRating(document.querySelector('#rating3'), {
//   maxStars: 5,
//   initialRating: 4.5,
//   allowHalf: true,
//   allowClear: true,
//   showValue: true,
//   size: 'large',
//   color: '#ff6b6b',
//   onChange: (rating) => console.log('Rating changed:', rating),
//   onHover: (rating) => console.log('Hovering:', rating)
// });

// HTML template
const ratingHTML = `
<div class="rating-container">
  <h3>Rate this product:</h3>
  <div id="product-rating" class="star-rating"></div>
  <p class="rating-text">Click to rate</p>
</div>
`;

// Form integration example
function setupRatingForm() {
  const form = document.querySelector('#review-form');
  const ratingElement = document.querySelector('#rating');
  
  const rating = new AdvancedStarRating(ratingElement, {
    maxStars: 5,
    allowHalf: false,
    showValue: true,
    onChange: (value) => {
      // Update hidden form field
      form.querySelector('input[name="rating"]').value = value;
    }
  });
  
  form.addEventListener('submit', (e) => {
    e.preventDefault();
    
    const ratingValue = rating.getRating();
    
    if (ratingValue === 0) {
      alert('Please select a rating');
      return;
    }
    
    console.log('Submitting rating:', ratingValue);
    // Submit form
  });
}
```

### **Bonus: Animated Stars**
```javascript
/**
 * Star rating with animations
 */
class AnimatedStarRating extends AdvancedStarRating {
  handleClick(event, starIndex) {
    super.handleClick(event, starIndex);
    
    // Add animation effect
    const star = this.stars[starIndex - 1];
    star.classList.add('star-pulse');
    
    setTimeout(() => {
      star.classList.remove('star-pulse');
    }, 600);
  }
}

// Animation CSS
const animationCSS = `
@keyframes star-pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.3);
  }
}

.star-pulse {
  animation: star-pulse 0.6s ease;
}
`;
```

**Interview Tips:**
- Star rating: interactive UI for user ratings (1-5 stars typical)
- Basic approach: click star to set rating, hover to preview
- Half stars: detect click position or separate click handlers
- Features: readonly mode, allow clear, show value, custom colors
- CSS: use absolute positioning for filled stars overlaying empty stars
- Events: click (set rating), hover (preview), change callback
- Accessibility: keyboard navigation, ARIA labels, focus states
- Considerations: mobile touch targets (larger size), form integration
- Common variations: hearts/thumbs, custom icons, animated feedback
- Libraries: react-rating, vue-star-rating for production use

</details>

58. Build an autocomplete/typeahead component

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Autocomplete**
```javascript
/**
 * Simple autocomplete component
 * 
 * HTML:
 * <div class="autocomplete">
 *   <input type="text" id="search" placeholder="Search...">
 *   <ul class="suggestions"></ul>
 * </div>
 */
class Autocomplete {
  constructor(input, options = {}) {
    this.input = input;
    this.suggestions = options.suggestions || [];
    this.onSelect = options.onSelect || (() => {});
    this.minChars = options.minChars || 1;
    
    this.suggestionsList = this.createSuggestionsList();
    this.currentFocus = -1;
    
    this.init();
  }
  
  createSuggestionsList() {
    const ul = document.createElement('ul');
    ul.className = 'autocomplete-suggestions';
    ul.style.display = 'none';
    this.input.parentNode.appendChild(ul);
    return ul;
  }
  
  init() {
    // Input event listener
    this.input.addEventListener('input', (e) => {
      this.handleInput(e.target.value);
    });
    
    // Keyboard navigation
    this.input.addEventListener('keydown', (e) => {
      this.handleKeydown(e);
    });
    
    // Close on click outside
    document.addEventListener('click', (e) => {
      if (!this.input.contains(e.target) && !this.suggestionsList.contains(e.target)) {
        this.closeSuggestions();
      }
    });
  }
  
  handleInput(value) {
    this.currentFocus = -1;
    
    if (value.length < this.minChars) {
      this.closeSuggestions();
      return;
    }
    
    const filtered = this.filterSuggestions(value);
    this.renderSuggestions(filtered, value);
  }
  
  filterSuggestions(value) {
    const lowerValue = value.toLowerCase();
    return this.suggestions.filter(item => 
      item.toLowerCase().includes(lowerValue)
    );
  }
  
  renderSuggestions(items, query) {
    this.suggestionsList.innerHTML = '';
    
    if (items.length === 0) {
      this.closeSuggestions();
      return;
    }
    
    items.forEach(item => {
      const li = document.createElement('li');
      li.textContent = item;
      li.className = 'suggestion-item';
      
      li.addEventListener('click', () => {
        this.selectItem(item);
      });
      
      this.suggestionsList.appendChild(li);
    });
    
    this.suggestionsList.style.display = 'block';
  }
  
  handleKeydown(e) {
    const items = this.suggestionsList.querySelectorAll('.suggestion-item');
    
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      this.currentFocus++;
      this.setActive(items);
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      this.currentFocus--;
      this.setActive(items);
    } else if (e.key === 'Enter') {
      e.preventDefault();
      if (this.currentFocus > -1 && items[this.currentFocus]) {
        this.selectItem(items[this.currentFocus].textContent);
      }
    } else if (e.key === 'Escape') {
      this.closeSuggestions();
    }
  }
  
  setActive(items) {
    if (!items.length) return;
    
    // Remove active class from all
    items.forEach(item => item.classList.remove('active'));
    
    // Wrap around
    if (this.currentFocus >= items.length) this.currentFocus = 0;
    if (this.currentFocus < 0) this.currentFocus = items.length - 1;
    
    // Add active class
    items[this.currentFocus].classList.add('active');
    items[this.currentFocus].scrollIntoView({ block: 'nearest' });
  }
  
  selectItem(item) {
    this.input.value = item;
    this.closeSuggestions();
    this.onSelect(item);
  }
  
  closeSuggestions() {
    this.suggestionsList.style.display = 'none';
    this.suggestionsList.innerHTML = '';
  }
  
  updateSuggestions(newSuggestions) {
    this.suggestions = newSuggestions;
  }
}

// CSS
const autocompleteCSS = `
.autocomplete {
  position: relative;
  width: 300px;
}

.autocomplete input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.autocomplete-suggestions {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  margin: 0;
  padding: 0;
  list-style: none;
  background: white;
  border: 1px solid #ddd;
  border-top: none;
  max-height: 300px;
  overflow-y: auto;
  z-index: 1000;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.suggestion-item {
  padding: 10px;
  cursor: pointer;
  transition: background 0.2s;
}

.suggestion-item:hover,
.suggestion-item.active {
  background: #f0f0f0;
}
`;

// Usage
// const countries = ['USA', 'UK', 'Canada', 'Australia', 'Germany', 'France'];
// const autocomplete = new Autocomplete(document.querySelector('#search'), {
//   suggestions: countries,
//   minChars: 2,
//   onSelect: (item) => console.log('Selected:', item)
// });
```

### **Approach 2: With Debounce and Highlighting**
```javascript
/**
 * Autocomplete with debounce and match highlighting
 */
class AdvancedAutocomplete {
  constructor(input, options = {}) {
    this.input = input;
    this.suggestions = options.suggestions || [];
    this.onSelect = options.onSelect || (() => {});
    this.minChars = options.minChars || 1;
    this.debounceDelay = options.debounceDelay || 300;
    this.maxResults = options.maxResults || 10;
    this.highlightMatches = options.highlightMatches !== false;
    
    this.suggestionsList = this.createSuggestionsList();
    this.currentFocus = -1;
    this.debounceTimer = null;
    
    this.init();
  }
  
  createSuggestionsList() {
    const ul = document.createElement('ul');
    ul.className = 'autocomplete-suggestions';
    ul.style.display = 'none';
    this.input.parentNode.insertBefore(ul, this.input.nextSibling);
    return ul;
  }
  
  init() {
    this.input.addEventListener('input', (e) => {
      this.handleInputDebounced(e.target.value);
    });
    
    this.input.addEventListener('keydown', (e) => {
      this.handleKeydown(e);
    });
    
    document.addEventListener('click', (e) => {
      if (!this.input.parentNode.contains(e.target)) {
        this.closeSuggestions();
      }
    });
    
    this.input.addEventListener('focus', () => {
      if (this.input.value.length >= this.minChars) {
        this.handleInput(this.input.value);
      }
    });
  }
  
  handleInputDebounced(value) {
    clearTimeout(this.debounceTimer);
    
    this.debounceTimer = setTimeout(() => {
      this.handleInput(value);
    }, this.debounceDelay);
  }
  
  handleInput(value) {
    this.currentFocus = -1;
    
    if (value.length < this.minChars) {
      this.closeSuggestions();
      return;
    }
    
    const filtered = this.filterSuggestions(value);
    this.renderSuggestions(filtered, value);
  }
  
  filterSuggestions(value) {
    const lowerValue = value.toLowerCase();
    
    // Filter and rank by match position
    const matches = this.suggestions
      .map(item => {
        const lowerItem = item.toLowerCase();
        const startsWith = lowerItem.startsWith(lowerValue);
        const includes = lowerItem.includes(lowerValue);
        const index = lowerItem.indexOf(lowerValue);
        
        return {
          item,
          startsWith,
          includes,
          index: includes ? index : Infinity
        };
      })
      .filter(match => match.includes)
      .sort((a, b) => {
        // Prioritize starts-with matches
        if (a.startsWith && !b.startsWith) return -1;
        if (!a.startsWith && b.startsWith) return 1;
        // Then by position
        return a.index - b.index;
      })
      .slice(0, this.maxResults)
      .map(match => match.item);
    
    return matches;
  }
  
  highlightMatch(text, query) {
    if (!this.highlightMatches) return text;
    
    const index = text.toLowerCase().indexOf(query.toLowerCase());
    
    if (index === -1) return text;
    
    const before = text.substring(0, index);
    const match = text.substring(index, index + query.length);
    const after = text.substring(index + query.length);
    
    return `${before}<strong>${match}</strong>${after}`;
  }
  
  renderSuggestions(items, query) {
    this.suggestionsList.innerHTML = '';
    
    if (items.length === 0) {
      const li = document.createElement('li');
      li.className = 'no-results';
      li.textContent = 'No results found';
      this.suggestionsList.appendChild(li);
      this.suggestionsList.style.display = 'block';
      return;
    }
    
    items.forEach(item => {
      const li = document.createElement('li');
      li.className = 'suggestion-item';
      li.innerHTML = this.highlightMatch(item, query);
      li.dataset.value = item;
      
      li.addEventListener('click', () => {
        this.selectItem(item);
      });
      
      this.suggestionsList.appendChild(li);
    });
    
    this.suggestionsList.style.display = 'block';
  }
  
  handleKeydown(e) {
    const items = this.suggestionsList.querySelectorAll('.suggestion-item');
    
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      this.currentFocus++;
      this.setActive(items);
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      this.currentFocus--;
      this.setActive(items);
    } else if (e.key === 'Enter') {
      e.preventDefault();
      if (this.currentFocus > -1 && items[this.currentFocus]) {
        this.selectItem(items[this.currentFocus].dataset.value);
      }
    } else if (e.key === 'Escape') {
      this.closeSuggestions();
    }
  }
  
  setActive(items) {
    if (!items.length) return;
    
    items.forEach(item => item.classList.remove('active'));
    
    if (this.currentFocus >= items.length) this.currentFocus = 0;
    if (this.currentFocus < 0) this.currentFocus = items.length - 1;
    
    items[this.currentFocus].classList.add('active');
    items[this.currentFocus].scrollIntoView({ block: 'nearest' });
  }
  
  selectItem(item) {
    this.input.value = item;
    this.closeSuggestions();
    this.onSelect(item);
  }
  
  closeSuggestions() {
    this.suggestionsList.style.display = 'none';
  }
  
  destroy() {
    clearTimeout(this.debounceTimer);
    this.suggestionsList.remove();
  }
}

// Enhanced CSS
const advancedAutocompleteCSS = `
.autocomplete {
  position: relative;
  width: 100%;
  max-width: 400px;
}

.autocomplete input {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.autocomplete input:focus {
  outline: none;
  border-color: #4285f4;
}

.autocomplete-suggestions {
  position: absolute;
  top: calc(100% + 4px);
  left: 0;
  right: 0;
  margin: 0;
  padding: 0;
  list-style: none;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  max-height: 300px;
  overflow-y: auto;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.suggestion-item {
  padding: 12px 16px;
  cursor: pointer;
  transition: background 0.2s;
  border-bottom: 1px solid #f0f0f0;
}

.suggestion-item:last-child {
  border-bottom: none;
}

.suggestion-item:hover,
.suggestion-item.active {
  background: #f5f5f5;
}

.suggestion-item strong {
  color: #4285f4;
  font-weight: 600;
}

.no-results {
  padding: 12px 16px;
  color: #999;
  text-align: center;
}
`;
```

### **Approach 3: Async with API Integration**
```javascript
/**
 * Autocomplete with async data fetching
 */
class AsyncAutocomplete {
  constructor(input, options = {}) {
    this.input = input;
    this.fetchSuggestions = options.fetchSuggestions;
    this.onSelect = options.onSelect || (() => {});
    this.minChars = options.minChars || 2;
    this.debounceDelay = options.debounceDelay || 300;
    this.maxResults = options.maxResults || 10;
    this.cache = options.cache !== false;
    this.showLoader = options.showLoader !== false;
    
    this.suggestionsList = this.createSuggestionsList();
    this.loader = this.createLoader();
    this.currentFocus = -1;
    this.debounceTimer = null;
    this.abortController = null;
    this.cacheStore = new Map();
    
    this.init();
  }
  
  createSuggestionsList() {
    const ul = document.createElement('ul');
    ul.className = 'autocomplete-suggestions';
    ul.style.display = 'none';
    this.input.parentNode.appendChild(ul);
    return ul;
  }
  
  createLoader() {
    const loader = document.createElement('div');
    loader.className = 'autocomplete-loader';
    loader.innerHTML = '<div class="spinner"></div>';
    loader.style.display = 'none';
    this.input.parentNode.appendChild(loader);
    return loader;
  }
  
  init() {
    this.input.addEventListener('input', (e) => {
      this.handleInputDebounced(e.target.value);
    });
    
    this.input.addEventListener('keydown', (e) => {
      this.handleKeydown(e);
    });
    
    document.addEventListener('click', (e) => {
      if (!this.input.parentNode.contains(e.target)) {
        this.closeSuggestions();
      }
    });
  }
  
  handleInputDebounced(value) {
    clearTimeout(this.debounceTimer);
    
    // Cancel previous request
    if (this.abortController) {
      this.abortController.abort();
    }
    
    this.debounceTimer = setTimeout(() => {
      this.handleInput(value);
    }, this.debounceDelay);
  }
  
  async handleInput(value) {
    this.currentFocus = -1;
    
    if (value.length < this.minChars) {
      this.closeSuggestions();
      return;
    }
    
    // Check cache
    if (this.cache && this.cacheStore.has(value)) {
      this.renderSuggestions(this.cacheStore.get(value), value);
      return;
    }
    
    // Show loader
    if (this.showLoader) {
      this.loader.style.display = 'block';
    }
    
    try {
      this.abortController = new AbortController();
      
      const results = await this.fetchSuggestions(value, {
        signal: this.abortController.signal,
        maxResults: this.maxResults
      });
      
      // Cache results
      if (this.cache) {
        this.cacheStore.set(value, results);
      }
      
      this.renderSuggestions(results, value);
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error('Error fetching suggestions:', error);
        this.renderError();
      }
    } finally {
      if (this.showLoader) {
        this.loader.style.display = 'none';
      }
    }
  }
  
  renderSuggestions(items, query) {
    this.suggestionsList.innerHTML = '';
    
    if (!items || items.length === 0) {
      const li = document.createElement('li');
      li.className = 'no-results';
      li.textContent = 'No results found';
      this.suggestionsList.appendChild(li);
      this.suggestionsList.style.display = 'block';
      return;
    }
    
    items.forEach(item => {
      const li = document.createElement('li');
      li.className = 'suggestion-item';
      
      // Handle different item formats
      if (typeof item === 'string') {
        li.textContent = item;
        li.dataset.value = item;
      } else {
        li.innerHTML = this.formatItem(item);
        li.dataset.value = item.value || item.name || item.label;
      }
      
      li.addEventListener('click', () => {
        this.selectItem(li.dataset.value, item);
      });
      
      this.suggestionsList.appendChild(li);
    });
    
    this.suggestionsList.style.display = 'block';
  }
  
  formatItem(item) {
    // Customize based on item structure
    if (item.title && item.subtitle) {
      return `
        <div class="suggestion-main">${item.title}</div>
        <div class="suggestion-sub">${item.subtitle}</div>
      `;
    }
    return item.name || item.label || item.value || String(item);
  }
  
  renderError() {
    this.suggestionsList.innerHTML = `
      <li class="error-message">Failed to load suggestions</li>
    `;
    this.suggestionsList.style.display = 'block';
  }
  
  handleKeydown(e) {
    const items = this.suggestionsList.querySelectorAll('.suggestion-item');
    
    if (e.key === 'ArrowDown') {
      e.preventDefault();
      this.currentFocus++;
      this.setActive(items);
    } else if (e.key === 'ArrowUp') {
      e.preventDefault();
      this.currentFocus--;
      this.setActive(items);
    } else if (e.key === 'Enter') {
      e.preventDefault();
      if (this.currentFocus > -1 && items[this.currentFocus]) {
        items[this.currentFocus].click();
      }
    } else if (e.key === 'Escape') {
      this.closeSuggestions();
    }
  }
  
  setActive(items) {
    if (!items.length) return;
    
    items.forEach(item => item.classList.remove('active'));
    
    if (this.currentFocus >= items.length) this.currentFocus = 0;
    if (this.currentFocus < 0) this.currentFocus = items.length - 1;
    
    items[this.currentFocus].classList.add('active');
    items[this.currentFocus].scrollIntoView({ block: 'nearest' });
  }
  
  selectItem(value, originalItem) {
    this.input.value = value;
    this.closeSuggestions();
    this.onSelect(value, originalItem);
  }
  
  closeSuggestions() {
    this.suggestionsList.style.display = 'none';
    if (this.showLoader) {
      this.loader.style.display = 'none';
    }
  }
  
  clearCache() {
    this.cacheStore.clear();
  }
  
  destroy() {
    clearTimeout(this.debounceTimer);
    if (this.abortController) {
      this.abortController.abort();
    }
    this.suggestionsList.remove();
    this.loader.remove();
  }
}

// Complete CSS with loader
const asyncAutocompleteCSS = `
.autocomplete {
  position: relative;
  width: 100%;
  max-width: 500px;
}

.autocomplete input {
  width: 100%;
  padding: 12px 40px 12px 16px;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.autocomplete input:focus {
  outline: none;
  border-color: #4285f4;
}

.autocomplete-loader {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
}

.spinner {
  width: 20px;
  height: 20px;
  border: 2px solid #f3f3f3;
  border-top: 2px solid #4285f4;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.autocomplete-suggestions {
  position: absolute;
  top: calc(100% + 4px);
  left: 0;
  right: 0;
  margin: 0;
  padding: 0;
  list-style: none;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  max-height: 400px;
  overflow-y: auto;
  z-index: 1000;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.suggestion-item {
  padding: 12px 16px;
  cursor: pointer;
  transition: background 0.2s;
  border-bottom: 1px solid #f5f5f5;
}

.suggestion-item:last-child {
  border-bottom: none;
}

.suggestion-item:hover,
.suggestion-item.active {
  background: #f5f5f5;
}

.suggestion-main {
  font-weight: 500;
  color: #333;
}

.suggestion-sub {
  font-size: 12px;
  color: #666;
  margin-top: 4px;
}

.no-results,
.error-message {
  padding: 12px 16px;
  color: #999;
  text-align: center;
}

.error-message {
  color: #d32f2f;
}
`;

// Usage examples
console.log('=== Basic Autocomplete ===');
// const countries = ['USA', 'UK', 'Canada', 'Australia', 'Germany', 'France'];
// const autocomplete1 = new Autocomplete(document.querySelector('#search'), {
//   suggestions: countries,
//   onSelect: (item) => console.log('Selected:', item)
// });

console.log('\n=== Advanced with Highlighting ===');
// const autocomplete2 = new AdvancedAutocomplete(document.querySelector('#search2'), {
//   suggestions: countries,
//   minChars: 2,
//   debounceDelay: 300,
//   highlightMatches: true,
//   onSelect: (item) => console.log('Selected:', item)
// });

console.log('\n=== Async with API ===');
// const autocomplete3 = new AsyncAutocomplete(document.querySelector('#search3'), {
//   minChars: 2,
//   debounceDelay: 300,
//   cache: true,
//   fetchSuggestions: async (query, options) => {
//     const response = await fetch(
//       `https://api.example.com/search?q=${encodeURIComponent(query)}`,
//       { signal: options.signal }
//     );
//     const data = await response.json();
//     return data.results.slice(0, options.maxResults);
//   },
//   onSelect: (value, item) => {
//     console.log('Selected:', value, item);
//   }
// });

// Real-world example: GitHub repository search
async function githubRepoSearch(query, options) {
  const response = await fetch(
    `https://api.github.com/search/repositories?q=${encodeURIComponent(query)}&per_page=${options.maxResults}`,
    { signal: options.signal }
  );
  
  const data = await response.json();
  
  return data.items.map(repo => ({
    title: repo.full_name,
    subtitle: repo.description || 'No description',
    value: repo.full_name,
    url: repo.html_url
  }));
}

// const githubAutocomplete = new AsyncAutocomplete(document.querySelector('#github-search'), {
//   fetchSuggestions: githubRepoSearch,
//   onSelect: (value, item) => {
//     window.location.href = item.url;
//   }
// });
```

### **Bonus: Multi-Select Autocomplete**
```javascript
/**
 * Multi-select autocomplete (tags input)
 */
class MultiSelectAutocomplete extends AsyncAutocomplete {
  constructor(input, options = {}) {
    super(input, options);
    this.selectedItems = [];
    this.createTagContainer();
  }
  
  createTagContainer() {
    this.tagContainer = document.createElement('div');
    this.tagContainer.className = 'tag-container';
    this.input.parentNode.insertBefore(this.tagContainer, this.input);
  }
  
  selectItem(value, originalItem) {
    if (!this.selectedItems.find(item => item.value === value)) {
      this.selectedItems.push({ value, originalItem });
      this.addTag(value);
    }
    
    this.input.value = '';
    this.closeSuggestions();
    this.onSelect(this.selectedItems);
  }
  
  addTag(value) {
    const tag = document.createElement('span');
    tag.className = 'tag';
    tag.innerHTML = `
      ${value}
      <button class="tag-remove" aria-label="Remove ${value}">×</button>
    `;
    
    tag.querySelector('.tag-remove').addEventListener('click', () => {
      this.removeTag(value);
    });
    
    this.tagContainer.appendChild(tag);
  }
  
  removeTag(value) {
    this.selectedItems = this.selectedItems.filter(item => item.value !== value);
    
    const tags = this.tagContainer.querySelectorAll('.tag');
    tags.forEach(tag => {
      if (tag.textContent.trim().startsWith(value)) {
        tag.remove();
      }
    });
    
    this.onSelect(this.selectedItems);
  }
  
  getSelectedValues() {
    return this.selectedItems.map(item => item.value);
  }
  
  clear() {
    this.selectedItems = [];
    this.tagContainer.innerHTML = '';
  }
}

const multiSelectCSS = `
.tag-container {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 8px;
}

.tag {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  background: #e3f2fd;
  border-radius: 16px;
  font-size: 14px;
  color: #1976d2;
}

.tag-remove {
  background: none;
  border: none;
  font-size: 20px;
  line-height: 1;
  cursor: pointer;
  color: #1976d2;
  padding: 0;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: background 0.2s;
}

.tag-remove:hover {
  background: rgba(25, 118, 210, 0.1);
}
`;
```

**Interview Tips:**
- Autocomplete/typeahead suggests completions as user types
- Key features: debounce input (300ms typical), min characters (2-3)
- Keyboard navigation: arrow keys, Enter to select, Escape to close
- Highlight matching text for better UX
- Async: debounce, abort previous requests, cache results
- Performance: limit max results (10-20), virtual scrolling for large lists
- Ranking: prioritize starts-with matches, then includes
- Accessibility: ARIA attributes, keyboard support, screen reader announcements
- Edge cases: no results, loading state, error handling, click outside
- Common libraries: react-autosuggest, downshift, @reach/combobox
- Real-world: Google search, GitHub search, address autocomplete

</details>

59. Implement a modal/dialog component

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Modal**
```javascript
/**
 * Simple modal/dialog component
 * 
 * HTML:
 * <div id="myModal" class="modal">
 *   <div class="modal-content">
 *     <span class="close">&times;</span>
 *     <h2>Modal Title</h2>
 *     <p>Modal content goes here</p>
 *   </div>
 * </div>
 */
class Modal {
  constructor(element, options = {}) {
    this.modal = element;
    this.closeButton = element.querySelector('.close');
    this.onOpen = options.onOpen || (() => {});
    this.onClose = options.onClose || (() => {});
    this.closeOnOverlay = options.closeOnOverlay !== false;
    this.closeOnEsc = options.closeOnEsc !== false;
    
    this.init();
  }
  
  init() {
    // Close button
    if (this.closeButton) {
      this.closeButton.addEventListener('click', () => this.close());
    }
    
    // Click overlay to close
    if (this.closeOnOverlay) {
      this.modal.addEventListener('click', (e) => {
        if (e.target === this.modal) {
          this.close();
        }
      });
    }
    
    // ESC key to close
    if (this.closeOnEsc) {
      this.handleEsc = (e) => {
        if (e.key === 'Escape' && this.isOpen()) {
          this.close();
        }
      };
      document.addEventListener('keydown', this.handleEsc);
    }
  }
  
  open() {
    this.modal.style.display = 'block';
    document.body.style.overflow = 'hidden'; // Prevent background scroll
    this.onOpen();
  }
  
  close() {
    this.modal.style.display = 'none';
    document.body.style.overflow = ''; // Restore scroll
    this.onClose();
  }
  
  isOpen() {
    return this.modal.style.display === 'block';
  }
  
  toggle() {
    if (this.isOpen()) {
      this.close();
    } else {
      this.open();
    }
  }
  
  destroy() {
    if (this.handleEsc) {
      document.removeEventListener('keydown', this.handleEsc);
    }
  }
}

// CSS
const modalCSS = `
.modal {
  display: none;
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  overflow: auto;
}

.modal-content {
  background-color: white;
  margin: 10% auto;
  padding: 20px;
  border-radius: 8px;
  width: 80%;
  max-width: 600px;
  position: relative;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}

.close {
  position: absolute;
  right: 20px;
  top: 20px;
  font-size: 28px;
  font-weight: bold;
  color: #aaa;
  cursor: pointer;
  line-height: 1;
}

.close:hover,
.close:focus {
  color: #000;
}
`;

// Usage
// const modal = new Modal(document.querySelector('#myModal'), {
//   onOpen: () => console.log('Modal opened'),
//   onClose: () => console.log('Modal closed')
// });
// 
// document.querySelector('#openModalBtn').addEventListener('click', () => {
//   modal.open();
// });
```

### **Approach 2: With Animations and Focus Trap**
```javascript
/**
 * Modal with animations and accessibility features
 */
class AnimatedModal {
  constructor(element, options = {}) {
    this.modal = element;
    this.modalContent = element.querySelector('.modal-content');
    this.closeButton = element.querySelector('.close');
    this.onOpen = options.onOpen || (() => {});
    this.onClose = options.onClose || (() => {});
    this.closeOnOverlay = options.closeOnOverlay !== false;
    this.closeOnEsc = options.closeOnEsc !== false;
    this.animation = options.animation || 'fade'; // fade, slide, zoom
    
    this.isAnimating = false;
    this.previousActiveElement = null;
    this.focusableElements = [];
    
    this.init();
  }
  
  init() {
    // Add animation class
    this.modal.classList.add(`modal-${this.animation}`);
    
    // Close button
    if (this.closeButton) {
      this.closeButton.addEventListener('click', () => this.close());
    }
    
    // Click overlay to close
    if (this.closeOnOverlay) {
      this.modal.addEventListener('click', (e) => {
        if (e.target === this.modal && !this.isAnimating) {
          this.close();
        }
      });
    }
    
    // ESC key to close
    if (this.closeOnEsc) {
      this.handleEsc = (e) => {
        if (e.key === 'Escape' && this.isOpen()) {
          this.close();
        }
      };
      document.addEventListener('keydown', this.handleEsc);
    }
    
    // Focus trap
    this.handleTab = (e) => {
      if (e.key === 'Tab' && this.isOpen()) {
        this.trapFocus(e);
      }
    };
    document.addEventListener('keydown', this.handleTab);
  }
  
  async open() {
    if (this.isAnimating || this.isOpen()) return;
    
    this.isAnimating = true;
    this.previousActiveElement = document.activeElement;
    
    // Show modal
    this.modal.style.display = 'block';
    this.modal.classList.add('modal-opening');
    
    // Prevent background scroll
    document.body.style.overflow = 'hidden';
    
    // Trigger animation
    requestAnimationFrame(() => {
      this.modal.classList.add('modal-open');
    });
    
    // Wait for animation
    await this.waitForAnimation();
    
    this.modal.classList.remove('modal-opening');
    this.isAnimating = false;
    
    // Set up focus trap
    this.setupFocusTrap();
    
    // Focus first focusable element
    if (this.focusableElements.length > 0) {
      this.focusableElements[0].focus();
    }
    
    this.onOpen();
  }
  
  async close() {
    if (this.isAnimating || !this.isOpen()) return;
    
    this.isAnimating = true;
    this.modal.classList.add('modal-closing');
    this.modal.classList.remove('modal-open');
    
    // Wait for animation
    await this.waitForAnimation();
    
    this.modal.style.display = 'none';
    this.modal.classList.remove('modal-closing');
    this.isAnimating = false;
    
    // Restore scroll
    document.body.style.overflow = '';
    
    // Restore focus
    if (this.previousActiveElement) {
      this.previousActiveElement.focus();
    }
    
    this.onClose();
  }
  
  isOpen() {
    return this.modal.style.display === 'block';
  }
  
  waitForAnimation() {
    return new Promise(resolve => {
      const duration = parseFloat(getComputedStyle(this.modal).transitionDuration) * 1000;
      setTimeout(resolve, duration || 300);
    });
  }
  
  setupFocusTrap() {
    // Get all focusable elements
    const focusableSelectors = [
      'a[href]',
      'button:not([disabled])',
      'textarea:not([disabled])',
      'input:not([disabled])',
      'select:not([disabled])',
      '[tabindex]:not([tabindex="-1"])'
    ].join(', ');
    
    this.focusableElements = Array.from(
      this.modalContent.querySelectorAll(focusableSelectors)
    );
  }
  
  trapFocus(e) {
    if (this.focusableElements.length === 0) return;
    
    const firstElement = this.focusableElements[0];
    const lastElement = this.focusableElements[this.focusableElements.length - 1];
    
    if (e.shiftKey) {
      // Shift + Tab
      if (document.activeElement === firstElement) {
        e.preventDefault();
        lastElement.focus();
      }
    } else {
      // Tab
      if (document.activeElement === lastElement) {
        e.preventDefault();
        firstElement.focus();
      }
    }
  }
  
  destroy() {
    if (this.handleEsc) {
      document.removeEventListener('keydown', this.handleEsc);
    }
    if (this.handleTab) {
      document.removeEventListener('keydown', this.handleTab);
    }
  }
}

// CSS with animations
const animatedModalCSS = `
.modal {
  display: none;
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0);
  overflow: auto;
  transition: background-color 0.3s ease;
}

.modal-open {
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background-color: white;
  margin: 10% auto;
  padding: 30px;
  border-radius: 12px;
  width: 90%;
  max-width: 600px;
  position: relative;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

/* Fade animation */
.modal-fade .modal-content {
  opacity: 0;
  transition: opacity 0.3s ease;
}

.modal-fade.modal-open .modal-content {
  opacity: 1;
}

/* Slide animation */
.modal-slide .modal-content {
  transform: translateY(-50px);
  opacity: 0;
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.modal-slide.modal-open .modal-content {
  transform: translateY(0);
  opacity: 1;
}

/* Zoom animation */
.modal-zoom .modal-content {
  transform: scale(0.7);
  opacity: 0;
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.modal-zoom.modal-open .modal-content {
  transform: scale(1);
  opacity: 1;
}

.close {
  position: absolute;
  right: 20px;
  top: 20px;
  font-size: 28px;
  font-weight: bold;
  color: #aaa;
  cursor: pointer;
  line-height: 1;
  transition: color 0.2s;
}

.close:hover,
.close:focus {
  color: #000;
}
`;
```

### **Approach 3: Production-Ready with Multiple Features**
```javascript
/**
 * Full-featured modal component
 */
class AdvancedModal {
  constructor(options = {}) {
    this.title = options.title || '';
    this.content = options.content || '';
    this.size = options.size || 'medium'; // small, medium, large, full
    this.showCloseButton = options.showCloseButton !== false;
    this.closeOnOverlay = options.closeOnOverlay !== false;
    this.closeOnEsc = options.closeOnEsc !== false;
    this.animation = options.animation || 'fade';
    this.footer = options.footer || null;
    this.className = options.className || '';
    this.onOpen = options.onOpen || (() => {});
    this.onClose = options.onClose || (() => {});
    this.beforeClose = options.beforeClose || null;
    
    this.modal = null;
    this.isAnimating = false;
    this.previousActiveElement = null;
    
    this.create();
  }
  
  create() {
    // Create modal structure
    this.modal = document.createElement('div');
    this.modal.className = `modal modal-${this.animation} modal-${this.size} ${this.className}`;
    this.modal.setAttribute('role', 'dialog');
    this.modal.setAttribute('aria-modal', 'true');
    
    if (this.title) {
      this.modal.setAttribute('aria-labelledby', 'modal-title');
    }
    
    const modalContent = document.createElement('div');
    modalContent.className = 'modal-content';
    
    // Header
    if (this.title || this.showCloseButton) {
      const header = document.createElement('div');
      header.className = 'modal-header';
      
      if (this.title) {
        const titleElement = document.createElement('h2');
        titleElement.id = 'modal-title';
        titleElement.className = 'modal-title';
        titleElement.textContent = this.title;
        header.appendChild(titleElement);
      }
      
      if (this.showCloseButton) {
        const closeBtn = document.createElement('button');
        closeBtn.className = 'modal-close';
        closeBtn.innerHTML = '&times;';
        closeBtn.setAttribute('aria-label', 'Close modal');
        closeBtn.addEventListener('click', () => this.close());
        header.appendChild(closeBtn);
      }
      
      modalContent.appendChild(header);
    }
    
    // Body
    const body = document.createElement('div');
    body.className = 'modal-body';
    
    if (typeof this.content === 'string') {
      body.innerHTML = this.content;
    } else if (this.content instanceof HTMLElement) {
      body.appendChild(this.content);
    }
    
    modalContent.appendChild(body);
    
    // Footer
    if (this.footer) {
      const footer = document.createElement('div');
      footer.className = 'modal-footer';
      
      if (typeof this.footer === 'string') {
        footer.innerHTML = this.footer;
      } else if (this.footer instanceof HTMLElement) {
        footer.appendChild(this.footer);
      } else if (Array.isArray(this.footer)) {
        this.footer.forEach(button => footer.appendChild(button));
      }
      
      modalContent.appendChild(footer);
    }
    
    this.modal.appendChild(modalContent);
    document.body.appendChild(this.modal);
    
    this.init();
  }
  
  init() {
    // Click overlay to close
    if (this.closeOnOverlay) {
      this.modal.addEventListener('click', (e) => {
        if (e.target === this.modal && !this.isAnimating) {
          this.close();
        }
      });
    }
    
    // ESC key to close
    if (this.closeOnEsc) {
      this.handleEsc = (e) => {
        if (e.key === 'Escape' && this.isOpen()) {
          this.close();
        }
      };
      document.addEventListener('keydown', this.handleEsc);
    }
    
    // Focus trap
    this.handleTab = (e) => {
      if (e.key === 'Tab' && this.isOpen()) {
        this.trapFocus(e);
      }
    };
    document.addEventListener('keydown', this.handleTab);
  }
  
  async open() {
    if (this.isAnimating || this.isOpen()) return;
    
    this.isAnimating = true;
    this.previousActiveElement = document.activeElement;
    
    this.modal.style.display = 'block';
    this.modal.classList.add('modal-opening');
    document.body.style.overflow = 'hidden';
    
    requestAnimationFrame(() => {
      this.modal.classList.add('modal-open');
    });
    
    await this.waitForAnimation();
    
    this.modal.classList.remove('modal-opening');
    this.isAnimating = false;
    
    this.setupFocusTrap();
    this.focusFirstElement();
    
    this.onOpen();
  }
  
  async close() {
    if (this.isAnimating || !this.isOpen()) return;
    
    // Call beforeClose hook
    if (this.beforeClose) {
      const canClose = await this.beforeClose();
      if (canClose === false) return;
    }
    
    this.isAnimating = true;
    this.modal.classList.add('modal-closing');
    this.modal.classList.remove('modal-open');
    
    await this.waitForAnimation();
    
    this.modal.style.display = 'none';
    this.modal.classList.remove('modal-closing');
    this.isAnimating = false;
    
    document.body.style.overflow = '';
    
    if (this.previousActiveElement) {
      this.previousActiveElement.focus();
    }
    
    this.onClose();
  }
  
  isOpen() {
    return this.modal.style.display === 'block';
  }
  
  waitForAnimation() {
    return new Promise(resolve => {
      const duration = parseFloat(getComputedStyle(this.modal).transitionDuration) * 1000;
      setTimeout(resolve, duration || 300);
    });
  }
  
  setupFocusTrap() {
    const focusableSelectors = [
      'a[href]',
      'button:not([disabled])',
      'textarea:not([disabled])',
      'input:not([disabled])',
      'select:not([disabled])',
      '[tabindex]:not([tabindex="-1"])'
    ].join(', ');
    
    this.focusableElements = Array.from(
      this.modal.querySelectorAll(focusableSelectors)
    );
  }
  
  focusFirstElement() {
    if (this.focusableElements && this.focusableElements.length > 0) {
      this.focusableElements[0].focus();
    }
  }
  
  trapFocus(e) {
    if (!this.focusableElements || this.focusableElements.length === 0) return;
    
    const firstElement = this.focusableElements[0];
    const lastElement = this.focusableElements[this.focusableElements.length - 1];
    
    if (e.shiftKey) {
      if (document.activeElement === firstElement) {
        e.preventDefault();
        lastElement.focus();
      }
    } else {
      if (document.activeElement === lastElement) {
        e.preventDefault();
        firstElement.focus();
      }
    }
  }
  
  setContent(content) {
    const body = this.modal.querySelector('.modal-body');
    if (body) {
      if (typeof content === 'string') {
        body.innerHTML = content;
      } else if (content instanceof HTMLElement) {
        body.innerHTML = '';
        body.appendChild(content);
      }
    }
  }
  
  setTitle(title) {
    const titleElement = this.modal.querySelector('.modal-title');
    if (titleElement) {
      titleElement.textContent = title;
    }
  }
  
  destroy() {
    if (this.handleEsc) {
      document.removeEventListener('keydown', this.handleEsc);
    }
    if (this.handleTab) {
      document.removeEventListener('keydown', this.handleTab);
    }
    if (this.modal && this.modal.parentNode) {
      this.modal.parentNode.removeChild(this.modal);
    }
  }
}

// Complete CSS
const advancedModalCSS = `
.modal {
  display: none;
  position: fixed;
  z-index: 1000;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0);
  overflow: auto;
  transition: background-color 0.3s ease;
}

.modal-open {
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background-color: white;
  margin: 5% auto;
  border-radius: 12px;
  position: relative;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
  max-height: 90vh;
  display: flex;
  flex-direction: column;
}

/* Sizes */
.modal-small .modal-content {
  width: 90%;
  max-width: 400px;
}

.modal-medium .modal-content {
  width: 90%;
  max-width: 600px;
}

.modal-large .modal-content {
  width: 90%;
  max-width: 900px;
}

.modal-full .modal-content {
  width: 95%;
  max-width: none;
  height: 95%;
  margin: 2.5%;
}

/* Header */
.modal-header {
  padding: 20px 24px;
  border-bottom: 1px solid #e0e0e0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.modal-title {
  margin: 0;
  font-size: 24px;
  font-weight: 600;
  color: #333;
}

.modal-close {
  background: none;
  border: none;
  font-size: 32px;
  color: #aaa;
  cursor: pointer;
  padding: 0;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  transition: all 0.2s;
}

.modal-close:hover {
  background: #f5f5f5;
  color: #333;
}

/* Body */
.modal-body {
  padding: 24px;
  overflow-y: auto;
  flex: 1;
}

/* Footer */
.modal-footer {
  padding: 16px 24px;
  border-top: 1px solid #e0e0e0;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.modal-footer button {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.2s;
}

.modal-footer .btn-primary {
  background: #4285f4;
  color: white;
}

.modal-footer .btn-primary:hover {
  background: #3367d6;
}

.modal-footer .btn-secondary {
  background: #f5f5f5;
  color: #333;
}

.modal-footer .btn-secondary:hover {
  background: #e0e0e0;
}

/* Animations */
.modal-fade .modal-content {
  opacity: 0;
  transition: opacity 0.3s ease;
}

.modal-fade.modal-open .modal-content {
  opacity: 1;
}

.modal-slide .modal-content {
  transform: translateY(-50px);
  opacity: 0;
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.modal-slide.modal-open .modal-content {
  transform: translateY(0);
  opacity: 1;
}

.modal-zoom .modal-content {
  transform: scale(0.7);
  opacity: 0;
  transition: transform 0.3s ease, opacity 0.3s ease;
}

.modal-zoom.modal-open .modal-content {
  transform: scale(1);
  opacity: 1;
}

/* Responsive */
@media (max-width: 768px) {
  .modal-content {
    margin: 0;
    width: 100% !important;
    max-width: none !important;
    height: 100%;
    max-height: none;
    border-radius: 0;
  }
}
`;

// Usage examples
console.log('=== Basic Modal ===');
// const modal1 = new Modal(document.querySelector('#myModal'));
// document.querySelector('#openBtn').addEventListener('click', () => modal1.open());

console.log('\n=== Animated Modal ===');
// const modal2 = new AnimatedModal(document.querySelector('#animatedModal'), {
//   animation: 'slide',
//   onOpen: () => console.log('Opened'),
//   onClose: () => console.log('Closed')
// });

console.log('\n=== Advanced Modal ===');
// const modal3 = new AdvancedModal({
//   title: 'Confirm Action',
//   content: '<p>Are you sure you want to proceed?</p>',
//   size: 'medium',
//   animation: 'zoom',
//   footer: [
//     createButton('Cancel', () => modal3.close()),
//     createButton('Confirm', () => {
//       console.log('Confirmed');
//       modal3.close();
//     }, 'primary')
//   ],
//   beforeClose: async () => {
//     return confirm('Close modal?');
//   }
// });
// 
// modal3.open();

// Helper function to create buttons
function createButton(text, onClick, variant = 'secondary') {
  const button = document.createElement('button');
  button.textContent = text;
  button.className = `btn-${variant}`;
  button.addEventListener('click', onClick);
  return button;
}

// Common modal patterns
class ModalService {
  static confirm(options) {
    return new Promise((resolve) => {
      const modal = new AdvancedModal({
        title: options.title || 'Confirm',
        content: options.message || 'Are you sure?',
        size: 'small',
        footer: [
          createButton(options.cancelText || 'Cancel', () => {
            modal.close();
            resolve(false);
          }),
          createButton(options.confirmText || 'Confirm', () => {
            modal.close();
            resolve(true);
          }, 'primary')
        ]
      });
      modal.open();
    });
  }
  
  static alert(options) {
    return new Promise((resolve) => {
      const modal = new AdvancedModal({
        title: options.title || 'Alert',
        content: options.message || '',
        size: 'small',
        footer: [
          createButton(options.buttonText || 'OK', () => {
            modal.close();
            resolve();
          }, 'primary')
        ]
      });
      modal.open();
    });
  }
  
  static prompt(options) {
    return new Promise((resolve) => {
      const input = document.createElement('input');
      input.type = 'text';
      input.className = 'modal-input';
      input.value = options.defaultValue || '';
      input.placeholder = options.placeholder || '';
      
      const container = document.createElement('div');
      if (options.message) {
        const p = document.createElement('p');
        p.textContent = options.message;
        container.appendChild(p);
      }
      container.appendChild(input);
      
      const modal = new AdvancedModal({
        title: options.title || 'Input',
        content: container,
        size: 'small',
        footer: [
          createButton('Cancel', () => {
            modal.close();
            resolve(null);
          }),
          createButton('OK', () => {
            modal.close();
            resolve(input.value);
          }, 'primary')
        ]
      });
      
      modal.onOpen = () => input.focus();
      modal.open();
    });
  }
}

// Usage of modal service
// const confirmed = await ModalService.confirm({
//   title: 'Delete Item',
//   message: 'Are you sure you want to delete this item?',
//   confirmText: 'Delete',
//   cancelText: 'Cancel'
// });
// 
// if (confirmed) {
//   console.log('Item deleted');
// }

// await ModalService.alert({
//   title: 'Success',
//   message: 'Operation completed successfully!'
// });

// const name = await ModalService.prompt({
//   title: 'Enter Name',
//   message: 'Please enter your name:',
//   placeholder: 'John Doe',
//   defaultValue: ''
// });
```

### **Bonus: Stacked Modals**
```javascript
/**
 * Modal manager for handling multiple stacked modals
 */
class ModalManager {
  constructor() {
    this.modals = [];
    this.zIndexBase = 1000;
  }
  
  open(modal) {
    const zIndex = this.zIndexBase + (this.modals.length * 10);
    modal.modal.style.zIndex = zIndex;
    this.modals.push(modal);
    modal.open();
  }
  
  close(modal) {
    const index = this.modals.indexOf(modal);
    if (index > -1) {
      this.modals.splice(index, 1);
      modal.close();
    }
  }
  
  closeAll() {
    [...this.modals].reverse().forEach(modal => modal.close());
    this.modals = [];
  }
  
  getTopModal() {
    return this.modals[this.modals.length - 1];
  }
}

// Usage
// const manager = new ModalManager();
// 
// const modal1 = new AdvancedModal({ title: 'Modal 1', content: 'First modal' });
// const modal2 = new AdvancedModal({ title: 'Modal 2', content: 'Second modal' });
// 
// manager.open(modal1);
// manager.open(modal2); // Opens on top of modal1
```

**Interview Tips:**
- Modal/dialog: overlay window on top of main content
- Key features: close on overlay click, ESC key, close button
- Accessibility: focus trap, restore focus on close, ARIA attributes
- Prevent background scroll: set body overflow to hidden when open
- Animations: fade, slide, zoom using CSS transitions
- Focus management: trap Tab key, focus first element on open
- Common patterns: confirm, alert, prompt dialogs
- beforeClose hook: allow async validation before closing
- Multiple modals: z-index management, stacking context
- Responsive: full-screen on mobile devices
- Libraries: react-modal, dialog element (native), Material-UI Dialog
- HTML5: `<dialog>` element with showModal() API

</details>

60. Create a tabs component

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Tabs**
```javascript
/**
 * Simple tabs component
 * 
 * HTML:
 * <div class="tabs">
 *   <div class="tab-list">
 *     <button class="tab-button active" data-tab="tab1">Tab 1</button>
 *     <button class="tab-button" data-tab="tab2">Tab 2</button>
 *     <button class="tab-button" data-tab="tab3">Tab 3</button>
 *   </div>
 *   <div class="tab-panels">
 *     <div class="tab-panel active" id="tab1">Content 1</div>
 *     <div class="tab-panel" id="tab2">Content 2</div>
 *     <div class="tab-panel" id="tab3">Content 3</div>
 *   </div>
 * </div>
 */
class Tabs {
  constructor(element, options = {}) {
    this.container = element;
    this.tabs = Array.from(element.querySelectorAll('.tab-button'));
    this.panels = Array.from(element.querySelectorAll('.tab-panel'));
    this.activeIndex = 0;
    this.onChange = options.onChange || (() => {});
    
    this.init();
  }
  
  init() {
    // Set up click handlers
    this.tabs.forEach((tab, index) => {
      tab.addEventListener('click', () => {
        this.switchTab(index);
      });
    });
    
    // Find initially active tab
    const activeTab = this.tabs.findIndex(tab => tab.classList.contains('active'));
    if (activeTab !== -1) {
      this.activeIndex = activeTab;
    }
  }
  
  switchTab(index) {
    if (index === this.activeIndex || index < 0 || index >= this.tabs.length) {
      return;
    }
    
    // Remove active class from current tab and panel
    this.tabs[this.activeIndex].classList.remove('active');
    this.panels[this.activeIndex].classList.remove('active');
    
    // Add active class to new tab and panel
    this.activeIndex = index;
    this.tabs[this.activeIndex].classList.add('active');
    this.panels[this.activeIndex].classList.add('active');
    
    // Call onChange callback
    this.onChange(index, this.tabs[index].dataset.tab);
  }
  
  getActiveIndex() {
    return this.activeIndex;
  }
  
  getActiveTab() {
    return this.tabs[this.activeIndex];
  }
}

// CSS
const tabsCSS = `
.tabs {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

.tab-list {
  display: flex;
  border-bottom: 2px solid #e0e0e0;
  gap: 8px;
}

.tab-button {
  padding: 12px 24px;
  background: none;
  border: none;
  border-bottom: 3px solid transparent;
  cursor: pointer;
  font-size: 16px;
  color: #666;
  transition: all 0.3s;
  position: relative;
  bottom: -2px;
}

.tab-button:hover {
  color: #333;
  background: #f5f5f5;
}

.tab-button.active {
  color: #4285f4;
  border-bottom-color: #4285f4;
  font-weight: 600;
}

.tab-panels {
  padding: 24px;
  background: white;
}

.tab-panel {
  display: none;
}

.tab-panel.active {
  display: block;
}
`;

// Usage
// const tabs = new Tabs(document.querySelector('.tabs'), {
//   onChange: (index, tabId) => {
//     console.log('Switched to tab:', index, tabId);
//   }
// });
```

### **Approach 2: With Keyboard Navigation and ARIA**
```javascript
/**
 * Accessible tabs with keyboard navigation
 */
class AccessibleTabs {
  constructor(element, options = {}) {
    this.container = element;
    this.tabList = element.querySelector('.tab-list');
    this.tabs = Array.from(element.querySelectorAll('.tab-button'));
    this.panels = Array.from(element.querySelectorAll('.tab-panel'));
    this.activeIndex = 0;
    this.onChange = options.onChange || (() => {});
    
    this.init();
  }
  
  init() {
    // Set ARIA attributes
    this.tabList.setAttribute('role', 'tablist');
    
    this.tabs.forEach((tab, index) => {
      // ARIA attributes for tabs
      tab.setAttribute('role', 'tab');
      tab.setAttribute('id', `tab-${index}`);
      tab.setAttribute('aria-controls', `panel-${index}`);
      tab.setAttribute('tabindex', index === this.activeIndex ? '0' : '-1');
      tab.setAttribute('aria-selected', index === this.activeIndex ? 'true' : 'false');
      
      // Click handler
      tab.addEventListener('click', () => {
        this.switchTab(index);
      });
      
      // Keyboard navigation
      tab.addEventListener('keydown', (e) => {
        this.handleKeyboard(e, index);
      });
    });
    
    this.panels.forEach((panel, index) => {
      // ARIA attributes for panels
      panel.setAttribute('role', 'tabpanel');
      panel.setAttribute('id', `panel-${index}`);
      panel.setAttribute('aria-labelledby', `tab-${index}`);
      panel.setAttribute('tabindex', '0');
    });
  }
  
  handleKeyboard(event, currentIndex) {
    let newIndex = currentIndex;
    
    switch (event.key) {
      case 'ArrowLeft':
        event.preventDefault();
        newIndex = currentIndex > 0 ? currentIndex - 1 : this.tabs.length - 1;
        break;
      
      case 'ArrowRight':
        event.preventDefault();
        newIndex = currentIndex < this.tabs.length - 1 ? currentIndex + 1 : 0;
        break;
      
      case 'Home':
        event.preventDefault();
        newIndex = 0;
        break;
      
      case 'End':
        event.preventDefault();
        newIndex = this.tabs.length - 1;
        break;
      
      default:
        return;
    }
    
    this.switchTab(newIndex);
    this.tabs[newIndex].focus();
  }
  
  switchTab(index) {
    if (index < 0 || index >= this.tabs.length) return;
    
    // Update previous tab
    this.tabs[this.activeIndex].classList.remove('active');
    this.tabs[this.activeIndex].setAttribute('tabindex', '-1');
    this.tabs[this.activeIndex].setAttribute('aria-selected', 'false');
    this.panels[this.activeIndex].classList.remove('active');
    
    // Update new tab
    this.activeIndex = index;
    this.tabs[this.activeIndex].classList.add('active');
    this.tabs[this.activeIndex].setAttribute('tabindex', '0');
    this.tabs[this.activeIndex].setAttribute('aria-selected', 'true');
    this.panels[this.activeIndex].classList.add('active');
    
    this.onChange(index, this.tabs[index].dataset.tab);
  }
}

// Enhanced CSS with focus styles
const accessibleTabsCSS = `
.tabs {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

.tab-list {
  display: flex;
  border-bottom: 2px solid #e0e0e0;
  gap: 4px;
}

.tab-button {
  padding: 12px 24px;
  background: none;
  border: none;
  border-bottom: 3px solid transparent;
  cursor: pointer;
  font-size: 16px;
  color: #666;
  transition: all 0.2s;
  position: relative;
  bottom: -2px;
}

.tab-button:hover {
  color: #333;
  background: #f5f5f5;
}

.tab-button:focus {
  outline: 2px solid #4285f4;
  outline-offset: -2px;
  z-index: 1;
}

.tab-button.active {
  color: #4285f4;
  border-bottom-color: #4285f4;
  font-weight: 600;
  background: white;
}

.tab-panels {
  padding: 24px;
  background: white;
  border: 1px solid #e0e0e0;
  border-top: none;
}

.tab-panel {
  display: none;
}

.tab-panel.active {
  display: block;
  animation: fadeIn 0.3s ease;
}

.tab-panel:focus {
  outline: 2px solid #4285f4;
  outline-offset: 4px;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
`;
```

### **Approach 3: Dynamic Tabs with Features**
```javascript
/**
 * Full-featured tabs component
 */
class AdvancedTabs {
  constructor(element, options = {}) {
    this.container = element;
    this.activeIndex = options.defaultTab || 0;
    this.orientation = options.orientation || 'horizontal'; // horizontal, vertical
    this.variant = options.variant || 'underline'; // underline, enclosed, pills
    this.onChange = options.onChange || (() => {});
    this.onBeforeChange = options.onBeforeChange || null;
    this.lazy = options.lazy || false; // Lazy load panel content
    this.closeable = options.closeable || false;
    
    this.tabData = [];
    this.tabList = null;
    this.panelContainer = null;
    
    this.init();
  }
  
  init() {
    // Add classes based on options
    this.container.className = `tabs tabs-${this.orientation} tabs-${this.variant}`;
    
    // Create tab list
    this.tabList = document.createElement('div');
    this.tabList.className = 'tab-list';
    this.tabList.setAttribute('role', 'tablist');
    this.tabList.setAttribute('aria-orientation', this.orientation);
    this.container.appendChild(this.tabList);
    
    // Create panel container
    this.panelContainer = document.createElement('div');
    this.panelContainer.className = 'tab-panels';
    this.container.appendChild(this.panelContainer);
    
    // Extract existing tabs if any
    this.extractExistingTabs();
  }
  
  extractExistingTabs() {
    const existingButtons = this.container.querySelectorAll('.tab-button');
    const existingPanels = this.container.querySelectorAll('.tab-panel');
    
    existingButtons.forEach((button, index) => {
      const panel = existingPanels[index];
      if (panel) {
        this.addTab({
          id: button.dataset.tab || `tab-${index}`,
          label: button.textContent,
          content: panel.innerHTML,
          closeable: button.dataset.closeable === 'true'
        }, false);
      }
    });
    
    if (this.tabData.length > 0) {
      this.render();
      this.switchTab(this.activeIndex);
    }
  }
  
  addTab(config, render = true) {
    const tabId = config.id || `tab-${Date.now()}`;
    
    this.tabData.push({
      id: tabId,
      label: config.label || 'New Tab',
      content: config.content || '',
      closeable: config.closeable !== undefined ? config.closeable : this.closeable,
      disabled: config.disabled || false,
      badge: config.badge || null
    });
    
    if (render) {
      this.render();
      // Switch to new tab
      this.switchTab(this.tabData.length - 1);
    }
    
    return tabId;
  }
  
  removeTab(index) {
    if (this.tabData.length <= 1) return; // Keep at least one tab
    
    this.tabData.splice(index, 1);
    
    // Adjust active index if needed
    if (this.activeIndex >= this.tabData.length) {
      this.activeIndex = this.tabData.length - 1;
    }
    
    this.render();
    this.switchTab(this.activeIndex);
  }
  
  updateTab(index, updates) {
    if (index < 0 || index >= this.tabData.length) return;
    
    Object.assign(this.tabData[index], updates);
    this.render();
  }
  
  render() {
    this.tabList.innerHTML = '';
    this.panelContainer.innerHTML = '';
    
    this.tabData.forEach((tab, index) => {
      this.renderTab(tab, index);
      this.renderPanel(tab, index);
    });
  }
  
  renderTab(tab, index) {
    const button = document.createElement('button');
    button.className = 'tab-button';
    button.setAttribute('role', 'tab');
    button.setAttribute('id', `tab-${index}`);
    button.setAttribute('aria-controls', `panel-${index}`);
    button.setAttribute('aria-selected', index === this.activeIndex ? 'true' : 'false');
    button.setAttribute('tabindex', index === this.activeIndex ? '0' : '-1');
    
    if (tab.disabled) {
      button.disabled = true;
      button.classList.add('disabled');
    }
    
    if (index === this.activeIndex) {
      button.classList.add('active');
    }
    
    // Tab label
    const label = document.createElement('span');
    label.className = 'tab-label';
    label.textContent = tab.label;
    button.appendChild(label);
    
    // Badge
    if (tab.badge) {
      const badge = document.createElement('span');
      badge.className = 'tab-badge';
      badge.textContent = tab.badge;
      button.appendChild(badge);
    }
    
    // Close button
    if (tab.closeable && this.tabData.length > 1) {
      const closeBtn = document.createElement('span');
      closeBtn.className = 'tab-close';
      closeBtn.innerHTML = '×';
      closeBtn.setAttribute('aria-label', `Close ${tab.label}`);
      
      closeBtn.addEventListener('click', (e) => {
        e.stopPropagation();
        this.removeTab(index);
      });
      
      button.appendChild(closeBtn);
    }
    
    // Event listeners
    button.addEventListener('click', () => {
      if (!tab.disabled) {
        this.switchTab(index);
      }
    });
    
    button.addEventListener('keydown', (e) => {
      this.handleKeyboard(e, index);
    });
    
    this.tabList.appendChild(button);
  }
  
  renderPanel(tab, index) {
    const panel = document.createElement('div');
    panel.className = 'tab-panel';
    panel.setAttribute('role', 'tabpanel');
    panel.setAttribute('id', `panel-${index}`);
    panel.setAttribute('aria-labelledby', `tab-${index}`);
    panel.setAttribute('tabindex', '0');
    
    if (index === this.activeIndex) {
      panel.classList.add('active');
      
      // Render content (lazy loading support)
      if (typeof tab.content === 'function') {
        panel.innerHTML = tab.content();
      } else {
        panel.innerHTML = tab.content;
      }
    } else if (!this.lazy) {
      // Pre-render all panels
      if (typeof tab.content === 'function') {
        panel.innerHTML = tab.content();
      } else {
        panel.innerHTML = tab.content;
      }
    }
    
    this.panelContainer.appendChild(panel);
  }
  
  async switchTab(index) {
    if (index === this.activeIndex || index < 0 || index >= this.tabData.length) {
      return;
    }
    
    if (this.tabData[index].disabled) {
      return;
    }
    
    // Call beforeChange hook
    if (this.onBeforeChange) {
      const canSwitch = await this.onBeforeChange(this.activeIndex, index);
      if (canSwitch === false) return;
    }
    
    const previousIndex = this.activeIndex;
    
    // Update tabs
    const tabs = this.tabList.querySelectorAll('.tab-button');
    tabs[previousIndex].classList.remove('active');
    tabs[previousIndex].setAttribute('aria-selected', 'false');
    tabs[previousIndex].setAttribute('tabindex', '-1');
    
    tabs[index].classList.add('active');
    tabs[index].setAttribute('aria-selected', 'true');
    tabs[index].setAttribute('tabindex', '0');
    
    // Update panels
    const panels = this.panelContainer.querySelectorAll('.tab-panel');
    panels[previousIndex].classList.remove('active');
    
    panels[index].classList.add('active');
    
    // Lazy load content if needed
    if (this.lazy && !panels[index].innerHTML) {
      const tab = this.tabData[index];
      if (typeof tab.content === 'function') {
        panels[index].innerHTML = tab.content();
      } else {
        panels[index].innerHTML = tab.content;
      }
    }
    
    this.activeIndex = index;
    this.onChange(index, this.tabData[index].id);
  }
  
  handleKeyboard(event, currentIndex) {
    let newIndex = currentIndex;
    
    switch (event.key) {
      case 'ArrowLeft':
      case 'ArrowUp':
        event.preventDefault();
        newIndex = currentIndex > 0 ? currentIndex - 1 : this.tabData.length - 1;
        // Skip disabled tabs
        while (this.tabData[newIndex].disabled && newIndex !== currentIndex) {
          newIndex = newIndex > 0 ? newIndex - 1 : this.tabData.length - 1;
        }
        break;
      
      case 'ArrowRight':
      case 'ArrowDown':
        event.preventDefault();
        newIndex = currentIndex < this.tabData.length - 1 ? currentIndex + 1 : 0;
        // Skip disabled tabs
        while (this.tabData[newIndex].disabled && newIndex !== currentIndex) {
          newIndex = newIndex < this.tabData.length - 1 ? newIndex + 1 : 0;
        }
        break;
      
      case 'Home':
        event.preventDefault();
        newIndex = 0;
        break;
      
      case 'End':
        event.preventDefault();
        newIndex = this.tabData.length - 1;
        break;
      
      default:
        return;
    }
    
    if (newIndex !== currentIndex && !this.tabData[newIndex].disabled) {
      this.switchTab(newIndex);
      this.tabList.querySelectorAll('.tab-button')[newIndex].focus();
    }
  }
  
  getActiveIndex() {
    return this.activeIndex;
  }
  
  getActiveTab() {
    return this.tabData[this.activeIndex];
  }
  
  destroy() {
    this.container.innerHTML = '';
  }
}

// Complete CSS
const advancedTabsCSS = `
.tabs {
  width: 100%;
  max-width: 900px;
  margin: 0 auto;
}

/* Horizontal tabs */
.tabs-horizontal {
  display: flex;
  flex-direction: column;
}

.tabs-horizontal .tab-list {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

/* Vertical tabs */
.tabs-vertical {
  display: flex;
  flex-direction: row;
}

.tabs-vertical .tab-list {
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 200px;
}

/* Underline variant */
.tabs-underline .tab-list {
  border-bottom: 2px solid #e0e0e0;
}

.tabs-underline .tab-button {
  padding: 12px 20px;
  background: none;
  border: none;
  border-bottom: 3px solid transparent;
  position: relative;
  bottom: -2px;
}

.tabs-underline .tab-button.active {
  border-bottom-color: #4285f4;
}

/* Enclosed variant */
.tabs-enclosed .tab-button {
  padding: 12px 20px;
  background: #f5f5f5;
  border: 1px solid #e0e0e0;
  border-bottom: none;
  border-radius: 8px 8px 0 0;
}

.tabs-enclosed .tab-button.active {
  background: white;
  border-bottom-color: white;
  position: relative;
  z-index: 1;
  margin-bottom: -1px;
}

.tabs-enclosed .tab-panels {
  border: 1px solid #e0e0e0;
  border-top-color: #e0e0e0;
}

/* Pills variant */
.tabs-pills .tab-button {
  padding: 10px 20px;
  background: #f5f5f5;
  border: none;
  border-radius: 20px;
}

.tabs-pills .tab-button.active {
  background: #4285f4;
  color: white;
}

/* Tab button common styles */
.tab-button {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-size: 15px;
  color: #666;
  transition: all 0.2s;
  white-space: nowrap;
}

.tab-button:hover:not(.disabled) {
  color: #333;
  background: #f5f5f5;
}

.tab-button:focus {
  outline: 2px solid #4285f4;
  outline-offset: 2px;
  z-index: 2;
}

.tab-button.active {
  color: #4285f4;
  font-weight: 600;
}

.tab-button.disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.tab-label {
  flex: 1;
}

.tab-badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 20px;
  height: 20px;
  padding: 0 6px;
  background: #e3f2fd;
  color: #1976d2;
  border-radius: 10px;
  font-size: 12px;
  font-weight: 600;
}

.tab-button.active .tab-badge {
  background: #4285f4;
  color: white;
}

.tab-close {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  font-size: 18px;
  line-height: 1;
  transition: background 0.2s;
}

.tab-close:hover {
  background: rgba(0, 0, 0, 0.1);
}

/* Tab panels */
.tab-panels {
  padding: 24px;
  background: white;
}

.tabs-vertical .tab-panels {
  flex: 1;
  border-left: 1px solid #e0e0e0;
}

.tab-panel {
  display: none;
}

.tab-panel.active {
  display: block;
  animation: fadeIn 0.2s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Responsive */
@media (max-width: 768px) {
  .tabs-vertical {
    flex-direction: column;
  }
  
  .tabs-vertical .tab-list {
    min-width: auto;
    flex-direction: row;
    overflow-x: auto;
  }
  
  .tabs-vertical .tab-panels {
    border-left: none;
    border-top: 1px solid #e0e0e0;
  }
}
`;

// Usage examples
console.log('=== Basic Tabs ===');
// const tabs1 = new Tabs(document.querySelector('.tabs'));

console.log('\n=== Accessible Tabs ===');
// const tabs2 = new AccessibleTabs(document.querySelector('.tabs'));

console.log('\n=== Advanced Tabs ===');
// const tabs3 = new AdvancedTabs(document.querySelector('#advanced-tabs'), {
//   variant: 'underline', // underline, enclosed, pills
//   orientation: 'horizontal',
//   closeable: false,
//   lazy: true,
//   onChange: (index, tabId) => {
//     console.log('Tab changed:', index, tabId);
//   }
// });
// 
// // Add dynamic tabs
// tabs3.addTab({
//   id: 'profile',
//   label: 'Profile',
//   content: '<h3>Profile Content</h3>',
//   badge: '5'
// });
// 
// tabs3.addTab({
//   id: 'settings',
//   label: 'Settings',
//   content: () => {
//     // Lazy load content
//     return '<h3>Settings loaded dynamically</h3>';
//   },
//   closeable: true
// });

// Real-world example: Browser-like tabs
class BrowserTabs extends AdvancedTabs {
  constructor(element) {
    super(element, {
      variant: 'enclosed',
      closeable: true,
      lazy: true
    });
    
    // Add "New Tab" button
    this.addNewTabButton();
  }
  
  addNewTabButton() {
    const newTabBtn = document.createElement('button');
    newTabBtn.className = 'new-tab-button';
    newTabBtn.textContent = '+';
    newTabBtn.setAttribute('aria-label', 'New tab');
    
    newTabBtn.addEventListener('click', () => {
      this.addTab({
        label: 'New Tab',
        content: '<p>Empty tab</p>',
        closeable: true
      });
    });
    
    this.tabList.appendChild(newTabBtn);
  }
}

// const browserTabs = new BrowserTabs(document.querySelector('#browser-tabs'));
```

**Interview Tips:**
- Tabs: UI pattern for organizing content in separate panels
- Key features: single active tab, keyboard navigation (arrows, Home, End)
- Accessibility: ARIA roles (tab, tablist, tabpanel), aria-selected, focus management
- Keyboard: Arrow keys to navigate, Space/Enter to activate
- Variants: underline (Google), enclosed (traditional), pills (Bootstrap)
- Orientations: horizontal (common) or vertical (sidebar)
- Advanced features: closeable tabs, badges, disabled state, lazy loading
- Dynamic: add/remove tabs programmatically
- State management: track active index, handle tab changes
- Common patterns: browser tabs, settings pages, dashboards
- Libraries: react-tabs, @reach/tabs, Material-UI Tabs
- Considerations: mobile responsiveness, overflow handling, animations

</details>

