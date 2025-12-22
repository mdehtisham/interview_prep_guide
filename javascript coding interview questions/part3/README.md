
---

## **EXTRA HARD LEVEL (96-120)**

### **Complex Data Structures & Algorithms**

96. Implement a Trie (Prefix Tree) for autocomplete

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Trie Implementation**
```javascript
/**
 * Basic Trie (Prefix Tree) for word storage and search
 * Time Complexity: 
 *   - Insert: O(m) where m is word length
 *   - Search: O(m)
 *   - StartsWith: O(p) where p is prefix length
 * Space Complexity: O(n * m) where n is number of words
 */
class TrieNode {
  constructor() {
    this.children = {};
    this.isEndOfWord = false;
  }
}

class Trie {
  constructor() {
    this.root = new TrieNode();
  }
  
  insert(word) {
    let node = this.root;
    
    for (const char of word) {
      if (!node.children[char]) {
        node.children[char] = new TrieNode();
      }
      node = node.children[char];
    }
    
    node.isEndOfWord = true;
  }
  
  search(word) {
    let node = this.root;
    
    for (const char of word) {
      if (!node.children[char]) {
        return false;
      }
      node = node.children[char];
    }
    
    return node.isEndOfWord;
  }
  
  startsWith(prefix) {
    let node = this.root;
    
    for (const char of prefix) {
      if (!node.children[char]) {
        return false;
      }
      node = node.children[char];
    }
    
    return true;
  }
}

// Test
console.log('=== Basic Trie ===');

const trie = new Trie();

trie.insert('apple');
trie.insert('app');
trie.insert('application');

console.log('Search "apple":', trie.search('apple')); // true
console.log('Search "app":', trie.search('app')); // true
console.log('Search "appl":', trie.search('appl')); // false
console.log('StartsWith "app":', trie.startsWith('app')); // true
console.log('StartsWith "apl":', trie.startsWith('apl')); // false
```

### **Approach 2: Trie with Autocomplete**
```javascript
/**
 * Trie with autocomplete/suggestion functionality
 */
class AutocompleteTrie {
  constructor() {
    this.root = new TrieNode();
  }
  
  insert(word) {
    let node = this.root;
    
    for (const char of word.toLowerCase()) {
      if (!node.children[char]) {
        node.children[char] = new TrieNode();
      }
      node = node.children[char];
    }
    
    node.isEndOfWord = true;
  }
  
  // Get all words with given prefix
  autocomplete(prefix) {
    let node = this.root;
    prefix = prefix.toLowerCase();
    
    // Navigate to prefix node
    for (const char of prefix) {
      if (!node.children[char]) {
        return [];
      }
      node = node.children[char];
    }
    
    // Collect all words from this node
    const results = [];
    this._collectWords(node, prefix, results);
    return results;
  }
  
  _collectWords(node, currentWord, results) {
    if (node.isEndOfWord) {
      results.push(currentWord);
    }
    
    for (const char in node.children) {
      this._collectWords(
        node.children[char],
        currentWord + char,
        results
      );
    }
  }
}

// Test
console.log('\n=== Autocomplete Trie ===');

const autoTrie = new AutocompleteTrie();

const words = ['apple', 'app', 'application', 'apply', 'apricot', 'banana'];
words.forEach(word => autoTrie.insert(word));

console.log('Autocomplete "app":', autoTrie.autocomplete('app'));
// ['app', 'apple', 'application', 'apply']

console.log('Autocomplete "apr":', autoTrie.autocomplete('apr'));
// ['apricot']

console.log('Autocomplete "ban":', autoTrie.autocomplete('ban'));
// ['banana']

console.log('Autocomplete "xyz":', autoTrie.autocomplete('xyz'));
// []
```

### **Approach 3: Trie with Word Frequency**
```javascript
/**
 * Trie that tracks word frequency for ranking suggestions
 */
class TrieNodeWithFreq {
  constructor() {
    this.children = {};
    this.isEndOfWord = false;
    this.frequency = 0;
  }
}

class FrequencyTrie {
  constructor() {
    this.root = new TrieNodeWithFreq();
  }
  
  insert(word) {
    let node = this.root;
    
    for (const char of word.toLowerCase()) {
      if (!node.children[char]) {
        node.children[char] = new TrieNodeWithFreq();
      }
      node = node.children[char];
    }
    
    node.isEndOfWord = true;
    node.frequency++;
  }
  
  autocomplete(prefix, limit = 5) {
    let node = this.root;
    prefix = prefix.toLowerCase();
    
    // Navigate to prefix
    for (const char of prefix) {
      if (!node.children[char]) {
        return [];
      }
      node = node.children[char];
    }
    
    // Collect words with frequencies
    const results = [];
    this._collectWithFreq(node, prefix, results);
    
    // Sort by frequency (descending) and limit results
    return results
      .sort((a, b) => b.frequency - a.frequency)
      .slice(0, limit)
      .map(item => item.word);
  }
  
  _collectWithFreq(node, currentWord, results) {
    if (node.isEndOfWord) {
      results.push({ word: currentWord, frequency: node.frequency });
    }
    
    for (const char in node.children) {
      this._collectWithFreq(
        node.children[char],
        currentWord + char,
        results
      );
    }
  }
  
  getFrequency(word) {
    let node = this.root;
    
    for (const char of word.toLowerCase()) {
      if (!node.children[char]) {
        return 0;
      }
      node = node.children[char];
    }
    
    return node.isEndOfWord ? node.frequency : 0;
  }
}

// Test
console.log('\n=== Frequency Trie ===');

const freqTrie = new FrequencyTrie();

// Insert with varying frequencies
freqTrie.insert('apple');
freqTrie.insert('apple');
freqTrie.insert('apple');
freqTrie.insert('app');
freqTrie.insert('app');
freqTrie.insert('application');
freqTrie.insert('apply');

console.log('Frequency of "apple":', freqTrie.getFrequency('apple')); // 3
console.log('Frequency of "app":', freqTrie.getFrequency('app')); // 2

console.log('Top autocomplete "app":', freqTrie.autocomplete('app', 3));
// ['apple', 'app', 'application'] (sorted by frequency)
```

### **Approach 4: Trie with Delete Operation**
```javascript
/**
 * Trie with delete functionality
 */
class DeletableTrie {
  constructor() {
    this.root = new TrieNode();
  }
  
  insert(word) {
    let node = this.root;
    
    for (const char of word) {
      if (!node.children[char]) {
        node.children[char] = new TrieNode();
      }
      node = node.children[char];
    }
    
    node.isEndOfWord = true;
  }
  
  delete(word) {
    return this._deleteHelper(this.root, word, 0);
  }
  
  _deleteHelper(node, word, index) {
    if (index === word.length) {
      // Reached end of word
      if (!node.isEndOfWord) {
        return false; // Word doesn't exist
      }
      
      node.isEndOfWord = false;
      
      // Delete node if it has no children
      return Object.keys(node.children).length === 0;
    }
    
    const char = word[index];
    const childNode = node.children[char];
    
    if (!childNode) {
      return false; // Word doesn't exist
    }
    
    const shouldDeleteChild = this._deleteHelper(childNode, word, index + 1);
    
    if (shouldDeleteChild) {
      delete node.children[char];
      
      // Delete current node if it has no children and isn't end of word
      return Object.keys(node.children).length === 0 && !node.isEndOfWord;
    }
    
    return false;
  }
  
  search(word) {
    let node = this.root;
    
    for (const char of word) {
      if (!node.children[char]) {
        return false;
      }
      node = node.children[char];
    }
    
    return node.isEndOfWord;
  }
}

// Test
console.log('\n=== Deletable Trie ===');

const delTrie = new DeletableTrie();

delTrie.insert('apple');
delTrie.insert('app');
delTrie.insert('application');

console.log('Before delete:');
console.log('  Search "apple":', delTrie.search('apple')); // true
console.log('  Search "app":', delTrie.search('app')); // true

delTrie.delete('apple');

console.log('After delete "apple":');
console.log('  Search "apple":', delTrie.search('apple')); // false
console.log('  Search "app":', delTrie.search('app')); // true (still exists)
```

### **Approach 5: Advanced Trie with Multiple Features**
```javascript
/**
 * Feature-rich Trie implementation
 */
class AdvancedTrieNode {
  constructor() {
    this.children = {};
    this.isEndOfWord = false;
    this.frequency = 0;
    this.data = null; // Store additional data
  }
}

class AdvancedTrie {
  constructor() {
    this.root = new AdvancedTrieNode();
    this.wordCount = 0;
  }
  
  insert(word, data = null) {
    let node = this.root;
    word = word.toLowerCase();
    
    for (const char of word) {
      if (!node.children[char]) {
        node.children[char] = new AdvancedTrieNode();
      }
      node = node.children[char];
    }
    
    if (!node.isEndOfWord) {
      this.wordCount++;
    }
    
    node.isEndOfWord = true;
    node.frequency++;
    node.data = data;
  }
  
  search(word) {
    const node = this._findNode(word.toLowerCase());
    return node ? node.isEndOfWord : false;
  }
  
  startsWith(prefix) {
    return this._findNode(prefix.toLowerCase()) !== null;
  }
  
  autocomplete(prefix, options = {}) {
    const {
      limit = 10,
      sortBy = 'frequency', // 'frequency' or 'alphabetical'
      includeData = false
    } = options;
    
    const node = this._findNode(prefix.toLowerCase());
    if (!node) return [];
    
    const results = [];
    this._collectWords(node, prefix.toLowerCase(), results, includeData);
    
    // Sort results
    if (sortBy === 'frequency') {
      results.sort((a, b) => b.frequency - a.frequency);
    } else {
      results.sort((a, b) => a.word.localeCompare(b.word));
    }
    
    return results.slice(0, limit).map(item => 
      includeData ? item : item.word
    );
  }
  
  delete(word) {
    if (this._deleteHelper(this.root, word.toLowerCase(), 0)) {
      this.wordCount--;
      return true;
    }
    return false;
  }
  
  getAllWords() {
    const results = [];
    this._collectWords(this.root, '', results, false);
    return results.map(item => item.word);
  }
  
  getWordCount() {
    return this.wordCount;
  }
  
  longestCommonPrefix() {
    let prefix = '';
    let node = this.root;
    
    while (Object.keys(node.children).length === 1 && !node.isEndOfWord) {
      const char = Object.keys(node.children)[0];
      prefix += char;
      node = node.children[char];
    }
    
    return prefix;
  }
  
  _findNode(word) {
    let node = this.root;
    
    for (const char of word) {
      if (!node.children[char]) {
        return null;
      }
      node = node.children[char];
    }
    
    return node;
  }
  
  _collectWords(node, currentWord, results, includeData) {
    if (node.isEndOfWord) {
      const result = {
        word: currentWord,
        frequency: node.frequency
      };
      
      if (includeData) {
        result.data = node.data;
      }
      
      results.push(result);
    }
    
    for (const char in node.children) {
      this._collectWords(
        node.children[char],
        currentWord + char,
        results,
        includeData
      );
    }
  }
  
  _deleteHelper(node, word, index) {
    if (index === word.length) {
      if (!node.isEndOfWord) return false;
      node.isEndOfWord = false;
      node.frequency = 0;
      node.data = null;
      return Object.keys(node.children).length === 0;
    }
    
    const char = word[index];
    if (!node.children[char]) return false;
    
    const shouldDelete = this._deleteHelper(node.children[char], word, index + 1);
    
    if (shouldDelete) {
      delete node.children[char];
      return Object.keys(node.children).length === 0 && !node.isEndOfWord;
    }
    
    return false;
  }
}

// Test
console.log('\n=== Advanced Trie ===');

const advTrie = new AdvancedTrie();

// Insert with data
advTrie.insert('apple', { price: 1.99, category: 'fruit' });
advTrie.insert('apple'); // Increment frequency
advTrie.insert('application', { type: 'software' });
advTrie.insert('app', { type: 'mobile' });

console.log('Word count:', advTrie.getWordCount());
console.log('All words:', advTrie.getAllWords());

console.log('Autocomplete "app" (by frequency):',
  advTrie.autocomplete('app', { limit: 3, sortBy: 'frequency' }));

console.log('Autocomplete "app" (alphabetical):',
  advTrie.autocomplete('app', { limit: 3, sortBy: 'alphabetical' }));

console.log('Autocomplete with data:',
  advTrie.autocomplete('app', { includeData: true }));
```

### **Bonus: Compressed Trie (PATRICIA Trie)**
```javascript
/**
 * Compressed Trie that stores strings on edges
 * More space-efficient for sparse tries
 */
class CompressedTrieNode {
  constructor() {
    this.children = new Map();
    this.isEndOfWord = false;
  }
}

class CompressedTrie {
  constructor() {
    this.root = new CompressedTrieNode();
  }
  
  insert(word) {
    this._insertHelper(this.root, word, 0);
  }
  
  _insertHelper(node, word, index) {
    if (index === word.length) {
      node.isEndOfWord = true;
      return;
    }
    
    const char = word[index];
    
    // Try to find matching edge
    for (const [edge, child] of node.children) {
      const commonLength = this._getCommonPrefixLength(edge, word.slice(index));
      
      if (commonLength > 0) {
        if (commonLength === edge.length) {
          // Full edge match, continue down
          this._insertHelper(child, word, index + commonLength);
          return;
        } else {
          // Partial match, need to split edge
          const commonPrefix = edge.slice(0, commonLength);
          const oldSuffix = edge.slice(commonLength);
          const newSuffix = word.slice(index + commonLength);
          
          // Create intermediate node
          const intermediateNode = new CompressedTrieNode();
          
          // Move old child
          node.children.delete(edge);
          node.children.set(commonPrefix, intermediateNode);
          intermediateNode.children.set(oldSuffix, child);
          
          // Add new path
          if (newSuffix.length > 0) {
            intermediateNode.children.set(newSuffix, new CompressedTrieNode());
            intermediateNode.children.get(newSuffix).isEndOfWord = true;
          } else {
            intermediateNode.isEndOfWord = true;
          }
          
          return;
        }
      }
    }
    
    // No matching edge, create new
    const newNode = new CompressedTrieNode();
    newNode.isEndOfWord = true;
    node.children.set(word.slice(index), newNode);
  }
  
  search(word) {
    return this._searchHelper(this.root, word, 0);
  }
  
  _searchHelper(node, word, index) {
    if (index === word.length) {
      return node.isEndOfWord;
    }
    
    for (const [edge, child] of node.children) {
      if (word.slice(index).startsWith(edge)) {
        return this._searchHelper(child, word, index + edge.length);
      }
    }
    
    return false;
  }
  
  _getCommonPrefixLength(str1, str2) {
    let i = 0;
    while (i < str1.length && i < str2.length && str1[i] === str2[i]) {
      i++;
    }
    return i;
  }
}

// Test
console.log('\n=== Compressed Trie ===');

const compTrie = new CompressedTrie();

compTrie.insert('romane');
compTrie.insert('romanus');
compTrie.insert('romulus');
compTrie.insert('rubens');
compTrie.insert('ruber');
compTrie.insert('rubicon');
compTrie.insert('rubicundus');

console.log('Search "romane":', compTrie.search('romane')); // true
console.log('Search "roman":', compTrie.search('roman')); // false
console.log('Search "rubicon":', compTrie.search('rubicon')); // true
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications of Trie
 */****

// 1. Search Engine Autocomplete
console.log('\n=== Search Autocomplete ===');

class SearchEngine {
  constructor() {
    this.trie = new FrequencyTrie();
  }
  
  indexQuery(query) {
    this.trie.insert(query);
  }
  
  suggest(prefix, limit = 5) {
    return this.trie.autocomplete(prefix, limit);
  }
}

const search = new SearchEngine();

// Simulate user searches
['javascript tutorial', 'javascript array', 'javascript array methods',
 'javascript async', 'java programming', 'python basics'].forEach(query => {
  search.indexQuery(query);
});

// Some searches are more popular
search.indexQuery('javascript tutorial');
search.indexQuery('javascript tutorial');

console.log('Suggestions for "java":', search.suggest('java', 3));
console.log('Suggestions for "javascript a":', search.suggest('javascript a', 3));

// 2. Spell Checker
console.log('\n=== Spell Checker ===');

class SpellChecker {
  constructor(dictionary) {
    this.trie = new Trie();
    dictionary.forEach(word => this.trie.insert(word.toLowerCase()));
  }
  
  isCorrect(word) {
    return this.trie.search(word.toLowerCase());
  }
  
  suggest(word) {
    word = word.toLowerCase();
    
    // If correct, no suggestions needed
    if (this.isCorrect(word)) {
      return [];
    }
    
    // Find similar words (simple approach)
    const suggestions = [];
    
    // Try removing each character
    for (let i = 0; i < word.length; i++) {
      const candidate = word.slice(0, i) + word.slice(i + 1);
      if (this.isCorrect(candidate)) {
        suggestions.push(candidate);
      }
    }
    
    // Try replacing each character
    for (let i = 0; i < word.length; i++) {
      for (let c = 97; c <= 122; c++) { // a-z
        const char = String.fromCharCode(c);
        const candidate = word.slice(0, i) + char + word.slice(i + 1);
        if (this.isCorrect(candidate) && !suggestions.includes(candidate)) {
          suggestions.push(candidate);
        }
      }
    }
    
    return suggestions.slice(0, 5);
  }
}

const dictionary = ['apple', 'application', 'apply', 'banana', 'band', 'can'];
const spellChecker = new SpellChecker(dictionary);

console.log('Is "apple" correct?', spellChecker.isCorrect('apple'));
console.log('Is "aple" correct?', spellChecker.isCorrect('aple'));
console.log('Suggestions for "aple":', spellChecker.suggest('aple'));

// 3. IP Routing Table
console.log('\n=== IP Routing ===');

class IPRouter {
  constructor() {
    this.trie = new AdvancedTrie();
  }
  
  addRoute(ipPrefix, gateway) {
    // Convert IP to binary string for trie
    const binaryIP = ipPrefix.split('.')
      .map(octet => parseInt(octet).toString(2).padStart(8, '0'))
      .join('');
    
    this.trie.insert(binaryIP, { gateway, originalIP: ipPrefix });
  }
  
  findRoute(ip) {
    const binaryIP = ip.split('.')
      .map(octet => parseInt(octet).toString(2).padStart(8, '0'))
      .join('');
    
    // Find longest prefix match
    let longestMatch = null;
    let maxLength = 0;
    
    for (let i = 1; i <= binaryIP.length; i++) {
      const prefix = binaryIP.slice(0, i);
      const node = this.trie._findNode(prefix);
      
      if (node && node.isEndOfWord) {
        if (i > maxLength) {
          maxLength = i;
          longestMatch = node.data;
        }
      }
    }
    
    return longestMatch;
  }
}

const router = new IPRouter();
router.addRoute('192.168', 'Gateway A');
router.addRoute('192.168.1', 'Gateway B');
router.addRoute('10', 'Gateway C');

console.log('Route for 192.168.1.100:', router.findRoute('192.168.1.100'));
console.log('Route for 192.168.5.1:', router.findRoute('192.168.5.1'));
console.log('Route for 10.0.0.1:', router.findRoute('10.0.0.1'));

// 4. Contact Search
console.log('\n=== Contact Search ===');

class ContactList {
  constructor() {
    this.trie = new AdvancedTrie();
  }
  
  addContact(name, phone, email) {
    // Index by name
    this.trie.insert(name.toLowerCase(), { name, phone, email });
    
    // Also index by phone
    this.trie.insert(phone, { name, phone, email });
  }
  
  search(query) {
    return this.trie.autocomplete(query.toLowerCase(), {
      limit: 10,
      includeData: true,
      sortBy: 'alphabetical'
    });
  }
}

const contacts = new ContactList();
contacts.addContact('Alice Smith', '555-0100', 'alice@email.com');
contacts.addContact('Alice Johnson', '555-0101', 'alicej@email.com');
contacts.addContact('Bob Wilson', '555-0200', 'bob@email.com');

console.log('Search "alice":', contacts.search('alice'));
console.log('Search "555":', contacts.search('555'));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

const edgeTrie = new Trie();

// Empty string
edgeTrie.insert('');
console.log('Search empty string:', edgeTrie.search('')); // true

// Single character
edgeTrie.insert('a');
console.log('Search "a":', edgeTrie.search('a')); // true

// Long word
const longWord = 'a'.repeat(1000);
edgeTrie.insert(longWord);
console.log('Search long word:', edgeTrie.search(longWord)); // true

// Case sensitivity
const caseTrie = new AutocompleteTrie();
caseTrie.insert('Apple');
caseTrie.insert('APPLE');
console.log('Case insensitive autocomplete:', caseTrie.autocomplete('app'));

// Special characters
const specialTrie = new Trie();
specialTrie.insert('hello-world');
specialTrie.insert('hello_world');
console.log('Search with dash:', specialTrie.search('hello-world')); // true

// Overlapping words
const overlapTrie = new Trie();
overlapTrie.insert('test');
overlapTrie.insert('testing');
overlapTrie.insert('tester');
console.log('Search "test":', overlapTrie.search('test')); // true
console.log('Search "testing":', overlapTrie.search('testing')); // true
```

### **Performance Benchmarks**
```javascript
/**
 * Compare Trie vs Array vs Object
 */
console.log('\n=== Performance Comparison ===');

const words = [];
for (let i = 0; i < 1000; i++) {
  words.push('word' + i);
}

// Trie
const perfTrie = new Trie();
console.time('Trie insert 1000 words');
words.forEach(word => perfTrie.insert(word));
console.timeEnd('Trie insert 1000 words');

console.time('Trie search 1000 words');
words.forEach(word => perfTrie.search(word));
console.timeEnd('Trie search 1000 words');

// Array
console.time('Array insert 1000 words');
const arr = [...words];
console.timeEnd('Array insert 1000 words');

console.time('Array search 1000 words');
words.forEach(word => arr.includes(word));
console.timeEnd('Array search 1000 words');

// Set
console.time('Set insert 1000 words');
const set = new Set(words);
console.timeEnd('Set insert 1000 words');

console.time('Set search 1000 words');
words.forEach(word => set.has(word));
console.timeEnd('Set search 1000 words');

console.log(`
Performance Analysis:
┌──────────────┬────────────┬────────────┬────────────────┐
│ Operation    │ Trie       │ Array      │ Set            │
├──────────────┼────────────┼────────────┼────────────────┤
│ Insert       │ O(m)       │ O(1)       │ O(1)           │
│ Search       │ O(m)       │ O(n)       │ O(1) avg       │
│ Prefix       │ O(p)       │ O(n*m)     │ O(n*m)         │
│ Autocomplete │ O(p+k)     │ O(n*m)     │ O(n*m)         │
│ Space        │ O(n*m)     │ O(n*m)     │ O(n*m)         │
└──────────────┴────────────┴────────────┴────────────────┘

Where:
  m = average word length
  n = number of words
  p = prefix length
  k = number of matching words

Best Use Cases:
• Trie: Prefix search, autocomplete, spell checking
• Array: Small datasets, insertion order matters
• Set: Fast exact match lookup, no prefix operations
`);
```

**Interview Tips:**
- Trie (prefix tree): tree where each node represents a character
- Each path from root to leaf represents a word
- Children stored in map/object: { 'a': node, 'b': node }
- isEndOfWord flag: marks complete words vs prefixes
- Insert: traverse/create nodes for each character, mark last as end
- Search: traverse nodes, check if final node has isEndOfWord=true
- StartsWith: traverse nodes for prefix, return true if path exists
- Autocomplete: navigate to prefix node, collect all words below it
- Time: O(m) for insert/search where m is word length
- Space: O(ALPHABET_SIZE * n * m) worst case, typically O(n * m)
- Advantages: fast prefix operations, memory efficient for common prefixes
- Applications: autocomplete, spell checker, IP routing, T9 predictive text
- Frequency: track insertion count for ranking suggestions
- Delete: recursive removal, clean up unused nodes
- Compressed Trie (PATRICIA): store strings on edges, more space-efficient
- Ternary Search Tree: uses less memory than standard trie
- Edge cases: empty string, single char, case sensitivity, special characters
- Optimization: use arrays instead of objects for fixed alphabet
- Alternative: suffix tree for substring search
- Clarify: case sensitive? special characters? delete needed? frequency ranking?
- Follow-ups: add wildcard search, implement prefix count, optimize memory

</details>

97. Implement a Binary Search Tree with insert, delete, and search

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic BST Implementation**
```javascript
/**
 * Basic Binary Search Tree
 * Time Complexity:
 *   - Insert: O(h) where h is height, O(log n) average, O(n) worst
 *   - Search: O(h)
 *   - Delete: O(h)
 * Space Complexity: O(n) for n nodes
 */
class TreeNode {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null;
  }
}

class BinarySearchTree {
  constructor() {
    this.root = null;
  }
  
  insert(value) {
    const newNode = new TreeNode(value);
    
    if (!this.root) {
      this.root = newNode;
      return this;
    }
    
    let current = this.root;
    
    while (true) {
      if (value === current.value) {
        return this; // Duplicate, ignore
      }
      
      if (value < current.value) {
        if (!current.left) {
          current.left = newNode;
          return this;
        }
        current = current.left;
      } else {
        if (!current.right) {
          current.right = newNode;
          return this;
        }
        current = current.right;
      }
    }
  }
  
  search(value) {
    let current = this.root;
    
    while (current) {
      if (value === current.value) {
        return true;
      }
      
      if (value < current.value) {
        current = current.left;
      } else {
        current = current.right;
      }
    }
    
    return false;
  }
  
  delete(value) {
    this.root = this._deleteNode(this.root, value);
    return this;
  }
  
  _deleteNode(node, value) {
    if (!node) {
      return null;
    }
    
    if (value < node.value) {
      node.left = this._deleteNode(node.left, value);
      return node;
    }
    
    if (value > node.value) {
      node.right = this._deleteNode(node.right, value);
      return node;
    }
    
    // Found node to delete
    
    // Case 1: No children (leaf node)
    if (!node.left && !node.right) {
      return null;
    }
    
    // Case 2: One child
    if (!node.left) {
      return node.right;
    }
    
    if (!node.right) {
      return node.left;
    }
    
    // Case 3: Two children
    // Find in-order successor (smallest in right subtree)
    let successor = node.right;
    while (successor.left) {
      successor = successor.left;
    }
    
    // Replace node value with successor value
    node.value = successor.value;
    
    // Delete successor
    node.right = this._deleteNode(node.right, successor.value);
    
    return node;
  }
}

// Test
console.log('=== Basic BST ===');

const bst = new BinarySearchTree();

// Insert values
[50, 30, 70, 20, 40, 60, 80].forEach(val => bst.insert(val));

console.log('Search 40:', bst.search(40)); // true
console.log('Search 25:', bst.search(25)); // false

bst.delete(20); // Delete leaf
console.log('After delete 20, search 20:', bst.search(20)); // false

bst.delete(30); // Delete node with two children
console.log('After delete 30, search 30:', bst.search(30)); // false
console.log('Search 40:', bst.search(40)); // true (still exists)
```

### **Approach 2: Recursive BST**
```javascript
/**
 * BST with recursive implementations
 */
class RecursiveBST {
  constructor() {
    this.root = null;
  }
  
  insert(value) {
    this.root = this._insertRecursive(this.root, value);
    return this;
  }
  
  _insertRecursive(node, value) {
    if (!node) {
      return new TreeNode(value);
    }
    
    if (value < node.value) {
      node.left = this._insertRecursive(node.left, value);
    } else if (value > node.value) {
      node.right = this._insertRecursive(node.right, value);
    }
    
    return node;
  }
  
  search(value) {
    return this._searchRecursive(this.root, value);
  }
  
  _searchRecursive(node, value) {
    if (!node) {
      return false;
    }
    
    if (value === node.value) {
      return true;
    }
    
    if (value < node.value) {
      return this._searchRecursive(node.left, value);
    }
    
    return this._searchRecursive(node.right, value);
  }
  
  delete(value) {
    this.root = this._deleteRecursive(this.root, value);
    return this;
  }
  
  _deleteRecursive(node, value) {
    if (!node) {
      return null;
    }
    
    if (value < node.value) {
      node.left = this._deleteRecursive(node.left, value);
    } else if (value > node.value) {
      node.right = this._deleteRecursive(node.right, value);
    } else {
      // Node found
      
      if (!node.left && !node.right) {
        return null;
      }
      
      if (!node.left) {
        return node.right;
      }
      
      if (!node.right) {
        return node.left;
      }
      
      // Two children: find min in right subtree
      const minRight = this._findMin(node.right);
      node.value = minRight.value;
      node.right = this._deleteRecursive(node.right, minRight.value);
    }
    
    return node;
  }
  
  _findMin(node) {
    while (node.left) {
      node = node.left;
    }
    return node;
  }
  
  findMin() {
    if (!this.root) return null;
    return this._findMin(this.root).value;
  }
  
  findMax() {
    if (!this.root) return null;
    let current = this.root;
    while (current.right) {
      current = current.right;
    }
    return current.value;
  }
}

// Test
console.log('\n=== Recursive BST ===');

const recBst = new RecursiveBST();

[50, 30, 70, 20, 40, 60, 80].forEach(val => recBst.insert(val));

console.log('Min value:', recBst.findMin()); // 20
console.log('Max value:', recBst.findMax()); // 80
console.log('Search 40:', recBst.search(40)); // true

recBst.delete(50); // Delete root
console.log('After delete 50:', recBst.search(50)); // false
```

### **Approach 3: BST with Traversals**
```javascript
/**
 * BST with different traversal methods
 */
class TraversalBST extends RecursiveBST {
  inOrderTraversal(callback) {
    this._inOrder(this.root, callback);
  }
  
  _inOrder(node, callback) {
    if (!node) return;
    
    this._inOrder(node.left, callback);
    callback(node.value);
    this._inOrder(node.right, callback);
  }
  
  preOrderTraversal(callback) {
    this._preOrder(this.root, callback);
  }
  
  _preOrder(node, callback) {
    if (!node) return;
    
    callback(node.value);
    this._preOrder(node.left, callback);
    this._preOrder(node.right, callback);
  }
  
  postOrderTraversal(callback) {
    this._postOrder(this.root, callback);
  }
  
  _postOrder(node, callback) {
    if (!node) return;
    
    this._postOrder(node.left, callback);
    this._postOrder(node.right, callback);
    callback(node.value);
  }
  
  levelOrderTraversal(callback) {
    if (!this.root) return;
    
    const queue = [this.root];
    
    while (queue.length > 0) {
      const node = queue.shift();
      callback(node.value);
      
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
  
  toArray(order = 'inorder') {
    const result = [];
    const callback = (val) => result.push(val);
    
    switch (order) {
      case 'inorder':
        this.inOrderTraversal(callback);
        break;
      case 'preorder':
        this.preOrderTraversal(callback);
        break;
      case 'postorder':
        this.postOrderTraversal(callback);
        break;
      case 'levelorder':
        this.levelOrderTraversal(callback);
        break;
    }
    
    return result;
  }
}

// Test
console.log('\n=== BST Traversals ===');

const travBst = new TraversalBST();
[50, 30, 70, 20, 40, 60, 80].forEach(val => travBst.insert(val));

console.log('In-order (sorted):', travBst.toArray('inorder'));
// [20, 30, 40, 50, 60, 70, 80]

console.log('Pre-order:', travBst.toArray('preorder'));
// [50, 30, 20, 40, 70, 60, 80]

console.log('Post-order:', travBst.toArray('postorder'));
// [20, 40, 30, 60, 80, 70, 50]

console.log('Level-order:', travBst.toArray('levelorder'));
// [50, 30, 70, 20, 40, 60, 80]
```

### **Approach 4: BST with Additional Operations**
```javascript
/**
 * Feature-rich BST
 */
class AdvancedBST extends TraversalBST {
  getHeight() {
    return this._getHeightRecursive(this.root);
  }
  
  _getHeightRecursive(node) {
    if (!node) return -1;
    
    return 1 + Math.max(
      this._getHeightRecursive(node.left),
      this._getHeightRecursive(node.right)
    );
  }
  
  isBalanced() {
    return this._checkBalance(this.root) !== -1;
  }
  
  _checkBalance(node) {
    if (!node) return 0;
    
    const leftHeight = this._checkBalance(node.left);
    if (leftHeight === -1) return -1;
    
    const rightHeight = this._checkBalance(node.right);
    if (rightHeight === -1) return -1;
    
    if (Math.abs(leftHeight - rightHeight) > 1) {
      return -1; // Unbalanced
    }
    
    return 1 + Math.max(leftHeight, rightHeight);
  }
  
  countNodes() {
    return this._countNodesRecursive(this.root);
  }
  
  _countNodesRecursive(node) {
    if (!node) return 0;
    
    return 1 + 
           this._countNodesRecursive(node.left) + 
           this._countNodesRecursive(node.right);
  }
  
  findClosest(target) {
    if (!this.root) return null;
    
    let closest = this.root.value;
    let minDiff = Math.abs(target - closest);
    let current = this.root;
    
    while (current) {
      const diff = Math.abs(target - current.value);
      
      if (diff < minDiff) {
        minDiff = diff;
        closest = current.value;
      }
      
      if (target === current.value) {
        return current.value;
      }
      
      current = target < current.value ? current.left : current.right;
    }
    
    return closest;
  }
  
  findKthSmallest(k) {
    const result = { count: 0, value: null };
    
    this._findKthHelper(this.root, k, result);
    
    return result.value;
  }
  
  _findKthHelper(node, k, result) {
    if (!node || result.value !== null) return;
    
    this._findKthHelper(node.left, k, result);
    
    result.count++;
    if (result.count === k) {
      result.value = node.value;
      return;
    }
    
    this._findKthHelper(node.right, k, result);
  }
  
  getRange(min, max) {
    const result = [];
    this._getRangeHelper(this.root, min, max, result);
    return result;
  }
  
  _getRangeHelper(node, min, max, result) {
    if (!node) return;
    
    if (node.value > min) {
      this._getRangeHelper(node.left, min, max, result);
    }
    
    if (node.value >= min && node.value <= max) {
      result.push(node.value);
    }
    
    if (node.value < max) {
      this._getRangeHelper(node.right, min, max, result);
    }
  }
  
  validate() {
    return this._isValidBST(this.root, -Infinity, Infinity);
  }
  
  _isValidBST(node, min, max) {
    if (!node) return true;
    
    if (node.value <= min || node.value >= max) {
      return false;
    }
    
    return this._isValidBST(node.left, min, node.value) &&
           this._isValidBST(node.right, node.value, max);
  }
}

// Test
console.log('\n=== Advanced BST ===');

const advBst = new AdvancedBST();
[50, 30, 70, 20, 40, 60, 80, 10, 25, 35, 45].forEach(val => advBst.insert(val));

console.log('Height:', advBst.getHeight()); // 3
console.log('Node count:', advBst.countNodes()); // 11
console.log('Is balanced:', advBst.isBalanced()); // false (unbalanced)
console.log('Is valid BST:', advBst.validate()); // true

console.log('Closest to 33:', advBst.findClosest(33)); // 30 or 35
console.log('3rd smallest:', advBst.findKthSmallest(3)); // 25
console.log('Range [25, 50]:', advBst.getRange(25, 50));
// [25, 30, 35, 40, 45, 50]
```

### **Approach 5: Self-Balancing BST (AVL Tree Basics)**
```javascript
/**
 * Simple AVL tree with rotations
 * (Simplified version for demonstration)
 */
class AVLNode extends TreeNode {
  constructor(value) {
    super(value);
    this.height = 1;
  }
}

class AVLTree {
  constructor() {
    this.root = null;
  }
  
  getHeight(node) {
    return node ? node.height : 0;
  }
  
  getBalance(node) {
    return node ? this.getHeight(node.left) - this.getHeight(node.right) : 0;
  }
  
  updateHeight(node) {
    if (node) {
      node.height = 1 + Math.max(
        this.getHeight(node.left),
        this.getHeight(node.right)
      );
    }
  }
  
  rotateRight(y) {
    const x = y.left;
    const T2 = x.right;
    
    x.right = y;
    y.left = T2;
    
    this.updateHeight(y);
    this.updateHeight(x);
    
    return x;
  }
  
  rotateLeft(x) {
    const y = x.right;
    const T2 = y.left;
    
    y.left = x;
    x.right = T2;
    
    this.updateHeight(x);
    this.updateHeight(y);
    
    return y;
  }
  
  insert(value) {
    this.root = this._insertAVL(this.root, value);
    return this;
  }
  
  _insertAVL(node, value) {
    // Standard BST insert
    if (!node) {
      return new AVLNode(value);
    }
    
    if (value < node.value) {
      node.left = this._insertAVL(node.left, value);
    } else if (value > node.value) {
      node.right = this._insertAVL(node.right, value);
    } else {
      return node; // Duplicate
    }
    
    // Update height
    this.updateHeight(node);
    
    // Get balance factor
    const balance = this.getBalance(node);
    
    // Left Left Case
    if (balance > 1 && value < node.left.value) {
      return this.rotateRight(node);
    }
    
    // Right Right Case
    if (balance < -1 && value > node.right.value) {
      return this.rotateLeft(node);
    }
    
    // Left Right Case
    if (balance > 1 && value > node.left.value) {
      node.left = this.rotateLeft(node.left);
      return this.rotateRight(node);
    }
    
    // Right Left Case
    if (balance < -1 && value < node.right.value) {
      node.right = this.rotateRight(node.right);
      return this.rotateLeft(node);
    }
    
    return node;
  }
  
  search(value) {
    let current = this.root;
    
    while (current) {
      if (value === current.value) return true;
      current = value < current.value ? current.left : current.right;
    }
    
    return false;
  }
  
  inOrder() {
    const result = [];
    this._inOrderHelper(this.root, result);
    return result;
  }
  
  _inOrderHelper(node, result) {
    if (!node) return;
    
    this._inOrderHelper(node.left, result);
    result.push(node.value);
    this._inOrderHelper(node.right, result);
  }
}

// Test
console.log('\n=== AVL Tree (Self-Balancing) ===');

const avl = new AVLTree();

// Insert sequence that would create unbalanced BST
[10, 20, 30, 40, 50, 25].forEach(val => avl.insert(val));

console.log('In-order traversal:', avl.inOrder());
console.log('Root value (balanced):', avl.root.value); // Should be 30
console.log('Height:', avl.getHeight(avl.root)); // Much shorter than unbalanced
```

### **Real-World Use Cases**
```javascript
/**
 * Practical BST applications
 */

// 1. Range Query System
console.log('\n=== Range Query ===');

class RangeQueryDB {
  constructor() {
    this.bst = new AdvancedBST();
  }
  
  insert(key, value) {
    // In real system, store key-value pairs
    this.bst.insert(key);
  }
  
  rangeQuery(min, max) {
    return this.bst.getRange(min, max);
  }
}

const db = new RangeQueryDB();
[15, 10, 20, 8, 12, 16, 25].forEach(val => db.insert(val));

console.log('Range query [10, 20]:', db.rangeQuery(10, 20));

// 2. Priority Queue (using BST)
console.log('\n=== Priority Queue ===');

class PriorityQueueBST {
  constructor() {
    this.bst = new RecursiveBST();
  }
  
  enqueue(priority) {
    this.bst.insert(priority);
  }
  
  dequeue() {
    // Remove and return highest priority (max)
    const max = this.bst.findMax();
    if (max !== null) {
      this.bst.delete(max);
    }
    return max;
  }
  
  peek() {
    return this.bst.findMax();
  }
  
  isEmpty() {
    return this.bst.root === null;
  }
}

const pq = new PriorityQueueBST();
[5, 1, 9, 3, 7].forEach(p => pq.enqueue(p));

console.log('Dequeue (highest):', pq.dequeue()); // 9
console.log('Dequeue next:', pq.dequeue()); // 7
console.log('Peek:', pq.peek()); // 5

// 3. Sorted Set
console.log('\n=== Sorted Set ===');

class SortedSet {
  constructor() {
    this.bst = new TraversalBST();
  }
  
  add(value) {
    this.bst.insert(value);
  }
  
  has(value) {
    return this.bst.search(value);
  }
  
  delete(value) {
    this.bst.delete(value);
  }
  
  toArray() {
    return this.bst.toArray('inorder');
  }
  
  min() {
    return this.bst.findMin();
  }
  
  max() {
    return this.bst.findMax();
  }
}

const sortedSet = new SortedSet();
[5, 2, 8, 1, 9].forEach(v => sortedSet.add(v));

console.log('Sorted values:', sortedSet.toArray()); // [1, 2, 5, 8, 9]
console.log('Has 5:', sortedSet.has(5)); // true
console.log('Min:', sortedSet.min()); // 1
console.log('Max:', sortedSet.max()); // 9
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

const edgeBst = new AdvancedBST();

// Empty tree operations
console.log('Search in empty:', edgeBst.search(5)); // false
console.log('Min in empty:', edgeBst.findMin()); // null
console.log('Height of empty:', edgeBst.getHeight()); // -1

// Single node
edgeBst.insert(50);
console.log('Single node height:', edgeBst.getHeight()); // 0

// Duplicate insertions
edgeBst.insert(30);
edgeBst.insert(30); // Should be ignored
console.log('Node count (with duplicate):', edgeBst.countNodes()); // 2

// Delete root
edgeBst.insert(70);
edgeBst.delete(50);
console.log('After delete root:', edgeBst.toArray('inorder'));

// Skewed tree (worst case)
const skewedBst = new AdvancedBST();
[1, 2, 3, 4, 5].forEach(v => skewedBst.insert(v));
console.log('Skewed tree height:', skewedBst.getHeight()); // 4 (O(n))
console.log('Is skewed balanced:', skewedBst.isBalanced()); // false

// Delete non-existent
const testBst = new AdvancedBST();
[5, 3, 7].forEach(v => testBst.insert(v));
testBst.delete(10); // No error
console.log('After delete non-existent:', testBst.toArray('inorder'));
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Time Complexity:
┌──────────────┬─────────────┬─────────────┬─────────────┐
│ Operation    │ BST Average │ BST Worst   │ AVL Tree    │
├──────────────┼─────────────┼─────────────┼─────────────┤
│ Insert       │ O(log n)    │ O(n)        │ O(log n)    │
│ Search       │ O(log n)    │ O(n)        │ O(log n)    │
│ Delete       │ O(log n)    │ O(n)        │ O(log n)    │
│ Min/Max      │ O(log n)    │ O(n)        │ O(log n)    │
│ In-order     │ O(n)        │ O(n)        │ O(n)        │
│ Range Query  │ O(log n + k)│ O(n)        │ O(log n + k)│
└──────────────┴─────────────┴─────────────┴─────────────┘

Space Complexity: O(n) for all

Best Cases:
• Balanced BST: height = log(n), all operations O(log n)
• Skewed BST: height = n, degenerates to linked list O(n)

Advantages:
• Sorted order maintained
• Efficient search, insert, delete
• No rehashing like hash tables
• Range queries efficient

Disadvantages:
• Can become unbalanced (worst case O(n))
• More complex than arrays
• Extra memory for pointers

Solutions:
• AVL Tree: self-balancing, strict balance (height diff ≤ 1)
• Red-Black Tree: self-balancing, relaxed rules, faster insert
• B-Tree: for disk-based databases, multiple keys per node
`);
```

**Interview Tips:**
- BST: binary tree where left < node < right for all nodes
- Insert: traverse tree comparing values, add at appropriate leaf
- Search: traverse left if target < node, right if target > node
- Delete: three cases - leaf (remove), one child (replace), two children (find successor)
- Successor: smallest value in right subtree (leftmost node)
- Predecessor: largest value in left subtree (rightmost node)
- Time: O(h) where h is height, O(log n) average, O(n) worst (skewed)
- Space: O(n) for n nodes, O(h) call stack for recursion
- In-order traversal: yields sorted order (left, node, right)
- Pre-order: node, left, right (useful for copying tree)
- Post-order: left, right, node (useful for deletion)
- Level-order: breadth-first, use queue
- Height: longest path from root to leaf
- Balanced: height difference between subtrees ≤ 1 at every node
- Skewed: all nodes on one side, degenerates to linked list
- Applications: sorted sets, priority queues, range queries, databases
- Alternatives: AVL (strict balance), Red-Black (relaxed balance), B-Tree (databases)
- Edge cases: empty tree, single node, duplicates, delete root, skewed tree
- Optimization: keep height/size in nodes, iterative vs recursive, parent pointers
- Clarify: duplicates allowed? need balancing? additional operations? parent pointers?
- Follow-ups: serialize/deserialize tree, find LCA, convert to balanced BST, add iterator

</details>

98. Implement breadth-first search (BFS) on a graph

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic BFS with Adjacency List**
```javascript
/**
 * Breadth-First Search using queue
 * Time Complexity: O(V + E) where V is vertices, E is edges
 * Space Complexity: O(V) for queue and visited set
 */
class Graph {
  constructor() {
    this.adjacencyList = {};
  }
  
  addVertex(vertex) {
    if (!this.adjacencyList[vertex]) {
      this.adjacencyList[vertex] = [];
    }
  }
  
  addEdge(v1, v2) {
    this.adjacencyList[v1].push(v2);
    this.adjacencyList[v2].push(v1); // Undirected graph
  }
  
  bfs(start) {
    const visited = new Set();
    const queue = [start];
    const result = [];
    
    visited.add(start);
    
    while (queue.length > 0) {
      const vertex = queue.shift();
      result.push(vertex);
      
      for (const neighbor of this.adjacencyList[vertex]) {
        if (!visited.has(neighbor)) {
          visited.add(neighbor);
          queue.push(neighbor);
        }
      }
    }
    
    return result;
  }
}

// Test
console.log('=== Basic BFS ===');

const graph = new Graph();

// Build graph:
//     A
//    / \
//   B   C
//  / \   \
// D   E   F

['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph.addVertex(v));
graph.addEdge('A', 'B');
graph.addEdge('A', 'C');
graph.addEdge('B', 'D');
graph.addEdge('B', 'E');
graph.addEdge('C', 'F');

console.log('BFS from A:', graph.bfs('A'));
// ['A', 'B', 'C', 'D', 'E', 'F']

console.log('BFS from C:', graph.bfs('C'));
// ['C', 'A', 'F', 'B', 'D', 'E']
```

### **Approach 2: BFS with Level Tracking**
```javascript
/**
 * BFS that tracks levels/distances from start
 */
function bfsWithLevels(graph, start) {
  const visited = new Set();
  const queue = [[start, 0]]; // [vertex, level]
  const levels = {};
  
  visited.add(start);
  levels[start] = 0;
  
  while (queue.length > 0) {
    const [vertex, level] = queue.shift();
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        levels[neighbor] = level + 1;
        queue.push([neighbor, level + 1]);
      }
    }
  }
  
  return levels;
}

// Test
console.log('\n=== BFS with Levels ===');

const graph2 = new Graph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph2.addVertex(v));
graph2.addEdge('A', 'B');
graph2.addEdge('A', 'C');
graph2.addEdge('B', 'D');
graph2.addEdge('B', 'E');
graph2.addEdge('C', 'F');

console.log('Levels from A:', bfsWithLevels(graph2, 'A'));
// { A: 0, B: 1, C: 1, D: 2, E: 2, F: 2 }
```

### **Approach 3: BFS for Shortest Path**
```javascript
/**
 * BFS to find shortest path between two vertices
 */
function bfsShortestPath(graph, start, end) {
  const visited = new Set();
  const queue = [[start, [start]]]; // [vertex, path]
  
  visited.add(start);
  
  while (queue.length > 0) {
    const [vertex, path] = queue.shift();
    
    if (vertex === end) {
      return path;
    }
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        queue.push([neighbor, [...path, neighbor]]);
      }
    }
  }
  
  return null; // No path found
}

// Test
console.log('\n=== BFS Shortest Path ===');

const graph3 = new Graph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph3.addVertex(v));
graph3.addEdge('A', 'B');
graph3.addEdge('A', 'C');
graph3.addEdge('B', 'D');
graph3.addEdge('C', 'E');
graph3.addEdge('D', 'F');
graph3.addEdge('E', 'F');

console.log('Shortest path A to F:', bfsShortestPath(graph3, 'A', 'F'));
// ['A', 'B', 'D', 'F'] or ['A', 'C', 'E', 'F']
```

### **Approach 4: BFS for Disconnected Graphs**
```javascript
/**
 * BFS that handles disconnected graphs
 */
function bfsAllComponents(graph) {
  const visited = new Set();
  const components = [];
  
  function bfsComponent(start) {
    const queue = [start];
    const component = [];
    
    visited.add(start);
    
    while (queue.length > 0) {
      const vertex = queue.shift();
      component.push(vertex);
      
      for (const neighbor of graph.adjacencyList[vertex]) {
        if (!visited.has(neighbor)) {
          visited.add(neighbor);
          queue.push(neighbor);
        }
      }
    }
    
    return component;
  }
  
  // Visit all vertices
  for (const vertex in graph.adjacencyList) {
    if (!visited.has(vertex)) {
      components.push(bfsComponent(vertex));
    }
  }
  
  return components;
}

// Test
console.log('\n=== BFS All Components ===');

const disconnectedGraph = new Graph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => disconnectedGraph.addVertex(v));

// Component 1: A - B - C
disconnectedGraph.addEdge('A', 'B');
disconnectedGraph.addEdge('B', 'C');

// Component 2: D - E
disconnectedGraph.addEdge('D', 'E');

// Component 3: F (isolated)

console.log('All components:', bfsAllComponents(disconnectedGraph));
// [['A', 'B', 'C'], ['D', 'E'], ['F']]
```

### **Approach 5: BFS on Matrix (Grid)**
```javascript
/**
 * BFS on 2D grid (common in problems)
 */
function bfsGrid(grid, startRow, startCol) {
  const rows = grid.length;
  const cols = grid[0].length;
  const visited = Array.from({ length: rows }, () => Array(cols).fill(false));
  const queue = [[startRow, startCol]];
  const result = [];
  
  visited[startRow][startCol] = true;
  
  // Directions: up, right, down, left
  const directions = [[-1, 0], [0, 1], [1, 0], [0, -1]];
  
  while (queue.length > 0) {
    const [row, col] = queue.shift();
    result.push([row, col]);
    
    for (const [dr, dc] of directions) {
      const newRow = row + dr;
      const newCol = col + dc;
      
      if (
        newRow >= 0 && newRow < rows &&
        newCol >= 0 && newCol < cols &&
        !visited[newRow][newCol] &&
        grid[newRow][newCol] !== 0 // 0 represents obstacle
      ) {
        visited[newRow][newCol] = true;
        queue.push([newRow, newCol]);
      }
    }
  }
  
  return result;
}

// Test
console.log('\n=== BFS on Grid ===');

const grid = [
  [1, 1, 0, 1],
  [1, 1, 1, 0],
  [0, 1, 1, 1],
  [1, 0, 1, 1]
];

console.log('BFS from (0,0):');
const gridResult = bfsGrid(grid, 0, 0);
gridResult.forEach(([r, c]) => console.log(`  (${r},${c})`));
```

### **Bonus: Multi-Source BFS**
```javascript
/**
 * BFS from multiple starting points simultaneously
 * Useful for: rotten oranges, walls and gates problems
 */
function multiSourceBFS(graph, sources) {
  const visited = new Set();
  const queue = [];
  const distances = {};
  
  // Initialize queue with all sources
  for (const source of sources) {
    queue.push([source, 0]);
    visited.add(source);
    distances[source] = 0;
  }
  
  while (queue.length > 0) {
    const [vertex, distance] = queue.shift();
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        distances[neighbor] = distance + 1;
        queue.push([neighbor, distance + 1]);
      }
    }
  }
  
  return distances;
}

// Test
console.log('\n=== Multi-Source BFS ===');

const graph4 = new Graph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph4.addVertex(v));
graph4.addEdge('A', 'B');
graph4.addEdge('B', 'C');
graph4.addEdge('C', 'D');
graph4.addEdge('D', 'E');
graph4.addEdge('E', 'F');

// Start from both ends
console.log('Distances from [A, F]:', multiSourceBFS(graph4, ['A', 'F']));
// Nearest source to each vertex
```

### **Bonus: Bidirectional BFS**
```javascript
/**
 * Bidirectional BFS for faster pathfinding
 * Search from both start and end simultaneously
 */
function bidirectionalBFS(graph, start, end) {
  if (start === end) return [start];
  
  const visitedStart = new Map([[start, [start]]]);
  const visitedEnd = new Map([[end, [end]]]);
  const queueStart = [start];
  const queueEnd = [end];
  
  while (queueStart.length > 0 || queueEnd.length > 0) {
    // Expand from start
    if (queueStart.length > 0) {
      const vertex = queueStart.shift();
      
      for (const neighbor of graph.adjacencyList[vertex]) {
        if (visitedEnd.has(neighbor)) {
          // Paths met!
          return [
            ...visitedStart.get(vertex),
            ...visitedEnd.get(neighbor).reverse()
          ];
        }
        
        if (!visitedStart.has(neighbor)) {
          visitedStart.set(neighbor, [...visitedStart.get(vertex), neighbor]);
          queueStart.push(neighbor);
        }
      }
    }
    
    // Expand from end
    if (queueEnd.length > 0) {
      const vertex = queueEnd.shift();
      
      for (const neighbor of graph.adjacencyList[vertex]) {
        if (visitedStart.has(neighbor)) {
          // Paths met!
          return [
            ...visitedStart.get(neighbor),
            ...visitedEnd.get(vertex).reverse()
          ];
        }
        
        if (!visitedEnd.has(neighbor)) {
          visitedEnd.set(neighbor, [...visitedEnd.get(vertex), neighbor]);
          queueEnd.push(neighbor);
        }
      }
    }
  }
  
  return null; // No path
}

// Test
console.log('\n=== Bidirectional BFS ===');

const graph5 = new Graph();
for (let i = 0; i < 10; i++) {
  graph5.addVertex(i);
}

// Linear chain: 0-1-2-3-4-5-6-7-8-9
for (let i = 0; i < 9; i++) {
  graph5.addEdge(i, i + 1);
}

console.log('Bidirectional path 0 to 9:', bidirectionalBFS(graph5, 0, 9));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical BFS applications
 */

// 1. Social Network - Degrees of Separation
console.log('\n=== Social Network ===');

class SocialNetwork {
  constructor() {
    this.graph = new Graph();
  }
  
  addUser(user) {
    this.graph.addVertex(user);
  }
  
  addFriendship(user1, user2) {
    this.graph.addEdge(user1, user2);
  }
  
  degreesOfSeparation(user1, user2) {
    const levels = bfsWithLevels(this.graph, user1);
    return levels[user2] !== undefined ? levels[user2] : -1;
  }
  
  findMutualFriends(user1, user2) {
    const friends1 = new Set(this.graph.adjacencyList[user1]);
    const friends2 = new Set(this.graph.adjacencyList[user2]);
    
    return [...friends1].filter(friend => friends2.has(friend));
  }
}

const social = new SocialNetwork();
['Alice', 'Bob', 'Charlie', 'David', 'Eve'].forEach(u => social.addUser(u));
social.addFriendship('Alice', 'Bob');
social.addFriendship('Bob', 'Charlie');
social.addFriendship('Charlie', 'David');
social.addFriendship('Alice', 'Eve');
social.addFriendship('Eve', 'David');

console.log('Degrees Alice to David:', social.degreesOfSeparation('Alice', 'David'));
console.log('Mutual friends Alice & David:', social.findMutualFriends('Alice', 'David'));

// 2. Web Crawler
console.log('\n=== Web Crawler ===');

class WebCrawler {
  constructor(maxDepth = 3) {
    this.maxDepth = maxDepth;
    this.visited = new Set();
  }
  
  crawl(startUrl, links) {
    const queue = [[startUrl, 0]]; // [url, depth]
    const crawled = [];
    
    this.visited.add(startUrl);
    
    while (queue.length > 0) {
      const [url, depth] = queue.shift();
      
      console.log(`  Crawling: ${url} (depth ${depth})`);
      crawled.push(url);
      
      if (depth < this.maxDepth && links[url]) {
        for (const link of links[url]) {
          if (!this.visited.has(link)) {
            this.visited.add(link);
            queue.push([link, depth + 1]);
          }
        }
      }
    }
    
    return crawled;
  }
}

const links = {
  'home': ['about', 'contact', 'blog'],
  'about': ['team', 'careers'],
  'blog': ['post1', 'post2'],
  'team': ['member1', 'member2']
};

const crawler = new WebCrawler(2);
console.log('Crawled pages:', crawler.crawl('home', links));

// 3. Game - Find Shortest Path
console.log('\n=== Game Pathfinding ===');

function findShortestPath(maze, start, end) {
  const [startRow, startCol] = start;
  const [endRow, endCol] = end;
  
  const rows = maze.length;
  const cols = maze[0].length;
  const visited = Array.from({ length: rows }, () => Array(cols).fill(false));
  const queue = [[startRow, startCol, 0, [[startRow, startCol]]]]; // row, col, dist, path
  
  visited[startRow][startCol] = true;
  
  const directions = [[-1, 0], [0, 1], [1, 0], [0, -1]];
  
  while (queue.length > 0) {
    const [row, col, dist, path] = queue.shift();
    
    if (row === endRow && col === endCol) {
      return { distance: dist, path };
    }
    
    for (const [dr, dc] of directions) {
      const newRow = row + dr;
      const newCol = col + dc;
      
      if (
        newRow >= 0 && newRow < rows &&
        newCol >= 0 && newCol < cols &&
        !visited[newRow][newCol] &&
        maze[newRow][newCol] === 0 // 0 = passable
      ) {
        visited[newRow][newCol] = true;
        queue.push([newRow, newCol, dist + 1, [...path, [newRow, newCol]]]);
      }
    }
  }
  
  return null; // No path
}

const maze = [
  [0, 0, 1, 0, 0],
  [0, 0, 1, 0, 0],
  [0, 0, 0, 0, 1],
  [1, 1, 0, 1, 0],
  [0, 0, 0, 0, 0]
];

const pathResult = findShortestPath(maze, [0, 0], [4, 4]);
console.log('Shortest path:', pathResult);

// 4. Network Broadcasting
console.log('\n=== Network Broadcasting ===');

function broadcastTime(network, source) {
  const visited = new Set();
  const queue = [[source, 0]];
  const receiveTimes = {};
  
  visited.add(source);
  receiveTimes[source] = 0;
  
  while (queue.length > 0) {
    const [node, time] = queue.shift();
    
    for (const [neighbor, delay] of network[node] || []) {
      const arrivalTime = time + delay;
      
      if (!visited.has(neighbor)) {
        visited.add(neighbor);
        receiveTimes[neighbor] = arrivalTime;
        queue.push([neighbor, arrivalTime]);
      }
    }
  }
  
  return receiveTimes;
}

const network = {
  'A': [['B', 1], ['C', 2]],
  'B': [['D', 3]],
  'C': [['D', 1], ['E', 2]],
  'D': [['E', 1]]
};

console.log('Broadcast from A:', broadcastTime(network, 'A'));

// 5. Flood Fill (Paint Bucket)
console.log('\n=== Flood Fill ===');

function floodFill(image, sr, sc, newColor) {
  const rows = image.length;
  const cols = image[0].length;
  const originalColor = image[sr][sc];
  
  if (originalColor === newColor) return image;
  
  const queue = [[sr, sc]];
  image[sr][sc] = newColor;
  
  const directions = [[-1, 0], [0, 1], [1, 0], [0, -1]];
  
  while (queue.length > 0) {
    const [row, col] = queue.shift();
    
    for (const [dr, dc] of directions) {
      const newRow = row + dr;
      const newCol = col + dc;
      
      if (
        newRow >= 0 && newRow < rows &&
        newCol >= 0 && newCol < cols &&
        image[newRow][newCol] === originalColor
      ) {
        image[newRow][newCol] = newColor;
        queue.push([newRow, newCol]);
      }
    }
  }
  
  return image;
}

const image = [
  [1, 1, 1],
  [1, 1, 0],
  [1, 0, 1]
];

console.log('Original image:', image);
const filled = floodFill(JSON.parse(JSON.stringify(image)), 1, 1, 2);
console.log('After flood fill (1,1) with 2:', filled);
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Single node
const singleGraph = new Graph();
singleGraph.addVertex('A');
console.log('BFS single node:', singleGraph.bfs('A')); // ['A']

// No edges
const isolatedGraph = new Graph();
['A', 'B', 'C'].forEach(v => isolatedGraph.addVertex(v));
console.log('BFS isolated nodes:', isolatedGraph.bfs('A')); // ['A']

// Cyclic graph
const cyclicGraph = new Graph();
['A', 'B', 'C', 'D'].forEach(v => cyclicGraph.addVertex(v));
cyclicGraph.addEdge('A', 'B');
cyclicGraph.addEdge('B', 'C');
cyclicGraph.addEdge('C', 'D');
cyclicGraph.addEdge('D', 'A'); // Creates cycle
console.log('BFS cyclic graph:', cyclicGraph.bfs('A'));

// Self-loop
const selfLoopGraph = new Graph();
selfLoopGraph.addVertex('A');
selfLoopGraph.adjacencyList['A'].push('A'); // Self-loop
console.log('BFS with self-loop:', selfLoopGraph.bfs('A'));

// Non-existent start
const testGraph = new Graph();
testGraph.addVertex('A');
console.log('BFS non-existent start:', testGraph.bfs('Z')); // []
```

### **Performance Analysis**
```javascript
console.log('\n=== Performance ===');

console.log(`
Time Complexity Analysis:
• BFS: O(V + E) where V = vertices, E = edges
  - Each vertex visited once: O(V)
  - Each edge explored once: O(E)
  - Total: O(V + E)

• Space Complexity: O(V)
  - Queue size: O(V) worst case
  - Visited set: O(V)
  - Path storage: O(V) per path

Grid BFS:
• Time: O(rows × cols)
• Space: O(rows × cols)

Optimization Techniques:
1. Use deque instead of array.shift() for O(1) queue operations
2. Bidirectional BFS: reduces time from O(b^d) to O(b^(d/2))
3. Early termination when target found
4. Use visited set, not array search (O(1) vs O(n))
5. Store only indices for grid, not full coordinates

BFS vs DFS:
┌─────────────────┬─────────────┬─────────────┐
│ Aspect          │ BFS         │ DFS         │
├─────────────────┼─────────────┼─────────────┤
│ Data Structure  │ Queue       │ Stack/Recursion│
│ Shortest Path   │ Yes         │ No          │
│ Memory          │ O(V)        │ O(h)        │
│ Complete        │ Yes         │ No (cycles) │
│ Optimal         │ Yes (unweighted)│ No       │
└─────────────────┴─────────────┴─────────────┘

When to use BFS:
• Finding shortest path (unweighted)
• Level-order traversal
• Social network analysis
• Web crawling
• Broadcasting/flooding
• Minimum steps problems
`);
```

**Interview Tips:**
- BFS: explore level by level using queue (FIFO)
- Algorithm: add start to queue, mark visited, dequeue, process, enqueue unvisited neighbors
- Data structures: queue for traversal, Set for visited tracking
- Time: O(V + E) visit each vertex once, explore each edge once
- Space: O(V) for queue and visited set
- Shortest path: BFS finds shortest path in unweighted graph
- Level tracking: store depth/distance with each vertex in queue
- Grid BFS: treat as implicit graph, use 4 or 8 directions
- Applications: shortest path, social networks, web crawling, flood fill, game pathfinding
- Multi-source: start from multiple nodes simultaneously
- Bidirectional: search from both ends, meet in middle, faster for single source-target
- Queue operations: use deque or circular buffer for O(1) enqueue/dequeue (array.shift() is O(n))
- Visited timing: mark as visited when enqueuing, not when dequeuing (prevents duplicates in queue)
- Disconnected graphs: run BFS from each unvisited vertex to find all components
- Edge cases: empty graph, single node, cycles, self-loops, disconnected components
- vs DFS: BFS uses more memory but finds shortest paths, DFS uses less memory but explores deep first
- Optimization: early termination, bidirectional search, efficient queue implementation
- Clarify: directed or undirected? weighted? need path or just reachability? single target or all vertices?
- Follow-ups: implement with adjacency matrix, add edge weights, find all paths, detect bipartite

</details>

99. Implement depth-first search (DFS) on a graph

<details>
<summary><b>Solution</b></summary>

### **Approach 1: DFS with Recursion**
```javascript
/**
 * Depth-First Search using recursion
 * Time Complexity: O(V + E) where V is vertices, E is edges
 * Space Complexity: O(V) for recursion stack and visited set
 */
class Graph {
  constructor() {
    this.adjacencyList = {};
  }
  
  addVertex(vertex) {
    if (!this.adjacencyList[vertex]) {
      this.adjacencyList[vertex] = [];
    }
  }
  
  addEdge(v1, v2) {
    this.adjacencyList[v1].push(v2);
    this.adjacencyList[v2].push(v1); // Undirected
  }
  
  dfsRecursive(start) {
    const visited = new Set();
    const result = [];
    
    const dfs = (vertex) => {
      visited.add(vertex);
      result.push(vertex);
      
      for (const neighbor of this.adjacencyList[vertex]) {
        if (!visited.has(neighbor)) {
          dfs(neighbor);
        }
      }
    };
    
    dfs(start);
    return result;
  }
}

// Test
console.log('=== Recursive DFS ===');

const graph = new Graph();

// Build graph:
//     A
//    / \
//   B   C
//  / \   \
// D   E   F

['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph.addVertex(v));
graph.addEdge('A', 'B');
graph.addEdge('A', 'C');
graph.addEdge('B', 'D');
graph.addEdge('B', 'E');
graph.addEdge('C', 'F');

console.log('DFS from A:', graph.dfsRecursive('A'));
// ['A', 'B', 'D', 'E', 'C', 'F'] (depth-first order)
```

### **Approach 2: DFS with Iterative Stack**
```javascript
/**
 * Iterative DFS using explicit stack
 */
class GraphIterative extends Graph {
  dfsIterative(start) {
    const visited = new Set();
    const stack = [start];
    const result = [];
    
    visited.add(start);
    
    while (stack.length > 0) {
      const vertex = stack.pop();
      result.push(vertex);
      
      // Add neighbors to stack (reverse order for same result as recursive)
      const neighbors = [...this.adjacencyList[vertex]].reverse();
      
      for (const neighbor of neighbors) {
        if (!visited.has(neighbor)) {
          visited.add(neighbor);
          stack.push(neighbor);
        }
      }
    }
    
    return result;
  }
}

// Test
console.log('\n=== Iterative DFS ===');

const graph2 = new GraphIterative();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph2.addVertex(v));
graph2.addEdge('A', 'B');
graph2.addEdge('A', 'C');
graph2.addEdge('B', 'D');
graph2.addEdge('B', 'E');
graph2.addEdge('C', 'F');

console.log('DFS iterative from A:', graph2.dfsIterative('A'));
```

### **Approach 3: DFS with Path Finding**
```javascript
/**
 * DFS to find a path between two vertices
 */
function dfsPath(graph, start, end, visited = new Set(), path = []) {
  visited.add(start);
  path.push(start);
  
  if (start === end) {
    return [...path]; // Found path
  }
  
  for (const neighbor of graph.adjacencyList[start]) {
    if (!visited.has(neighbor)) {
      const result = dfsPath(graph, neighbor, end, visited, path);
      if (result) {
        return result;
      }
    }
  }
  
  path.pop(); // Backtrack
  return null;
}

// Find all paths
function dfsAllPaths(graph, start, end, visited = new Set(), path = [], allPaths = []) {
  visited.add(start);
  path.push(start);
  
  if (start === end) {
    allPaths.push([...path]);
  } else {
    for (const neighbor of graph.adjacencyList[start]) {
      if (!visited.has(neighbor)) {
        dfsAllPaths(graph, neighbor, end, visited, path, allPaths);
      }
    }
  }
  
  path.pop();
  visited.delete(start); // Allow revisiting for other paths
  
  return allPaths;
}

// Test
console.log('\n=== DFS Path Finding ===');

const graph3 = new Graph();
['A', 'B', 'C', 'D', 'E'].forEach(v => graph3.addVertex(v));
graph3.addEdge('A', 'B');
graph3.addEdge('A', 'C');
graph3.addEdge('B', 'D');
graph3.addEdge('C', 'D');
graph3.addEdge('D', 'E');

console.log('One path A to E:', dfsPath(graph3, 'A', 'E'));
console.log('All paths A to E:', dfsAllPaths(graph3, 'A', 'E'));
```

### **Approach 4: DFS for Cycle Detection**
```javascript
/**
 * DFS to detect cycles in undirected graph
 */
function hasCycleUndirected(graph) {
  const visited = new Set();
  
  function dfs(vertex, parent) {
    visited.add(vertex);
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (!visited.has(neighbor)) {
        if (dfs(neighbor, vertex)) {
          return true;
        }
      } else if (neighbor !== parent) {
        // Visited and not parent = cycle
        return true;
      }
    }
    
    return false;
  }
  
  // Check all components
  for (const vertex in graph.adjacencyList) {
    if (!visited.has(vertex)) {
      if (dfs(vertex, null)) {
        return true;
      }
    }
  }
  
  return false;
}

// DFS for cycle detection in directed graph
function hasCycleDirected(graph) {
  const visited = new Set();
  const recStack = new Set(); // Recursion stack
  
  function dfs(vertex) {
    visited.add(vertex);
    recStack.add(vertex);
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (!visited.has(neighbor)) {
        if (dfs(neighbor)) {
          return true;
        }
      } else if (recStack.has(neighbor)) {
        // In current path = cycle
        return true;
      }
    }
    
    recStack.delete(vertex);
    return false;
  }
  
  for (const vertex in graph.adjacencyList) {
    if (!visited.has(vertex)) {
      if (dfs(vertex)) {
        return true;
      }
    }
  }
  
  return false;
}

// Test
console.log('\n=== Cycle Detection ===');

const cyclicGraph = new Graph();
['A', 'B', 'C', 'D'].forEach(v => cyclicGraph.addVertex(v));
cyclicGraph.addEdge('A', 'B');
cyclicGraph.addEdge('B', 'C');
cyclicGraph.addEdge('C', 'D');
cyclicGraph.addEdge('D', 'A'); // Creates cycle

console.log('Has cycle (undirected):', hasCycleUndirected(cyclicGraph)); // true

const acyclicGraph = new Graph();
['A', 'B', 'C', 'D'].forEach(v => acyclicGraph.addVertex(v));
acyclicGraph.addEdge('A', 'B');
acyclicGraph.addEdge('B', 'C');
acyclicGraph.addEdge('C', 'D');

console.log('Has cycle (acyclic):', hasCycleUndirected(acyclicGraph)); // false
```

### **Approach 5: DFS with Timestamps (Discovery/Finish)**
```javascript
/**
 * DFS with discovery and finish timestamps
 * Useful for topological sorting and finding SCCs
 */
function dfsWithTimestamps(graph) {
  const visited = new Set();
  const discovery = {};
  const finish = {};
  let time = 0;
  
  function dfs(vertex) {
    visited.add(vertex);
    discovery[vertex] = ++time;
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (!visited.has(neighbor)) {
        dfs(neighbor);
      }
    }
    
    finish[vertex] = ++time;
  }
  
  for (const vertex in graph.adjacencyList) {
    if (!visited.has(vertex)) {
      dfs(vertex);
    }
  }
  
  return { discovery, finish };
}

// Test
console.log('\n=== DFS with Timestamps ===');

const graph4 = new Graph();
['A', 'B', 'C', 'D'].forEach(v => graph4.addVertex(v));
graph4.addEdge('A', 'B');
graph4.addEdge('B', 'C');
graph4.addEdge('A', 'D');

const timestamps = dfsWithTimestamps(graph4);
console.log('Discovery times:', timestamps.discovery);
console.log('Finish times:', timestamps.finish);
```

### **Bonus: DFS on Grid/Matrix**
```javascript
/**
 * DFS on 2D grid
 */
function dfsGrid(grid, row, col, visited = null) {
  const rows = grid.length;
  const cols = grid[0].length;
  
  if (!visited) {
    visited = Array.from({ length: rows }, () => Array(cols).fill(false));
  }
  
  const result = [];
  
  function dfs(r, c) {
    if (
      r < 0 || r >= rows ||
      c < 0 || c >= cols ||
      visited[r][c] ||
      grid[r][c] === 0 // 0 = obstacle
    ) {
      return;
    }
    
    visited[r][c] = true;
    result.push([r, c]);
    
    // Visit all 4 directions
    dfs(r - 1, c); // Up
    dfs(r, c + 1); // Right
    dfs(r + 1, c); // Down
    dfs(r, c - 1); // Left
  }
  
  dfs(row, col);
  return result;
}

// Count islands problem
function countIslands(grid) {
  const rows = grid.length;
  const cols = grid[0].length;
  const visited = Array.from({ length: rows }, () => Array(cols).fill(false));
  let count = 0;
  
  function dfs(r, c) {
    if (
      r < 0 || r >= rows ||
      c < 0 || c >= cols ||
      visited[r][c] ||
      grid[r][c] === 0
    ) {
      return;
    }
    
    visited[r][c] = true;
    
    dfs(r - 1, c);
    dfs(r, c + 1);
    dfs(r + 1, c);
    dfs(r, c - 1);
  }
  
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (grid[r][c] === 1 && !visited[r][c]) {
        dfs(r, c);
        count++;
      }
    }
  }
  
  return count;
}

// Test
console.log('\n=== DFS on Grid ===');

const grid = [
  [1, 1, 0, 0],
  [1, 0, 0, 1],
  [0, 0, 1, 1],
  [0, 0, 0, 1]
];

console.log('DFS from (0,0):', dfsGrid(grid, 0, 0));

const islandGrid = [
  [1, 1, 0, 0, 0],
  [1, 1, 0, 0, 1],
  [0, 0, 1, 0, 0],
  [0, 0, 0, 1, 1]
];

console.log('Number of islands:', countIslands(islandGrid)); // 4
```

### **Real-World Use Cases**
```javascript
/**
 * Practical DFS applications
 */

// 1. Maze Solver
console.log('\n=== Maze Solver ===');

function solveMaze(maze, start, end) {
  const [startRow, startCol] = start;
  const [endRow, endCol] = end;
  const rows = maze.length;
  const cols = maze[0].length;
  const visited = Array.from({ length: rows }, () => Array(cols).fill(false));
  
  function dfs(r, c, path) {
    if (r < 0 || r >= rows || c < 0 || c >= cols || visited[r][c] || maze[r][c] === 1) {
      return null;
    }
    
    visited[r][c] = true;
    path.push([r, c]);
    
    if (r === endRow && c === endCol) {
      return path;
    }
    
    // Try all directions
    const result = 
      dfs(r - 1, c, path) ||
      dfs(r, c + 1, path) ||
      dfs(r + 1, c, path) ||
      dfs(r, c - 1, path);
    
    if (result) return result;
    
    path.pop(); // Backtrack
    return null;
  }
  
  return dfs(startRow, startCol, []);
}

const maze = [
  [0, 0, 1, 0, 0],
  [0, 0, 1, 0, 0],
  [0, 0, 0, 0, 1],
  [1, 1, 0, 1, 0],
  [0, 0, 0, 0, 0]
];

console.log('Maze solution:', solveMaze(maze, [0, 0], [4, 4]));

// 2. File System Traversal
console.log('\n=== File System ===');

class FileSystem {
  constructor() {
    this.structure = {
      name: 'root',
      type: 'folder',
      children: []
    };
  }
  
  dfsTraversal(node = this.structure, depth = 0) {
    const indent = '  '.repeat(depth);
    console.log(`${indent}${node.name} (${node.type})`);
    
    if (node.children) {
      for (const child of node.children) {
        this.dfsTraversal(child, depth + 1);
      }
    }
  }
  
  findFile(name, node = this.structure) {
    if (node.name === name) {
      return node;
    }
    
    if (node.children) {
      for (const child of node.children) {
        const result = this.findFile(name, child);
        if (result) return result;
      }
    }
    
    return null;
  }
}

const fs = new FileSystem();
fs.structure.children = [
  {
    name: 'documents',
    type: 'folder',
    children: [
      { name: 'file1.txt', type: 'file' },
      { name: 'file2.txt', type: 'file' }
    ]
  },
  {
    name: 'photos',
    type: 'folder',
    children: [
      { name: 'pic1.jpg', type: 'file' }
    ]
  }
];

console.log('File system structure:');
fs.dfsTraversal();

console.log('\nFind file "pic1.jpg":', fs.findFile('pic1.jpg'));

// 3. Dependency Resolution
console.log('\n=== Dependency Resolution ===');

class DependencyGraph {
  constructor() {
    this.graph = {};
  }
  
  addPackage(pkg, dependencies = []) {
    this.graph[pkg] = dependencies;
  }
  
  getInstallOrder(pkg, visited = new Set(), stack = []) {
    if (visited.has(pkg)) {
      return;
    }
    
    visited.add(pkg);
    
    for (const dep of this.graph[pkg] || []) {
      this.getInstallOrder(dep, visited, stack);
    }
    
    stack.push(pkg);
    return stack;
  }
  
  hasCyclicDependency(pkg, visited = new Set(), recStack = new Set()) {
    visited.add(pkg);
    recStack.add(pkg);
    
    for (const dep of this.graph[pkg] || []) {
      if (!visited.has(dep)) {
        if (this.hasCyclicDependency(dep, visited, recStack)) {
          return true;
        }
      } else if (recStack.has(dep)) {
        return true; // Cycle detected
      }
    }
    
    recStack.delete(pkg);
    return false;
  }
}

const deps = new DependencyGraph();
deps.addPackage('app', ['express', 'mongoose']);
deps.addPackage('express', ['body-parser']);
deps.addPackage('mongoose', ['mongodb']);
deps.addPackage('body-parser', []);
deps.addPackage('mongodb', []);

console.log('Install order for app:', deps.getInstallOrder('app'));
console.log('Has cyclic dependency:', deps.hasCyclicDependency('app'));

// 4. Connected Components
console.log('\n=== Connected Components ===');

function findConnectedComponents(graph) {
  const visited = new Set();
  const components = [];
  
  function dfs(vertex, component) {
    visited.add(vertex);
    component.push(vertex);
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (!visited.has(neighbor)) {
        dfs(neighbor, component);
      }
    }
  }
  
  for (const vertex in graph.adjacencyList) {
    if (!visited.has(vertex)) {
      const component = [];
      dfs(vertex, component);
      components.push(component);
    }
  }
  
  return components;
}

const disconnected = new Graph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => disconnected.addVertex(v));
disconnected.addEdge('A', 'B');
disconnected.addEdge('B', 'C');
disconnected.addEdge('D', 'E');

console.log('Connected components:', findConnectedComponents(disconnected));

// 5. Sudoku Solver
console.log('\n=== Sudoku Solver ===');

function solveSudoku(board) {
  function isValid(board, row, col, num) {
    // Check row
    for (let c = 0; c < 9; c++) {
      if (board[row][c] === num) return false;
    }
    
    // Check column
    for (let r = 0; r < 9; r++) {
      if (board[r][col] === num) return false;
    }
    
    // Check 3x3 box
    const boxRow = Math.floor(row / 3) * 3;
    const boxCol = Math.floor(col / 3) * 3;
    for (let r = boxRow; r < boxRow + 3; r++) {
      for (let c = boxCol; c < boxCol + 3; c++) {
        if (board[r][c] === num) return false;
      }
    }
    
    return true;
  }
  
  function solve() {
    for (let row = 0; row < 9; row++) {
      for (let col = 0; col < 9; col++) {
        if (board[row][col] === 0) {
          for (let num = 1; num <= 9; num++) {
            if (isValid(board, row, col, num)) {
              board[row][col] = num;
              
              if (solve()) {
                return true;
              }
              
              board[row][col] = 0; // Backtrack
            }
          }
          return false;
        }
      }
    }
    return true;
  }
  
  solve();
  return board;
}

const sudoku = [
  [5, 3, 0, 0, 7, 0, 0, 0, 0],
  [6, 0, 0, 1, 9, 5, 0, 0, 0],
  [0, 9, 8, 0, 0, 0, 0, 6, 0],
  [8, 0, 0, 0, 6, 0, 0, 0, 3],
  [4, 0, 0, 8, 0, 3, 0, 0, 1],
  [7, 0, 0, 0, 2, 0, 0, 0, 6],
  [0, 6, 0, 0, 0, 0, 2, 8, 0],
  [0, 0, 0, 4, 1, 9, 0, 0, 5],
  [0, 0, 0, 0, 8, 0, 0, 7, 9]
];

console.log('Sudoku solved:', solveSudoku(sudoku)[0].join(' '));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty graph
const emptyGraph = new Graph();
console.log('DFS empty graph:', emptyGraph.dfsRecursive('A')); // []

// Single vertex
const singleGraph = new Graph();
singleGraph.addVertex('A');
console.log('DFS single vertex:', singleGraph.dfsRecursive('A')); // ['A']

// Self-loop
const selfLoop = new Graph();
selfLoop.addVertex('A');
selfLoop.adjacencyList['A'].push('A');
console.log('DFS self-loop:', selfLoop.dfsRecursive('A')); // ['A']

// Dense graph
const denseGraph = new Graph();
for (let i = 0; i < 5; i++) {
  denseGraph.addVertex(i);
  for (let j = 0; j < i; j++) {
    denseGraph.addEdge(i, j);
  }
}
console.log('DFS dense graph:', denseGraph.dfsRecursive(0));

// Stack overflow protection (very deep recursion)
const deepGraph = new Graph();
for (let i = 0; i < 100; i++) {
  deepGraph.addVertex(i);
  if (i > 0) {
    deepGraph.addEdge(i - 1, i);
  }
}
console.log('DFS deep graph (first 5):', deepGraph.dfsRecursive(0).slice(0, 5));
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Time & Space Complexity:
┌──────────────────┬─────────────┬─────────────┐
│ Aspect           │ DFS         │ BFS         │
├──────────────────┼─────────────┼─────────────┤
│ Time             │ O(V + E)    │ O(V + E)    │
│ Space (recursive)│ O(V)        │ O(V)        │
│ Space (iterative)│ O(V)        │ O(V)        │
│ Data Structure   │ Stack/Call  │ Queue       │
│ Shortest Path    │ No          │ Yes         │
│ Memory (worst)   │ O(h) height │ O(w) width  │
└──────────────────┴─────────────┴─────────────┘

When to use DFS:
• Pathfinding (any path, not shortest)
• Cycle detection
• Topological sorting
• Strongly connected components
• Maze solving with backtracking
• Exhaustive search (all solutions)
• Tree traversals (pre/in/post-order)
• Dependency resolution

DFS Advantages:
• Less memory for narrow/tall graphs
• Natural for recursion
• Good for exhaustive search
• Backtracking friendly

DFS Disadvantages:
• Doesn't find shortest path
• Can get stuck in infinite paths (needs cycle detection)
• Stack overflow risk for deep recursion

Optimization:
• Use iterative version for very deep graphs
• Tail recursion optimization
• Early termination when target found
• Path compression for repeated searches
`);
```

**Interview Tips:**
- DFS: explore as deep as possible before backtracking
- Implementation: recursion (implicit stack) or explicit stack (iterative)
- Recursive: simpler code, natural backtracking, risk of stack overflow
- Iterative: explicit stack, more control, no recursion limit
- Time: O(V + E) visit each vertex once, explore each edge once
- Space: O(V) for visited set, O(h) for recursion stack where h is height
- Applications: pathfinding, cycle detection, topological sort, maze solving, dependency resolution
- Backtracking: DFS naturally supports undo operations (pop from path)
- Pre-order: process before children (useful for copying)
- Post-order: process after children (useful for deletion, topological sort)
- Cycle detection: track recursion stack (directed) or parent (undirected)
- All paths: remove from visited on backtrack to allow revisiting
- Grid DFS: check bounds, visited, and obstacles; use recursion for 4/8 directions
- vs BFS: DFS uses less memory for tall graphs, doesn't guarantee shortest path
- Stack overflow: switch to iterative for very deep graphs (>10k depth)
- Edge cases: empty graph, single node, cycles, self-loops, disconnected components
- Mark visited: when entering function, not when adding to stack
- Optimization: iterative for depth, memoization for repeated subproblems
- Clarify: directed or undirected? need all paths or just one? detect cycles? max depth limit?
- Follow-ups: find all paths, detect cycle and return it, add path cost, implement iteratively

</details>

100. Find the shortest path in a graph (Dijkstra's algorithm)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Dijkstra with Array**
```javascript
/**
 * Dijkstra's Algorithm for shortest path in weighted graph
 * Time Complexity: O(V²) with array implementation
 * Space Complexity: O(V)
 */
class WeightedGraph {
  constructor() {
    this.adjacencyList = {};
  }
  
  addVertex(vertex) {
    if (!this.adjacencyList[vertex]) {
      this.adjacencyList[vertex] = [];
    }
  }
  
  addEdge(v1, v2, weight) {
    this.adjacencyList[v1].push({ node: v2, weight });
    this.adjacencyList[v2].push({ node: v1, weight }); // Undirected
  }
  
  dijkstra(start, end) {
    const distances = {};
    const previous = {};
    const unvisited = new Set();
    
    // Initialize
    for (const vertex in this.adjacencyList) {
      distances[vertex] = vertex === start ? 0 : Infinity;
      previous[vertex] = null;
      unvisited.add(vertex);
    }
    
    while (unvisited.size > 0) {
      // Find vertex with minimum distance
      let minVertex = null;
      let minDistance = Infinity;
      
      for (const vertex of unvisited) {
        if (distances[vertex] < minDistance) {
          minDistance = distances[vertex];
          minVertex = vertex;
        }
      }
      
      if (minVertex === null || minDistance === Infinity) {
        break; // No path to remaining vertices
      }
      
      unvisited.delete(minVertex);
      
      // Early termination if we reached the end
      if (minVertex === end) {
        break;
      }
      
      // Update distances to neighbors
      for (const neighbor of this.adjacencyList[minVertex]) {
        if (unvisited.has(neighbor.node)) {
          const newDistance = distances[minVertex] + neighbor.weight;
          
          if (newDistance < distances[neighbor.node]) {
            distances[neighbor.node] = newDistance;
            previous[neighbor.node] = minVertex;
          }
        }
      }
    }
    
    // Reconstruct path
    const path = [];
    let current = end;
    
    while (current !== null) {
      path.unshift(current);
      current = previous[current];
    }
    
    return {
      distance: distances[end],
      path: distances[end] === Infinity ? [] : path
    };
  }
}

// Test
console.log('=== Basic Dijkstra ===');

const graph = new WeightedGraph();

//      A
//    /   \
//   4     2
//  /       \
// B----1----C
//  \       /
//   3     5
//    \   /
//      D

['A', 'B', 'C', 'D'].forEach(v => graph.addVertex(v));
graph.addEdge('A', 'B', 4);
graph.addEdge('A', 'C', 2);
graph.addEdge('B', 'C', 1);
graph.addEdge('B', 'D', 3);
graph.addEdge('C', 'D', 5);

console.log('Shortest path A to D:', graph.dijkstra('A', 'D'));
// { distance: 7, path: ['A', 'B', 'D'] }
```

### **Approach 2: Dijkstra with Priority Queue (Min Heap)**
```javascript
/**
 * Optimized Dijkstra using priority queue
 * Time Complexity: O((V + E) log V)
 * Space Complexity: O(V)
 */
class MinHeap {
  constructor() {
    this.heap = [];
  }
  
  push(value, priority) {
    this.heap.push({ value, priority });
    this.bubbleUp(this.heap.length - 1);
  }
  
  pop() {
    if (this.heap.length === 0) return null;
    if (this.heap.length === 1) return this.heap.pop();
    
    const min = this.heap[0];
    this.heap[0] = this.heap.pop();
    this.bubbleDown(0);
    
    return min;
  }
  
  bubbleUp(index) {
    while (index > 0) {
      const parentIndex = Math.floor((index - 1) / 2);
      
      if (this.heap[index].priority >= this.heap[parentIndex].priority) {
        break;
      }
      
      [this.heap[index], this.heap[parentIndex]] = 
        [this.heap[parentIndex], this.heap[index]];
      
      index = parentIndex;
    }
  }
  
  bubbleDown(index) {
    while (true) {
      const leftChild = 2 * index + 1;
      const rightChild = 2 * index + 2;
      let smallest = index;
      
      if (
        leftChild < this.heap.length &&
        this.heap[leftChild].priority < this.heap[smallest].priority
      ) {
        smallest = leftChild;
      }
      
      if (
        rightChild < this.heap.length &&
        this.heap[rightChild].priority < this.heap[smallest].priority
      ) {
        smallest = rightChild;
      }
      
      if (smallest === index) break;
      
      [this.heap[index], this.heap[smallest]] = 
        [this.heap[smallest], this.heap[index]];
      
      index = smallest;
    }
  }
  
  isEmpty() {
    return this.heap.length === 0;
  }
}

class OptimizedWeightedGraph extends WeightedGraph {
  dijkstraOptimized(start, end) {
    const distances = {};
    const previous = {};
    const pq = new MinHeap();
    
    // Initialize
    for (const vertex in this.adjacencyList) {
      distances[vertex] = vertex === start ? 0 : Infinity;
      previous[vertex] = null;
    }
    
    pq.push(start, 0);
    
    while (!pq.isEmpty()) {
      const { value: current, priority: currentDistance } = pq.pop();
      
      // Early termination
      if (current === end) {
        break;
      }
      
      // Skip if we've found a better path already
      if (currentDistance > distances[current]) {
        continue;
      }
      
      // Check neighbors
      for (const neighbor of this.adjacencyList[current]) {
        const newDistance = distances[current] + neighbor.weight;
        
        if (newDistance < distances[neighbor.node]) {
          distances[neighbor.node] = newDistance;
          previous[neighbor.node] = current;
          pq.push(neighbor.node, newDistance);
        }
      }
    }
    
    // Reconstruct path
    const path = [];
    let current = end;
    
    while (current !== null) {
      path.unshift(current);
      current = previous[current];
    }
    
    return {
      distance: distances[end],
      path: distances[end] === Infinity ? [] : path
    };
  }
}

// Test
console.log('\n=== Optimized Dijkstra ===');

const graph2 = new OptimizedWeightedGraph();
['A', 'B', 'C', 'D', 'E'].forEach(v => graph2.addVertex(v));
graph2.addEdge('A', 'B', 4);
graph2.addEdge('A', 'C', 2);
graph2.addEdge('B', 'C', 1);
graph2.addEdge('B', 'D', 5);
graph2.addEdge('C', 'D', 8);
graph2.addEdge('C', 'E', 10);
graph2.addEdge('D', 'E', 2);

console.log('Shortest path A to E:', graph2.dijkstraOptimized('A', 'E'));
// { distance: 11, path: ['A', 'B', 'D', 'E'] }
```

### **Approach 3: Dijkstra with All Paths**
```javascript
/**
 * Find shortest distances to all vertices from source
 */
function dijkstraAll(graph, start) {
  const distances = {};
  const previous = {};
  const visited = new Set();
  const pq = new MinHeap();
  
  // Initialize
  for (const vertex in graph.adjacencyList) {
    distances[vertex] = vertex === start ? 0 : Infinity;
    previous[vertex] = null;
  }
  
  pq.push(start, 0);
  
  while (!pq.isEmpty()) {
    const { value: current } = pq.pop();
    
    if (visited.has(current)) continue;
    visited.add(current);
    
    for (const neighbor of graph.adjacencyList[current]) {
      const newDistance = distances[current] + neighbor.weight;
      
      if (newDistance < distances[neighbor.node]) {
        distances[neighbor.node] = newDistance;
        previous[neighbor.node] = current;
        pq.push(neighbor.node, newDistance);
      }
    }
  }
  
  return { distances, previous };
}

// Helper to reconstruct any path
function reconstructPath(previous, start, end) {
  const path = [];
  let current = end;
  
  while (current !== null) {
    path.unshift(current);
    current = previous[current];
  }
  
  return path[0] === start ? path : [];
}

// Test
console.log('\n=== Dijkstra All Paths ===');

const graph3 = new OptimizedWeightedGraph();
['A', 'B', 'C', 'D', 'E'].forEach(v => graph3.addVertex(v));
graph3.addEdge('A', 'B', 4);
graph3.addEdge('A', 'C', 2);
graph3.addEdge('B', 'D', 5);
graph3.addEdge('C', 'D', 8);
graph3.addEdge('C', 'E', 10);
graph3.addEdge('D', 'E', 2);

const { distances, previous } = dijkstraAll(graph3, 'A');
console.log('Distances from A:', distances);
console.log('Path to E:', reconstructPath(previous, 'A', 'E'));
```

### **Approach 4: Dijkstra on Grid**
```javascript
/**
 * Dijkstra for weighted grid (common in games)
 */
function dijkstraGrid(grid, start, end) {
  const [startRow, startCol] = start;
  const [endRow, endCol] = end;
  const rows = grid.length;
  const cols = grid[0].length;
  
  const distances = Array.from({ length: rows }, () => 
    Array(cols).fill(Infinity)
  );
  
  const previous = Array.from({ length: rows }, () => 
    Array(cols).fill(null)
  );
  
  const pq = new MinHeap();
  
  distances[startRow][startCol] = 0;
  pq.push([startRow, startCol], 0);
  
  const directions = [[-1, 0], [0, 1], [1, 0], [0, -1]];
  
  while (!pq.isEmpty()) {
    const { value: [row, col], priority: dist } = pq.pop();
    
    if (row === endRow && col === endCol) {
      break;
    }
    
    if (dist > distances[row][col]) {
      continue;
    }
    
    for (const [dr, dc] of directions) {
      const newRow = row + dr;
      const newCol = col + dc;
      
      if (
        newRow >= 0 && newRow < rows &&
        newCol >= 0 && newCol < cols &&
        grid[newRow][newCol] !== -1 // -1 = obstacle
      ) {
        const newDistance = distances[row][col] + grid[newRow][newCol];
        
        if (newDistance < distances[newRow][newCol]) {
          distances[newRow][newCol] = newDistance;
          previous[newRow][newCol] = [row, col];
          pq.push([newRow, newCol], newDistance);
        }
      }
    }
  }
  
  // Reconstruct path
  const path = [];
  let current = [endRow, endCol];
  
  while (current !== null) {
    path.unshift(current);
    current = previous[current[0]][current[1]];
  }
  
  return {
    distance: distances[endRow][endCol],
    path: distances[endRow][endCol] === Infinity ? [] : path
  };
}

// Test
console.log('\n=== Dijkstra on Grid ===');

// Grid values represent cost to enter that cell
const weightedGrid = [
  [1, 3, 1, 2],
  [2, 1, 4, 1],
  [1, 2, 1, 3],
  [3, 1, 2, 1]
];

const gridResult = dijkstraGrid(weightedGrid, [0, 0], [3, 3]);
console.log('Shortest path cost:', gridResult.distance);
console.log('Path:', gridResult.path);
```

### **Approach 5: A* Algorithm (Dijkstra with Heuristic)**
```javascript
/**
 * A* algorithm - Dijkstra with heuristic for faster pathfinding
 */
function aStarGrid(grid, start, end) {
  const [startRow, startCol] = start;
  const [endRow, endCol] = end;
  const rows = grid.length;
  const cols = grid[0].length;
  
  // Heuristic: Manhattan distance
  const heuristic = (row, col) => {
    return Math.abs(row - endRow) + Math.abs(col - endCol);
  };
  
  const gScore = Array.from({ length: rows }, () => 
    Array(cols).fill(Infinity)
  );
  
  const fScore = Array.from({ length: rows }, () => 
    Array(cols).fill(Infinity)
  );
  
  const previous = Array.from({ length: rows }, () => 
    Array(cols).fill(null)
  );
  
  gScore[startRow][startCol] = 0;
  fScore[startRow][startCol] = heuristic(startRow, startCol);
  
  const pq = new MinHeap();
  pq.push([startRow, startCol], fScore[startRow][startCol]);
  
  const directions = [[-1, 0], [0, 1], [1, 0], [0, -1]];
  
  while (!pq.isEmpty()) {
    const { value: [row, col] } = pq.pop();
    
    if (row === endRow && col === endCol) {
      break;
    }
    
    for (const [dr, dc] of directions) {
      const newRow = row + dr;
      const newCol = col + dc;
      
      if (
        newRow >= 0 && newRow < rows &&
        newCol >= 0 && newCol < cols &&
        grid[newRow][newCol] !== -1
      ) {
        const tentativeG = gScore[row][col] + grid[newRow][newCol];
        
        if (tentativeG < gScore[newRow][newCol]) {
          previous[newRow][newCol] = [row, col];
          gScore[newRow][newCol] = tentativeG;
          fScore[newRow][newCol] = tentativeG + heuristic(newRow, newCol);
          pq.push([newRow, newCol], fScore[newRow][newCol]);
        }
      }
    }
  }
  
  // Reconstruct path
  const path = [];
  let current = [endRow, endCol];
  
  while (current !== null) {
    path.unshift(current);
    current = previous[current[0]][current[1]];
  }
  
  return {
    distance: gScore[endRow][endCol],
    path: gScore[endRow][endCol] === Infinity ? [] : path
  };
}

// Test
console.log('\n=== A* Algorithm ===');

const astarGrid = [
  [1, 1, 1, 1, 1],
  [1, -1, -1, -1, 1],
  [1, 1, 1, 1, 1],
  [1, -1, -1, -1, 1],
  [1, 1, 1, 1, 1]
];

const astarResult = aStarGrid(astarGrid, [0, 0], [4, 4]);
console.log('A* shortest path:', astarResult.path);
```

### **Real-World Use Cases**
```javascript
/**
 * Practical Dijkstra applications
 */

// 1. GPS Navigation System
console.log('\n=== GPS Navigation ===');

class NavigationSystem {
  constructor() {
    this.map = new OptimizedWeightedGraph();
  }
  
  addLocation(name) {
    this.map.addVertex(name);
  }
  
  addRoute(from, to, distance, traffic = 1) {
    // Weight considers distance and traffic
    const weight = distance * traffic;
    this.map.addEdge(from, to, weight);
  }
  
  findRoute(start, end) {
    const result = this.map.dijkstraOptimized(start, end);
    return {
      ...result,
      estimatedTime: Math.round(result.distance / 60) // Convert to minutes
    };
  }
}

const gps = new NavigationSystem();

['Home', 'Work', 'Store', 'School', 'Park'].forEach(loc => 
  gps.addLocation(loc)
);

gps.addRoute('Home', 'Work', 10, 1.5); // 10km, heavy traffic
gps.addRoute('Home', 'Store', 5, 1);
gps.addRoute('Store', 'Work', 8, 1.2);
gps.addRoute('Store', 'School', 3, 1);
gps.addRoute('School', 'Work', 6, 1);

console.log('Best route Home to Work:', gps.findRoute('Home', 'Work'));

// 2. Network Routing
console.log('\n=== Network Routing ===');

class NetworkRouter {
  constructor() {
    this.network = new OptimizedWeightedGraph();
  }
  
  addNode(node) {
    this.network.addVertex(node);
  }
  
  addConnection(node1, node2, latency) {
    this.network.addEdge(node1, node2, latency);
  }
  
  findFastestRoute(source, destination) {
    return this.network.dijkstraOptimized(source, destination);
  }
}

const router = new NetworkRouter();

['Server-A', 'Server-B', 'Server-C', 'Server-D', 'Client'].forEach(n => 
  router.addNode(n)
);

router.addConnection('Client', 'Server-A', 10);
router.addConnection('Client', 'Server-B', 15);
router.addConnection('Server-A', 'Server-C', 5);
router.addConnection('Server-B', 'Server-C', 8);
router.addConnection('Server-C', 'Server-D', 3);

console.log('Fastest route Client to Server-D:', 
  router.findFastestRoute('Client', 'Server-D')
);

// 3. Flight Booking System
console.log('\n=== Flight Booking ===');

class FlightBooking {
  constructor() {
    this.routes = new OptimizedWeightedGraph();
  }
  
  addAirport(code) {
    this.routes.addVertex(code);
  }
  
  addFlight(from, to, price) {
    this.routes.addEdge(from, to, price);
  }
  
  findCheapestRoute(from, to) {
    const result = this.routes.dijkstraOptimized(from, to);
    return {
      route: result.path,
      totalPrice: result.distance,
      stops: result.path.length - 2
    };
  }
}

const booking = new FlightBooking();

['NYC', 'LAX', 'CHI', 'MIA', 'SEA'].forEach(city => 
  booking.addAirport(city)
);

booking.addFlight('NYC', 'CHI', 200);
booking.addFlight('NYC', 'MIA', 300);
booking.addFlight('CHI', 'LAX', 150);
booking.addFlight('MIA', 'LAX', 250);
booking.addFlight('CHI', 'SEA', 180);
booking.addFlight('SEA', 'LAX', 100);

console.log('Cheapest NYC to LAX:', booking.findCheapestRoute('NYC', 'LAX'));

// 4. Game Pathfinding with Obstacles
console.log('\n=== Game Pathfinding ===');

function findGamePath(map, player, goal) {
  // Convert character positions to grid coordinates
  const rows = map.length;
  const cols = map[0].length;
  
  // Build weighted grid (different terrain costs)
  const grid = map.map(row => 
    row.map(cell => {
      if (cell === '#') return -1; // Wall
      if (cell === 'W') return 5;  // Water (slow)
      if (cell === 'M') return 3;  // Mountain (medium)
      return 1; // Plain
    })
  );
  
  return dijkstraGrid(grid, player, goal);
}

const gameMap = [
  ['.', '.', 'W', '.', '.'],
  ['.', '#', 'W', '#', '.'],
  ['.', '.', '.', '.', '.'],
  ['M', 'M', '.', 'M', 'M'],
  ['.', '.', '.', '.', '.']
];

console.log('Game path (0,0) to (4,4):', 
  findGamePath(gameMap, [0, 0], [4, 4])
);

// 5. Delivery Route Optimization
console.log('\n=== Delivery Routes ===');

class DeliveryRouter {
  constructor() {
    this.cityMap = new OptimizedWeightedGraph();
  }
  
  addLocation(location) {
    this.cityMap.addVertex(location);
  }
  
  addRoad(loc1, loc2, distance) {
    this.cityMap.addEdge(loc1, loc2, distance);
  }
  
  optimizeRoute(warehouse, deliveries) {
    let currentLocation = warehouse;
    let totalDistance = 0;
    const route = [warehouse];
    const remaining = new Set(deliveries);
    
    // Greedy: always go to nearest unvisited delivery
    while (remaining.size > 0) {
      let nearest = null;
      let minDistance = Infinity;
      
      for (const delivery of remaining) {
        const result = this.cityMap.dijkstraOptimized(currentLocation, delivery);
        
        if (result.distance < minDistance) {
          minDistance = result.distance;
          nearest = delivery;
        }
      }
      
      if (nearest) {
        route.push(nearest);
        totalDistance += minDistance;
        currentLocation = nearest;
        remaining.delete(nearest);
      }
    }
    
    // Return to warehouse
    const returnTrip = this.cityMap.dijkstraOptimized(currentLocation, warehouse);
    route.push(warehouse);
    totalDistance += returnTrip.distance;
    
    return { route, totalDistance };
  }
}

const delivery = new DeliveryRouter();

['Warehouse', 'A', 'B', 'C', 'D'].forEach(loc => 
  delivery.addLocation(loc)
);

delivery.addRoad('Warehouse', 'A', 5);
delivery.addRoad('Warehouse', 'B', 10);
delivery.addRoad('A', 'C', 3);
delivery.addRoad('B', 'C', 2);
delivery.addRoad('C', 'D', 4);
delivery.addRoad('B', 'D', 8);

console.log('Optimized delivery route:', 
  delivery.optimizeRoute('Warehouse', ['A', 'C', 'D'])
);
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// No path exists
const disconnected = new OptimizedWeightedGraph();
['A', 'B', 'C', 'D'].forEach(v => disconnected.addVertex(v));
disconnected.addEdge('A', 'B', 1);
disconnected.addEdge('C', 'D', 1);

console.log('No path A to D:', disconnected.dijkstraOptimized('A', 'D'));
// { distance: Infinity, path: [] }

// Single vertex
const single = new OptimizedWeightedGraph();
single.addVertex('A');
console.log('Path to self:', single.dijkstraOptimized('A', 'A'));
// { distance: 0, path: ['A'] }

// Negative weights (Dijkstra doesn't work correctly)
const negative = new OptimizedWeightedGraph();
['A', 'B', 'C'].forEach(v => negative.addVertex(v));
negative.addEdge('A', 'B', 5);
negative.addEdge('B', 'C', -3); // Negative weight!
console.log('With negative weight:', negative.dijkstraOptimized('A', 'C'));
console.log('Note: Dijkstra incorrect with negative weights. Use Bellman-Ford instead.');

// Multiple equal shortest paths
const multiple = new OptimizedWeightedGraph();
['A', 'B', 'C', 'D'].forEach(v => multiple.addVertex(v));
multiple.addEdge('A', 'B', 1);
multiple.addEdge('A', 'C', 1);
multiple.addEdge('B', 'D', 1);
multiple.addEdge('C', 'D', 1);

console.log('Multiple equal paths A to D:', multiple.dijkstraOptimized('A', 'D'));
// Returns one of the equal-cost paths

// Zero weight edges
const zeroWeight = new OptimizedWeightedGraph();
['A', 'B', 'C'].forEach(v => zeroWeight.addVertex(v));
zeroWeight.addEdge('A', 'B', 0);
zeroWeight.addEdge('B', 'C', 5);

console.log('With zero weight:', zeroWeight.dijkstraOptimized('A', 'C'));
```

### **Performance Analysis**
```javascript
console.log('\n=== Performance ===');

console.log(`
Time Complexity:
┌──────────────────────┬─────────────────┬─────────────────┐
│ Implementation       │ Time            │ Space           │
├──────────────────────┼─────────────────┼─────────────────┤
│ Array (naive)        │ O(V²)           │ O(V)            │
│ Min Heap             │ O((V+E) log V)  │ O(V)            │
│ Fibonacci Heap       │ O(E + V log V)  │ O(V)            │
└──────────────────────┴─────────────────┴─────────────────┘

Where:
  V = number of vertices
  E = number of edges

Algorithm Comparison:
┌──────────────────┬─────────────┬─────────────┬─────────────┐
│ Algorithm        │ Time        │ Negative    │ Use Case    │
│                  │             │ Weights     │             │
├──────────────────┼─────────────┼─────────────┼─────────────┤
│ Dijkstra         │ O(E log V)  │ No          │ General     │
│ Bellman-Ford     │ O(VE)       │ Yes         │ Neg weights │
│ Floyd-Warshall   │ O(V³)       │ Yes         │ All pairs   │
│ BFS              │ O(V + E)    │ Unweighted  │ Unweighted  │
│ A*               │ O(E log V)  │ No          │ With heur   │
└──────────────────┴─────────────┴─────────────┴─────────────┘

Optimization Techniques:
1. Use priority queue (min heap) instead of array
2. Early termination when target found
3. Bidirectional search from both ends
4. A* with good heuristic for specific targets
5. Skip already-visited vertices
6. Fibonacci heap for theoretical improvement

Memory Optimization:
• Store only essential data in heap
• Use indices instead of full vertex objects
• Lazy deletion from priority queue
• Compress path reconstruction data
`);
```

**Interview Tips:**
- Dijkstra: finds shortest path in weighted graph with non-negative weights
- Greedy algorithm: always picks vertex with minimum distance
- Data structures: distances array, priority queue (min heap), previous/parent tracking
- Algorithm: initialize distances to infinity except start (0), repeatedly extract min, relax neighbors
- Relaxation: if dist[u] + weight(u,v) < dist[v], update dist[v] and previous[v]
- Time: O(V²) with array, O((V+E) log V) with min heap (better for sparse graphs)
- Space: O(V) for distances and previous arrays
- Early termination: stop when target vertex is dequeued
- Path reconstruction: backtrack through previous pointers from end to start
- Applications: GPS navigation, network routing, game pathfinding, flight booking
- vs BFS: Dijkstra handles weighted graphs, BFS only unweighted
- Limitation: doesn't work with negative weights (use Bellman-Ford instead)
- A* optimization: add heuristic for faster search to specific target
- Priority queue: critical for performance, use min heap not array
- Edge cases: no path (return Infinity), negative weights (undefined behavior), single vertex
- Grid version: treat cells as vertices, neighbors as edges, use 4 or 8 directions
- Bidirectional: search from both ends simultaneously, faster for single target
- All-pairs: run Dijkstra from each vertex or use Floyd-Warshall
- Clarify: negative weights? single path or all? need actual path or just distance? directed/undirected?
- Follow-ups: implement with Fibonacci heap, handle negative weights (Bellman-Ford), find k shortest paths, add turn costs

</details>

101. Implement a Min Heap / Max Heap

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Min Heap**
```javascript
/**
 * Min Heap - parent smaller than children
 * Time Complexity:
 *   - Insert: O(log n)
 *   - Extract Min: O(log n)
 *   - Peek: O(1)
 * Space Complexity: O(n)
 */
class MinHeap {
  constructor() {
    this.heap = [];
  }
  
  getParentIndex(i) {
    return Math.floor((i - 1) / 2);
  }
  
  getLeftChildIndex(i) {
    return 2 * i + 1;
  }
  
  getRightChildIndex(i) {
    return 2 * i + 2;
  }
  
  swap(i, j) {
    [this.heap[i], this.heap[j]] = [this.heap[j], this.heap[i]];
  }
  
  insert(value) {
    this.heap.push(value);
    this.bubbleUp(this.heap.length - 1);
  }
  
  bubbleUp(index) {
    while (index > 0) {
      const parentIndex = this.getParentIndex(index);
      
      if (this.heap[index] >= this.heap[parentIndex]) {
        break;
      }
      
      this.swap(index, parentIndex);
      index = parentIndex;
    }
  }
  
  extractMin() {
    if (this.heap.length === 0) {
      return null;
    }
    
    if (this.heap.length === 1) {
      return this.heap.pop();
    }
    
    const min = this.heap[0];
    this.heap[0] = this.heap.pop();
    this.bubbleDown(0);
    
    return min;
  }
  
  bubbleDown(index) {
    while (true) {
      const leftChild = this.getLeftChildIndex(index);
      const rightChild = this.getRightChildIndex(index);
      let smallest = index;
      
      if (
        leftChild < this.heap.length &&
        this.heap[leftChild] < this.heap[smallest]
      ) {
        smallest = leftChild;
      }
      
      if (
        rightChild < this.heap.length &&
        this.heap[rightChild] < this.heap[smallest]
      ) {
        smallest = rightChild;
      }
      
      if (smallest === index) {
        break;
      }
      
      this.swap(index, smallest);
      index = smallest;
    }
  }
  
  peek() {
    return this.heap.length > 0 ? this.heap[0] : null;
  }
  
  size() {
    return this.heap.length;
  }
  
  isEmpty() {
    return this.heap.length === 0;
  }
}

// Test
console.log('=== Min Heap ===');

const minHeap = new MinHeap();

[5, 3, 7, 1, 4, 6].forEach(val => minHeap.insert(val));

console.log('Min Heap:', minHeap.heap);
console.log('Extract min:', minHeap.extractMin()); // 1
console.log('Extract min:', minHeap.extractMin()); // 3
console.log('Peek:', minHeap.peek()); // 4
console.log('Size:', minHeap.size()); // 4
```

### **Approach 2: Max Heap**
```javascript
/**
 * Max Heap - parent larger than children
 */
class MaxHeap {
  constructor() {
    this.heap = [];
  }
  
  getParentIndex(i) {
    return Math.floor((i - 1) / 2);
  }
  
  getLeftChildIndex(i) {
    return 2 * i + 1;
  }
  
  getRightChildIndex(i) {
    return 2 * i + 2;
  }
  
  swap(i, j) {
    [this.heap[i], this.heap[j]] = [this.heap[j], this.heap[i]];
  }
  
  insert(value) {
    this.heap.push(value);
    this.bubbleUp(this.heap.length - 1);
  }
  
  bubbleUp(index) {
    while (index > 0) {
      const parentIndex = this.getParentIndex(index);
      
      if (this.heap[index] <= this.heap[parentIndex]) {
        break;
      }
      
      this.swap(index, parentIndex);
      index = parentIndex;
    }
  }
  
  extractMax() {
    if (this.heap.length === 0) {
      return null;
    }
    
    if (this.heap.length === 1) {
      return this.heap.pop();
    }
    
    const max = this.heap[0];
    this.heap[0] = this.heap.pop();
    this.bubbleDown(0);
    
    return max;
  }
  
  bubbleDown(index) {
    while (true) {
      const leftChild = this.getLeftChildIndex(index);
      const rightChild = this.getRightChildIndex(index);
      let largest = index;
      
      if (
        leftChild < this.heap.length &&
        this.heap[leftChild] > this.heap[largest]
      ) {
        largest = leftChild;
      }
      
      if (
        rightChild < this.heap.length &&
        this.heap[rightChild] > this.heap[largest]
      ) {
        largest = rightChild;
      }
      
      if (largest === index) {
        break;
      }
      
      this.swap(index, largest);
      index = largest;
    }
  }
  
  peek() {
    return this.heap.length > 0 ? this.heap[0] : null;
  }
  
  size() {
    return this.heap.length;
  }
}

// Test
console.log('\n=== Max Heap ===');

const maxHeap = new MaxHeap();

[5, 3, 7, 1, 4, 6].forEach(val => maxHeap.insert(val));

console.log('Max Heap:', maxHeap.heap);
console.log('Extract max:', maxHeap.extractMax()); // 7
console.log('Extract max:', maxHeap.extractMax()); // 6
console.log('Peek:', maxHeap.peek()); // 5
```

### **Approach 3: Generic Heap with Comparator**
```javascript
/**
 * Generic heap that works as min or max based on comparator
 */
class Heap {
  constructor(compareFn = (a, b) => a - b) {
    this.heap = [];
    this.compare = compareFn;
  }
  
  getParentIndex(i) {
    return Math.floor((i - 1) / 2);
  }
  
  getLeftChildIndex(i) {
    return 2 * i + 1;
  }
  
  getRightChildIndex(i) {
    return 2 * i + 2;
  }
  
  swap(i, j) {
    [this.heap[i], this.heap[j]] = [this.heap[j], this.heap[i]];
  }
  
  insert(value) {
    this.heap.push(value);
    this.bubbleUp(this.heap.length - 1);
  }
  
  bubbleUp(index) {
    while (index > 0) {
      const parentIndex = this.getParentIndex(index);
      
      if (this.compare(this.heap[index], this.heap[parentIndex]) >= 0) {
        break;
      }
      
      this.swap(index, parentIndex);
      index = parentIndex;
    }
  }
  
  extract() {
    if (this.heap.length === 0) return null;
    if (this.heap.length === 1) return this.heap.pop();
    
    const root = this.heap[0];
    this.heap[0] = this.heap.pop();
    this.bubbleDown(0);
    
    return root;
  }
  
  bubbleDown(index) {
    while (true) {
      const leftChild = this.getLeftChildIndex(index);
      const rightChild = this.getRightChildIndex(index);
      let target = index;
      
      if (
        leftChild < this.heap.length &&
        this.compare(this.heap[leftChild], this.heap[target]) < 0
      ) {
        target = leftChild;
      }
      
      if (
        rightChild < this.heap.length &&
        this.compare(this.heap[rightChild], this.heap[target]) < 0
      ) {
        target = rightChild;
      }
      
      if (target === index) break;
      
      this.swap(index, target);
      index = target;
    }
  }
  
  peek() {
    return this.heap.length > 0 ? this.heap[0] : null;
  }
  
  size() {
    return this.heap.length;
  }
  
  isEmpty() {
    return this.heap.length === 0;
  }
  
  toArray() {
    return [...this.heap];
  }
}

// Test
console.log('\n=== Generic Heap ===');

// Min heap for numbers
const minHeap2 = new Heap((a, b) => a - b);
[5, 3, 7, 1].forEach(v => minHeap2.insert(v));
console.log('Min heap:', minHeap2.toArray());

// Max heap for numbers
const maxHeap2 = new Heap((a, b) => b - a);
[5, 3, 7, 1].forEach(v => maxHeap2.insert(v));
console.log('Max heap:', maxHeap2.toArray());

// Custom comparator for objects
const taskHeap = new Heap((a, b) => a.priority - b.priority);
taskHeap.insert({ task: 'Low', priority: 3 });
taskHeap.insert({ task: 'High', priority: 1 });
taskHeap.insert({ task: 'Medium', priority: 2 });

console.log('Next task:', taskHeap.extract());
```

### **Approach 4: Heap with Additional Operations**
```javascript
/**
 * Enhanced heap with more operations
 */
class EnhancedHeap extends Heap {
  constructor(compareFn = (a, b) => a - b) {
    super(compareFn);
    this.indexMap = new Map(); // Track element indices
  }
  
  insert(value) {
    this.heap.push(value);
    const index = this.heap.length - 1;
    this.indexMap.set(value, index);
    this.bubbleUp(index);
  }
  
  extract() {
    if (this.heap.length === 0) return null;
    if (this.heap.length === 1) {
      const value = this.heap.pop();
      this.indexMap.delete(value);
      return value;
    }
    
    const root = this.heap[0];
    this.indexMap.delete(root);
    
    const last = this.heap.pop();
    this.heap[0] = last;
    this.indexMap.set(last, 0);
    this.bubbleDown(0);
    
    return root;
  }
  
  remove(value) {
    const index = this.indexMap.get(value);
    if (index === undefined) return false;
    
    if (index === this.heap.length - 1) {
      this.heap.pop();
      this.indexMap.delete(value);
      return true;
    }
    
    const last = this.heap.pop();
    this.indexMap.delete(value);
    
    this.heap[index] = last;
    this.indexMap.set(last, index);
    
    // May need to bubble up or down
    const parentIndex = this.getParentIndex(index);
    if (index > 0 && this.compare(this.heap[index], this.heap[parentIndex]) < 0) {
      this.bubbleUp(index);
    } else {
      this.bubbleDown(index);
    }
    
    return true;
  }
  
  updatePriority(oldValue, newValue) {
    this.remove(oldValue);
    this.insert(newValue);
  }
  
  contains(value) {
    return this.indexMap.has(value);
  }
  
  clear() {
    this.heap = [];
    this.indexMap.clear();
  }
  
  // Build heap from array in O(n)
  heapify(array) {
    this.heap = [...array];
    this.indexMap.clear();
    
    array.forEach((val, idx) => this.indexMap.set(val, idx));
    
    // Start from last non-leaf node
    for (let i = Math.floor(this.heap.length / 2) - 1; i >= 0; i--) {
      this.bubbleDown(i);
    }
  }
}

// Test
console.log('\n=== Enhanced Heap ===');

const enhanced = new EnhancedHeap((a, b) => a - b);

[5, 3, 7, 1, 4].forEach(v => enhanced.insert(v));
console.log('Contains 3:', enhanced.contains(3)); // true

enhanced.remove(3);
console.log('After remove 3:', enhanced.toArray());

// Heapify
const arr = [9, 5, 6, 2, 3];
enhanced.heapify(arr);
console.log('Heapified:', enhanced.toArray());
```

### **Approach 5: Heap Sort Implementation**
```javascript
/**
 * Heap sort using heap
 */
function heapSort(array) {
  const heap = new MaxHeap();
  
  // Build heap
  array.forEach(val => heap.insert(val));
  
  // Extract all elements
  const sorted = [];
  while (heap.size() > 0) {
    sorted.unshift(heap.extractMax());
  }
  
  return sorted;
}

// In-place heap sort
function heapSortInPlace(array) {
  const n = array.length;
  
  // Build max heap
  for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
    heapifyDown(array, n, i);
  }
  
  // Extract elements one by one
  for (let i = n - 1; i > 0; i--) {
    [array[0], array[i]] = [array[i], array[0]];
    heapifyDown(array, i, 0);
  }
  
  return array;
}

function heapifyDown(array, heapSize, index) {
  let largest = index;
  const left = 2 * index + 1;
  const right = 2 * index + 2;
  
  if (left < heapSize && array[left] > array[largest]) {
    largest = left;
  }
  
  if (right < heapSize && array[right] > array[largest]) {
    largest = right;
  }
  
  if (largest !== index) {
    [array[index], array[largest]] = [array[largest], array[index]];
    heapifyDown(array, heapSize, largest);
  }
}

// Test
console.log('\n=== Heap Sort ===');

console.log('Sorted:', heapSort([5, 3, 7, 1, 4, 6]));
// [1, 3, 4, 5, 6, 7]

const arr = [5, 3, 7, 1, 4, 6];
heapSortInPlace(arr);
console.log('In-place sorted:', arr);
```

### **Real-World Use Cases**
```javascript
/**
 * Practical heap applications
 */

// 1. Priority Queue for Task Scheduling
console.log('\n=== Task Scheduler ===');

class TaskScheduler {
  constructor() {
    this.tasks = new Heap((a, b) => a.priority - b.priority);
  }
  
  addTask(name, priority) {
    this.tasks.insert({ name, priority, timestamp: Date.now() });
  }
  
  getNextTask() {
    return this.tasks.extract();
  }
  
  peekNextTask() {
    return this.tasks.peek();
  }
  
  isEmpty() {
    return this.tasks.isEmpty();
  }
}

const scheduler = new TaskScheduler();

scheduler.addTask('Send email', 3);
scheduler.addTask('Fix critical bug', 1);
scheduler.addTask('Code review', 2);
scheduler.addTask('Meeting', 2);

console.log('Next task:', scheduler.getNextTask().name); // Fix critical bug
console.log('Next task:', scheduler.getNextTask().name); // Code review

// 2. Find K Largest/Smallest Elements
console.log('\n=== K Largest Elements ===');

function findKLargest(array, k) {
  const minHeap = new MinHeap();
  
  for (const num of array) {
    minHeap.insert(num);
    
    if (minHeap.size() > k) {
      minHeap.extractMin();
    }
  }
  
  return minHeap.heap.sort((a, b) => b - a);
}

console.log('3 largest in [3,1,5,12,2,11]:', findKLargest([3, 1, 5, 12, 2, 11], 3));
// [12, 11, 5]

// 3. Merge K Sorted Arrays
console.log('\n=== Merge K Sorted Arrays ===');

function mergeKSortedArrays(arrays) {
  const heap = new Heap((a, b) => a.value - b.value);
  const result = [];
  
  // Initialize heap with first element from each array
  arrays.forEach((arr, arrayIndex) => {
    if (arr.length > 0) {
      heap.insert({ value: arr[0], arrayIndex, elementIndex: 0 });
    }
  });
  
  while (!heap.isEmpty()) {
    const { value, arrayIndex, elementIndex } = heap.extract();
    result.push(value);
    
    // Add next element from same array
    const nextIndex = elementIndex + 1;
    if (nextIndex < arrays[arrayIndex].length) {
      heap.insert({
        value: arrays[arrayIndex][nextIndex],
        arrayIndex,
        elementIndex: nextIndex
      });
    }
  }
  
  return result;
}

const sortedArrays = [
  [1, 4, 7],
  [2, 5, 8],
  [3, 6, 9]
];

console.log('Merged:', mergeKSortedArrays(sortedArrays));
// [1, 2, 3, 4, 5, 6, 7, 8, 9]

// 4. Running Median
console.log('\n=== Running Median ===');

class MedianFinder {
  constructor() {
    this.maxHeap = new Heap((a, b) => b - a); // Lower half
    this.minHeap = new Heap((a, b) => a - b); // Upper half
  }
  
  addNum(num) {
    // Add to max heap first
    this.maxHeap.insert(num);
    
    // Balance: move largest from maxHeap to minHeap
    this.minHeap.insert(this.maxHeap.extract());
    
    // Ensure maxHeap has equal or one more element
    if (this.maxHeap.size() < this.minHeap.size()) {
      this.maxHeap.insert(this.minHeap.extract());
    }
  }
  
  findMedian() {
    if (this.maxHeap.size() > this.minHeap.size()) {
      return this.maxHeap.peek();
    }
    
    return (this.maxHeap.peek() + this.minHeap.peek()) / 2;
  }
}

const medianFinder = new MedianFinder();

[1, 2, 3, 4, 5].forEach(num => {
  medianFinder.addNum(num);
  console.log(`After adding ${num}, median:`, medianFinder.findMedian());
});

// 5. CPU Task Scheduling with Cooldown
console.log('\n=== CPU Scheduler ===');

function leastInterval(tasks, cooldown) {
  const freq = {};
  
  // Count frequencies
  for (const task of tasks) {
    freq[task] = (freq[task] || 0) + 1;
  }
  
  // Max heap by frequency
  const heap = new Heap((a, b) => b - a);
  Object.values(freq).forEach(f => heap.insert(f));
  
  let time = 0;
  const queue = []; // [frequency, availableTime]
  
  while (!heap.isEmpty() || queue.length > 0) {
    time++;
    
    if (!heap.isEmpty()) {
      const freq = heap.extract() - 1;
      
      if (freq > 0) {
        queue.push([freq, time + cooldown]);
      }
    }
    
    if (queue.length > 0 && queue[0][1] === time) {
      heap.insert(queue.shift()[0]);
    }
  }
  
  return time;
}

console.log('Min intervals for [A,A,A,B,B,B] with cooldown 2:', 
  leastInterval(['A','A','A','B','B','B'], 2)
);
// 8 (A->B->idle->A->B->idle->A->B)
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty heap
const empty = new MinHeap();
console.log('Extract from empty:', empty.extractMin()); // null
console.log('Peek empty:', empty.peek()); // null

// Single element
const single = new MinHeap();
single.insert(5);
console.log('Single element:', single.extractMin()); // 5
console.log('After extract:', single.isEmpty()); // true

// Duplicate values
const duplicates = new MinHeap();
[5, 3, 5, 1, 3].forEach(v => duplicates.insert(v));
console.log('With duplicates:', duplicates.toArray());

// All same values
const same = new MinHeap();
[5, 5, 5, 5].forEach(v => same.insert(v));
console.log('All same:', same.toArray());

// Large heap
const large = new MinHeap();
for (let i = 1000; i > 0; i--) {
  large.insert(i);
}
console.log('Large heap min:', large.peek()); // 1
console.log('Large heap size:', large.size()); // 1000
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Time Complexity:
┌──────────────────┬─────────────┬─────────────┐
│ Operation        │ Time        │ Note        │
├──────────────────┼─────────────┼─────────────┤
│ Insert           │ O(log n)    │ Bubble up   │
│ Extract Min/Max  │ O(log n)    │ Bubble down │
│ Peek             │ O(1)        │ Access root │
│ Remove arbitrary │ O(log n)    │ Find + fix  │
│ Build heap       │ O(n)        │ Heapify     │
│ Heap sort        │ O(n log n)  │ n extracts  │
└──────────────────┴─────────────┴─────────────┘

Space Complexity: O(n)

Heap vs Other Structures:
┌────────────────┬──────────┬──────────┬──────────┐
│ Operation      │ Heap     │ Array    │ BST      │
├────────────────┼──────────┼──────────┼──────────┤
│ Find min/max   │ O(1)     │ O(n)     │ O(log n) │
│ Insert         │ O(log n) │ O(1)     │ O(log n) │
│ Delete min/max │ O(log n) │ O(n)     │ O(log n) │
│ Search         │ O(n)     │ O(n)     │ O(log n) │
└────────────────┴──────────┴──────────┴──────────┘

Use Cases:
• Priority Queue: task scheduling, event handling
• Heap Sort: O(n log n) sorting, in-place possible
• K largest/smallest: maintain heap of size k
• Median finding: two heaps (max + min)
• Graph algorithms: Dijkstra, Prim's MST
• Merge K sorted: heap of size k
`);
```

**Interview Tips:**
- Heap: complete binary tree stored as array
- Min heap: parent ≤ children, root is minimum
- Max heap: parent ≥ children, root is maximum
- Array representation: parent at i, children at 2i+1 and 2i+2
- Insert: add to end, bubble up to maintain heap property
- Extract: remove root, move last to root, bubble down
- Bubble up: swap with parent while violating heap property
- Bubble down: swap with smaller/larger child while violating property
- Time: O(log n) for insert/extract, O(1) for peek
- Space: O(n) for storing n elements
- Heapify: build heap from array in O(n) time (bottom-up)
- Applications: priority queue, heap sort, k largest/smallest, running median
- Priority queue: heap is ideal implementation
- Heap sort: O(n log n) time, O(1) extra space (in-place)
- K largest: use min heap of size k, keep removing smallest if size > k
- Running median: use two heaps, max heap for lower half, min heap for upper half
- Generic heap: use comparator function for flexibility
- Edge cases: empty heap, single element, duplicates, all same
- Complete tree: all levels full except last, filled left to right
- Not for searching: heaps don't maintain full sorted order, only heap property
- vs BST: heap faster for min/max, BST faster for search
- Clarify: min or max heap? need to remove arbitrary elements? custom comparison?
- Follow-ups: implement decrease-key operation, merge two heaps, make it persistent

</details>

102. Detect cycle in a directed graph

<details>
<summary><b>Solution</b></summary>

### **Approach 1: DFS with Recursion Stack**
```javascript
/**
 * Cycle detection using DFS with recursion stack
 * Time Complexity: O(V + E)
 * Space Complexity: O(V)
 */
class DirectedGraph {
  constructor() {
    this.adjacencyList = {};
  }
  
  addVertex(vertex) {
    if (!this.adjacencyList[vertex]) {
      this.adjacencyList[vertex] = [];
    }
  }
  
  addEdge(from, to) {
    this.adjacencyList[from].push(to); // Directed edge
  }
  
  hasCycle() {
    const visited = new Set();
    const recStack = new Set(); // Recursion stack
    
    const dfs = (vertex) => {
      visited.add(vertex);
      recStack.add(vertex);
      
      for (const neighbor of this.adjacencyList[vertex]) {
        if (!visited.has(neighbor)) {
          if (dfs(neighbor)) {
            return true;
          }
        } else if (recStack.has(neighbor)) {
          // Found back edge to vertex in current path
          return true;
        }
      }
      
      recStack.delete(vertex);
      return false;
    };
    
    // Check all vertices (for disconnected components)
    for (const vertex in this.adjacencyList) {
      if (!visited.has(vertex)) {
        if (dfs(vertex)) {
          return true;
        }
      }
    }
    
    return false;
  }
}

// Test
console.log('=== Cycle Detection (DFS) ===');

const graph1 = new DirectedGraph();
['A', 'B', 'C', 'D'].forEach(v => graph1.addVertex(v));
graph1.addEdge('A', 'B');
graph1.addEdge('B', 'C');
graph1.addEdge('C', 'D');

console.log('Has cycle (no):', graph1.hasCycle()); // false

graph1.addEdge('D', 'A'); // Create cycle
console.log('Has cycle (yes):', graph1.hasCycle()); // true
```

### **Approach 2: DFS with Three Colors**
```javascript
/**
 * Three-color DFS: white (unvisited), gray (processing), black (done)
 */
class ColoredGraph extends DirectedGraph {
  hasCycleWithColors() {
    const WHITE = 0; // Unvisited
    const GRAY = 1;  // In progress
    const BLACK = 2; // Completed
    
    const colors = {};
    
    // Initialize all vertices as white
    for (const vertex in this.adjacencyList) {
      colors[vertex] = WHITE;
    }
    
    const dfs = (vertex) => {
      colors[vertex] = GRAY;
      
      for (const neighbor of this.adjacencyList[vertex]) {
        if (colors[neighbor] === GRAY) {
          return true; // Back edge to gray node = cycle
        }
        
        if (colors[neighbor] === WHITE) {
          if (dfs(neighbor)) {
            return true;
          }
        }
      }
      
      colors[vertex] = BLACK;
      return false;
    };
    
    for (const vertex in this.adjacencyList) {
      if (colors[vertex] === WHITE) {
        if (dfs(vertex)) {
          return true;
        }
      }
    }
    
    return false;
  }
}

// Test
console.log('\n=== Three-Color DFS ===');

const graph2 = new ColoredGraph();
['A', 'B', 'C', 'D'].forEach(v => graph2.addVertex(v));
graph2.addEdge('A', 'B');
graph2.addEdge('B', 'C');
graph2.addEdge('C', 'A'); // Cycle

console.log('Has cycle:', graph2.hasCycleWithColors()); // true
```

### **Approach 3: Find Cycle and Return Path**
```javascript
/**
 * Detect cycle and return the actual cycle path
 */
function findCycle(graph) {
  const visited = new Set();
  const recStack = new Set();
  const parent = {};
  let cycleStart = null;
  let cycleEnd = null;
  
  function dfs(vertex) {
    visited.add(vertex);
    recStack.add(vertex);
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (!visited.has(neighbor)) {
        parent[neighbor] = vertex;
        
        if (dfs(neighbor)) {
          return true;
        }
      } else if (recStack.has(neighbor)) {
        cycleStart = neighbor;
        cycleEnd = vertex;
        return true;
      }
    }
    
    recStack.delete(vertex);
    return false;
  }
  
  // Find cycle
  for (const vertex in graph.adjacencyList) {
    if (!visited.has(vertex)) {
      if (dfs(vertex)) {
        break;
      }
    }
  }
  
  if (!cycleStart) {
    return null; // No cycle
  }
  
  // Reconstruct cycle path
  const cycle = [cycleStart];
  let current = cycleEnd;
  
  while (current !== cycleStart) {
    cycle.push(current);
    current = parent[current];
  }
  
  cycle.push(cycleStart); // Complete the cycle
  cycle.reverse();
  
  return cycle;
}

// Test
console.log('\n=== Find Cycle Path ===');

const graph3 = new DirectedGraph();
['A', 'B', 'C', 'D', 'E'].forEach(v => graph3.addVertex(v));
graph3.addEdge('A', 'B');
graph3.addEdge('B', 'C');
graph3.addEdge('C', 'D');
graph3.addEdge('D', 'B'); // Cycle: B -> C -> D -> B
graph3.addEdge('D', 'E');

const cycle = findCycle(graph3);
console.log('Cycle found:', cycle); // ['B', 'C', 'D', 'B']
```

### **Approach 4: Kahn's Algorithm (Topological Sort)**
```javascript
/**
 * Detect cycle using Kahn's algorithm (topological sort)
 * If topological sort completes, no cycle exists
 */
function hasCycleKahn(graph) {
  const inDegree = {};
  const queue = [];
  let processedCount = 0;
  
  // Initialize in-degrees
  for (const vertex in graph.adjacencyList) {
    inDegree[vertex] = 0;
  }
  
  // Calculate in-degrees
  for (const vertex in graph.adjacencyList) {
    for (const neighbor of graph.adjacencyList[vertex]) {
      inDegree[neighbor]++;
    }
  }
  
  // Add vertices with in-degree 0 to queue
  for (const vertex in inDegree) {
    if (inDegree[vertex] === 0) {
      queue.push(vertex);
    }
  }
  
  // Process vertices
  while (queue.length > 0) {
    const vertex = queue.shift();
    processedCount++;
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      inDegree[neighbor]--;
      
      if (inDegree[neighbor] === 0) {
        queue.push(neighbor);
      }
    }
  }
  
  // If not all vertices processed, cycle exists
  const totalVertices = Object.keys(graph.adjacencyList).length;
  return processedCount !== totalVertices;
}

// Test
console.log('\n=== Kahn\'s Algorithm ===');

const graph4 = new DirectedGraph();
['A', 'B', 'C', 'D'].forEach(v => graph4.addVertex(v));
graph4.addEdge('A', 'B');
graph4.addEdge('B', 'C');
graph4.addEdge('C', 'D');

console.log('Has cycle (no):', hasCycleKahn(graph4)); // false

graph4.addEdge('D', 'A'); // Create cycle
console.log('Has cycle (yes):', hasCycleKahn(graph4)); // true
```

### **Approach 5: Union-Find for DAG Verification**
```javascript
/**
 * Union-Find to detect cycles (works for adding edges one by one)
 */
class UnionFind {
  constructor(vertices) {
    this.parent = {};
    this.rank = {};
    
    for (const v of vertices) {
      this.parent[v] = v;
      this.rank[v] = 0;
    }
  }
  
  find(x) {
    if (this.parent[x] !== x) {
      this.parent[x] = this.find(this.parent[x]); // Path compression
    }
    return this.parent[x];
  }
  
  union(x, y) {
    const rootX = this.find(x);
    const rootY = this.find(y);
    
    if (rootX === rootY) {
      return false; // Already in same set
    }
    
    // Union by rank
    if (this.rank[rootX] < this.rank[rootY]) {
      this.parent[rootX] = rootY;
    } else if (this.rank[rootX] > this.rank[rootY]) {
      this.parent[rootY] = rootX;
    } else {
      this.parent[rootY] = rootX;
      this.rank[rootX]++;
    }
    
    return true;
  }
}

// Note: Union-Find only works for undirected graphs or
// directed acyclic graphs being built incrementally
function canAddEdgeWithoutCycle(vertices, edges, newEdge) {
  const uf = new UnionFind(vertices);
  
  // Process existing edges
  for (const [from, to] of edges) {
    uf.union(from, to);
  }
  
  // Check if new edge creates cycle
  const [from, to] = newEdge;
  return uf.find(from) !== uf.find(to);
}

// Test
console.log('\n=== Union-Find ===');

const vertices = ['A', 'B', 'C', 'D'];
const edges = [['A', 'B'], ['B', 'C']];

console.log('Can add A->D:', canAddEdgeWithoutCycle(vertices, edges, ['A', 'D'])); // true
console.log('Can add C->A:', canAddEdgeWithoutCycle(vertices, edges, ['C', 'A'])); // false (creates cycle)
```

### **Bonus: All Cycles Detection**
```javascript
/**
 * Find all cycles in directed graph
 */
function findAllCycles(graph) {
  const visited = new Set();
  const recStack = new Set();
  const path = [];
  const allCycles = [];
  
  function dfs(vertex) {
    visited.add(vertex);
    recStack.add(vertex);
    path.push(vertex);
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (!visited.has(neighbor)) {
        dfs(neighbor);
      } else if (recStack.has(neighbor)) {
        // Found cycle
        const cycleStart = path.indexOf(neighbor);
        const cycle = path.slice(cycleStart);
        cycle.push(neighbor); // Complete the cycle
        allCycles.push([...cycle]);
      }
    }
    
    path.pop();
    recStack.delete(vertex);
  }
  
  for (const vertex in graph.adjacencyList) {
    if (!visited.has(vertex)) {
      dfs(vertex);
    }
  }
  
  return allCycles;
}

// Test
console.log('\n=== All Cycles ===');

const graph5 = new DirectedGraph();
['A', 'B', 'C', 'D', 'E'].forEach(v => graph5.addVertex(v));
graph5.addEdge('A', 'B');
graph5.addEdge('B', 'C');
graph5.addEdge('C', 'A'); // Cycle 1: A->B->C->A
graph5.addEdge('C', 'D');
graph5.addEdge('D', 'E');
graph5.addEdge('E', 'C'); // Cycle 2: C->D->E->C

console.log('All cycles:', findAllCycles(graph5));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical cycle detection applications
 */

// 1. Dependency Resolution
console.log('\n=== Dependency Resolution ===');

class DependencyManager {
  constructor() {
    this.graph = new DirectedGraph();
  }
  
  addPackage(pkg) {
    this.graph.addVertex(pkg);
  }
  
  addDependency(pkg, dependency) {
    this.graph.addEdge(pkg, dependency);
  }
  
  hasCircularDependency() {
    return this.graph.hasCycle();
  }
  
  findCircularDependency() {
    return findCycle(this.graph);
  }
}

const deps = new DependencyManager();

['app', 'db', 'auth', 'logger'].forEach(pkg => deps.addPackage(pkg));
deps.addDependency('app', 'db');
deps.addDependency('app', 'auth');
deps.addDependency('auth', 'logger');
deps.addDependency('db', 'logger');

console.log('Has circular dependency:', deps.hasCircularDependency()); // false

deps.addDependency('logger', 'auth'); // Creates cycle
console.log('After adding logger->auth:', deps.hasCircularDependency()); // true
console.log('Circular dependency:', deps.findCircularDependency());

// 2. Deadlock Detection
console.log('\n=== Deadlock Detection ===');

class DeadlockDetector {
  constructor() {
    this.waitGraph = new DirectedGraph();
  }
  
  addProcess(process) {
    this.waitGraph.addVertex(process);
  }
  
  processWaitsFor(process1, process2) {
    this.waitGraph.addEdge(process1, process2);
  }
  
  detectDeadlock() {
    return this.waitGraph.hasCycle();
  }
  
  getDeadlockedProcesses() {
    return findCycle(this.waitGraph);
  }
}

const detector = new DeadlockDetector();

['P1', 'P2', 'P3'].forEach(p => detector.addProcess(p));
detector.processWaitsFor('P1', 'P2'); // P1 waits for P2's resource
detector.processWaitsFor('P2', 'P3'); // P2 waits for P3's resource

console.log('Deadlock:', detector.detectDeadlock()); // false

detector.processWaitsFor('P3', 'P1'); // P3 waits for P1's resource - deadlock!
console.log('After P3->P1:', detector.detectDeadlock()); // true
console.log('Deadlocked processes:', detector.getDeadlockedProcesses());

// 3. Build System
console.log('\n=== Build System ===');

class BuildSystem {
  constructor() {
    this.targets = new DirectedGraph();
  }
  
  addTarget(target) {
    this.targets.addVertex(target);
  }
  
  addDependency(target, dependency) {
    this.targets.addEdge(target, dependency);
  }
  
  canBuild() {
    if (this.targets.hasCycle()) {
      return { success: false, error: 'Circular dependency detected' };
    }
    return { success: true };
  }
}

const build = new BuildSystem();

['main.o', 'utils.o', 'lib.a'].forEach(t => build.addTarget(t));
build.addDependency('main.o', 'utils.o');
build.addDependency('utils.o', 'lib.a');

console.log('Can build:', build.canBuild());

build.addDependency('lib.a', 'main.o'); // Circular dependency
console.log('After circular dep:', build.canBuild());

// 4. Course Prerequisites
console.log('\n=== Course Prerequisites ===');

class CourseScheduler {
  constructor() {
    this.prerequisites = new DirectedGraph();
  }
  
  addCourse(course) {
    this.prerequisites.addVertex(course);
  }
  
  addPrerequisite(course, prerequisite) {
    this.prerequisites.addEdge(course, prerequisite);
  }
  
  canFinishAllCourses() {
    return !this.prerequisites.hasCycle();
  }
  
  findProblematicCourses() {
    if (!this.canFinishAllCourses()) {
      return findCycle(this.prerequisites);
    }
    return null;
  }
}

const scheduler = new CourseScheduler();

['CS101', 'CS201', 'CS301', 'MATH101'].forEach(c => scheduler.addCourse(c));
scheduler.addPrerequisite('CS201', 'CS101');
scheduler.addPrerequisite('CS301', 'CS201');
scheduler.addPrerequisite('CS301', 'MATH101');

console.log('Can finish all courses:', scheduler.canFinishAllCourses()); // true

scheduler.addPrerequisite('CS101', 'CS301'); // Creates impossible situation
console.log('After impossible prereq:', scheduler.canFinishAllCourses()); // false
console.log('Problematic courses:', scheduler.findProblematicCourses());

// 5. Workflow Validation
console.log('\n=== Workflow Validation ===');

class WorkflowValidator {
  constructor() {
    this.workflow = new DirectedGraph();
  }
  
  addTask(task) {
    this.workflow.addVertex(task);
  }
  
  addTransition(from, to) {
    this.workflow.addEdge(from, to);
  }
  
  validateWorkflow() {
    if (this.workflow.hasCycle()) {
      return {
        valid: false,
        error: 'Workflow contains infinite loop',
        cycle: findCycle(this.workflow)
      };
    }
    
    return { valid: true };
  }
}

const workflow = new WorkflowValidator();

['Start', 'Review', 'Approve', 'Deploy', 'End'].forEach(t => workflow.addTask(t));
workflow.addTransition('Start', 'Review');
workflow.addTransition('Review', 'Approve');
workflow.addTransition('Approve', 'Deploy');
workflow.addTransition('Deploy', 'End');

console.log('Workflow valid:', workflow.validateWorkflow());

workflow.addTransition('Deploy', 'Review'); // Can retry after deploy
console.log('After retry loop:', workflow.validateWorkflow());
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Single vertex with self-loop
const selfLoop = new DirectedGraph();
selfLoop.addVertex('A');
selfLoop.addEdge('A', 'A');
console.log('Self-loop:', selfLoop.hasCycle()); // true

// Two vertices with mutual edges
const mutual = new DirectedGraph();
['A', 'B'].forEach(v => mutual.addVertex(v));
mutual.addEdge('A', 'B');
mutual.addEdge('B', 'A');
console.log('Mutual edges:', mutual.hasCycle()); // true

// Disconnected components with one cycle
const disconnected = new DirectedGraph();
['A', 'B', 'C', 'D', 'E'].forEach(v => disconnected.addVertex(v));
disconnected.addEdge('A', 'B');
disconnected.addEdge('B', 'A'); // Cycle in component 1
disconnected.addEdge('C', 'D'); // Component 2, no cycle
console.log('Disconnected with cycle:', disconnected.hasCycle()); // true

// Empty graph
const empty = new DirectedGraph();
console.log('Empty graph:', empty.hasCycle()); // false

// Single vertex, no edges
const singleVertex = new DirectedGraph();
singleVertex.addVertex('A');
console.log('Single vertex:', singleVertex.hasCycle()); // false

// Long chain (no cycle)
const chain = new DirectedGraph();
for (let i = 0; i < 100; i++) {
  chain.addVertex(i);
  if (i > 0) {
    chain.addEdge(i - 1, i);
  }
}
console.log('Long chain (no cycle):', chain.hasCycle()); // false

// Complex graph with multiple paths
const complex = new DirectedGraph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => complex.addVertex(v));
complex.addEdge('A', 'B');
complex.addEdge('A', 'C');
complex.addEdge('B', 'D');
complex.addEdge('C', 'D');
complex.addEdge('D', 'E');
complex.addEdge('E', 'F');
console.log('Complex DAG:', complex.hasCycle()); // false

complex.addEdge('F', 'A'); // Add cycle
console.log('Complex with cycle:', complex.hasCycle()); // true
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Algorithm Comparison:
┌──────────────────────┬─────────────┬─────────────┬────────────┐
│ Algorithm            │ Time        │ Space       │ Best For   │
├──────────────────────┼─────────────┼─────────────┼────────────┤
│ DFS (recursion)      │ O(V + E)    │ O(V)        │ General    │
│ DFS (iterative)      │ O(V + E)    │ O(V)        │ Deep graphs│
│ Three-color DFS      │ O(V + E)    │ O(V)        │ Cleaner    │
│ Kahn's algorithm     │ O(V + E)    │ O(V)        │ Topo sort  │
│ Union-Find           │ O(E α(V))   │ O(V)        │ Incremental│
└──────────────────────┴─────────────┴─────────────┴────────────┘

Where:
  V = number of vertices
  E = number of edges
  α(V) = inverse Ackermann (nearly constant)

Key Differences:
• DFS with recursion stack: Most common, intuitive
• Three-color: Cleaner state tracking
• Kahn's: Bonus of topological order if no cycle
• Union-Find: Good for dynamic graph construction

Optimization:
• Early termination when cycle found
• Track visited to avoid redundant work
• Path compression in Union-Find
• Iterative DFS for very deep graphs (avoid stack overflow)

Space Optimization:
• Reuse visited set across components
• In-place color marking if vertices are numbered
• Bit vectors for large graphs
`);
```

**Interview Tips:**
- Directed graph cycle: use DFS with recursion stack tracking
- Key insight: back edge to vertex in current recursion path = cycle
- Recursion stack: tracks current DFS path, different from visited
- Algorithm: DFS, mark vertex as "in progress", check neighbors, unmark when done
- If neighbor is "in progress" (in recursion stack), cycle found
- Time: O(V + E) visit each vertex once, check each edge once
- Space: O(V) for visited and recursion stack sets
- Three colors: white (unvisited), gray (processing), black (done)
- Gray vertex: currently in DFS path (recursion stack)
- Black vertex: fully explored, no cycle through it
- Applications: dependency resolution, deadlock detection, build systems, prerequisites
- vs undirected: undirected uses parent tracking, directed uses recursion stack
- Kahn's algorithm: alternative using topological sort, if all vertices processed then DAG
- Return cycle: track parent pointers, reconstruct path when cycle found
- All cycles: continue DFS after finding each cycle, collect all
- Edge cases: self-loop (cycle), mutual edges, disconnected components, empty graph
- Disconnected: check all components by iterating unvisited vertices
- Union-Find: works for incremental edge addition in undirected/DAG scenarios
- Optimization: early termination, avoid processing same vertices multiple times
- Clarify: need cycle path or just detection? all cycles or first? graph properties?
- Follow-ups: find shortest cycle, count cycles, remove minimum edges to break cycles

</details>

103. Implement topological sort

<details>
<summary><b>Solution</b></summary>

### **Approach 1: DFS with Post-Order**
```javascript
/**
 * Topological sort using DFS
 * Time Complexity: O(V + E)
 * Space Complexity: O(V)
 */
class DirectedGraph {
  constructor() {
    this.adjacencyList = {};
  }
  
  addVertex(vertex) {
    if (!this.adjacencyList[vertex]) {
      this.adjacencyList[vertex] = [];
    }
  }
  
  addEdge(from, to) {
    this.adjacencyList[from].push(to);
  }
  
  topologicalSort() {
    const visited = new Set();
    const stack = [];
    
    const dfs = (vertex) => {
      visited.add(vertex);
      
      for (const neighbor of this.adjacencyList[vertex]) {
        if (!visited.has(neighbor)) {
          dfs(neighbor);
        }
      }
      
      // Add to result after visiting all descendants
      stack.push(vertex);
    };
    
    // Visit all vertices
    for (const vertex in this.adjacencyList) {
      if (!visited.has(vertex)) {
        dfs(vertex);
      }
    }
    
    // Reverse to get correct order
    return stack.reverse();
  }
}

// Test
console.log('=== Topological Sort (DFS) ===');

const graph = new DirectedGraph();

// Example: course prerequisites
// CS301 -> CS201 -> CS101
// CS301 -> MATH101
['CS101', 'CS201', 'CS301', 'MATH101'].forEach(v => graph.addVertex(v));
graph.addEdge('CS301', 'CS201');
graph.addEdge('CS201', 'CS101');
graph.addEdge('CS301', 'MATH101');

console.log('Topological order:', graph.topologicalSort());
// ['CS101', 'MATH101', 'CS201', 'CS301'] or ['MATH101', 'CS101', 'CS201', 'CS301']
```

### **Approach 2: Kahn's Algorithm (BFS)**
```javascript
/**
 * Kahn's algorithm using BFS and in-degrees
 */
class KahnGraph extends DirectedGraph {
  topologicalSortKahn() {
    const inDegree = {};
    const result = [];
    const queue = [];
    
    // Initialize in-degrees
    for (const vertex in this.adjacencyList) {
      inDegree[vertex] = 0;
    }
    
    // Calculate in-degrees
    for (const vertex in this.adjacencyList) {
      for (const neighbor of this.adjacencyList[vertex]) {
        inDegree[neighbor] = (inDegree[neighbor] || 0) + 1;
      }
    }
    
    // Add vertices with in-degree 0 to queue
    for (const vertex in inDegree) {
      if (inDegree[vertex] === 0) {
        queue.push(vertex);
      }
    }
    
    // Process vertices
    while (queue.length > 0) {
      const vertex = queue.shift();
      result.push(vertex);
      
      // Reduce in-degree for neighbors
      for (const neighbor of this.adjacencyList[vertex]) {
        inDegree[neighbor]--;
        
        if (inDegree[neighbor] === 0) {
          queue.push(neighbor);
        }
      }
    }
    
    // Check for cycle
    if (result.length !== Object.keys(this.adjacencyList).length) {
      throw new Error('Graph contains a cycle - topological sort not possible');
    }
    
    return result;
  }
}

// Test
console.log('\n=== Kahn\'s Algorithm ===');

const graph2 = new KahnGraph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph2.addVertex(v));
graph2.addEdge('A', 'C');
graph2.addEdge('A', 'D');
graph2.addEdge('B', 'D');
graph2.addEdge('B', 'E');
graph2.addEdge('C', 'F');
graph2.addEdge('D', 'F');
graph2.addEdge('E', 'F');

console.log('Kahn\'s topological order:', graph2.topologicalSortKahn());
```

### **Approach 3: All Possible Topological Orders**
```javascript
/**
 * Find all valid topological orderings
 */
function allTopologicalSorts(graph) {
  const inDegree = {};
  const allOrders = [];
  
  // Calculate in-degrees
  for (const vertex in graph.adjacencyList) {
    inDegree[vertex] = 0;
  }
  
  for (const vertex in graph.adjacencyList) {
    for (const neighbor of graph.adjacencyList[vertex]) {
      inDegree[neighbor] = (inDegree[neighbor] || 0) + 1;
    }
  }
  
  function backtrack(current, remaining) {
    if (remaining === 0) {
      allOrders.push([...current]);
      return;
    }
    
    // Try all vertices with in-degree 0
    for (const vertex in inDegree) {
      if (inDegree[vertex] === 0) {
        // Choose vertex
        current.push(vertex);
        
        // Temporarily reduce in-degrees
        const neighbors = graph.adjacencyList[vertex];
        for (const neighbor of neighbors) {
          inDegree[neighbor]--;
        }
        
        inDegree[vertex] = -1; // Mark as used
        
        // Recurse
        backtrack(current, remaining - 1);
        
        // Backtrack
        inDegree[vertex] = 0;
        for (const neighbor of neighbors) {
          inDegree[neighbor]++;
        }
        current.pop();
      }
    }
  }
  
  const vertexCount = Object.keys(graph.adjacencyList).length;
  backtrack([], vertexCount);
  
  return allOrders;
}

// Test
console.log('\n=== All Topological Orders ===');

const graph3 = new DirectedGraph();
['A', 'B', 'C', 'D'].forEach(v => graph3.addVertex(v));
graph3.addEdge('A', 'C');
graph3.addEdge('B', 'C');
graph3.addEdge('C', 'D');

console.log('All valid orders:', allTopologicalSorts(graph3));
// [['A', 'B', 'C', 'D'], ['B', 'A', 'C', 'D']]
```

### **Approach 4: Topological Sort with Levels**
```javascript
/**
 * Topological sort grouped by levels (parallel processing)
 */
function topologicalSortWithLevels(graph) {
  const inDegree = {};
  const levels = [];
  
  // Initialize in-degrees
  for (const vertex in graph.adjacencyList) {
    inDegree[vertex] = 0;
  }
  
  for (const vertex in graph.adjacencyList) {
    for (const neighbor of graph.adjacencyList[vertex]) {
      inDegree[neighbor] = (inDegree[neighbor] || 0) + 1;
    }
  }
  
  let currentLevel = [];
  
  // Find vertices with in-degree 0
  for (const vertex in inDegree) {
    if (inDegree[vertex] === 0) {
      currentLevel.push(vertex);
    }
  }
  
  while (currentLevel.length > 0) {
    levels.push([...currentLevel]);
    const nextLevel = [];
    
    for (const vertex of currentLevel) {
      for (const neighbor of graph.adjacencyList[vertex]) {
        inDegree[neighbor]--;
        
        if (inDegree[neighbor] === 0) {
          nextLevel.push(neighbor);
        }
      }
    }
    
    currentLevel = nextLevel;
  }
  
  return levels;
}

// Test
console.log('\n=== Topological Sort with Levels ===');

const graph4 = new DirectedGraph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph4.addVertex(v));
graph4.addEdge('A', 'C');
graph4.addEdge('A', 'D');
graph4.addEdge('B', 'D');
graph4.addEdge('C', 'E');
graph4.addEdge('D', 'E');
graph4.addEdge('D', 'F');
graph4.addEdge('E', 'F');

const levels = topologicalSortWithLevels(graph4);
console.log('Levels:', levels);
// [[A, B], [C, D], [E], [F]]
// Level 0: A, B can run in parallel
// Level 1: C, D can run after A, B
// etc.
```

### **Approach 5: Lexicographically Smallest Topological Sort**
```javascript
/**
 * Find lexicographically smallest topological order
 * Uses priority queue (min heap) instead of regular queue
 */
class MinHeap {
  constructor(compareFn = (a, b) => a.localeCompare(b)) {
    this.heap = [];
    this.compare = compareFn;
  }
  
  push(val) {
    this.heap.push(val);
    this.bubbleUp(this.heap.length - 1);
  }
  
  pop() {
    if (this.heap.length === 0) return null;
    if (this.heap.length === 1) return this.heap.pop();
    
    const min = this.heap[0];
    this.heap[0] = this.heap.pop();
    this.bubbleDown(0);
    return min;
  }
  
  bubbleUp(index) {
    while (index > 0) {
      const parent = Math.floor((index - 1) / 2);
      if (this.compare(this.heap[index], this.heap[parent]) >= 0) break;
      [this.heap[index], this.heap[parent]] = [this.heap[parent], this.heap[index]];
      index = parent;
    }
  }
  
  bubbleDown(index) {
    while (true) {
      let smallest = index;
      const left = 2 * index + 1;
      const right = 2 * index + 2;
      
      if (left < this.heap.length && this.compare(this.heap[left], this.heap[smallest]) < 0) {
        smallest = left;
      }
      
      if (right < this.heap.length && this.compare(this.heap[right], this.heap[smallest]) < 0) {
        smallest = right;
      }
      
      if (smallest === index) break;
      
      [this.heap[index], this.heap[smallest]] = [this.heap[smallest], this.heap[index]];
      index = smallest;
    }
  }
  
  isEmpty() {
    return this.heap.length === 0;
  }
}

function lexicographicalTopologicalSort(graph) {
  const inDegree = {};
  const result = [];
  const pq = new MinHeap();
  
  // Initialize and calculate in-degrees
  for (const vertex in graph.adjacencyList) {
    inDegree[vertex] = 0;
  }
  
  for (const vertex in graph.adjacencyList) {
    for (const neighbor of graph.adjacencyList[vertex]) {
      inDegree[neighbor] = (inDegree[neighbor] || 0) + 1;
    }
  }
  
  // Add vertices with in-degree 0
  for (const vertex in inDegree) {
    if (inDegree[vertex] === 0) {
      pq.push(vertex);
    }
  }
  
  while (!pq.isEmpty()) {
    const vertex = pq.pop();
    result.push(vertex);
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      inDegree[neighbor]--;
      
      if (inDegree[neighbor] === 0) {
        pq.push(neighbor);
      }
    }
  }
  
  return result;
}

// Test
console.log('\n=== Lexicographical Sort ===');

const graph5 = new DirectedGraph();
['C', 'A', 'B', 'D'].forEach(v => graph5.addVertex(v));
graph5.addEdge('C', 'D');
graph5.addEdge('A', 'D');
graph5.addEdge('B', 'D');

console.log('Lexicographical order:', lexicographicalTopologicalSort(graph5));
// ['A', 'B', 'C', 'D'] (alphabetically smallest)
```

### **Real-World Use Cases**
```javascript
/**
 * Practical topological sort applications
 */

// 1. Course Scheduling
console.log('\n=== Course Scheduling ===');

class CourseScheduler {
  constructor() {
    this.graph = new KahnGraph();
  }
  
  addCourse(course) {
    this.graph.addVertex(course);
  }
  
  addPrerequisite(course, prerequisite) {
    // prerequisite must be taken before course
    this.graph.addEdge(prerequisite, course);
  }
  
  getCourseOrder() {
    try {
      return this.graph.topologicalSortKahn();
    } catch (error) {
      return { error: 'Circular prerequisites detected' };
    }
  }
  
  getCourseScheduleByLevel() {
    return topologicalSortWithLevels(this.graph);
  }
}

const courses = new CourseScheduler();

['CS101', 'CS201', 'CS301', 'MATH101', 'MATH201'].forEach(c => 
  courses.addCourse(c)
);

courses.addPrerequisite('CS201', 'CS101');
courses.addPrerequisite('CS301', 'CS201');
courses.addPrerequisite('MATH201', 'MATH101');
courses.addPrerequisite('CS301', 'MATH201');

console.log('Course order:', courses.getCourseOrder());
console.log('Semester plan:', courses.getCourseScheduleByLevel());

// 2. Build System
console.log('\n=== Build System ===');

class BuildSystem {
  constructor() {
    this.targets = new DirectedGraph();
  }
  
  addTarget(target) {
    this.targets.addVertex(target);
  }
  
  addDependency(target, dependency) {
    // dependency must be built before target
    this.targets.addEdge(dependency, target);
  }
  
  getBuildOrder() {
    return this.targets.topologicalSort();
  }
  
  getParallelBuildStages() {
    return topologicalSortWithLevels(this.targets);
  }
}

const build = new BuildSystem();

['main.o', 'utils.o', 'lib.a', 'app'].forEach(t => build.addTarget(t));
build.addDependency('main.o', 'utils.o');
build.addDependency('main.o', 'lib.a');
build.addDependency('app', 'main.o');

console.log('Build order:', build.getBuildOrder());
console.log('Parallel stages:', build.getParallelBuildStages());

// 3. Task Scheduling
console.log('\n=== Task Scheduling ===');

class TaskScheduler {
  constructor() {
    this.tasks = new DirectedGraph();
    this.durations = {};
  }
  
  addTask(task, duration = 1) {
    this.tasks.addVertex(task);
    this.durations[task] = duration;
  }
  
  addDependency(task, dependency) {
    this.tasks.addEdge(dependency, task);
  }
  
  getExecutionOrder() {
    return this.tasks.topologicalSort();
  }
  
  getCriticalPath() {
    const order = this.tasks.topologicalSort();
    const earliestStart = {};
    const latestStart = {};
    
    // Forward pass: earliest start times
    for (const task of order) {
      let maxPredecessorEnd = 0;
      
      // Find max end time of dependencies
      for (const vertex in this.tasks.adjacencyList) {
        if (this.tasks.adjacencyList[vertex].includes(task)) {
          const predEnd = earliestStart[vertex] + this.durations[vertex];
          maxPredecessorEnd = Math.max(maxPredecessorEnd, predEnd);
        }
      }
      
      earliestStart[task] = maxPredecessorEnd;
    }
    
    // Project duration
    const projectDuration = Math.max(
      ...order.map(task => earliestStart[task] + this.durations[task])
    );
    
    console.log('Project duration:', projectDuration);
    console.log('Earliest start times:', earliestStart);
    
    return { earliestStart, projectDuration };
  }
}

const scheduler = new TaskScheduler();

scheduler.addTask('Design', 3);
scheduler.addTask('Code', 5);
scheduler.addTask('Test', 2);
scheduler.addTask('Deploy', 1);

scheduler.addDependency('Code', 'Design');
scheduler.addDependency('Test', 'Code');
scheduler.addDependency('Deploy', 'Test');

console.log('Execution order:', scheduler.getExecutionOrder());
scheduler.getCriticalPath();

// 4. Package Manager
console.log('\n=== Package Manager ===');

class PackageManager {
  constructor() {
    this.packages = new DirectedGraph();
  }
  
  addPackage(pkg) {
    this.packages.addVertex(pkg);
  }
  
  addDependency(pkg, dependency) {
    this.packages.addEdge(dependency, pkg);
  }
  
  getInstallOrder(targetPackage) {
    // Get all reachable packages using DFS
    const reachable = new Set();
    
    const dfs = (vertex) => {
      if (reachable.has(vertex)) return;
      reachable.add(vertex);
      
      for (const dep of this.packages.adjacencyList[vertex]) {
        dfs(dep);
      }
    };
    
    // Reverse edges for DFS
    const reversed = {};
    for (const vertex in this.packages.adjacencyList) {
      reversed[vertex] = [];
    }
    
    for (const vertex in this.packages.adjacencyList) {
      for (const neighbor of this.packages.adjacencyList[vertex]) {
        reversed[neighbor].push(vertex);
      }
    }
    
    // DFS from target
    const dfsFromTarget = (vertex) => {
      if (reachable.has(vertex)) return;
      reachable.add(vertex);
      
      for (const dep of reversed[vertex]) {
        dfsFromTarget(dep);
      }
    };
    
    dfsFromTarget(targetPackage);
    
    // Get topological order of reachable packages
    const fullOrder = this.packages.topologicalSort();
    return fullOrder.filter(pkg => reachable.has(pkg));
  }
}

const pm = new PackageManager();

['express', 'body-parser', 'cookie-parser', 'morgan'].forEach(p => 
  pm.addPackage(p)
);

pm.addDependency('express', 'body-parser');
pm.addDependency('express', 'cookie-parser');

console.log('Install order for express:', pm.getInstallOrder('express'));

// 5. Data Pipeline
console.log('\n=== Data Pipeline ===');

class DataPipeline {
  constructor() {
    this.stages = new DirectedGraph();
  }
  
  addStage(stage) {
    this.stages.addVertex(stage);
  }
  
  addDataFlow(from, to) {
    this.stages.addEdge(from, to);
  }
  
  getExecutionPlan() {
    return topologicalSortWithLevels(this.stages);
  }
}

const pipeline = new DataPipeline();

['Extract', 'Transform1', 'Transform2', 'Validate', 'Load'].forEach(s => 
  pipeline.addStage(s)
);

pipeline.addDataFlow('Extract', 'Transform1');
pipeline.addDataFlow('Extract', 'Transform2');
pipeline.addDataFlow('Transform1', 'Validate');
pipeline.addDataFlow('Transform2', 'Validate');
pipeline.addDataFlow('Validate', 'Load');

console.log('Pipeline stages (parallel):', pipeline.getExecutionPlan());
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty graph
const empty = new DirectedGraph();
console.log('Empty graph:', empty.topologicalSort()); // []

// Single vertex
const single = new DirectedGraph();
single.addVertex('A');
console.log('Single vertex:', single.topologicalSort()); // ['A']

// Linear chain
const chain = new DirectedGraph();
['A', 'B', 'C', 'D'].forEach(v => chain.addVertex(v));
chain.addEdge('A', 'B');
chain.addEdge('B', 'C');
chain.addEdge('C', 'D');
console.log('Linear chain:', chain.topologicalSort()); // ['A', 'B', 'C', 'D']

// Complete DAG (all possible edges)
const complete = new DirectedGraph();
['A', 'B', 'C'].forEach(v => complete.addVertex(v));
complete.addEdge('A', 'B');
complete.addEdge('A', 'C');
complete.addEdge('B', 'C');
console.log('Complete DAG:', complete.topologicalSort()); // ['A', 'B', 'C']

// Disconnected components
const disconnected = new DirectedGraph();
['A', 'B', 'C', 'D'].forEach(v => disconnected.addVertex(v));
disconnected.addEdge('A', 'B');
disconnected.addEdge('C', 'D');
console.log('Disconnected:', disconnected.topologicalSort());
// Multiple valid orders

// Graph with cycle (should fail with Kahn's)
const cyclic = new KahnGraph();
['A', 'B', 'C'].forEach(v => cyclic.addVertex(v));
cyclic.addEdge('A', 'B');
cyclic.addEdge('B', 'C');
cyclic.addEdge('C', 'A');

try {
  cyclic.topologicalSortKahn();
} catch (error) {
  console.log('Cyclic graph error:', error.message);
}
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Algorithm Comparison:
┌──────────────────┬─────────────┬─────────────┬────────────────┐
│ Algorithm        │ Time        │ Space       │ Advantages     │
├──────────────────┼─────────────┼─────────────┼────────────────┤
│ DFS              │ O(V + E)    │ O(V)        │ Simple, one    │
│                  │             │             │ valid order    │
├──────────────────┼─────────────┼─────────────┼────────────────┤
│ Kahn's (BFS)     │ O(V + E)    │ O(V)        │ Detects cycles,│
│                  │             │             │ level tracking │
├──────────────────┼─────────────┼─────────────┼────────────────┤
│ All orders       │ O(V! × E)   │ O(V)        │ Find all valid │
│                  │             │             │ orderings      │
├──────────────────┼─────────────┼─────────────┼────────────────┤
│ Lexicographical  │ O((V+E)logV)│ O(V)        │ Smallest order │
└──────────────────┴─────────────┴─────────────┴────────────────┘

When to use:
• DFS: Default choice, simple and efficient
• Kahn's: Need cycle detection or level grouping
• All orders: Count valid schedules, enumerate options
• Lexicographical: Specific ordering requirement

Applications:
• Course scheduling: plan semester, detect prereq cycles
• Build systems: compilation order, parallel builds
• Task scheduling: project management, critical path
• Package managers: installation order, dependency resolution
• Data pipelines: ETL stages, parallel processing
`);
```

**Interview Tips:**
- Topological sort: linear ordering of vertices where u comes before v for every edge u→v
- Only possible for DAG (Directed Acyclic Graph)
- Two main algorithms: DFS (post-order) and Kahn's (BFS with in-degrees)
- DFS approach: run DFS, add vertex to result after visiting all descendants, reverse result
- Kahn's approach: repeatedly remove vertices with in-degree 0, reduce neighbors' in-degrees
- Time: O(V + E) for both approaches
- Space: O(V) for auxiliary data structures
- Post-order: in DFS, add vertex after processing all children (finish time order)
- In-degree: number of incoming edges to a vertex
- Applications: course prerequisites, build order, task scheduling, dependency resolution
- Cycle detection: Kahn's algorithm naturally detects cycles (not all vertices processed)
- Multiple valid orders: most DAGs have multiple topological orderings
- Levels: vertices at same level have no dependencies between them (can run in parallel)
- Critical path: in project scheduling, longest path determines minimum project duration
- Lexicographical: use priority queue instead of regular queue in Kahn's algorithm
- All orders: backtracking to enumerate all valid topological sorts
- Edge cases: empty graph, single vertex, disconnected components, cycles
- DFS vs Kahn's: DFS simpler, Kahn's better for cycle detection and level tracking
- Optimization: early termination if only one valid order needed
- Clarify: need one order or all? detect cycles? level grouping? lexicographical order?
- Follow-ups: find all valid orders, minimum time with parallel execution, longest path

</details>

104. Find all strongly connected components in a graph

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Kosaraju's Algorithm (Two DFS)**
```javascript
/**
 * Find SCCs using Kosaraju's algorithm
 * Time Complexity: O(V + E)
 * Space Complexity: O(V)
 */
class DirectedGraph {
  constructor() {
    this.adjacencyList = {};
  }
  
  addVertex(vertex) {
    if (!this.adjacencyList[vertex]) {
      this.adjacencyList[vertex] = [];
    }
  }
  
  addEdge(from, to) {
    this.adjacencyList[from].push(to);
  }
  
  // Kosaraju's Algorithm
  findStronglyConnectedComponents() {
    const visited = new Set();
    const stack = [];
    
    // Step 1: Fill stack with vertices in order of finish time (DFS)
    const dfs1 = (vertex) => {
      visited.add(vertex);
      
      for (const neighbor of this.adjacencyList[vertex]) {
        if (!visited.has(neighbor)) {
          dfs1(neighbor);
        }
      }
      
      stack.push(vertex); // Add after visiting all descendants
    };
    
    // Run DFS on all unvisited vertices
    for (const vertex in this.adjacencyList) {
      if (!visited.has(vertex)) {
        dfs1(vertex);
      }
    }
    
    // Step 2: Create transpose (reverse) graph
    const transpose = {};
    for (const vertex in this.adjacencyList) {
      transpose[vertex] = [];
    }
    
    for (const vertex in this.adjacencyList) {
      for (const neighbor of this.adjacencyList[vertex]) {
        transpose[neighbor].push(vertex);
      }
    }
    
    // Step 3: DFS on transpose in order from stack
    visited.clear();
    const sccs = [];
    
    const dfs2 = (vertex, component) => {
      visited.add(vertex);
      component.push(vertex);
      
      for (const neighbor of transpose[vertex]) {
        if (!visited.has(neighbor)) {
          dfs2(neighbor, component);
        }
      }
    };
    
    while (stack.length > 0) {
      const vertex = stack.pop();
      
      if (!visited.has(vertex)) {
        const component = [];
        dfs2(vertex, component);
        sccs.push(component);
      }
    }
    
    return sccs;
  }
}

// Test
console.log('=== Kosaraju\'s Algorithm ===');

const graph = new DirectedGraph();
['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H'].forEach(v => graph.addVertex(v));

// Create SCCs: {A, B, C}, {D, E}, {F, G}, {H}
graph.addEdge('A', 'B');
graph.addEdge('B', 'C');
graph.addEdge('C', 'A'); // SCC 1

graph.addEdge('B', 'D');
graph.addEdge('D', 'E');
graph.addEdge('E', 'D'); // SCC 2

graph.addEdge('C', 'F');
graph.addEdge('F', 'G');
graph.addEdge('G', 'F'); // SCC 3

graph.addEdge('E', 'H'); // SCC 4 (single node)

const sccs = graph.findStronglyConnectedComponents();
console.log('SCCs:', sccs);
```

### **Approach 2: Tarjan's Algorithm (Single DFS)**
```javascript
/**
 * Find SCCs using Tarjan's algorithm with single DFS
 * More efficient as it only does one DFS pass
 */
class TarjanGraph extends DirectedGraph {
  findSCCsTarjan() {
    let index = 0;
    const indices = {};
    const lowLinks = {};
    const onStack = new Set();
    const stack = [];
    const sccs = [];
    
    const strongConnect = (vertex) => {
      // Set the depth index for this vertex
      indices[vertex] = index;
      lowLinks[vertex] = index;
      index++;
      
      stack.push(vertex);
      onStack.add(vertex);
      
      // Consider successors
      for (const neighbor of this.adjacencyList[vertex]) {
        if (indices[neighbor] === undefined) {
          // Successor not yet visited; recurse
          strongConnect(neighbor);
          lowLinks[vertex] = Math.min(lowLinks[vertex], lowLinks[neighbor]);
        } else if (onStack.has(neighbor)) {
          // Successor is on stack (part of current SCC)
          lowLinks[vertex] = Math.min(lowLinks[vertex], indices[neighbor]);
        }
      }
      
      // If vertex is a root node, pop the stack to find SCC
      if (lowLinks[vertex] === indices[vertex]) {
        const scc = [];
        let w;
        
        do {
          w = stack.pop();
          onStack.delete(w);
          scc.push(w);
        } while (w !== vertex);
        
        sccs.push(scc);
      }
    };
    
    // Run for all unvisited vertices
    for (const vertex in this.adjacencyList) {
      if (indices[vertex] === undefined) {
        strongConnect(vertex);
      }
    }
    
    return sccs;
  }
}

// Test
console.log('\n=== Tarjan\'s Algorithm ===');

const graph2 = new TarjanGraph();
['0', '1', '2', '3', '4', '5', '6', '7'].forEach(v => graph2.addVertex(v));

graph2.addEdge('0', '1');
graph2.addEdge('1', '2');
graph2.addEdge('2', '0'); // SCC: 0, 1, 2

graph2.addEdge('1', '3');
graph2.addEdge('3', '4');
graph2.addEdge('4', '5');
graph2.addEdge('5', '3'); // SCC: 3, 4, 5

graph2.addEdge('4', '6');
graph2.addEdge('6', '7'); // SCC: 6, SCC: 7

const sccs2 = graph2.findSCCsTarjan();
console.log('SCCs (Tarjan):', sccs2);
```

### **Approach 3: Path-Based Algorithm**
```javascript
/**
 * Path-based strong component algorithm
 * Alternative to Tarjan's with two stacks
 */
function findSCCsPathBased(graph) {
  let preorderCounter = 0;
  const preorder = {};
  const assigned = {};
  const stack = [];
  const boundaryStack = [];
  const sccs = [];
  
  function dfs(vertex) {
    preorder[vertex] = preorderCounter++;
    stack.push(vertex);
    boundaryStack.push(vertex);
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      if (preorder[neighbor] === undefined) {
        dfs(neighbor);
      } else if (assigned[neighbor] === undefined) {
        // Remove vertices with higher preorder than neighbor
        while (preorder[boundaryStack[boundaryStack.length - 1]] > preorder[neighbor]) {
          boundaryStack.pop();
        }
      }
    }
    
    // Check if vertex is boundary
    if (boundaryStack[boundaryStack.length - 1] === vertex) {
      boundaryStack.pop();
      const scc = [];
      let w;
      
      do {
        w = stack.pop();
        assigned[w] = true;
        scc.push(w);
      } while (w !== vertex);
      
      sccs.push(scc);
    }
  }
  
  for (const vertex in graph.adjacencyList) {
    if (preorder[vertex] === undefined) {
      dfs(vertex);
    }
  }
  
  return sccs;
}

// Test
console.log('\n=== Path-Based Algorithm ===');

const graph3 = new DirectedGraph();
['A', 'B', 'C', 'D', 'E'].forEach(v => graph3.addVertex(v));

graph3.addEdge('A', 'B');
graph3.addEdge('B', 'C');
graph3.addEdge('C', 'A'); // SCC: A, B, C

graph3.addEdge('B', 'D');
graph3.addEdge('D', 'E');
graph3.addEdge('E', 'D'); // SCC: D, E

console.log('SCCs (Path-based):', findSCCsPathBased(graph3));
```

### **Approach 4: SCC Condensation Graph**
```javascript
/**
 * Create condensation graph (DAG of SCCs)
 */
function createCondensationGraph(graph) {
  const sccs = graph.findStronglyConnectedComponents();
  
  // Map each vertex to its SCC index
  const vertexToSCC = {};
  sccs.forEach((scc, index) => {
    scc.forEach(vertex => {
      vertexToSCC[vertex] = index;
    });
  });
  
  // Build condensation graph
  const condensation = new DirectedGraph();
  
  // Add vertices (SCC indices)
  for (let i = 0; i < sccs.length; i++) {
    condensation.addVertex(i);
  }
  
  // Add edges between different SCCs
  const addedEdges = new Set();
  
  for (const vertex in graph.adjacencyList) {
    const fromSCC = vertexToSCC[vertex];
    
    for (const neighbor of graph.adjacencyList[vertex]) {
      const toSCC = vertexToSCC[neighbor];
      
      if (fromSCC !== toSCC) {
        const edgeKey = `${fromSCC}->${toSCC}`;
        if (!addedEdges.has(edgeKey)) {
          condensation.addEdge(fromSCC, toSCC);
          addedEdges.add(edgeKey);
        }
      }
    }
  }
  
  return { sccs, condensation, vertexToSCC };
}

// Test
console.log('\n=== Condensation Graph ===');

const graph4 = new DirectedGraph();
['A', 'B', 'C', 'D', 'E', 'F'].forEach(v => graph4.addVertex(v));

graph4.addEdge('A', 'B');
graph4.addEdge('B', 'A'); // SCC 0: A, B

graph4.addEdge('B', 'C');
graph4.addEdge('C', 'D');
graph4.addEdge('D', 'C'); // SCC 1: C, D

graph4.addEdge('D', 'E');
graph4.addEdge('E', 'F');
graph4.addEdge('F', 'E'); // SCC 2: E, F

const { sccs: sccs4, condensation, vertexToSCC } = createCondensationGraph(graph4);
console.log('SCCs:', sccs4);
console.log('Vertex to SCC mapping:', vertexToSCC);
console.log('Condensation edges:', condensation.adjacencyList);
```

### **Approach 5: Finding Bridges and Articulation Points**
```javascript
/**
 * Related: Find bridges (edges whose removal disconnects graph)
 * and articulation points (vertices whose removal disconnects)
 */
class BridgeArticulationFinder {
  constructor(graph) {
    this.graph = graph;
    this.time = 0;
  }
  
  findBridges() {
    const visited = new Set();
    const disc = {}; // Discovery time
    const low = {}; // Lowest reachable
    const parent = {};
    const bridges = [];
    
    const dfs = (u) => {
      visited.add(u);
      disc[u] = low[u] = this.time++;
      
      for (const v of this.graph.adjacencyList[u]) {
        if (!visited.has(v)) {
          parent[v] = u;
          dfs(v);
          
          low[u] = Math.min(low[u], low[v]);
          
          // If lowest reachable from v is greater than discovery of u
          if (low[v] > disc[u]) {
            bridges.push([u, v]);
          }
        } else if (v !== parent[u]) {
          low[u] = Math.min(low[u], disc[v]);
        }
      }
    };
    
    for (const vertex in this.graph.adjacencyList) {
      if (!visited.has(vertex)) {
        dfs(vertex);
      }
    }
    
    return bridges;
  }
  
  findArticulationPoints() {
    const visited = new Set();
    const disc = {};
    const low = {};
    const parent = {};
    const ap = new Set();
    
    const dfs = (u) => {
      let children = 0;
      visited.add(u);
      disc[u] = low[u] = this.time++;
      
      for (const v of this.graph.adjacencyList[u]) {
        if (!visited.has(v)) {
          children++;
          parent[v] = u;
          dfs(v);
          
          low[u] = Math.min(low[u], low[v]);
          
          // u is root with multiple children OR
          // u is not root and low[v] >= disc[u]
          if (parent[u] === undefined && children > 1) {
            ap.add(u);
          }
          
          if (parent[u] !== undefined && low[v] >= disc[u]) {
            ap.add(u);
          }
        } else if (v !== parent[u]) {
          low[u] = Math.min(low[u], disc[v]);
        }
      }
    };
    
    for (const vertex in this.graph.adjacencyList) {
      if (!visited.has(vertex)) {
        dfs(vertex);
      }
    }
    
    return Array.from(ap);
  }
}

// Test
console.log('\n=== Bridges and Articulation Points ===');

const graph5 = new DirectedGraph();
['0', '1', '2', '3', '4'].forEach(v => graph5.addVertex(v));

graph5.addEdge('0', '1');
graph5.addEdge('1', '0');
graph5.addEdge('1', '2');
graph5.addEdge('2', '1');
graph5.addEdge('1', '3');
graph5.addEdge('3', '1');
graph5.addEdge('3', '4');
graph5.addEdge('4', '3');

const finder = new BridgeArticulationFinder(graph5);
console.log('Bridges:', finder.findBridges());
console.log('Articulation points:', finder.findArticulationPoints());
```

### **Real-World Use Cases**
```javascript
/**
 * Practical SCC applications
 */

// 1. Social Network Analysis
console.log('\n=== Social Network Analysis ===');

class SocialNetwork {
  constructor() {
    this.follows = new DirectedGraph();
  }
  
  addUser(user) {
    this.follows.addVertex(user);
  }
  
  follow(follower, followee) {
    this.follows.addEdge(follower, followee);
  }
  
  findCommunities() {
    // Communities are SCCs - groups where everyone can reach everyone
    return this.follows.findStronglyConnectedComponents();
  }
  
  findInfluencers() {
    const { sccs, condensation, vertexToSCC } = createCondensationGraph(this.follows);
    
    // Calculate in-degree for each SCC in condensation
    const inDegree = {};
    for (const scc in condensation.adjacencyList) {
      inDegree[scc] = 0;
    }
    
    for (const scc in condensation.adjacencyList) {
      for (const neighbor of condensation.adjacencyList[scc]) {
        inDegree[neighbor]++;
      }
    }
    
    // Find SCCs with high in-degree (many communities follow them)
    const influencers = [];
    for (const scc in inDegree) {
      if (inDegree[scc] >= 2) {
        influencers.push({ scc: parseInt(scc), users: sccs[scc], influence: inDegree[scc] });
      }
    }
    
    return influencers;
  }
}

const network = new SocialNetwork();

['Alice', 'Bob', 'Charlie', 'David', 'Eve', 'Frank'].forEach(u => network.addUser(u));

// Community 1: Alice, Bob, Charlie (mutual follows)
network.follow('Alice', 'Bob');
network.follow('Bob', 'Charlie');
network.follow('Charlie', 'Alice');

// Community 2: David, Eve (mutual follows)
network.follow('David', 'Eve');
network.follow('Eve', 'David');

// Frank follows both communities
network.follow('Frank', 'Alice');
network.follow('Frank', 'David');

// Communities follow Frank back
network.follow('Charlie', 'Frank');
network.follow('Eve', 'Frank');

console.log('Communities:', network.findCommunities());
console.log('Influencers:', network.findInfluencers());

// 2. Web Page Ranking
console.log('\n=== Web Page Ranking ===');

class WebGraph {
  constructor() {
    this.links = new DirectedGraph();
  }
  
  addPage(page) {
    this.links.addVertex(page);
  }
  
  addLink(from, to) {
    this.links.addEdge(from, to);
  }
  
  findStronglyLinkedGroups() {
    // Pages that link to each other form strong groups
    return this.links.findStronglyConnectedComponents();
  }
  
  identifyLinkFarms() {
    const sccs = this.links.findStronglyConnectedComponents();
    
    // Link farms are large SCCs with high internal linking
    return sccs.filter(scc => {
      if (scc.length < 3) return false;
      
      // Count internal links
      let internalLinks = 0;
      for (const page of scc) {
        for (const link of this.links.adjacencyList[page]) {
          if (scc.includes(link)) {
            internalLinks++;
          }
        }
      }
      
      // High internal link density suggests link farm
      const maxPossibleLinks = scc.length * (scc.length - 1);
      return internalLinks / maxPossibleLinks > 0.5;
    });
  }
}

const web = new WebGraph();

['pageA', 'pageB', 'pageC', 'pageD', 'pageE'].forEach(p => web.addPage(p));

// Legitimate group
web.addLink('pageA', 'pageB');
web.addLink('pageB', 'pageA');

// Link farm (suspicious)
web.addLink('pageC', 'pageD');
web.addLink('pageD', 'pageE');
web.addLink('pageE', 'pageC');
web.addLink('pageC', 'pageE');

console.log('Strongly linked groups:', web.findStronglyLinkedGroups());
console.log('Potential link farms:', web.identifyLinkFarms());

// 3. Code Dependency Analysis
console.log('\n=== Code Dependency Analysis ===');

class CodeAnalyzer {
  constructor() {
    this.dependencies = new DirectedGraph();
  }
  
  addModule(module) {
    this.dependencies.addVertex(module);
  }
  
  addImport(importer, imported) {
    this.dependencies.addEdge(importer, imported);
  }
  
  findCircularDependencies() {
    const sccs = this.dependencies.findStronglyConnectedComponents();
    
    // Circular dependencies are SCCs with more than one module
    return sccs.filter(scc => scc.length > 1);
  }
  
  suggestRefactoring() {
    const circular = this.findCircularDependencies();
    
    return circular.map(cycle => ({
      modules: cycle,
      suggestion: `Consider extracting common code from ${cycle.join(', ')} into a new module`,
      severity: cycle.length > 3 ? 'high' : 'medium'
    }));
  }
}

const analyzer = new CodeAnalyzer();

['auth', 'user', 'profile', 'settings', 'utils'].forEach(m => analyzer.addModule(m));

// Good dependencies
analyzer.addImport('auth', 'utils');
analyzer.addImport('user', 'utils');

// Circular dependency
analyzer.addImport('profile', 'user');
analyzer.addImport('user', 'settings');
analyzer.addImport('settings', 'profile'); // Creates cycle

console.log('Circular dependencies:', analyzer.findCircularDependencies());
console.log('Refactoring suggestions:', analyzer.suggestRefactoring());

// 4. Database Referential Integrity
console.log('\n=== Database Referential Integrity ===');

class DatabaseAnalyzer {
  constructor() {
    this.foreignKeys = new DirectedGraph();
  }
  
  addTable(table) {
    this.foreignKeys.addVertex(table);
  }
  
  addForeignKey(fromTable, toTable) {
    this.foreignKeys.addEdge(fromTable, toTable);
  }
  
  findMutualDependencies() {
    const sccs = this.foreignKeys.findStronglyConnectedComponents();
    return sccs.filter(scc => scc.length > 1);
  }
  
  getDeletionOrder() {
    // For tables without circular references, find safe deletion order
    const sccs = this.foreignKeys.findStronglyConnectedComponents();
    
    // Build condensation and find topological order
    const { condensation } = createCondensationGraph(this.foreignKeys);
    
    // Reverse topological sort gives deletion order
    const visited = new Set();
    const order = [];
    
    const dfs = (vertex) => {
      visited.add(vertex);
      for (const neighbor of condensation.adjacencyList[vertex]) {
        if (!visited.has(neighbor)) {
          dfs(neighbor);
        }
      }
      order.push(sccs[vertex]);
    };
    
    for (const vertex in condensation.adjacencyList) {
      if (!visited.has(vertex)) {
        dfs(parseInt(vertex));
      }
    }
    
    return order;
  }
}

const db = new DatabaseAnalyzer();

['users', 'orders', 'products', 'reviews', 'categories'].forEach(t => db.addTable(t));

db.addForeignKey('orders', 'users');
db.addForeignKey('orders', 'products');
db.addForeignKey('reviews', 'users');
db.addForeignKey('reviews', 'products');
db.addForeignKey('products', 'categories');

console.log('Mutual dependencies:', db.findMutualDependencies());
console.log('Safe deletion order:', db.getDeletionOrder());

// 5. Compiler Optimization
console.log('\n=== Compiler Optimization ===');

class ControlFlowAnalyzer {
  constructor() {
    this.cfg = new TarjanGraph(); // Control Flow Graph
  }
  
  addBasicBlock(block) {
    this.cfg.addVertex(block);
  }
  
  addJump(from, to) {
    this.cfg.addEdge(from, to);
  }
  
  findLoops() {
    // Loops are SCCs in control flow graph
    const sccs = this.cfg.findSCCsTarjan();
    return sccs.filter(scc => scc.length > 1 || this.hasSelfLoop(scc[0]));
  }
  
  hasSelfLoop(block) {
    return this.cfg.adjacencyList[block].includes(block);
  }
  
  identifyHotPaths() {
    const loops = this.findLoops();
    
    return loops.map(loop => ({
      blocks: loop,
      optimization: 'Loop unrolling, inlining, or hoisting candidates',
      complexity: loop.length
    }));
  }
}

const compiler = new ControlFlowAnalyzer();

['entry', 'cond1', 'loop_body', 'cond2', 'exit'].forEach(b => compiler.addBasicBlock(b));

compiler.addJump('entry', 'cond1');
compiler.addJump('cond1', 'loop_body');
compiler.addJump('loop_body', 'cond2');
compiler.addJump('cond2', 'loop_body'); // Loop back
compiler.addJump('cond2', 'exit');
compiler.addJump('cond1', 'exit');

console.log('Detected loops:', compiler.findLoops());
console.log('Hot paths for optimization:', compiler.identifyHotPaths());
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Single vertex (SCC of size 1)
const single = new DirectedGraph();
single.addVertex('A');
console.log('Single vertex:', single.findStronglyConnectedComponents()); // [['A']]

// Self-loop
const selfLoop = new DirectedGraph();
selfLoop.addVertex('A');
selfLoop.addEdge('A', 'A');
console.log('Self-loop:', selfLoop.findStronglyConnectedComponents()); // [['A']]

// Two vertices with bidirectional edges
const bidirectional = new DirectedGraph();
['A', 'B'].forEach(v => bidirectional.addVertex(v));
bidirectional.addEdge('A', 'B');
bidirectional.addEdge('B', 'A');
console.log('Bidirectional:', bidirectional.findStronglyConnectedComponents()); // [['A', 'B']]

// Complete graph (all vertices mutually reachable)
const complete = new DirectedGraph();
['A', 'B', 'C'].forEach(v => complete.addVertex(v));
complete.addEdge('A', 'B');
complete.addEdge('B', 'C');
complete.addEdge('C', 'A');
complete.addEdge('A', 'C');
complete.addEdge('C', 'B');
complete.addEdge('B', 'A');
console.log('Complete graph:', complete.findStronglyConnectedComponents()); // [['A', 'B', 'C']]

// DAG (each vertex is its own SCC)
const dag = new DirectedGraph();
['A', 'B', 'C', 'D'].forEach(v => dag.addVertex(v));
dag.addEdge('A', 'B');
dag.addEdge('B', 'C');
dag.addEdge('A', 'D');
dag.addEdge('D', 'C');
console.log('DAG:', dag.findStronglyConnectedComponents());
// Each vertex is separate SCC

// Disconnected components
const disconnected = new DirectedGraph();
['A', 'B', 'C', 'D'].forEach(v => disconnected.addVertex(v));
disconnected.addEdge('A', 'B');
disconnected.addEdge('B', 'A');
disconnected.addEdge('C', 'D');
disconnected.addEdge('D', 'C');
console.log('Disconnected:', disconnected.findStronglyConnectedComponents());
// [['A', 'B'], ['C', 'D']]

// Empty graph
const empty = new DirectedGraph();
console.log('Empty graph:', empty.findStronglyConnectedComponents()); // []

// Large single SCC
const largeSCC = new DirectedGraph();
for (let i = 0; i < 10; i++) {
  largeSCC.addVertex(i);
}
for (let i = 0; i < 10; i++) {
  largeSCC.addEdge(i, (i + 1) % 10); // Circular
}
console.log('Large single SCC:', largeSCC.findStronglyConnectedComponents());
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Algorithm Comparison:
┌────────────────────┬─────────────┬─────────────┬────────────────┐
│ Algorithm          │ Time        │ Space       │ Advantages     │
├────────────────────┼─────────────┼─────────────┼────────────────┤
│ Kosaraju's         │ O(V + E)    │ O(V)        │ Easy to        │
│                    │             │             │ understand     │
├────────────────────┼─────────────┼─────────────┼────────────────┤
│ Tarjan's           │ O(V + E)    │ O(V)        │ Single DFS,    │
│                    │             │             │ more efficient │
├────────────────────┼─────────────┼─────────────┼────────────────┤
│ Path-Based         │ O(V + E)    │ O(V)        │ Alternative    │
│                    │             │             │ single-pass    │
└────────────────────┴─────────────┴─────────────┴────────────────┘

Kosaraju's Algorithm:
• Two DFS passes (original graph, then transpose)
• Easy to implement and understand
• Good for teaching
• Creates transpose graph explicitly

Tarjan's Algorithm:
• Single DFS pass
• More efficient in practice (no transpose needed)
• Uses discovery time and low-link values
• Stack-based approach
• Preferred for production use

Applications:
• Social networks: community detection
• Web analysis: page ranking, link farms
• Code analysis: circular dependencies
• Database: referential integrity, deletion order
• Compilers: loop detection, optimization
• Security: vulnerability chains

Condensation Graph:
• DAG of SCCs
• Useful for meta-analysis
• Enables hierarchical processing
• Reveals high-level structure

Related Algorithms:
• Bridges: edges whose removal disconnects graph
• Articulation points: vertices whose removal disconnects
• Both use similar DFS with low-link values
`);
```

**Interview Tips:**
- SCC: maximal set of vertices where every vertex is reachable from every other
- Two main algorithms: Kosaraju's (two DFS) and Tarjan's (one DFS)
- Kosaraju's: DFS to get finish times, transpose graph, DFS in reverse finish order
- Tarjan's: single DFS with discovery time and low-link values, stack for current path
- Time: O(V + E) for both algorithms
- Space: O(V) for auxiliary structures
- Condensation graph: DAG where each node is an SCC, edges between SCCs
- Applications: social network communities, web page ranking, circular dependency detection
- Low-link value: lowest vertex reachable from subtree in DFS
- Discovery time: when vertex is first visited in DFS
- SCC root: vertex where low-link equals discovery time
- Transpose: reverse all edges in directed graph
- Finish time: when DFS completes processing a vertex (post-order)
- Why transpose: allows finding vertices that can reach a given vertex
- Stack in Tarjan's: maintains vertices in current SCC being explored
- Single vertex: can be SCC (trivial SCC)
- DAG: each vertex is its own SCC
- Complete graph: all vertices form single SCC
- Bridges and articulation points: related concepts for connectivity
- Edge cases: empty graph, single vertex, self-loops, disconnected components
- Kosaraju vs Tarjan: Kosaraju easier to understand, Tarjan more efficient
- Follow-ups: count SCCs, largest SCC, condensation graph, bridges/articulation points
- Clarify: need SCC members? condensation graph? just count? specific vertex's SCC?

</details>

105. Implement an LRU (Least Recently Used) Cache

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Hash Map + Doubly Linked List**
```javascript
/**
 * LRU Cache with O(1) operations
 * Time Complexity: O(1) for get and put
 * Space Complexity: O(capacity)
 */
class Node {
  constructor(key, value) {
    this.key = key;
    this.value = value;
    this.prev = null;
    this.next = null;
  }
}

class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
    
    // Dummy head and tail for easier operations
    this.head = new Node(0, 0);
    this.tail = new Node(0, 0);
    this.head.next = this.tail;
    this.tail.prev = this.head;
  }
  
  get(key) {
    if (!this.cache.has(key)) {
      return -1;
    }
    
    const node = this.cache.get(key);
    
    // Move to front (most recently used)
    this.remove(node);
    this.addToFront(node);
    
    return node.value;
  }
  
  put(key, value) {
    if (this.cache.has(key)) {
      // Update existing
      const node = this.cache.get(key);
      node.value = value;
      this.remove(node);
      this.addToFront(node);
    } else {
      // Add new
      if (this.cache.size >= this.capacity) {
        // Remove least recently used (from tail)
        const lru = this.tail.prev;
        this.remove(lru);
        this.cache.delete(lru.key);
      }
      
      const newNode = new Node(key, value);
      this.cache.set(key, newNode);
      this.addToFront(newNode);
    }
  }
  
  remove(node) {
    node.prev.next = node.next;
    node.next.prev = node.prev;
  }
  
  addToFront(node) {
    node.next = this.head.next;
    node.prev = this.head;
    this.head.next.prev = node;
    this.head.next = node;
  }
}

// Test
console.log('=== LRU Cache ===');

const cache = new LRUCache(3);

cache.put(1, 'one');
cache.put(2, 'two');
cache.put(3, 'three');

console.log('Get 1:', cache.get(1)); // 'one' - moves to front
console.log('Get 2:', cache.get(2)); // 'two' - moves to front

cache.put(4, 'four'); // Evicts key 3 (LRU)

console.log('Get 3:', cache.get(3)); // -1 (evicted)
console.log('Get 4:', cache.get(4)); // 'four'
console.log('Get 1:', cache.get(1)); // 'one'
console.log('Get 2:', cache.get(2)); // 'two'
```

### **Approach 2: Using Map (Built-in Ordering)**
```javascript
/**
 * LRU Cache using JavaScript Map's insertion order
 * Map maintains insertion order, delete+set moves to end
 */
class LRUCacheMap {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }
  
  get(key) {
    if (!this.cache.has(key)) {
      return -1;
    }
    
    const value = this.cache.get(key);
    
    // Move to end (most recent)
    this.cache.delete(key);
    this.cache.set(key, value);
    
    return value;
  }
  
  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.capacity) {
      // Remove first entry (oldest)
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    this.cache.set(key, value);
  }
  
  keys() {
    return Array.from(this.cache.keys());
  }
}

// Test
console.log('\n=== LRU Cache (Map) ===');

const cache2 = new LRUCacheMap(3);

cache2.put(1, 'a');
cache2.put(2, 'b');
cache2.put(3, 'c');

console.log('Keys:', cache2.keys()); // [1, 2, 3]

cache2.get(1); // Access 1
console.log('After get(1):', cache2.keys()); // [2, 3, 1]

cache2.put(4, 'd'); // Evicts 2
console.log('After put(4, d):', cache2.keys()); // [3, 1, 4]
```

### **Approach 3: LRU with Expiration Time**
```javascript
/**
 * LRU Cache with TTL (Time To Live)
 */
class LRUCacheWithTTL extends LRUCache {
  constructor(capacity, ttl = Infinity) {
    super(capacity);
    this.ttl = ttl; // milliseconds
  }
  
  get(key) {
    if (!this.cache.has(key)) {
      return -1;
    }
    
    const node = this.cache.get(key);
    
    // Check expiration
    if (Date.now() - node.timestamp > this.ttl) {
      this.remove(node);
      this.cache.delete(key);
      return -1;
    }
    
    // Update timestamp and move to front
    node.timestamp = Date.now();
    this.remove(node);
    this.addToFront(node);
    
    return node.value;
  }
  
  put(key, value) {
    const timestamp = Date.now();
    
    if (this.cache.has(key)) {
      const node = this.cache.get(key);
      node.value = value;
      node.timestamp = timestamp;
      this.remove(node);
      this.addToFront(node);
    } else {
      if (this.cache.size >= this.capacity) {
        const lru = this.tail.prev;
        this.remove(lru);
        this.cache.delete(lru.key);
      }
      
      const newNode = new Node(key, value);
      newNode.timestamp = timestamp;
      this.cache.set(key, newNode);
      this.addToFront(newNode);
    }
  }
}

// Test
console.log('\n=== LRU Cache with TTL ===');

const cache3 = new LRUCacheWithTTL(3, 1000); // 1 second TTL

cache3.put('temp', 'value');
console.log('Immediate get:', cache3.get('temp')); // 'value'

setTimeout(() => {
  console.log('After 1.5s:', cache3.get('temp')); // -1 (expired)
}, 1500);
```

### **Approach 4: LRU with Frequency Tracking (LFU-LRU Hybrid)**
```javascript
/**
 * LRU Cache that also tracks frequency
 * Evicts least frequently used, then least recently used
 */
class LFULRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map(); // key -> {value, freq, timestamp}
  }
  
  get(key) {
    if (!this.cache.has(key)) {
      return -1;
    }
    
    const entry = this.cache.get(key);
    entry.freq++;
    entry.timestamp = Date.now();
    
    return entry.value;
  }
  
  put(key, value) {
    if (this.capacity === 0) return;
    
    if (this.cache.has(key)) {
      const entry = this.cache.get(key);
      entry.value = value;
      entry.freq++;
      entry.timestamp = Date.now();
    } else {
      if (this.cache.size >= this.capacity) {
        // Find LFU, then LRU
        let minFreq = Infinity;
        let minTime = Infinity;
        let lruKey = null;
        
        for (const [k, v] of this.cache) {
          if (v.freq < minFreq || (v.freq === minFreq && v.timestamp < minTime)) {
            minFreq = v.freq;
            minTime = v.timestamp;
            lruKey = k;
          }
        }
        
        this.cache.delete(lruKey);
      }
      
      this.cache.set(key, {
        value,
        freq: 1,
        timestamp: Date.now()
      });
    }
  }
}

// Test
console.log('\n=== LFU-LRU Hybrid ===');

const cache4 = new LFULRUCache(3);

cache4.put(1, 'one');
cache4.put(2, 'two');
cache4.put(3, 'three');

cache4.get(1); // freq = 2
cache4.get(1); // freq = 3
cache4.get(2); // freq = 2

cache4.put(4, 'four'); // Evicts 3 (freq = 1, least frequent)

console.log('Get 3:', cache4.get(3)); // -1 (evicted)
console.log('Get 1:', cache4.get(1)); // 'one'
```

### **Approach 5: Thread-Safe LRU Cache**
```javascript
/**
 * Thread-safe LRU Cache (for Node.js with async operations)
 * Uses locks to prevent race conditions
 */
class ThreadSafeLRUCache extends LRUCache {
  constructor(capacity) {
    super(capacity);
    this.locks = new Map();
  }
  
  async acquireLock(key) {
    while (this.locks.has(key)) {
      await new Promise(resolve => setTimeout(resolve, 10));
    }
    this.locks.set(key, true);
  }
  
  releaseLock(key) {
    this.locks.delete(key);
  }
  
  async getAsync(key) {
    await this.acquireLock(key);
    
    try {
      return this.get(key);
    } finally {
      this.releaseLock(key);
    }
  }
  
  async putAsync(key, value) {
    await this.acquireLock(key);
    
    try {
      this.put(key, value);
    } finally {
      this.releaseLock(key);
    }
  }
}

// Test
console.log('\n=== Thread-Safe LRU Cache ===');

const cache5 = new ThreadSafeLRUCache(3);

(async () => {
  await cache5.putAsync(1, 'one');
  await cache5.putAsync(2, 'two');
  
  const value = await cache5.getAsync(1);
  console.log('Async get:', value); // 'one'
})();
```

### **Real-World Use Cases**
```javascript
/**
 * Practical LRU Cache applications
 */

// 1. Web Browser Cache
console.log('\n=== Web Browser Cache ===');

class BrowserCache {
  constructor(maxPages = 50) {
    this.cache = new LRUCache(maxPages);
  }
  
  visitPage(url, content) {
    console.log(`Caching page: ${url}`);
    this.cache.put(url, content);
  }
  
  getPage(url) {
    const content = this.cache.get(url);
    
    if (content === -1) {
      console.log(`Cache miss for ${url} - fetching from server`);
      return null;
    }
    
    console.log(`Cache hit for ${url}`);
    return content;
  }
}

const browser = new BrowserCache(3);

browser.visitPage('page1.html', '<html>Page 1</html>');
browser.visitPage('page2.html', '<html>Page 2</html>');
browser.visitPage('page3.html', '<html>Page 3</html>');

browser.getPage('page1.html'); // Cache hit

browser.visitPage('page4.html', '<html>Page 4</html>'); // Evicts page2

browser.getPage('page2.html'); // Cache miss

// 2. Database Query Cache
console.log('\n=== Database Query Cache ===');

class QueryCache {
  constructor(maxQueries = 100) {
    this.cache = new LRUCacheMap(maxQueries);
    this.hits = 0;
    this.misses = 0;
  }
  
  async executeQuery(sql, params = []) {
    const cacheKey = JSON.stringify({ sql, params });
    const cached = this.cache.get(cacheKey);
    
    if (cached !== -1) {
      this.hits++;
      console.log(`Cache hit for query: ${sql.substring(0, 50)}...`);
      return cached;
    }
    
    this.misses++;
    console.log(`Cache miss - executing query: ${sql.substring(0, 50)}...`);
    
    // Simulate database query
    const result = await this.simulateDBQuery(sql, params);
    
    this.cache.put(cacheKey, result);
    return result;
  }
  
  async simulateDBQuery(sql, params) {
    // Simulate delay
    await new Promise(resolve => setTimeout(resolve, 100));
    return { rows: [], count: 0 };
  }
  
  getStats() {
    const total = this.hits + this.misses;
    const hitRate = total > 0 ? (this.hits / total * 100).toFixed(2) : 0;
    
    return {
      hits: this.hits,
      misses: this.misses,
      hitRate: `${hitRate}%`
    };
  }
}

const queryCache = new QueryCache(5);

(async () => {
  await queryCache.executeQuery('SELECT * FROM users WHERE id = ?', [1]);
  await queryCache.executeQuery('SELECT * FROM users WHERE id = ?', [1]); // Cache hit
  await queryCache.executeQuery('SELECT * FROM orders WHERE user_id = ?', [1]);
  
  console.log('Cache stats:', queryCache.getStats());
})();

// 3. API Response Cache
console.log('\n=== API Response Cache ===');

class APICache {
  constructor(maxEndpoints = 20, ttl = 60000) {
    this.cache = new LRUCacheWithTTL(maxEndpoints, ttl);
  }
  
  async fetch(endpoint, options = {}) {
    const cacheKey = JSON.stringify({ endpoint, options });
    const cached = this.cache.get(cacheKey);
    
    if (cached !== -1) {
      console.log(`Using cached response for ${endpoint}`);
      return cached;
    }
    
    console.log(`Fetching from API: ${endpoint}`);
    
    // Simulate API call
    const response = await this.simulateAPICall(endpoint, options);
    
    this.cache.put(cacheKey, response);
    return response;
  }
  
  async simulateAPICall(endpoint, options) {
    await new Promise(resolve => setTimeout(resolve, 200));
    return { data: `Response from ${endpoint}`, timestamp: Date.now() };
  }
}

const apiCache = new APICache(10, 5000); // 5 second TTL

(async () => {
  await apiCache.fetch('/api/users/1');
  await apiCache.fetch('/api/users/1'); // Cache hit
  
  setTimeout(async () => {
    await apiCache.fetch('/api/users/1'); // Cache miss after expiration
  }, 6000);
})();

// 4. Memoization Cache
console.log('\n=== Memoization Cache ===');

class MemoCache {
  constructor(fn, maxSize = 100) {
    this.fn = fn;
    this.cache = new LRUCacheMap(maxSize);
  }
  
  call(...args) {
    const key = JSON.stringify(args);
    const cached = this.cache.get(key);
    
    if (cached !== -1) {
      return cached;
    }
    
    const result = this.fn(...args);
    this.cache.put(key, result);
    
    return result;
  }
}

// Expensive Fibonacci calculation
function fibonacci(n) {
  console.log(`Calculating fib(${n})`);
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

const memoFib = new MemoCache(fibonacci, 50);

console.log('First call fib(10):', memoFib.call(10)); // Calculates
console.log('Second call fib(10):', memoFib.call(10)); // Cached

// 5. CDN Edge Cache
console.log('\n=== CDN Edge Cache ===');

class EdgeCache {
  constructor(maxAssets = 1000) {
    this.cache = new LRUCache(maxAssets);
    this.bandwidth = 0;
  }
  
  serve(assetPath) {
    const cached = this.cache.get(assetPath);
    
    if (cached !== -1) {
      console.log(`✓ Edge cache hit: ${assetPath}`);
      return cached;
    }
    
    console.log(`✗ Edge cache miss: ${assetPath} - fetching from origin`);
    
    // Simulate fetching from origin server
    const asset = this.fetchFromOrigin(assetPath);
    this.cache.put(assetPath, asset);
    this.bandwidth += asset.size;
    
    return asset;
  }
  
  fetchFromOrigin(path) {
    return {
      path,
      content: `Binary content of ${path}`,
      size: Math.random() * 1000 + 100, // KB
      timestamp: Date.now()
    };
  }
  
  getStats() {
    return {
      bandwidthUsed: `${this.bandwidth.toFixed(2)} KB`,
      cachedAssets: this.cache.cache.size
    };
  }
}

const cdn = new EdgeCache(5);

cdn.serve('/img/logo.png');
cdn.serve('/css/style.css');
cdn.serve('/img/logo.png'); // Cache hit
cdn.serve('/js/app.js');

console.log('CDN stats:', cdn.getStats());
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Capacity 1
const tiny = new LRUCache(1);
tiny.put(1, 'one');
tiny.put(2, 'two'); // Evicts 1
console.log('Capacity 1, get(1):', tiny.get(1)); // -1
console.log('Capacity 1, get(2):', tiny.get(2)); // 'two'

// Capacity 0 (edge case)
const zero = new LRUCache(0);
zero.put(1, 'one'); // No-op
console.log('Capacity 0:', zero.get(1)); // -1

// Update existing key
const update = new LRUCache(2);
update.put(1, 'one');
update.put(1, 'ONE'); // Update
console.log('Update:', update.get(1)); // 'ONE'

// Multiple operations
const multi = new LRUCache(2);
multi.put(1, 1);
multi.put(2, 2);
console.log('Get 1:', multi.get(1)); // 1 - moves to front
multi.put(3, 3); // Evicts 2
console.log('Get 2:', multi.get(2)); // -1 (evicted)
console.log('Get 3:', multi.get(3)); // 3

// Access pattern
const pattern = new LRUCache(3);
pattern.put('a', 1);
pattern.put('b', 2);
pattern.put('c', 3);
pattern.get('a'); // Access 'a'
pattern.get('b'); // Access 'b'
pattern.put('d', 4); // Evicts 'c' (least recently used)
console.log('Get c:', pattern.get('c')); // -1
```

### **Performance Analysis**
```javascript
console.log('\n=== Performance ===');

console.log(`
LRU Cache Comparison:
┌──────────────────────┬─────────┬─────────┬─────────────────┐
│ Implementation       │ Get     │ Put     │ Advantages      │
├──────────────────────┼─────────┼─────────┼─────────────────┤
│ HashMap + DLL        │ O(1)    │ O(1)    │ True O(1),      │
│                      │         │         │ predictable     │
├──────────────────────┼─────────┼─────────┼─────────────────┤
│ Map (JS)             │ O(1)*   │ O(1)*   │ Simpler code,   │
│                      │         │         │ uses built-in   │
├──────────────────────┼─────────┼─────────┼─────────────────┤
│ With TTL             │ O(1)    │ O(1)    │ Auto-expiration │
├──────────────────────┼─────────┼─────────┼─────────────────┤
│ LFU-LRU Hybrid       │ O(1)    │ O(n)**  │ Better eviction │
└──────────────────────┴─────────┴─────────┴─────────────────┘

* Amortized O(1) but may have overhead from delete+set
** O(n) to find LFU item, can be O(1) with more complex structure

Space Complexity: O(capacity) for all implementations

Applications:
• Browser cache: recently visited pages
• Database query cache: reduce DB load
• API response cache: reduce external API calls
• CDN edge cache: serve static assets quickly
• Memoization: cache expensive function results

Key Design Decisions:
• Doubly linked list: O(1) removal and insertion
• HashMap: O(1) lookup by key
• Dummy head/tail: simplify edge cases
• Move on access: maintain LRU order

Variations:
• LFU (Least Frequently Used): evict least accessed
• MRU (Most Recently Used): evict most recent
• FIFO: evict oldest insertion
• Random: evict random entry
• TTL: time-based expiration
`);
```

**Interview Tips:**
- LRU cache: evicts least recently used item when full
- Two data structures: hash map for O(1) lookup, doubly linked list for O(1) removal
- Get operation: return value and move to front (most recent)
- Put operation: add/update, move to front, evict from tail if full
- Doubly linked list: need prev and next pointers for O(1) removal
- Dummy head/tail nodes: simplify edge cases (no null checks)
- Time: O(1) for both get and put operations
- Space: O(capacity) to store at most capacity items
- Why doubly linked: need to remove from middle efficiently
- Hash map: stores key → node mapping for fast lookup
- Most recent: at head, least recent: at tail
- On access: remove from current position, add to head
- On eviction: remove from tail (least recently used)
- JavaScript Map: maintains insertion order, can use for simpler LRU
- Map.delete() + Map.set(): moves entry to end (most recent)
- Applications: browser cache, database query cache, CDN, memoization
- Variations: LFU (frequency-based), TTL (time-based), LRU-K (k-distance)
- Thread safety: add locks for concurrent access
- Edge cases: capacity 0 or 1, update existing key, empty cache
- Optimization: could use built-in Map but less educational
- Real-world: combine with TTL for time-based expiration
- Follow-ups: implement LFU, add TTL, thread-safe version, statistics/monitoring
- Clarify: capacity limit? what to return on miss? need statistics? thread-safe?

</details>

### **Advanced Problem Solving**

106. Solve the "Longest Palindromic Substring" problem

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Expand Around Center**
```javascript
/**
 * Longest palindromic substring by expanding around each center
 * Time Complexity: O(n²)
 * Space Complexity: O(1)
 */
function longestPalindrome(s) {
  if (!s || s.length < 1) return '';
  
  let start = 0;
  let end = 0;
  
  function expandAroundCenter(left, right) {
    while (left >= 0 && right < s.length && s[left] === s[right]) {
      left--;
      right++;
    }
    // Return length of palindrome
    return right - left - 1;
  }
  
  for (let i = 0; i < s.length; i++) {
    // Odd length palindrome (center is single character)
    const len1 = expandAroundCenter(i, i);
    
    // Even length palindrome (center is between two characters)
    const len2 = expandAroundCenter(i, i + 1);
    
    const len = Math.max(len1, len2);
    
    if (len > end - start) {
      start = i - Math.floor((len - 1) / 2);
      end = i + Math.floor(len / 2);
    }
  }
  
  return s.substring(start, end + 1);
}

// Test
console.log('=== Expand Around Center ===');

console.log(longestPalindrome('babad')); // 'bab' or 'aba'
console.log(longestPalindrome('cbbd')); // 'bb'
console.log(longestPalindrome('racecar')); // 'racecar'
console.log(longestPalindrome('a')); // 'a'
console.log(longestPalindrome('ac')); // 'a' or 'c'
```

### **Approach 2: Dynamic Programming**
```javascript
/**
 * DP approach: dp[i][j] = true if substring s[i...j] is palindrome
 * Time Complexity: O(n²)
 * Space Complexity: O(n²)
 */
function longestPalindromeDP(s) {
  const n = s.length;
  if (n < 2) return s;
  
  // dp[i][j] = true if s[i...j] is palindrome
  const dp = Array(n).fill(null).map(() => Array(n).fill(false));
  
  let start = 0;
  let maxLen = 1;
  
  // Every single character is a palindrome
  for (let i = 0; i < n; i++) {
    dp[i][i] = true;
  }
  
  // Check for length 2
  for (let i = 0; i < n - 1; i++) {
    if (s[i] === s[i + 1]) {
      dp[i][i + 1] = true;
      start = i;
      maxLen = 2;
    }
  }
  
  // Check for lengths greater than 2
  for (let len = 3; len <= n; len++) {
    for (let i = 0; i < n - len + 1; i++) {
      const j = i + len - 1; // Ending index
      
      // Check if s[i...j] is palindrome
      if (s[i] === s[j] && dp[i + 1][j - 1]) {
        dp[i][j] = true;
        
        if (len > maxLen) {
          start = i;
          maxLen = len;
        }
      }
    }
  }
  
  return s.substring(start, start + maxLen);
}

// Test
console.log('\n=== Dynamic Programming ===');

console.log(longestPalindromeDP('babad')); // 'bab' or 'aba'
console.log(longestPalindromeDP('cbbd')); // 'bb'
console.log(longestPalindromeDP('noon')); // 'noon'
console.log(longestPalindromeDP('abcdefg')); // any single char
```

### **Approach 3: Manacher's Algorithm (Optimal)**
```javascript
/**
 * Manacher's algorithm - linear time
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function longestPalindromeManacher(s) {
  if (!s || s.length === 0) return '';
  
  // Transform string to handle even/odd palindromes uniformly
  // "abc" -> "^#a#b#c#$"
  let T = '^';
  for (let i = 0; i < s.length; i++) {
    T += '#' + s[i];
  }
  T += '#$';
  
  const n = T.length;
  const P = new Array(n).fill(0); // P[i] = radius of palindrome centered at i
  let center = 0;
  let right = 0;
  
  for (let i = 1; i < n - 1; i++) {
    // Mirror of i with respect to center
    const mirror = 2 * center - i;
    
    if (i < right) {
      P[i] = Math.min(right - i, P[mirror]);
    }
    
    // Expand around i
    try {
      while (T[i + 1 + P[i]] === T[i - 1 - P[i]]) {
        P[i]++;
      }
    } catch (e) {
      // Out of bounds
    }
    
    // Update center and right if palindrome extends past right
    if (i + P[i] > right) {
      center = i;
      right = i + P[i];
    }
  }
  
  // Find longest palindrome
  let maxLen = 0;
  let centerIndex = 0;
  
  for (let i = 1; i < n - 1; i++) {
    if (P[i] > maxLen) {
      maxLen = P[i];
      centerIndex = i;
    }
  }
  
  // Extract palindrome from original string
  const start = Math.floor((centerIndex - maxLen) / 2);
  return s.substring(start, start + maxLen);
}

// Test
console.log('\n=== Manacher\'s Algorithm ===');

console.log(longestPalindromeManacher('babad')); // 'bab' or 'aba'
console.log(longestPalindromeManacher('cbbd')); // 'bb'
console.log(longestPalindromeManacher('racecar')); // 'racecar'
console.log(longestPalindromeManacher('abcde')); // any single char
```

### **Approach 4: Find All Palindromic Substrings**
```javascript
/**
 * Find all palindromic substrings and return longest
 */
function findAllPalindromes(s) {
  const palindromes = [];
  
  function expandAroundCenter(left, right) {
    while (left >= 0 && right < s.length && s[left] === s[right]) {
      palindromes.push(s.substring(left, right + 1));
      left--;
      right++;
    }
  }
  
  for (let i = 0; i < s.length; i++) {
    expandAroundCenter(i, i); // Odd length
    expandAroundCenter(i, i + 1); // Even length
  }
  
  // Return longest
  return palindromes.reduce((longest, current) => 
    current.length > longest.length ? current : longest
  , '');
}

// Test
console.log('\n=== All Palindromes ===');

const allPalindromes = [];

function getAllPalindromes(s) {
  const result = [];
  
  function expandAroundCenter(left, right) {
    while (left >= 0 && right < s.length && s[left] === s[right]) {
      result.push(s.substring(left, right + 1));
      left--;
      right++;
    }
  }
  
  for (let i = 0; i < s.length; i++) {
    expandAroundCenter(i, i);
    expandAroundCenter(i, i + 1);
  }
  
  return result;
}

console.log('All palindromes in "abba":', getAllPalindromes('abba'));
console.log('Longest:', findAllPalindromes('abba'));
```

### **Approach 5: Count Palindromic Substrings**
```javascript
/**
 * Count total number of palindromic substrings
 */
function countPalindromicSubstrings(s) {
  let count = 0;
  
  function expandAroundCenter(left, right) {
    while (left >= 0 && right < s.length && s[left] === s[right]) {
      count++;
      left--;
      right++;
    }
  }
  
  for (let i = 0; i < s.length; i++) {
    expandAroundCenter(i, i); // Odd length
    expandAroundCenter(i, i + 1); // Even length
  }
  
  return count;
}

// Test
console.log('\n=== Count Palindromes ===');

console.log('Count in "abc":', countPalindromicSubstrings('abc')); // 3 (a, b, c)
console.log('Count in "aaa":', countPalindromicSubstrings('aaa')); // 6 (a, a, a, aa, aa, aaa)
console.log('Count in "noon":', countPalindromicSubstrings('noon')); // 6
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications of palindrome detection
 */

// 1. DNA Sequence Analysis
console.log('\n=== DNA Sequence Analysis ===');

class DNAAnalyzer {
  constructor(sequence) {
    this.sequence = sequence;
  }
  
  findPalindromicSites() {
    // Palindromic sequences are important in genetics (restriction sites)
    const palindromes = [];
    
    function expandAroundCenter(s, left, right) {
      const found = [];
      while (left >= 0 && right < s.length && s[left] === s[right]) {
        if (right - left + 1 >= 4) { // Minimum length for restriction sites
          found.push({
            sequence: s.substring(left, right + 1),
            start: left,
            end: right,
            length: right - left + 1
          });
        }
        left--;
        right++;
      }
      return found;
    }
    
    for (let i = 0; i < this.sequence.length; i++) {
      palindromes.push(...expandAroundCenter(this.sequence, i, i));
      palindromes.push(...expandAroundCenter(this.sequence, i, i + 1));
    }
    
    // Sort by length descending
    return palindromes.sort((a, b) => b.length - a.length);
  }
  
  getLongestPalindrome() {
    return longestPalindrome(this.sequence);
  }
}

const dna = new DNAAnalyzer('GAATTCGAATTC');
console.log('Longest palindrome:', dna.getLongestPalindrome());
console.log('Palindromic sites:', dna.findPalindromicSites().slice(0, 3));

// 2. Text Processing - Detecting Symmetrical Patterns
console.log('\n=== Symmetrical Pattern Detection ===');

class TextAnalyzer {
  constructor(text) {
    this.text = text.toLowerCase().replace(/[^a-z0-9]/g, '');
  }
  
  findSymmetricalPhrases(minLength = 3) {
    const phrases = [];
    const text = this.text;
    
    function expandAroundCenter(left, right) {
      while (left >= 0 && right < text.length && text[left] === text[right]) {
        if (right - left + 1 >= minLength) {
          phrases.push({
            text: text.substring(left, right + 1),
            start: left,
            length: right - left + 1
          });
        }
        left--;
        right++;
      }
    }
    
    for (let i = 0; i < text.length; i++) {
      expandAroundCenter(i, i);
      expandAroundCenter(i, i + 1);
    }
    
    // Deduplicate and sort
    const unique = Array.from(
      new Map(phrases.map(p => [p.text, p])).values()
    );
    
    return unique.sort((a, b) => b.length - a.length);
  }
}

const analyzer = new TextAnalyzer('A man, a plan, a canal: Panama');
console.log('Symmetrical patterns:', analyzer.findSymmetricalPhrases().slice(0, 5));

// 3. Password Strength Validator
console.log('\n=== Password Strength ===');

class PasswordValidator {
  checkPalindromes(password) {
    const longest = longestPalindrome(password);
    
    // Penalize long palindromes (predictable patterns)
    return {
      longestPalindrome: longest,
      length: longest.length,
      warning: longest.length > 3 ? 
        'Contains palindromic pattern - less secure' : 
        'No significant palindromic patterns',
      strength: longest.length <= 3 ? 'Good' : 'Weak'
    };
  }
}

const validator = new PasswordValidator();

console.log(validator.checkPalindromes('abccba123')); // Weak
console.log(validator.checkPalindromes('x9k2!mQ7')); // Good

// 4. Compression Algorithm
console.log('\n=== String Compression ===');

class PalindromeCompressor {
  compress(s) {
    // Find longest palindrome to potentially compress
    const longest = longestPalindrome(s);
    const index = s.indexOf(longest);
    
    if (longest.length < 3) {
      return { compressed: s, ratio: 1.0 };
    }
    
    // Represent palindrome as center + radius notation
    const center = Math.floor(longest.length / 2);
    const compressed = 
      s.substring(0, index) + 
      `[PAL:${longest[center]}:${Math.floor(longest.length / 2)}]` +
      s.substring(index + longest.length);
    
    return {
      original: s,
      compressed,
      saved: s.length - compressed.length,
      ratio: (compressed.length / s.length).toFixed(2)
    };
  }
}

const compressor = new PalindromeCompressor();
console.log(compressor.compress('xyzracecarpqr'));
console.log(compressor.compress('abc123321xyz'));

// 5. Code Review - Detect Redundant Patterns
console.log('\n=== Code Pattern Detection ===');

class CodeAnalyzer {
  detectRedundantPatterns(code) {
    // Simplify code for analysis
    const simplified = code.replace(/\s+/g, '').toLowerCase();
    
    const palindromes = [];
    
    function expandAroundCenter(s, left, right) {
      while (left >= 0 && right < s.length && s[left] === s[right]) {
        if (right - left + 1 >= 5) { // Minimum pattern size
          palindromes.push({
            pattern: s.substring(left, right + 1),
            position: left,
            length: right - left + 1
          });
        }
        left--;
        right++;
      }
    }
    
    for (let i = 0; i < simplified.length; i++) {
      expandAroundCenter(simplified, i, i);
      expandAroundCenter(simplified, i, i + 1);
    }
    
    return palindromes.length > 0 ? {
      found: true,
      patterns: palindromes.slice(0, 3),
      suggestion: 'Consider refactoring repetitive code'
    } : {
      found: false,
      message: 'No obvious redundant patterns'
    };
  }
}

const codeAnalyzer = new CodeAnalyzer();
const code = 'if (x) { y = z; z = y; } else { y = z; z = y; }';
console.log(codeAnalyzer.detectRedundantPatterns(code));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty string
console.log('Empty string:', longestPalindrome('')); // ''

// Single character
console.log('Single char:', longestPalindrome('a')); // 'a'

// No palindrome (all different characters)
console.log('All different:', longestPalindrome('abcdef')); // Any single char

// Entire string is palindrome
console.log('Full palindrome:', longestPalindrome('racecar')); // 'racecar'

// Multiple same-length palindromes
console.log('Multiple:', longestPalindrome('abacabad')); // 'aba' or 'aca' or 'abad'

// Even length palindrome
console.log('Even length:', longestPalindrome('abba')); // 'abba'

// Odd length palindrome
console.log('Odd length:', longestPalindrome('aba')); // 'aba'

// Long string with small palindrome
const longStr = 'a'.repeat(1000) + 'racecar' + 'b'.repeat(1000);
console.log('Long string result length:', longestPalindrome(longStr).length); // 1000

// All same characters
console.log('All same:', longestPalindrome('aaaa')); // 'aaaa'

// Palindrome at start
console.log('At start:', longestPalindrome('abbaxy')); // 'abba'

// Palindrome at end
console.log('At end:', longestPalindrome('xyabba')); // 'abba'

// Nested palindromes
console.log('Nested:', longestPalindrome('abacabad')); // 'abacaba'
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

function benchmark(fn, name, input) {
  const start = performance.now();
  const result = fn(input);
  const end = performance.now();
  
  console.log(`${name}: ${result} (${(end - start).toFixed(3)}ms)`);
}

const testStr = 'forgeeksskeegfor';

benchmark(longestPalindrome, 'Expand Center', testStr);
benchmark(longestPalindromeDP, 'Dynamic Programming', testStr);
benchmark(longestPalindromeManacher, 'Manacher', testStr);

console.log(`
Algorithm Comparison:
┌────────────────────┬─────────────┬─────────────┬──────────────┐
│ Algorithm          │ Time        │ Space       │ Best For     │
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ Expand Center      │ O(n²)       │ O(1)        │ Simple,      │
│                    │             │             │ space-optimal│
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ Dynamic Programming│ O(n²)       │ O(n²)       │ All substrings│
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ Manacher's         │ O(n)        │ O(n)        │ Optimal,     │
│                    │             │             │ large inputs │
└────────────────────┴─────────────┴─────────────┴──────────────┘

Key Insights:
• Expand around center: most intuitive, good for interviews
• Consider both odd and even length palindromes
• DP: builds table of all substrings
• Manacher's: linear time but complex to implement
• For practical use: expand around center is usually sufficient

Optimization Tips:
• Early termination if max possible length exceeded
• Precompute character frequencies
• Skip positions that can't improve answer
• Use Manacher's for very long strings (10k+ chars)
`);
```

**Interview Tips:**
- Palindrome: reads same forwards and backwards
- Expand around center: most common interview approach, O(n²) time, O(1) space
- Key insight: every palindrome has a center (1 char for odd, 2 chars for even)
- For each position, expand left and right while characters match
- Time: O(n²) - n centers, each expansion O(n) worst case
- Space: O(1) - only tracking start/end indices
- DP approach: dp[i][j] = true if s[i..j] is palindrome
- DP recurrence: dp[i][j] = (s[i] == s[j]) && dp[i+1][j-1]
- DP time: O(n²), space: O(n²) for table
- Manacher's: optimal O(n) time but complex, rarely required in interviews
- Manacher's transforms string with delimiters to handle odd/even uniformly
- Two cases: odd length (center is char) and even length (center is between chars)
- Track longest palindrome: update start/end indices when found longer
- Applications: DNA analysis (restriction sites), text patterns, password validation
- Edge cases: empty string, single char, no palindrome, entire string, all same chars
- Multiple palindromes: problem usually asks for any longest, not all
- Optimization: can stop early if remaining string can't beat current max
- Common mistake: forgetting even-length palindromes
- Follow-ups: count all palindromes, longest palindromic subsequence (different!), check if palindrome
- Substring vs subsequence: substring is contiguous, subsequence allows gaps
- Clarify: return one or all longest? indices or string? ignore spaces/punctuation?

</details>

107. Implement a function to solve Sudoku

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Backtracking (Classic)**
```javascript
/**
 * Solve Sudoku using backtracking
 * Time Complexity: O(9^m) where m is number of empty cells
 * Space Complexity: O(1) if modifying in place
 */
function solveSudoku(board) {
  function isValid(board, row, col, num) {
    // Check row
    for (let x = 0; x < 9; x++) {
      if (board[row][x] === num) {
        return false;
      }
    }
    
    // Check column
    for (let x = 0; x < 9; x++) {
      if (board[x][col] === num) {
        return false;
      }
    }
    
    // Check 3x3 box
    const boxRow = Math.floor(row / 3) * 3;
    const boxCol = Math.floor(col / 3) * 3;
    
    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        if (board[boxRow + i][boxCol + j] === num) {
          return false;
        }
      }
    }
    
    return true;
  }
  
  function solve(board) {
    for (let row = 0; row < 9; row++) {
      for (let col = 0; col < 9; col++) {
        if (board[row][col] === '.') {
          for (let num = 1; num <= 9; num++) {
            const char = num.toString();
            
            if (isValid(board, row, col, char)) {
              board[row][col] = char;
              
              if (solve(board)) {
                return true;
              }
              
              // Backtrack
              board[row][col] = '.';
            }
          }
          
          return false; // No valid number found
        }
      }
    }
    
    return true; // All cells filled
  }
  
  solve(board);
  return board;
}

// Test
console.log('=== Sudoku Solver (Backtracking) ===');

const puzzle = [
  ["5","3",".",".","7",".",".",".","."],
  ["6",".",".","1","9","5",".",".","."],
  [".","9","8",".",".",".",".","6","."],
  ["8",".",".",".","6",".",".",".","3"],
  ["4",".",".","8",".","3",".",".","1"],
  ["7",".",".",".","2",".",".",".","6"],
  [".","6",".",".",".",".","2","8","."],
  [".",".",".","4","1","9",".",".","5"],
  [".",".",".",".","8",".",".","7","9"]
];

console.log('Original:');
puzzle.forEach(row => console.log(row.join(' ')));

solveSudoku(puzzle);

console.log('\nSolved:');
puzzle.forEach(row => console.log(row.join(' ')));
```

### **Approach 2: Backtracking with Optimization**
```javascript
/**
 * Optimized with constraint tracking
 */
class SudokuSolver {
  constructor(board) {
    this.board = board;
    this.rows = Array(9).fill(0).map(() => new Set());
    this.cols = Array(9).fill(0).map(() => new Set());
    this.boxes = Array(9).fill(0).map(() => new Set());
    
    // Initialize constraints
    for (let i = 0; i < 9; i++) {
      for (let j = 0; j < 9; j++) {
        if (board[i][j] !== '.') {
          const num = board[i][j];
          const boxIndex = Math.floor(i / 3) * 3 + Math.floor(j / 3);
          
          this.rows[i].add(num);
          this.cols[j].add(num);
          this.boxes[boxIndex].add(num);
        }
      }
    }
  }
  
  isValid(row, col, num) {
    const boxIndex = Math.floor(row / 3) * 3 + Math.floor(col / 3);
    
    return !this.rows[row].has(num) && 
           !this.cols[col].has(num) && 
           !this.boxes[boxIndex].has(num);
  }
  
  placeNumber(row, col, num) {
    const boxIndex = Math.floor(row / 3) * 3 + Math.floor(col / 3);
    
    this.board[row][col] = num;
    this.rows[row].add(num);
    this.cols[col].add(num);
    this.boxes[boxIndex].add(num);
  }
  
  removeNumber(row, col, num) {
    const boxIndex = Math.floor(row / 3) * 3 + Math.floor(col / 3);
    
    this.board[row][col] = '.';
    this.rows[row].delete(num);
    this.cols[col].delete(num);
    this.boxes[boxIndex].delete(num);
  }
  
  solve() {
    // Find next empty cell
    for (let row = 0; row < 9; row++) {
      for (let col = 0; col < 9; col++) {
        if (this.board[row][col] === '.') {
          // Try each number
          for (let num = 1; num <= 9; num++) {
            const char = num.toString();
            
            if (this.isValid(row, col, char)) {
              this.placeNumber(row, col, char);
              
              if (this.solve()) {
                return true;
              }
              
              this.removeNumber(row, col, char);
            }
          }
          
          return false;
        }
      }
    }
    
    return true;
  }
}

// Test
console.log('\n=== Optimized Solver ===');

const puzzle2 = [
  [".",".","9","7","4","8",".",".","."],
  ["7",".",".",".",".",".",".",".","."],
  [".","2",".","1",".","9",".",".","."],
  [".",".","7",".",".",".","2","4","."],
  [".","6","4",".","1",".","5","9","."],
  [".","9","8",".",".",".","3",".","."],
  [".",".",".","8",".","3",".","2","."],
  [".",".",".",".",".",".",".",".","6"],
  [".",".",".","2","7","5","9",".","."]
];

const solver = new SudokuSolver(puzzle2);
solver.solve();

console.log('Solved:');
solver.board.forEach(row => console.log(row.join(' ')));
```

### **Approach 3: Find All Solutions**
```javascript
/**
 * Find all possible solutions (some puzzles have multiple)
 */
function findAllSudokuSolutions(board) {
  const solutions = [];
  
  function isValid(board, row, col, num) {
    // Check row
    for (let x = 0; x < 9; x++) {
      if (board[row][x] === num) return false;
    }
    
    // Check column
    for (let x = 0; x < 9; x++) {
      if (board[x][col] === num) return false;
    }
    
    // Check box
    const boxRow = Math.floor(row / 3) * 3;
    const boxCol = Math.floor(col / 3) * 3;
    
    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        if (board[boxRow + i][boxCol + j] === num) return false;
      }
    }
    
    return true;
  }
  
  function solve(board) {
    for (let row = 0; row < 9; row++) {
      for (let col = 0; col < 9; col++) {
        if (board[row][col] === '.') {
          for (let num = 1; num <= 9; num++) {
            const char = num.toString();
            
            if (isValid(board, row, col, char)) {
              board[row][col] = char;
              solve(board);
              board[row][col] = '.';
            }
          }
          
          return;
        }
      }
    }
    
    // Found complete solution
    solutions.push(board.map(row => [...row]));
  }
  
  solve(board);
  return solutions;
}

// Test
console.log('\n=== Find All Solutions ===');

const simplePuzzle = [
  [".",".",".",".",".",".",".",".","1"],
  [".",".",".",".",".",".",".",".","2"],
  [".",".",".",".",".",".",".",".","3"],
  [".",".",".",".",".",".",".",".","4"],
  [".",".",".",".",".",".",".",".","5"],
  [".",".",".",".",".",".",".",".","6"],
  [".",".",".",".",".",".",".",".","7"],
  [".",".",".",".",".",".",".",".","8"],
  ["1","2","3","4","5","6","7","8","."]
];

const allSolutions = findAllSudokuSolutions(simplePuzzle);
console.log(`Found ${allSolutions.length} solution(s)`);
```

### **Approach 4: Validate Sudoku Board**
```javascript
/**
 * Validate if a Sudoku board is valid (not necessarily complete)
 */
function isValidSudoku(board) {
  // Check rows
  for (let row = 0; row < 9; row++) {
    const seen = new Set();
    for (let col = 0; col < 9; col++) {
      const num = board[row][col];
      if (num !== '.') {
        if (seen.has(num)) return false;
        seen.add(num);
      }
    }
  }
  
  // Check columns
  for (let col = 0; col < 9; col++) {
    const seen = new Set();
    for (let row = 0; row < 9; row++) {
      const num = board[row][col];
      if (num !== '.') {
        if (seen.has(num)) return false;
        seen.add(num);
      }
    }
  }
  
  // Check 3x3 boxes
  for (let box = 0; box < 9; box++) {
    const seen = new Set();
    const boxRow = Math.floor(box / 3) * 3;
    const boxCol = (box % 3) * 3;
    
    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        const num = board[boxRow + i][boxCol + j];
        if (num !== '.') {
          if (seen.has(num)) return false;
          seen.add(num);
        }
      }
    }
  }
  
  return true;
}

// Test
console.log('\n=== Validate Sudoku ===');

const validBoard = [
  ["5","3",".",".","7",".",".",".","."],
  ["6",".",".","1","9","5",".",".","."],
  [".","9","8",".",".",".",".","6","."],
  ["8",".",".",".","6",".",".",".","3"],
  ["4",".",".","8",".","3",".",".","1"],
  ["7",".",".",".","2",".",".",".","6"],
  [".","6",".",".",".",".","2","8","."],
  [".",".",".","4","1","9",".",".","5"],
  [".",".",".",".","8",".",".","7","9"]
];

const invalidBoard = [
  ["5","3",".",".","7",".",".",".","."],
  ["6",".",".","1","9","5",".",".","."],
  [".","9","8",".",".",".",".","6","."],
  ["8",".",".",".","6",".",".",".","3"],
  ["4",".",".","8",".","3",".",".","1"],
  ["7",".",".",".","2",".",".",".","6"],
  [".","6",".",".",".",".","2","8","."],
  [".",".",".","4","1","9",".",".","5"],
  ["5",".",".",".","8",".",".","7","9"] // Duplicate 5 in column 0
];

console.log('Valid board:', isValidSudoku(validBoard)); // true
console.log('Invalid board:', isValidSudoku(invalidBoard)); // false
```

### **Approach 5: Generate Sudoku Puzzle**
```javascript
/**
 * Generate a random valid Sudoku puzzle
 */
class SudokuGenerator {
  generateSolution() {
    const board = Array(9).fill(null).map(() => Array(9).fill('.'));
    this.fillBoard(board);
    return board;
  }
  
  fillBoard(board) {
    for (let row = 0; row < 9; row++) {
      for (let col = 0; col < 9; col++) {
        if (board[row][col] === '.') {
          const numbers = this.shuffleArray([1, 2, 3, 4, 5, 6, 7, 8, 9]);
          
          for (const num of numbers) {
            if (this.isValid(board, row, col, num.toString())) {
              board[row][col] = num.toString();
              
              if (this.fillBoard(board)) {
                return true;
              }
              
              board[row][col] = '.';
            }
          }
          
          return false;
        }
      }
    }
    
    return true;
  }
  
  isValid(board, row, col, num) {
    // Check row
    for (let x = 0; x < 9; x++) {
      if (board[row][x] === num) return false;
    }
    
    // Check column
    for (let x = 0; x < 9; x++) {
      if (board[x][col] === num) return false;
    }
    
    // Check box
    const boxRow = Math.floor(row / 3) * 3;
    const boxCol = Math.floor(col / 3) * 3;
    
    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        if (board[boxRow + i][boxCol + j] === num) return false;
      }
    }
    
    return true;
  }
  
  generatePuzzle(difficulty = 'medium') {
    const solution = this.generateSolution();
    const puzzle = solution.map(row => [...row]);
    
    // Remove cells based on difficulty
    const cellsToRemove = {
      easy: 30,
      medium: 40,
      hard: 50,
      expert: 55
    }[difficulty] || 40;
    
    let removed = 0;
    const cells = [];
    
    for (let i = 0; i < 9; i++) {
      for (let j = 0; j < 9; j++) {
        cells.push([i, j]);
      }
    }
    
    this.shuffleArray(cells);
    
    for (const [row, col] of cells) {
      if (removed >= cellsToRemove) break;
      
      puzzle[row][col] = '.';
      removed++;
    }
    
    return { puzzle, solution };
  }
  
  shuffleArray(array) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    return array;
  }
}

// Test
console.log('\n=== Generate Sudoku ===');

const generator = new SudokuGenerator();
const { puzzle: newPuzzle, solution } = generator.generatePuzzle('medium');

console.log('Generated puzzle:');
newPuzzle.forEach(row => console.log(row.join(' ')));

console.log('\nSolution:');
solution.forEach(row => console.log(row.join(' ')));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications beyond puzzles
 */

// 1. Constraint Satisfaction Problem Solver
console.log('\n=== CSP Solver ===');

class ConstraintSolver {
  constructor(variables, domains, constraints) {
    this.variables = variables;
    this.domains = domains;
    this.constraints = constraints;
  }
  
  solve(assignment = {}) {
    if (Object.keys(assignment).length === this.variables.length) {
      return assignment;
    }
    
    const variable = this.selectUnassignedVariable(assignment);
    
    for (const value of this.domains[variable]) {
      if (this.isConsistent(variable, value, assignment)) {
        assignment[variable] = value;
        
        const result = this.solve(assignment);
        if (result) return result;
        
        delete assignment[variable];
      }
    }
    
    return null;
  }
  
  selectUnassignedVariable(assignment) {
    for (const variable of this.variables) {
      if (!(variable in assignment)) {
        return variable;
      }
    }
  }
  
  isConsistent(variable, value, assignment) {
    for (const constraint of this.constraints) {
      if (!constraint(variable, value, assignment)) {
        return false;
      }
    }
    return true;
  }
}

// Example: Map coloring
const countries = ['A', 'B', 'C', 'D'];
const colors = { A: ['red', 'green', 'blue'], B: ['red', 'green', 'blue'], 
                 C: ['red', 'green', 'blue'], D: ['red', 'green', 'blue'] };
const adjacency = { A: ['B', 'C'], B: ['A', 'C', 'D'], C: ['A', 'B', 'D'], D: ['B', 'C'] };

const constraint = (var1, val1, assignment) => {
  for (const neighbor of adjacency[var1] || []) {
    if (neighbor in assignment && assignment[neighbor] === val1) {
      return false;
    }
  }
  return true;
};

const csp = new ConstraintSolver(countries, colors, [constraint]);
console.log('Map coloring solution:', csp.solve());

// 2. Schedule Generator
console.log('\n=== Schedule Generator ===');

class ScheduleGenerator {
  constructor(tasks, resources, constraints) {
    this.tasks = tasks;
    this.resources = resources;
    this.constraints = constraints;
  }
  
  generateSchedule() {
    const schedule = {};
    return this.backtrack(schedule, 0);
  }
  
  backtrack(schedule, taskIndex) {
    if (taskIndex === this.tasks.length) {
      return { ...schedule };
    }
    
    const task = this.tasks[taskIndex];
    
    for (const resource of this.resources) {
      for (const timeSlot of resource.availableSlots) {
        if (this.canSchedule(task, resource, timeSlot, schedule)) {
          schedule[task.id] = { resource: resource.id, time: timeSlot };
          
          const result = this.backtrack(schedule, taskIndex + 1);
          if (result) return result;
          
          delete schedule[task.id];
        }
      }
    }
    
    return null;
  }
  
  canSchedule(task, resource, timeSlot, schedule) {
    // Check resource capacity
    if (!resource.canHandle(task)) return false;
    
    // Check time conflicts
    for (const [taskId, assignment] of Object.entries(schedule)) {
      if (assignment.resource === resource.id && assignment.time === timeSlot) {
        return false;
      }
    }
    
    return true;
  }
}

// Example usage
const tasks = [
  { id: 'T1', duration: 2 },
  { id: 'T2', duration: 1 },
  { id: 'T3', duration: 3 }
];

const resources = [
  { id: 'R1', availableSlots: [9, 10, 11, 12], canHandle: (task) => true },
  { id: 'R2', availableSlots: [9, 10, 11, 12], canHandle: (task) => true }
];

const scheduler = new ScheduleGenerator(tasks, resources, []);
console.log('Schedule:', scheduler.generateSchedule());

// 3. Puzzle Game Engine
console.log('\n=== Puzzle Game Engine ===');

class PuzzleGame {
  constructor() {
    this.generator = new SudokuGenerator();
  }
  
  startGame(difficulty = 'medium') {
    const { puzzle, solution } = this.generator.generatePuzzle(difficulty);
    
    return {
      puzzle,
      solution,
      moves: [],
      hints: 3,
      startTime: Date.now()
    };
  }
  
  makeMove(game, row, col, value) {
    if (game.puzzle[row][col] !== '.') {
      return { success: false, error: 'Cell is fixed' };
    }
    
    game.puzzle[row][col] = value;
    game.moves.push({ row, col, value, time: Date.now() });
    
    const isCorrect = value === game.solution[row][col];
    
    return { 
      success: true, 
      correct: isCorrect,
      complete: this.isComplete(game.puzzle)
    };
  }
  
  getHint(game) {
    if (game.hints <= 0) {
      return { success: false, error: 'No hints remaining' };
    }
    
    // Find empty cell
    for (let i = 0; i < 9; i++) {
      for (let j = 0; j < 9; j++) {
        if (game.puzzle[i][j] === '.') {
          game.hints--;
          return {
            success: true,
            hint: { row: i, col: j, value: game.solution[i][j] },
            hintsRemaining: game.hints
          };
        }
      }
    }
    
    return { success: false, error: 'Puzzle already complete' };
  }
  
  isComplete(puzzle) {
    for (let i = 0; i < 9; i++) {
      for (let j = 0; j < 9; j++) {
        if (puzzle[i][j] === '.') return false;
      }
    }
    return true;
  }
}

const gameEngine = new PuzzleGame();
const game = gameEngine.startGame('easy');
console.log('Game started with', game.hints, 'hints available');

const hint = gameEngine.getHint(game);
console.log('Hint:', hint);
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty board (should fill entire board)
const emptyBoard = Array(9).fill(null).map(() => Array(9).fill('.'));
console.log('Can solve empty board:', solveSudoku(emptyBoard) !== null);

// Invalid board (no solution)
const invalidPuzzle = [
  ["5","5",".",".","7",".",".",".","."], // Two 5s in first row
  ["6",".",".","1","9","5",".",".","."],
  [".","9","8",".",".",".",".","6","."],
  ["8",".",".",".","6",".",".",".","3"],
  ["4",".",".","8",".","3",".",".","1"],
  ["7",".",".",".","2",".",".",".","6"],
  [".","6",".",".",".",".","2","8","."],
  [".",".",".","4","1","9",".",".","5"],
  [".",".",".",".","8",".",".","7","9"]
];

console.log('Invalid board validation:', isValidSudoku(invalidPuzzle)); // false

// Almost complete board (one cell empty)
const almostComplete = [
  ["5","3","4","6","7","8","9","1","2"],
  ["6","7","2","1","9","5","3","4","8"],
  ["1","9","8","3","4","2","5","6","7"],
  ["8","5","9","7","6","1","4","2","3"],
  ["4","2","6","8","5","3","7","9","1"],
  ["7","1","3","9","2","4","8","5","6"],
  ["9","6","1","5","3","7","2","8","4"],
  ["2","8","7","4","1","9","6","3","5"],
  ["3","4","5",".","8","6","1","7","9"] // Only last cell empty
];

console.log('Almost complete:');
solveSudoku(almostComplete);
console.log(almostComplete[8][3]); // Should be '2'
```

### **Performance Analysis**
```javascript
console.log('\n=== Performance ===');

console.log(`
Sudoku Solver Complexity:
┌──────────────────────┬─────────────┬─────────────┐
│ Operation            │ Time        │ Space       │
├──────────────────────┼─────────────┼─────────────┤
│ Backtracking         │ O(9^m)      │ O(m)        │
│ Validation           │ O(1)        │ O(1)        │
│ With constraints     │ O(9^m)*     │ O(81)       │
│ Generate puzzle      │ O(9^81)     │ O(81)       │
└──────────────────────┴─────────────┴─────────────┘

Where m = number of empty cells
* Constraint tracking provides constant-time validation

Optimizations:
• Maintain sets for rows/cols/boxes (O(1) validation)
• Choose cell with fewest candidates (MRV heuristic)
• Constraint propagation (naked singles, hidden singles)
• Dancing Links (Algorithm X) for advanced cases
• Bit manipulation for faster set operations

Heuristics:
• MRV: Choose variable with minimum remaining values
• Degree: Choose variable involved in most constraints
• LCV: Try least constraining value first

Applications:
• Constraint satisfaction problems
• Resource allocation
• Scheduling
• Graph coloring
• N-Queens problem
• Puzzle games
`);
```

**Interview Tips:**
- Sudoku: 9×9 grid, fill with 1-9, each row/col/box has unique numbers
- Backtracking: classic approach, try each number, recurse, backtrack if invalid
- Three constraints: row (no duplicates), column (no duplicates), 3×3 box (no duplicates)
- Algorithm: find empty cell, try 1-9, check validity, recurse, backtrack on failure
- Time: O(9^m) where m is empty cells, worst case 9^81
- Space: O(m) for recursion stack
- Validation: check row, column, and 3×3 box for duplicates
- Box calculation: boxRow = (row/3)*3, boxCol = (col/3)*3
- Optimization: track constraints in sets for O(1) validation vs O(27) checking
- Box index: floor(row/3)*3 + floor(col/3)
- Base case: no empty cells left = solution found
- Backtrack: if no valid number works, return false and undo
- In-place vs copy: can modify board in place or create copy
- Early termination: return immediately when solution found
- Applications: CSP solver, scheduling, resource allocation, game engine
- Validation: separate function to check if board state is valid
- Generation: fill board randomly with backtracking, then remove cells
- Difficulty: more cells removed = harder puzzle
- MRV heuristic: choose cell with minimum remaining valid values
- Edge cases: no solution, multiple solutions, invalid input
- Optimization: naked singles (only one possibility), hidden singles
- Follow-ups: count solutions, validate board, generate puzzle, solve variants (16×16, irregular)
- Clarify: modify in place? multiple solutions? validate only? generate puzzle?

</details>

108. Solve the "Word Ladder" problem

<details>
<summary><b>Solution</b></summary>

### **Approach 1: BFS (Shortest Path)**
```javascript
/**
 * Word Ladder using BFS to find shortest transformation
 * Time Complexity: O(M² × N) where M is word length, N is word list size
 * Space Complexity: O(N)
 */
function ladderLength(beginWord, endWord, wordList) {
  const wordSet = new Set(wordList);
  
  if (!wordSet.has(endWord)) {
    return 0; // No transformation possible
  }
  
  const queue = [[beginWord, 1]]; // [word, level]
  const visited = new Set([beginWord]);
  
  while (queue.length > 0) {
    const [word, level] = queue.shift();
    
    if (word === endWord) {
      return level;
    }
    
    // Try changing each character
    for (let i = 0; i < word.length; i++) {
      for (let c = 97; c <= 122; c++) { // a-z
        const char = String.fromCharCode(c);
        const newWord = word.slice(0, i) + char + word.slice(i + 1);
        
        if (wordSet.has(newWord) && !visited.has(newWord)) {
          queue.push([newWord, level + 1]);
          visited.add(newWord);
        }
      }
    }
  }
  
  return 0; // No transformation found
}

// Test
console.log('=== Word Ladder (BFS) ===');

console.log(ladderLength('hit', 'cog', ['hot', 'dot', 'dog', 'lot', 'log', 'cog'])); // 5
// hit -> hot -> dot -> dog -> cog

console.log(ladderLength('hit', 'cog', ['hot', 'dot', 'dog', 'lot', 'log'])); // 0
// endWord not in list

console.log(ladderLength('a', 'c', ['a', 'b', 'c'])); // 2
// a -> c
```

### **Approach 2: Bidirectional BFS (Optimized)**
```javascript
/**
 * Bidirectional BFS - search from both ends
 * More efficient for long paths
 */
function ladderLengthBidirectional(beginWord, endWord, wordList) {
  const wordSet = new Set(wordList);
  
  if (!wordSet.has(endWord)) {
    return 0;
  }
  
  let beginSet = new Set([beginWord]);
  let endSet = new Set([endWord]);
  const visited = new Set();
  let level = 1;
  
  while (beginSet.size > 0 && endSet.size > 0) {
    // Always expand smaller set
    if (beginSet.size > endSet.size) {
      [beginSet, endSet] = [endSet, beginSet];
    }
    
    const nextSet = new Set();
    
    for (const word of beginSet) {
      for (let i = 0; i < word.length; i++) {
        for (let c = 97; c <= 122; c++) {
          const char = String.fromCharCode(c);
          const newWord = word.slice(0, i) + char + word.slice(i + 1);
          
          if (endSet.has(newWord)) {
            return level + 1; // Found connection
          }
          
          if (wordSet.has(newWord) && !visited.has(newWord)) {
            nextSet.add(newWord);
            visited.add(newWord);
          }
        }
      }
    }
    
    beginSet = nextSet;
    level++;
  }
  
  return 0;
}

// Test
console.log('\n=== Bidirectional BFS ===');

console.log(ladderLengthBidirectional('hit', 'cog', ['hot', 'dot', 'dog', 'lot', 'log', 'cog'])); // 5
console.log(ladderLengthBidirectional('red', 'tax', ['ted', 'tex', 'red', 'tax', 'tad', 'den', 'rex', 'pee'])); // 4
```

### **Approach 3: Find All Shortest Paths**
```javascript
/**
 * Find all shortest transformation sequences
 */
function findLadders(beginWord, endWord, wordList) {
  const wordSet = new Set(wordList);
  
  if (!wordSet.has(endWord)) {
    return [];
  }
  
  // BFS to build parent map
  const parents = new Map();
  const queue = [beginWord];
  const visited = new Set([beginWord]);
  let found = false;
  
  while (queue.length > 0 && !found) {
    const levelSize = queue.length;
    const levelVisited = new Set();
    
    for (let i = 0; i < levelSize; i++) {
      const word = queue.shift();
      
      // Generate neighbors
      for (let j = 0; j < word.length; j++) {
        for (let c = 97; c <= 122; c++) {
          const char = String.fromCharCode(c);
          const newWord = word.slice(0, j) + char + word.slice(j + 1);
          
          if (newWord === endWord) {
            found = true;
          }
          
          if (wordSet.has(newWord) && !visited.has(newWord)) {
            if (!parents.has(newWord)) {
              parents.set(newWord, []);
            }
            parents.get(newWord).push(word);
            
            if (!levelVisited.has(newWord)) {
              queue.push(newWord);
              levelVisited.add(newWord);
            }
          }
        }
      }
    }
    
    // Mark all level words as visited after processing level
    for (const word of levelVisited) {
      visited.add(word);
    }
  }
  
  if (!found) {
    return [];
  }
  
  // Backtrack to build all paths
  const result = [];
  
  function backtrack(word, path) {
    if (word === beginWord) {
      result.push([...path].reverse());
      return;
    }
    
    if (parents.has(word)) {
      for (const parent of parents.get(word)) {
        path.push(parent);
        backtrack(parent, path);
        path.pop();
      }
    }
  }
  
  backtrack(endWord, [endWord]);
  return result;
}

// Test
console.log('\n=== All Shortest Paths ===');

const paths = findLadders('hit', 'cog', ['hot', 'dot', 'dog', 'lot', 'log', 'cog']);
console.log('All shortest paths:');
paths.forEach(path => console.log(path.join(' -> ')));
```

### **Approach 4: Word Ladder with Pre-computed Patterns**
```javascript
/**
 * Pre-compute generic patterns for faster neighbor lookup
 * Pattern: "h*t" matches "hit", "hot", "hat", etc.
 */
function ladderLengthWithPatterns(beginWord, endWord, wordList) {
  if (!wordList.includes(endWord)) {
    return 0;
  }
  
  // Build pattern map
  const patterns = new Map();
  const allWords = [beginWord, ...wordList];
  
  for (const word of allWords) {
    for (let i = 0; i < word.length; i++) {
      const pattern = word.slice(0, i) + '*' + word.slice(i + 1);
      
      if (!patterns.has(pattern)) {
        patterns.set(pattern, []);
      }
      patterns.get(pattern).push(word);
    }
  }
  
  // BFS with pattern lookup
  const queue = [[beginWord, 1]];
  const visited = new Set([beginWord]);
  
  while (queue.length > 0) {
    const [word, level] = queue.shift();
    
    if (word === endWord) {
      return level;
    }
    
    // Get all neighbors via patterns
    for (let i = 0; i < word.length; i++) {
      const pattern = word.slice(0, i) + '*' + word.slice(i + 1);
      
      if (patterns.has(pattern)) {
        for (const neighbor of patterns.get(pattern)) {
          if (!visited.has(neighbor)) {
            queue.push([neighbor, level + 1]);
            visited.add(neighbor);
          }
        }
      }
    }
  }
  
  return 0;
}

// Test
console.log('\n=== Pattern-Based BFS ===');

console.log(ladderLengthWithPatterns('hit', 'cog', ['hot', 'dot', 'dog', 'lot', 'log', 'cog'])); // 5
```

### **Approach 5: Build Word Graph**
```javascript
/**
 * Build explicit graph of word connections
 */
class WordGraph {
  constructor(wordList) {
    this.graph = new Map();
    this.buildGraph(wordList);
  }
  
  buildGraph(wordList) {
    // Initialize graph
    for (const word of wordList) {
      this.graph.set(word, []);
    }
    
    // Add edges between words that differ by one character
    for (let i = 0; i < wordList.length; i++) {
      for (let j = i + 1; j < wordList.length; j++) {
        if (this.differByOne(wordList[i], wordList[j])) {
          this.graph.get(wordList[i]).push(wordList[j]);
          this.graph.get(wordList[j]).push(wordList[i]);
        }
      }
    }
  }
  
  differByOne(word1, word2) {
    if (word1.length !== word2.length) {
      return false;
    }
    
    let diffs = 0;
    for (let i = 0; i < word1.length; i++) {
      if (word1[i] !== word2[i]) {
        diffs++;
        if (diffs > 1) return false;
      }
    }
    
    return diffs === 1;
  }
  
  shortestPath(start, end) {
    if (!this.graph.has(start) || !this.graph.has(end)) {
      return 0;
    }
    
    const queue = [[start, 1]];
    const visited = new Set([start]);
    
    while (queue.length > 0) {
      const [word, dist] = queue.shift();
      
      if (word === end) {
        return dist;
      }
      
      for (const neighbor of this.graph.get(word)) {
        if (!visited.has(neighbor)) {
          queue.push([neighbor, dist + 1]);
          visited.add(neighbor);
        }
      }
    }
    
    return 0;
  }
}

// Test
console.log('\n=== Word Graph ===');

const wordGraph = new WordGraph(['hot', 'dot', 'dog', 'lot', 'log', 'cog']);
wordGraph.graph.set('hit', ['hot']);

console.log('Shortest path (graph):', wordGraph.shortestPath('hit', 'cog'));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications of word transformation
 */

// 1. Spell Checker - Suggest Similar Words
console.log('\n=== Spell Checker ===');

class SpellChecker {
  constructor(dictionary) {
    this.dictionary = new Set(dictionary);
  }
  
  suggestCorrections(misspelled, maxDistance = 2) {
    const suggestions = [];
    
    // BFS to find words within edit distance
    const queue = [[misspelled, 0]];
    const visited = new Set([misspelled]);
    
    while (queue.length > 0) {
      const [word, dist] = queue.shift();
      
      if (dist > maxDistance) continue;
      
      if (this.dictionary.has(word) && word !== misspelled) {
        suggestions.push({ word, distance: dist });
      }
      
      // Generate one-edit neighbors
      for (let i = 0; i < word.length; i++) {
        for (let c = 97; c <= 122; c++) {
          const char = String.fromCharCode(c);
          const newWord = word.slice(0, i) + char + word.slice(i + 1);
          
          if (!visited.has(newWord)) {
            queue.push([newWord, dist + 1]);
            visited.add(newWord);
          }
        }
      }
    }
    
    return suggestions.sort((a, b) => a.distance - b.distance);
  }
}

const spellChecker = new SpellChecker(['cat', 'bat', 'hat', 'mat', 'rat', 'fat', 'sat']);
console.log('Suggestions for "cot":', spellChecker.suggestCorrections('cot'));

// 2. Text Transformation Chain
console.log('\n=== Text Transformation ===');

class TextTransformer {
  constructor(validWords) {
    this.validWords = new Set(validWords);
  }
  
  findTransformationChain(source, target) {
    if (!this.validWords.has(target)) {
      return null;
    }
    
    const queue = [[source, [source]]];
    const visited = new Set([source]);
    
    while (queue.length > 0) {
      const [word, path] = queue.shift();
      
      if (word === target) {
        return path;
      }
      
      for (let i = 0; i < word.length; i++) {
        for (let c = 97; c <= 122; c++) {
          const char = String.fromCharCode(c);
          const newWord = word.slice(0, i) + char + word.slice(i + 1);
          
          if (this.validWords.has(newWord) && !visited.has(newWord)) {
            queue.push([newWord, [...path, newWord]]);
            visited.add(newWord);
          }
        }
      }
    }
    
    return null;
  }
}

const transformer = new TextTransformer(['code', 'cede', 'core', 'care', 'cape', 'tape']);
const chain = transformer.findTransformationChain('code', 'tape');
console.log('Transformation chain:', chain ? chain.join(' -> ') : 'No path found');

// 3. Word Game Solver
console.log('\n=== Word Game Solver ===');

class WordGameSolver {
  constructor(dictionary) {
    this.dictionary = dictionary;
  }
  
  solveWordChain(start, end, maxMoves) {
    const wordSet = new Set(this.dictionary);
    
    if (!wordSet.has(end)) {
      return { possible: false };
    }
    
    const queue = [[start, 1, [start]]];
    const visited = new Set([start]);
    
    while (queue.length > 0) {
      const [word, moves, path] = queue.shift();
      
      if (moves > maxMoves) {
        continue;
      }
      
      if (word === end) {
        return {
          possible: true,
          moves: moves,
          path: path,
          score: this.calculateScore(path)
        };
      }
      
      for (let i = 0; i < word.length; i++) {
        for (let c = 97; c <= 122; c++) {
          const char = String.fromCharCode(c);
          const newWord = word.slice(0, i) + char + word.slice(i + 1);
          
          if (wordSet.has(newWord) && !visited.has(newWord)) {
            queue.push([newWord, moves + 1, [...path, newWord]]);
            visited.add(newWord);
          }
        }
      }
    }
    
    return { possible: false };
  }
  
  calculateScore(path) {
    // Score based on word rarity (length in this example)
    return path.reduce((sum, word) => sum + word.length, 0);
  }
}

const gameSolver = new WordGameSolver(['hit', 'hot', 'dot', 'dog', 'cog']);
const solution = gameSolver.solveWordChain('hit', 'cog', 5);
console.log('Game solution:', solution);

// 4. Password Strength Analyzer
console.log('\n=== Password Strength ===');

class PasswordAnalyzer {
  constructor(commonWords) {
    this.commonWords = new Set(commonWords);
  }
  
  analyzeStrength(password) {
    const strength = {
      score: 100,
      warnings: []
    };
    
    // Check proximity to common words
    const minDistance = this.minDistanceToCommon(password);
    
    if (minDistance <= 2) {
      strength.score -= 30;
      strength.warnings.push('Too similar to common words');
    }
    
    return strength;
  }
  
  minDistanceToCommon(password) {
    let minDist = Infinity;
    
    for (const word of this.commonWords) {
      if (word.length !== password.length) continue;
      
      const dist = this.hammingDistance(password, word);
      minDist = Math.min(minDist, dist);
    }
    
    return minDist;
  }
  
  hammingDistance(s1, s2) {
    let dist = 0;
    for (let i = 0; i < s1.length; i++) {
      if (s1[i] !== s2[i]) dist++;
    }
    return dist;
  }
}

const pwdAnalyzer = new PasswordAnalyzer(['password', 'qwerty', 'admin']);
console.log(pwdAnalyzer.analyzeStrength('passwprd')); // Too similar
console.log(pwdAnalyzer.analyzeStrength('x9k2mQ7!')); // Strong

// 5. Gene Mutation Tracker
console.log('\n=== Gene Mutation ===');

class GeneMutationTracker {
  constructor(validGenes) {
    this.validGenes = new Set(validGenes);
  }
  
  findMutationPath(startGene, endGene) {
    if (!this.validGenes.has(endGene)) {
      return null;
    }
    
    const queue = [[startGene, [startGene]]];
    const visited = new Set([startGene]);
    
    // Valid gene bases: A, C, G, T
    const bases = ['A', 'C', 'G', 'T'];
    
    while (queue.length > 0) {
      const [gene, path] = queue.shift();
      
      if (gene === endGene) {
        return {
          mutations: path.length - 1,
          path: path,
          steps: path.map((g, i) => ({
            step: i,
            gene: g,
            mutation: i > 0 ? this.getMutation(path[i-1], g) : null
          }))
        };
      }
      
      // Try single base mutations
      for (let i = 0; i < gene.length; i++) {
        for (const base of bases) {
          const mutated = gene.slice(0, i) + base + gene.slice(i + 1);
          
          if (this.validGenes.has(mutated) && !visited.has(mutated)) {
            queue.push([mutated, [...path, mutated]]);
            visited.add(mutated);
          }
        }
      }
    }
    
    return null;
  }
  
  getMutation(from, to) {
    for (let i = 0; i < from.length; i++) {
      if (from[i] !== to[i]) {
        return { position: i, from: from[i], to: to[i] };
      }
    }
    return null;
  }
}

const geneTracker = new GeneMutationTracker(['AAAT', 'AACT', 'ACCT', 'CCCT', 'CCCC']);
const mutationPath = geneTracker.findMutationPath('AAAT', 'CCCC');
console.log('Mutation path:', mutationPath);
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Same start and end
console.log('Same word:', ladderLength('hit', 'hit', ['hit'])); // 1

// No valid path
console.log('No path:', ladderLength('hit', 'cog', ['hot', 'dot', 'lot'])); // 0

// EndWord not in list
console.log('End not in list:', ladderLength('hit', 'cog', ['hot', 'dot'])); // 0

// Direct transformation (one step)
console.log('One step:', ladderLength('hit', 'hot', ['hot'])); // 2

// Empty word list
console.log('Empty list:', ladderLength('a', 'b', [])); // 0

// Single character words
console.log('Single char:', ladderLength('a', 'c', ['a', 'b', 'c'])); // 2

// Long words
const longWords = ['abcde', 'abcdf', 'abcef', 'abdef', 'abeef'];
console.log('Long words:', ladderLength('abcde', 'abeef', longWords)); // 3

// Large word list
const largeList = [];
for (let i = 0; i < 100; i++) {
  largeList.push('w' + i.toString().padStart(3, '0'));
}
console.log('Large list:', ladderLength('w000', 'w099', largeList)); // 0 (no connections)

// All words differ by one
const connected = ['aaa', 'aab', 'abb', 'bbb'];
console.log('Connected:', ladderLength('aaa', 'bbb', connected)); // 4
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Word Ladder Algorithms:
┌──────────────────────┬─────────────────┬─────────────┬────────────┐
│ Approach             │ Time            │ Space       │ Best For   │
├──────────────────────┼─────────────────┼─────────────┼────────────┤
│ BFS                  │ O(M² × N)       │ O(N)        │ Standard   │
│ Bidirectional BFS    │ O(M² × N/2)     │ O(N)        │ Long paths │
│ Pattern-based        │ O(M × N)        │ O(M × N)    │ Preprocessing│
│ Word Graph           │ O(N² × M)       │ O(N²)       │ Multiple   │
│                      │ build + O(E)    │             │ queries    │
└──────────────────────┴─────────────────┴─────────────┴────────────┘

Where:
  M = word length
  N = number of words in list
  E = edges in graph

Key Optimizations:
• Bidirectional BFS: expand from both ends
• Pattern matching: pre-compute generic patterns (*ot, h*t, ho*)
• Early termination: stop at first solution in BFS
• Use Set for O(1) word lookup
• Visited tracking to avoid cycles

Trade-offs:
• BFS: simple, finds shortest path
• Bidirectional: faster for long paths
• Graph: overhead to build but fast queries
• Pattern-based: faster neighbor generation

Applications:
• Spell checking and autocorrection
• Text transformation chains
• Word games (Wordle, Word Ladder)
• Gene sequence mutation analysis
• Password strength analysis
`);
```

**Interview Tips:**
- Word ladder: transform one word to another, changing one letter at a time
- Each intermediate word must exist in dictionary
- Goal: find shortest transformation sequence (BFS problem)
- BFS guarantees shortest path in unweighted graph
- Time: O(M² × N) - M for word comparison, M for generating neighbors, N words
- Space: O(N) for queue and visited set
- Key insight: treat as graph problem, words are nodes, edges between one-letter differences
- Generate neighbors: try replacing each position with a-z
- Use Set for O(1) dictionary lookup
- Track visited to avoid cycles
- Queue stores [word, level] or [word, path]
- Level/distance tracking: increment when dequeuing
- Bidirectional BFS: expand from both start and end, faster for long paths
- Always expand smaller frontier in bidirectional BFS
- Pattern approach: "h*t" matches all words differing at position 1
- Pre-computing patterns: O(M × N) space but faster neighbor lookup
- All shortest paths: use parent tracking during BFS, backtrack at end
- Edge cases: start equals end, no path, endWord not in list, empty list
- Common mistake: forgetting to check if endWord is in dictionary
- Return 0 if no path (vs -1 or null), check problem statement
- Applications: spell checker, text games, gene mutations, password analysis
- Follow-ups: find all shortest paths, count paths, with word costs, different alphabet
- Optimization: use bidirectional BFS for better performance
- Clarify: shortest path or any path? count transformations or return sequence?

</details>

109. Implement a regular expression matcher (basic)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Recursive Matching**
```javascript
/**
 * Basic regex matcher with . (any char) and * (zero or more)
 * Time Complexity: O(2^(m+n)) worst case
 * Space Complexity: O(m+n) for recursion
 */
function isMatch(text, pattern) {
  // Base case: empty pattern
  if (pattern.length === 0) {
    return text.length === 0;
  }
  
  // Check if first character matches
  const firstMatch = text.length > 0 && 
                     (pattern[0] === text[0] || pattern[0] === '.');
  
  // Handle * (zero or more of preceding character)
  if (pattern.length >= 2 && pattern[1] === '*') {
    // Try zero occurrence OR one+ occurrence
    return isMatch(text, pattern.slice(2)) || // zero
           (firstMatch && isMatch(text.slice(1), pattern)); // one or more
  } else {
    // Regular character or . (any single char)
    return firstMatch && isMatch(text.slice(1), pattern.slice(1));
  }
}

// Test
console.log('=== Recursive Matcher ===');

console.log(isMatch('aa', 'a')); // false
console.log(isMatch('aa', 'a*')); // true
console.log(isMatch('ab', '.*')); // true
console.log(isMatch('aab', 'c*a*b')); // true
console.log(isMatch('mississippi', 'mis*is*p*.')); // false
console.log(isMatch('abc', '.*c')); // true
```

### **Approach 2: Dynamic Programming**
```javascript
/**
 * DP approach for regex matching
 * dp[i][j] = true if text[0...i-1] matches pattern[0...j-1]
 */
function isMatchDP(text, pattern) {
  const m = text.length;
  const n = pattern.length;
  
  // dp[i][j] = does text[0..i-1] match pattern[0..j-1]
  const dp = Array(m + 1).fill(null).map(() => Array(n + 1).fill(false));
  
  // Empty text and empty pattern match
  dp[0][0] = true;
  
  // Handle patterns like a*, a*b*, etc. that can match empty string
  for (let j = 2; j <= n; j++) {
    if (pattern[j - 1] === '*') {
      dp[0][j] = dp[0][j - 2];
    }
  }
  
  // Fill the table
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      const textChar = text[i - 1];
      const patternChar = pattern[j - 1];
      
      if (patternChar === '*') {
        // Star can match zero or more of preceding character
        const prevPatternChar = pattern[j - 2];
        
        // Zero occurrence: ignore pattern[j-2] and pattern[j-1]
        dp[i][j] = dp[i][j - 2];
        
        // One or more occurrence: if chars match
        if (prevPatternChar === textChar || prevPatternChar === '.') {
          dp[i][j] = dp[i][j] || dp[i - 1][j];
        }
      } else if (patternChar === '.' || patternChar === textChar) {
        // Single character match
        dp[i][j] = dp[i - 1][j - 1];
      }
    }
  }
  
  return dp[m][n];
}

// Test
console.log('\n=== DP Matcher ===');

console.log(isMatchDP('aa', 'a')); // false
console.log(isMatchDP('aa', 'a*')); // true
console.log(isMatchDP('ab', '.*')); // true
console.log(isMatchDP('aab', 'c*a*b')); // true
console.log(isMatchDP('mississippi', 'mis*is*p*.')); // false
```

### **Approach 3: With Memoization**
```javascript
/**
 * Recursive with memoization for better performance
 */
function isMatchMemo(text, pattern) {
  const memo = new Map();
  
  function dp(i, j) {
    const key = `${i},${j}`;
    
    if (memo.has(key)) {
      return memo.get(key);
    }
    
    let result;
    
    if (j === pattern.length) {
      result = i === text.length;
    } else {
      const firstMatch = i < text.length && 
                        (pattern[j] === text[i] || pattern[j] === '.');
      
      if (j + 1 < pattern.length && pattern[j + 1] === '*') {
        result = dp(i, j + 2) || (firstMatch && dp(i + 1, j));
      } else {
        result = firstMatch && dp(i + 1, j + 1);
      }
    }
    
    memo.set(key, result);
    return result;
  }
  
  return dp(0, 0);
}

// Test
console.log('\n=== Memoized Matcher ===');

console.log(isMatchMemo('aa', 'a')); // false
console.log(isMatchMemo('aa', 'a*')); // true
console.log(isMatchMemo('aaaaaa', 'a*')); // true
console.log(isMatchMemo('ab', '.*')); // true
```

### **Approach 4: Extended Regex (+ operator)**
```javascript
/**
 * Extended matcher with + (one or more)
 */
function isMatchExtended(text, pattern) {
  // Convert + to equivalent pattern
  // a+ becomes aa*
  pattern = pattern.replace(/(.)\+/g, '$1$1*');
  
  return isMatchDP(text, pattern);
}

// Alternative: handle + directly
function isMatchWithPlus(text, pattern) {
  const memo = new Map();
  
  function dp(i, j) {
    const key = `${i},${j}`;
    if (memo.has(key)) return memo.get(key);
    
    let result;
    
    if (j === pattern.length) {
      result = i === text.length;
    } else {
      const firstMatch = i < text.length && 
                        (pattern[j] === text[i] || pattern[j] === '.');
      
      if (j + 1 < pattern.length) {
        const nextChar = pattern[j + 1];
        
        if (nextChar === '*') {
          // Zero or more
          result = dp(i, j + 2) || (firstMatch && dp(i + 1, j));
        } else if (nextChar === '+') {
          // One or more
          result = firstMatch && (dp(i + 1, j + 2) || dp(i + 1, j));
        } else {
          result = firstMatch && dp(i + 1, j + 1);
        }
      } else {
        result = firstMatch && dp(i + 1, j + 1);
      }
    }
    
    memo.set(key, result);
    return result;
  }
  
  return dp(0, 0);
}

// Test
console.log('\n=== Extended Matcher (+ operator) ===');

console.log(isMatchExtended('a', 'a+')); // true
console.log(isMatchExtended('aa', 'a+')); // true
console.log(isMatchExtended('', 'a+')); // false
console.log(isMatchExtended('ab', 'a+b')); // true
```

### **Approach 5: Full Regex Engine (?, [], ^, $)**
```javascript
/**
 * More complete regex engine
 */
class RegexEngine {
  constructor(pattern) {
    this.pattern = pattern;
  }
  
  match(text) {
    return this.matchRecursive(text, 0, this.pattern, 0);
  }
  
  matchRecursive(text, ti, pattern, pi) {
    // End of pattern
    if (pi === pattern.length) {
      return ti === text.length;
    }
    
    // Get current pattern character and next
    const pChar = pattern[pi];
    const nextChar = pi + 1 < pattern.length ? pattern[pi + 1] : null;
    
    // Handle quantifiers
    if (nextChar === '*') {
      // Zero occurrences
      if (this.matchRecursive(text, ti, pattern, pi + 2)) {
        return true;
      }
      
      // One or more occurrences
      let i = ti;
      while (i < text.length && this.matchChar(text[i], pChar)) {
        if (this.matchRecursive(text, i + 1, pattern, pi + 2)) {
          return true;
        }
        i++;
      }
      
      return false;
    } else if (nextChar === '?') {
      // Zero or one occurrence
      return this.matchRecursive(text, ti, pattern, pi + 2) ||
             (ti < text.length && this.matchChar(text[ti], pChar) &&
              this.matchRecursive(text, ti + 1, pattern, pi + 2));
    } else if (nextChar === '+') {
      // One or more occurrences
      if (ti >= text.length || !this.matchChar(text[ti], pChar)) {
        return false;
      }
      
      let i = ti + 1;
      while (i < text.length && this.matchChar(text[i], pChar)) {
        if (this.matchRecursive(text, i + 1, pattern, pi + 2)) {
          return true;
        }
        i++;
      }
      
      return this.matchRecursive(text, i, pattern, pi + 2);
    }
    
    // Regular character match
    if (ti < text.length && this.matchChar(text[ti], pChar)) {
      return this.matchRecursive(text, ti + 1, pattern, pi + 1);
    }
    
    return false;
  }
  
  matchChar(textChar, patternChar) {
    return patternChar === '.' || patternChar === textChar;
  }
}

// Test
console.log('\n=== Full Regex Engine ===');

const engine1 = new RegexEngine('a*b');
console.log(engine1.match('b')); // true
console.log(engine1.match('aab')); // true

const engine2 = new RegexEngine('a?b');
console.log(engine2.match('b')); // true
console.log(engine2.match('ab')); // true
console.log(engine2.match('aab')); // false

const engine3 = new RegexEngine('a+b');
console.log(engine3.match('ab')); // true
console.log(engine3.match('aab')); // true
console.log(engine3.match('b')); // false
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications of pattern matching
 */

// 1. Input Validation
console.log('\n=== Input Validation ===');

class InputValidator {
  validateEmail(email) {
    // Simplified email pattern
    const pattern = '.*@.*\\..*';
    return isMatchDP(email, pattern);
  }
  
  validatePhone(phone) {
    // Pattern: XXX-XXX-XXXX
    const pattern = '[0-9][0-9][0-9]-[0-9][0-9][0-9]-[0-9][0-9][0-9][0-9]';
    return this.matchBracket(phone, pattern);
  }
  
  matchBracket(text, pattern) {
    // Simple bracket matching [0-9] means any digit
    let expandedPattern = pattern;
    expandedPattern = expandedPattern.replace(/\[0-9\]/g, '[0123456789]');
    
    // Convert [abc] to (a|b|c) conceptually
    const matches = expandedPattern.match(/\[([^\]]+)\]/g);
    
    if (!matches) {
      return isMatchDP(text, expandedPattern);
    }
    
    // For simplicity, just check digit patterns
    for (let i = 0; i < text.length && i < pattern.length; i++) {
      if (pattern[i] === '[') {
        const closeIdx = pattern.indexOf(']', i);
        const chars = pattern.slice(i + 1, closeIdx);
        
        if (chars === '0-9' && !/\d/.test(text[i])) {
          return false;
        }
        
        i = closeIdx;
      } else if (pattern[i] !== text[i]) {
        return false;
      }
    }
    
    return true;
  }
}

const validator = new InputValidator();
console.log('Email valid:', validator.validateEmail('user@example.com')); // true
console.log('Email invalid:', validator.validateEmail('userexample.com')); // false

// 2. Search and Replace
console.log('\n=== Search and Replace ===');

class PatternReplacer {
  replace(text, pattern, replacement) {
    // Find all matches and replace
    const matches = this.findAll(text, pattern);
    
    let result = text;
    let offset = 0;
    
    for (const match of matches) {
      const before = result.slice(0, match.start + offset);
      const after = result.slice(match.end + offset);
      result = before + replacement + after;
      offset += replacement.length - (match.end - match.start);
    }
    
    return result;
  }
  
  findAll(text, pattern) {
    const matches = [];
    
    for (let i = 0; i < text.length; i++) {
      for (let j = i + 1; j <= text.length; j++) {
        const substring = text.slice(i, j);
        
        if (isMatchDP(substring, pattern)) {
          matches.push({ start: i, end: j, text: substring });
          break; // Take first match at this position
        }
      }
    }
    
    return matches;
  }
}

const replacer = new PatternReplacer();
console.log(replacer.replace('hello world', 'w.*d', 'universe'));

// 3. Log Parser
console.log('\n=== Log Parser ===');

class LogParser {
  parseLogLine(line) {
    // Pattern: [LEVEL] message
    const levelPattern = '\\[.*\\]';
    
    // Extract level
    const levelMatch = line.match(/\[([^\]]+)\]/);
    const level = levelMatch ? levelMatch[1] : 'UNKNOWN';
    
    // Extract message (everything after "]")
    const messageStart = line.indexOf(']') + 1;
    const message = line.slice(messageStart).trim();
    
    return { level, message };
  }
  
  filterByLevel(logs, levelPattern) {
    return logs.filter(log => {
      const parsed = this.parseLogLine(log);
      return isMatchDP(parsed.level, levelPattern);
    });
  }
}

const parser = new LogParser();
const logs = [
  '[INFO] Application started',
  '[ERROR] Connection failed',
  '[WARN] Deprecated API used',
  '[INFO] User logged in'
];

console.log('ERROR logs:', parser.filterByLevel(logs, 'ERROR'));
console.log('INFO logs:', parser.filterByLevel(logs, 'INFO'));

// 4. URL Router
console.log('\n=== URL Router ===');

class URLRouter {
  constructor() {
    this.routes = [];
  }
  
  addRoute(pattern, handler) {
    this.routes.push({ pattern, handler });
  }
  
  route(url) {
    for (const route of this.routes) {
      if (this.matchRoute(url, route.pattern)) {
        return route.handler(url);
      }
    }
    
    return 'Not Found';
  }
  
  matchRoute(url, pattern) {
    // Simple pattern matching
    // /user/:id becomes /user/.*
    let regexPattern = pattern.replace(/:[\w]+/g, '.*');
    
    return isMatchDP(url, regexPattern);
  }
}

const router = new URLRouter();
router.addRoute('/user/.*', (url) => `User page: ${url}`);
router.addRoute('/post/.*', (url) => `Post page: ${url}`);
router.addRoute('/about', (url) => 'About page');

console.log(router.route('/user/123')); // User page
console.log(router.route('/post/abc')); // Post page
console.log(router.route('/about')); // About page

// 5. Command Parser
console.log('\n=== Command Parser ===');

class CommandParser {
  parseCommand(input) {
    const commands = {
      'git add .*': { action: 'add', pattern: 'git add .*' },
      'git commit .*': { action: 'commit', pattern: 'git commit .*' },
      'npm install .*': { action: 'install', pattern: 'npm install .*' }
    };
    
    for (const [pattern, command] of Object.entries(commands)) {
      if (isMatchDP(input, pattern)) {
        return {
          valid: true,
          action: command.action,
          args: input.split(' ').slice(2)
        };
      }
    }
    
    return { valid: false };
  }
}

const cmdParser = new CommandParser();
console.log(cmdParser.parseCommand('git add file.txt'));
console.log(cmdParser.parseCommand('npm install react'));
console.log(cmdParser.parseCommand('invalid command'));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty strings
console.log('Empty text and pattern:', isMatch('', '')); // true
console.log('Empty text:', isMatch('', 'a*')); // true
console.log('Empty pattern:', isMatch('a', '')); // false

// Only wildcards
console.log('Only .*:', isMatch('anything', '.*')); // true
console.log('Only a*:', isMatch('', 'a*')); // true
console.log('Only a*:', isMatch('aaa', 'a*')); // true

// Complex patterns
console.log('Complex:', isMatch('aab', 'c*a*b')); // true (c* matches '', a* matches 'aa')
console.log('Complex:', isMatch('ab', '.*c')); // false

// Single characters
console.log('Single char:', isMatch('a', '.')); // true
console.log('Single char:', isMatch('a', 'a')); // true
console.log('Single char:', isMatch('a', 'b')); // false

// Multiple stars
console.log('Multiple *:', isMatch('aaa', 'a*a*a*')); // true
console.log('Multiple *:', isMatch('ab', 'a*b*')); // true

// Star at beginning
console.log('Star at start:', isMatch('aab', 'a*aab')); // true

// Long strings
const longText = 'a'.repeat(20);
console.log('Long string:', isMatch(longText, 'a*')); // true

// No match cases
console.log('No match:', isMatch('ab', 'a')); // false
console.log('No match:', isMatch('a', 'ab')); // false
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Regex Matcher Comparison:
┌────────────────────┬─────────────────┬─────────────┬─────────────┐
│ Approach           │ Time            │ Space       │ Best For    │
├────────────────────┼─────────────────┼─────────────┼─────────────┤
│ Recursive          │ O(2^(m+n))      │ O(m+n)      │ Simple cases│
│ DP (bottom-up)     │ O(m × n)        │ O(m × n)    │ General     │
│ Memoized           │ O(m × n)        │ O(m × n)    │ Many queries│
└────────────────────┴─────────────────┴─────────────┴─────────────┘

Where m = text length, n = pattern length

Supported Operations:
• . (dot): matches any single character
• * (star): matches zero or more of preceding element
• + (plus): matches one or more (extension)
• ? (question): matches zero or one (extension)

Key Insights:
• Star requires looking ahead in pattern
• Two choices with star: use it zero times or one+ times
• DP builds table of all subproblems
• Memoization avoids recomputing same states

Real-World Patterns:
• Email: .*@.*\\..*
• Phone: [0-9]{3}-[0-9]{3}-[0-9]{4}
• URL: http.*://.*
• Date: [0-9]{4}-[0-9]{2}-[0-9]{2}

Full Regex Engines:
• NFA (Non-deterministic Finite Automaton)
• DFA (Deterministic Finite Automaton)
• Backtracking with optimization
• Thompson's construction
`);
```

**Interview Tips:**
- Regex matching: pattern with . (any char) and * (zero or more of preceding)
- Classic problem: requires recursion or DP
- Key insight: * applies to preceding character, not following
- Two cases for *: match zero times (skip pattern[0:2]) or one+ times (consume text[0])
- First character match: text[0] == pattern[0] or pattern[0] == '.'
- Recursive: check first char match, then recursively match rest
- Base case: empty pattern → check if text is also empty
- Time: O(2^(m+n)) for pure recursion due to branching
- DP optimization: dp[i][j] = text[0..i-1] matches pattern[0..j-1]
- DP time: O(m × n), space: O(m × n)
- Empty pattern with *: "a*b*c*" can match empty text
- Handle * by checking pattern[j-2] (character before *)
- Memoization: cache results by (textIndex, patternIndex) pair
- Common mistake: forgetting * matches zero or MORE (not just zero or one)
- Extension: + (one or more), ? (zero or one), [] (character class)
- Applications: input validation, search/replace, log parsing, URL routing
- Edge cases: empty strings, only wildcards, multiple stars, no match
- Real implementation: use NFA/DFA for production regex engines
- Clarify: which operators supported? case sensitive? multiline?
- Follow-ups: add +, ?, [], ^, $, full regex engine, optimize with NFA

</details>

110. Solve the "Trapping Rain Water" problem

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Two Pointers**
```javascript
/**
 * Trap rain water using two pointers
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
function trap(height) {
  if (height.length === 0) return 0;
  
  let left = 0;
  let right = height.length - 1;
  let leftMax = 0;
  let rightMax = 0;
  let water = 0;
  
  while (left < right) {
    if (height[left] < height[right]) {
      if (height[left] >= leftMax) {
        leftMax = height[left];
      } else {
        water += leftMax - height[left];
      }
      left++;
    } else {
      if (height[right] >= rightMax) {
        rightMax = height[right];
      } else {
        water += rightMax - height[right];
      }
      right--;
    }
  }
  
  return water;
}

// Test
console.log('=== Two Pointers ===');

console.log(trap([0,1,0,2,1,0,1,3,2,1,2,1])); // 6
console.log(trap([4,2,0,3,2,5])); // 9
console.log(trap([3,0,2,0,4])); // 7
console.log(trap([5,4,1,2])); // 1
```

### **Approach 2: Dynamic Programming**
```javascript
/**
 * Pre-compute max heights on left and right
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function trapDP(height) {
  const n = height.length;
  if (n === 0) return 0;
  
  // leftMax[i] = max height from 0 to i
  const leftMax = new Array(n);
  leftMax[0] = height[0];
  
  for (let i = 1; i < n; i++) {
    leftMax[i] = Math.max(leftMax[i - 1], height[i]);
  }
  
  // rightMax[i] = max height from i to n-1
  const rightMax = new Array(n);
  rightMax[n - 1] = height[n - 1];
  
  for (let i = n - 2; i >= 0; i--) {
    rightMax[i] = Math.max(rightMax[i + 1], height[i]);
  }
  
  // Calculate water at each position
  let water = 0;
  for (let i = 0; i < n; i++) {
    const waterLevel = Math.min(leftMax[i], rightMax[i]);
    water += waterLevel - height[i];
  }
  
  return water;
}

// Test
console.log('\n=== Dynamic Programming ===');

console.log(trapDP([0,1,0,2,1,0,1,3,2,1,2,1])); // 6
console.log(trapDP([4,2,0,3,2,5])); // 9
console.log(trapDP([3,0,2,0,4])); // 7
```

### **Approach 3: Stack-Based**
```javascript
/**
 * Use stack to track potential water containers
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
function trapStack(height) {
  const stack = [];
  let water = 0;
  
  for (let i = 0; i < height.length; i++) {
    while (stack.length > 0 && height[i] > height[stack[stack.length - 1]]) {
      const top = stack.pop();
      
      if (stack.length === 0) {
        break;
      }
      
      const distance = i - stack[stack.length - 1] - 1;
      const boundedHeight = Math.min(height[i], height[stack[stack.length - 1]]) - height[top];
      
      water += distance * boundedHeight;
    }
    
    stack.push(i);
  }
  
  return water;
}

// Test
console.log('\n=== Stack-Based ===');

console.log(trapStack([0,1,0,2,1,0,1,3,2,1,2,1])); // 6
console.log(trapStack([4,2,0,3,2,5])); // 9
console.log(trapStack([3,0,2,0,4])); // 7
```

### **Approach 4: Visualize Water Levels**
```javascript
/**
 * Visualize the trapped water
 */
function trapVisualize(height) {
  const n = height.length;
  if (n === 0) return { water: 0, visualization: [] };
  
  // Calculate water at each position using DP
  const leftMax = new Array(n);
  leftMax[0] = height[0];
  for (let i = 1; i < n; i++) {
    leftMax[i] = Math.max(leftMax[i - 1], height[i]);
  }
  
  const rightMax = new Array(n);
  rightMax[n - 1] = height[n - 1];
  for (let i = n - 2; i >= 0; i--) {
    rightMax[i] = Math.max(rightMax[i + 1], height[i]);
  }
  
  // Build visualization
  const maxHeight = Math.max(...height, ...leftMax, ...rightMax);
  const visualization = [];
  let totalWater = 0;
  
  for (let level = maxHeight; level > 0; level--) {
    let row = '';
    for (let i = 0; i < n; i++) {
      const waterLevel = Math.min(leftMax[i], rightMax[i]);
      
      if (height[i] >= level) {
        row += '█'; // Block
      } else if (waterLevel >= level) {
        row += '≈'; // Water
        if (level === Math.ceil((waterLevel + height[i]) / 2)) {
          totalWater += waterLevel - height[i];
        }
      } else {
        row += ' '; // Air
      }
    }
    visualization.push(row);
  }
  
  return {
    water: totalWater,
    visualization: visualization
  };
}

// Test
console.log('\n=== Visualization ===');

const result = trapVisualize([0,1,0,2,1,0,1,3,2,1,2,1]);
console.log('Water trapped:', result.water);
console.log('Visualization:');
result.visualization.forEach(row => console.log(row));
```

### **Approach 5: 2D Rain Water Trapping**
```javascript
/**
 * Extended to 2D grid (harder version)
 * Given heightMap[i][j], find total water trapped
 */
function trapRainWater2D(heightMap) {
  if (!heightMap || heightMap.length === 0) return 0;
  
  const m = heightMap.length;
  const n = heightMap[0].length;
  
  // Priority queue (min heap)
  const pq = [];
  const visited = Array(m).fill(null).map(() => Array(n).fill(false));
  
  // Add boundary cells to priority queue
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (i === 0 || i === m - 1 || j === 0 || j === n - 1) {
        pq.push({ height: heightMap[i][j], row: i, col: j });
        visited[i][j] = true;
      }
    }
  }
  
  // Sort by height (min heap behavior)
  pq.sort((a, b) => a.height - b.height);
  
  const directions = [[0, 1], [1, 0], [0, -1], [-1, 0]];
  let water = 0;
  
  while (pq.length > 0) {
    const cell = pq.shift();
    
    // Check neighbors
    for (const [dr, dc] of directions) {
      const newRow = cell.row + dr;
      const newCol = cell.col + dc;
      
      if (newRow >= 0 && newRow < m && newCol >= 0 && newCol < n && !visited[newRow][newCol]) {
        visited[newRow][newCol] = true;
        
        // Water can be trapped if current max height > cell height
        water += Math.max(0, cell.height - heightMap[newRow][newCol]);
        
        // Add to queue with max height seen so far
        pq.push({
          height: Math.max(cell.height, heightMap[newRow][newCol]),
          row: newRow,
          col: newCol
        });
        
        // Re-sort (in real implementation, use proper min heap)
        pq.sort((a, b) => a.height - b.height);
      }
    }
  }
  
  return water;
}

// Test
console.log('\n=== 2D Rain Water ===');

const heightMap = [
  [1,4,3,1,3,2],
  [3,2,1,3,2,4],
  [2,3,3,2,3,1]
];

console.log('2D water trapped:', trapRainWater2D(heightMap)); // 4
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications of trapping rain water logic
 */

// 1. Rainfall Simulation
console.log('\n=== Rainfall Simulation ===');

class RainfallSimulator {
  constructor(terrain) {
    this.terrain = terrain;
  }
  
  simulateRainfall(rainfall) {
    const n = this.terrain.length;
    const leftMax = new Array(n);
    const rightMax = new Array(n);
    
    leftMax[0] = this.terrain[0];
    for (let i = 1; i < n; i++) {
      leftMax[i] = Math.max(leftMax[i - 1], this.terrain[i]);
    }
    
    rightMax[n - 1] = this.terrain[n - 1];
    for (let i = n - 2; i >= 0; i--) {
      rightMax[i] = Math.max(rightMax[i + 1], this.terrain[i]);
    }
    
    const waterLevels = [];
    let totalWater = 0;
    
    for (let i = 0; i < n; i++) {
      const waterLevel = Math.min(leftMax[i], rightMax[i]);
      const waterDepth = Math.max(0, waterLevel - this.terrain[i]);
      waterLevels.push(waterDepth);
      totalWater += waterDepth;
    }
    
    return {
      totalVolume: totalWater,
      waterLevels,
      capacity: totalWater,
      utilizationRate: totalWater / (n * Math.max(...this.terrain))
    };
  }
}

const simulator = new RainfallSimulator([3,0,2,0,4]);
console.log('Simulation:', simulator.simulateRainfall(1));

// 2. Flood Risk Assessment
console.log('\n=== Flood Risk Assessment ===');

class FloodRiskAnalyzer {
  constructor(elevationProfile) {
    this.elevationProfile = elevationProfile;
  }
  
  assessRisk() {
    const trapped = trap(this.elevationProfile);
    const avgElevation = this.elevationProfile.reduce((a, b) => a + b, 0) / this.elevationProfile.length;
    const maxElevation = Math.max(...this.elevationProfile);
    const minElevation = Math.min(...this.elevationProfile);
    
    // Find low-lying areas
    const vulnerableAreas = [];
    for (let i = 0; i < this.elevationProfile.length; i++) {
      if (this.elevationProfile[i] < avgElevation * 0.7) {
        vulnerableAreas.push({
          index: i,
          elevation: this.elevationProfile[i],
          risk: ((avgElevation - this.elevationProfile[i]) / avgElevation * 100).toFixed(1) + '%'
        });
      }
    }
    
    return {
      waterTrappingCapacity: trapped,
      averageElevation: avgElevation.toFixed(2),
      elevationRange: maxElevation - minElevation,
      vulnerableAreas,
      overallRisk: trapped > avgElevation ? 'HIGH' : trapped > avgElevation * 0.5 ? 'MEDIUM' : 'LOW'
    };
  }
}

const floodAnalyzer = new FloodRiskAnalyzer([5,3,1,2,4,3,2,4,5]);
console.log('Flood risk:', floodAnalyzer.assessRisk());

// 3. Water Storage Optimization
console.log('\n=== Water Storage Optimization ===');

class WaterStorageOptimizer {
  constructor(containers) {
    this.containers = containers;
  }
  
  optimizeStorage() {
    const currentCapacity = trap(this.containers);
    
    // Find best position to add height
    const improvements = [];
    
    for (let i = 0; i < this.containers.length; i++) {
      const modified = [...this.containers];
      modified[i] += 1;
      
      const newCapacity = trap(modified);
      const gain = newCapacity - currentCapacity;
      
      if (gain > 0) {
        improvements.push({
          position: i,
          currentHeight: this.containers[i],
          newHeight: modified[i],
          capacityGain: gain,
          roi: gain / 1 // cost of 1 unit height
        });
      }
    }
    
    improvements.sort((a, b) => b.roi - a.roi);
    
    return {
      currentCapacity,
      recommendations: improvements.slice(0, 3),
      maxPotentialCapacity: currentCapacity + (improvements[0]?.capacityGain || 0)
    };
  }
}

const optimizer = new WaterStorageOptimizer([3,0,2,0,4]);
console.log('Optimization:', optimizer.optimizeStorage());

// 4. Terrain Analysis
console.log('\n=== Terrain Analysis ===');

class TerrainAnalyzer {
  constructor(elevation) {
    this.elevation = elevation;
  }
  
  analyzeWaterflow() {
    const n = this.elevation.length;
    const leftMax = new Array(n);
    const rightMax = new Array(n);
    
    leftMax[0] = this.elevation[0];
    for (let i = 1; i < n; i++) {
      leftMax[i] = Math.max(leftMax[i - 1], this.elevation[i]);
    }
    
    rightMax[n - 1] = this.elevation[n - 1];
    for (let i = n - 2; i >= 0; i--) {
      rightMax[i] = Math.max(rightMax[i + 1], this.elevation[i]);
    }
    
    // Identify pools
    const pools = [];
    let currentPool = null;
    
    for (let i = 0; i < n; i++) {
      const waterLevel = Math.min(leftMax[i], rightMax[i]);
      const waterDepth = waterLevel - this.elevation[i];
      
      if (waterDepth > 0) {
        if (!currentPool) {
          currentPool = { start: i, end: i, depth: waterDepth, volume: waterDepth };
        } else {
          currentPool.end = i;
          currentPool.depth = Math.max(currentPool.depth, waterDepth);
          currentPool.volume += waterDepth;
        }
      } else if (currentPool) {
        pools.push(currentPool);
        currentPool = null;
      }
    }
    
    if (currentPool) {
      pools.push(currentPool);
    }
    
    return {
      numberOfPools: pools.length,
      pools,
      totalVolume: pools.reduce((sum, p) => sum + p.volume, 0),
      largestPool: pools.reduce((max, p) => p.volume > max.volume ? p : max, { volume: 0 })
    };
  }
}

const terrainAnalyzer = new TerrainAnalyzer([0,1,0,2,1,0,1,3,2,1,2,1]);
console.log('Terrain analysis:', terrainAnalyzer.analyzeWaterflow());

// 5. Building Foundation Planning
console.log('\n=== Foundation Planning ===');

class FoundationPlanner {
  constructor(groundProfile) {
    this.groundProfile = groundProfile;
  }
  
  planFoundation() {
    const n = this.groundProfile.length;
    const waterProblems = [];
    
    // Check each position for water accumulation risk
    for (let i = 1; i < n - 1; i++) {
      const leftMax = Math.max(...this.groundProfile.slice(0, i));
      const rightMax = Math.max(...this.groundProfile.slice(i + 1));
      
      const waterLevel = Math.min(leftMax, rightMax);
      const waterDepth = Math.max(0, waterLevel - this.groundProfile[i]);
      
      if (waterDepth > 0) {
        waterProblems.push({
          position: i,
          elevation: this.groundProfile[i],
          waterDepth,
          recommendation: waterDepth > 2 ? 'Install drainage system' : 'Minor grading needed'
        });
      }
    }
    
    return {
      waterAccumulationRisks: waterProblems.length,
      criticalAreas: waterProblems.filter(p => p.waterDepth > 2),
      recommendations: waterProblems,
      estimatedCost: waterProblems.reduce((sum, p) => sum + p.waterDepth * 100, 0)
    };
  }
}

const foundationPlanner = new FoundationPlanner([4,2,0,3,2,5]);
console.log('Foundation plan:', foundationPlanner.planFoundation());
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty array
console.log('Empty:', trap([])); // 0

// Single element
console.log('Single element:', trap([5])); // 0

// Two elements
console.log('Two elements:', trap([3, 5])); // 0

// All same height
console.log('All same:', trap([3,3,3,3])); // 0

// Ascending
console.log('Ascending:', trap([1,2,3,4,5])); // 0

// Descending
console.log('Descending:', trap([5,4,3,2,1])); // 0

// Valley
console.log('Valley:', trap([5,1,5])); // 4

// Multiple valleys
console.log('Multiple valleys:', trap([4,1,4,1,4])); // 6

// Peak in middle
console.log('Peak in middle:', trap([1,3,1])); // 0

// Complex terrain
console.log('Complex:', trap([5,2,1,2,1,5])); // 14

// Large numbers
console.log('Large numbers:', trap([100,50,100])); // 50

// Zero heights
console.log('With zeros:', trap([0,0,5,0,0])); // 0
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

function benchmark(fn, name, input) {
  const start = performance.now();
  const result = fn(input);
  const end = performance.now();
  console.log(`${name}: ${result} (${(end - start).toFixed(3)}ms)`);
}

const testArray = [0,1,0,2,1,0,1,3,2,1,2,1];

benchmark(trap, 'Two Pointers', testArray);
benchmark(trapDP, 'Dynamic Programming', testArray);
benchmark(trapStack, 'Stack-Based', testArray);

console.log(`
Algorithm Comparison:
┌────────────────────┬─────────────┬─────────────┬──────────────┐
│ Algorithm          │ Time        │ Space       │ Best For     │
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ Two Pointers       │ O(n)        │ O(1)        │ Optimal,     │
│                    │             │             │ space-efficient│
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ Dynamic Programming│ O(n)        │ O(n)        │ Intuitive,   │
│                    │             │             │ easier to    │
│                    │             │             │ understand   │
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ Stack-Based        │ O(n)        │ O(n)        │ Layer-by-layer│
│                    │             │             │ calculation  │
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ 2D Version         │ O(m×n×log k)│ O(m×n)      │ Grid problems│
│                    │             │             │ (harder)     │
└────────────────────┴─────────────┴─────────────┴──────────────┘

Key Insights:
• Water level at position i = min(leftMax[i], rightMax[i])
• Water trapped at i = waterLevel - height[i]
• Two pointers: most efficient, O(1) space
• DP: pre-compute left/right max arrays
• Stack: process layer by layer (horizontal layers)

Real-World Applications:
• Rainfall simulation and modeling
• Flood risk assessment
• Water storage optimization
• Terrain analysis for construction
• Foundation planning and drainage design

Optimization Tips:
• Two pointers eliminates need for extra arrays
• Process from both ends simultaneously
• Track max heights seen so far
• Water can only be trapped between two higher bars
`);
```

**Interview Tips:**
- Trapping rain water: calculate water trapped between elevation bars
- Key insight: water level at position = min(max_left, max_right)
- Water trapped = water_level - current_height (if positive)
- Three main approaches: two pointers (best), DP, stack
- Two pointers: O(n) time, O(1) space - optimal solution
- Track leftMax and rightMax while moving pointers inward
- Move pointer with smaller height (water limited by smaller side)
- DP approach: pre-compute leftMax and rightMax arrays, then iterate
- DP time: O(n), space: O(n) - easier to understand but uses extra space
- Stack approach: calculate water layer by layer horizontally
- Stack tracks indices of bars, pop when finding higher bar
- Applications: rainfall simulation, flood risk, water storage, terrain analysis
- Edge cases: empty array, single element, all same height, ascending/descending
- Visualization: water fills valleys between peaks
- 2D version: harder problem using priority queue (min heap)
- Common mistake: forgetting that water needs walls on both sides
- Optimization: two pointers preferred in interviews for O(1) space
- Follow-ups: 2D version, container with most water (different problem), visualize
- Clarify: 1D or 2D? return volume or visualization? constraints on height values?

</details>

111. Find the median of two sorted arrays

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Merge and Find Median**
```javascript
/**
 * Merge two sorted arrays and find median
 * Time Complexity: O(m + n)
 * Space Complexity: O(m + n)
 */
function findMedianSortedArrays(nums1, nums2) {
  const merged = [];
  let i = 0, j = 0;
  
  // Merge arrays
  while (i < nums1.length && j < nums2.length) {
    if (nums1[i] < nums2[j]) {
      merged.push(nums1[i++]);
    } else {
      merged.push(nums2[j++]);
    }
  }
  
  // Add remaining elements
  while (i < nums1.length) {
    merged.push(nums1[i++]);
  }
  
  while (j < nums2.length) {
    merged.push(nums2[j++]);
  }
  
  // Find median
  const len = merged.length;
  if (len % 2 === 0) {
    return (merged[len / 2 - 1] + merged[len / 2]) / 2;
  } else {
    return merged[Math.floor(len / 2)];
  }
}

// Test
console.log('=== Merge Approach ===');

console.log(findMedianSortedArrays([1,3], [2])); // 2
console.log(findMedianSortedArrays([1,2], [3,4])); // 2.5
console.log(findMedianSortedArrays([0,0], [0,0])); // 0
console.log(findMedianSortedArrays([], [1])); // 1
console.log(findMedianSortedArrays([2], [])); // 2
```

### **Approach 2: Binary Search (Optimal)**
```javascript
/**
 * Binary search on smaller array
 * Time Complexity: O(log(min(m,n)))
 * Space Complexity: O(1)
 */
function findMedianSortedArraysBinarySearch(nums1, nums2) {
  // Ensure nums1 is smaller array
  if (nums1.length > nums2.length) {
    [nums1, nums2] = [nums2, nums1];
  }
  
  const m = nums1.length;
  const n = nums2.length;
  let left = 0;
  let right = m;
  
  while (left <= right) {
    const partition1 = Math.floor((left + right) / 2);
    const partition2 = Math.floor((m + n + 1) / 2) - partition1;
    
    const maxLeft1 = partition1 === 0 ? -Infinity : nums1[partition1 - 1];
    const minRight1 = partition1 === m ? Infinity : nums1[partition1];
    
    const maxLeft2 = partition2 === 0 ? -Infinity : nums2[partition2 - 1];
    const minRight2 = partition2 === n ? Infinity : nums2[partition2];
    
    if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {
      // Found correct partition
      if ((m + n) % 2 === 0) {
        return (Math.max(maxLeft1, maxLeft2) + Math.min(minRight1, minRight2)) / 2;
      } else {
        return Math.max(maxLeft1, maxLeft2);
      }
    } else if (maxLeft1 > minRight2) {
      // Too far right in nums1
      right = partition1 - 1;
    } else {
      // Too far left in nums1
      left = partition1 + 1;
    }
  }
  
  throw new Error('Input arrays are not sorted');
}

// Test
console.log('\n=== Binary Search ===');

console.log(findMedianSortedArraysBinarySearch([1,3], [2])); // 2
console.log(findMedianSortedArraysBinarySearch([1,2], [3,4])); // 2.5
console.log(findMedianSortedArraysBinarySearch([0,0], [0,0])); // 0
console.log(findMedianSortedArraysBinarySearch([], [1])); // 1
console.log(findMedianSortedArraysBinarySearch([2], [])); // 2
```

### **Approach 3: K-th Element (Generalized)**
```javascript
/**
 * Find k-th smallest element in two sorted arrays
 * Can be used to find median
 */
function findKthElement(nums1, nums2, k) {
  function helper(arr1, start1, arr2, start2, k) {
    // Base cases
    if (start1 >= arr1.length) {
      return arr2[start2 + k - 1];
    }
    if (start2 >= arr2.length) {
      return arr1[start1 + k - 1];
    }
    if (k === 1) {
      return Math.min(arr1[start1], arr2[start2]);
    }
    
    // Compare k/2-th elements
    const mid = Math.floor(k / 2);
    const val1 = start1 + mid - 1 < arr1.length ? arr1[start1 + mid - 1] : Infinity;
    const val2 = start2 + mid - 1 < arr2.length ? arr2[start2 + mid - 1] : Infinity;
    
    if (val1 < val2) {
      return helper(arr1, start1 + mid, arr2, start2, k - mid);
    } else {
      return helper(arr1, start1, arr2, start2 + mid, k - mid);
    }
  }
  
  return helper(nums1, 0, nums2, 0, k);
}

function findMedianKth(nums1, nums2) {
  const total = nums1.length + nums2.length;
  
  if (total % 2 === 1) {
    return findKthElement(nums1, nums2, Math.floor(total / 2) + 1);
  } else {
    const mid1 = findKthElement(nums1, nums2, total / 2);
    const mid2 = findKthElement(nums1, nums2, total / 2 + 1);
    return (mid1 + mid2) / 2;
  }
}

// Test
console.log('\n=== K-th Element ===');

console.log(findMedianKth([1,3], [2])); // 2
console.log(findMedianKth([1,2], [3,4])); // 2.5
console.log(findKthElement([1,3,5,7], [2,4,6,8], 4)); // 4 (4th smallest)
```

### **Approach 4: Without Extra Space (Merge in Place)**
```javascript
/**
 * Find median without merging into new array
 */
function findMedianNoExtraSpace(nums1, nums2) {
  const total = nums1.length + nums2.length;
  const mid = Math.floor(total / 2);
  
  let i = 0, j = 0, count = 0;
  let prev = 0, curr = 0;
  
  while (count <= mid) {
    prev = curr;
    
    if (i < nums1.length && (j >= nums2.length || nums1[i] <= nums2[j])) {
      curr = nums1[i++];
    } else {
      curr = nums2[j++];
    }
    
    count++;
  }
  
  if (total % 2 === 0) {
    return (prev + curr) / 2;
  } else {
    return curr;
  }
}

// Test
console.log('\n=== No Extra Space ===');

console.log(findMedianNoExtraSpace([1,3], [2])); // 2
console.log(findMedianNoExtraSpace([1,2], [3,4])); // 2.5
```

### **Approach 5: Multiple Arrays Median**
```javascript
/**
 * Find median of multiple sorted arrays
 */
function findMedianMultipleArrays(arrays) {
  // Merge all arrays using min heap approach
  const merged = [];
  const pointers = new Array(arrays.length).fill(0);
  
  while (true) {
    let minVal = Infinity;
    let minIdx = -1;
    
    // Find minimum across all arrays
    for (let i = 0; i < arrays.length; i++) {
      if (pointers[i] < arrays[i].length && arrays[i][pointers[i]] < minVal) {
        minVal = arrays[i][pointers[i]];
        minIdx = i;
      }
    }
    
    if (minIdx === -1) break; // All arrays exhausted
    
    merged.push(minVal);
    pointers[minIdx]++;
  }
  
  // Find median
  const len = merged.length;
  if (len % 2 === 0) {
    return (merged[len / 2 - 1] + merged[len / 2]) / 2;
  } else {
    return merged[Math.floor(len / 2)];
  }
}

// Test
console.log('\n=== Multiple Arrays ===');

console.log(findMedianMultipleArrays([[1,3], [2,4], [5,6]])); // 3.5
console.log(findMedianMultipleArrays([[1], [2], [3]])); // 2
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Data Stream Median
console.log('\n=== Data Stream Median ===');

class MedianFinder {
  constructor() {
    this.small = []; // Max heap (simulate with negative values)
    this.large = []; // Min heap
  }
  
  addNum(num) {
    // Add to max heap (small)
    this.small.push(-num);
    this.small.sort((a, b) => a - b);
    
    // Balance: move largest from small to large
    if (this.small.length > 0) {
      this.large.push(-this.small.shift());
      this.large.sort((a, b) => a - b);
    }
    
    // Rebalance if large has too many
    if (this.large.length > this.small.length + 1) {
      this.small.push(-this.large.shift());
      this.small.sort((a, b) => a - b);
    }
  }
  
  findMedian() {
    if (this.large.length > this.small.length) {
      return this.large[0];
    } else {
      return (-this.small[0] + this.large[0]) / 2;
    }
  }
}

const medianFinder = new MedianFinder();
medianFinder.addNum(1);
medianFinder.addNum(2);
console.log('Median:', medianFinder.findMedian()); // 1.5
medianFinder.addNum(3);
console.log('Median:', medianFinder.findMedian()); // 2

// 2. Merge Performance Metrics
console.log('\n=== Performance Metrics ===');

class PerformanceAnalyzer {
  constructor() {
    this.serverALatencies = [];
    this.serverBLatencies = [];
  }
  
  recordLatency(server, latency) {
    if (server === 'A') {
      this.serverALatencies.push(latency);
      this.serverALatencies.sort((a, b) => a - b);
    } else {
      this.serverBLatencies.push(latency);
      this.serverBLatencies.sort((a, b) => a - b);
    }
  }
  
  getMedianLatency() {
    return findMedianSortedArraysBinarySearch(
      this.serverALatencies,
      this.serverBLatencies
    );
  }
  
  getStats() {
    const median = this.getMedianLatency();
    const total = this.serverALatencies.length + this.serverBLatencies.length;
    
    return {
      median,
      totalSamples: total,
      serverA: this.serverALatencies.length,
      serverB: this.serverBLatencies.length
    };
  }
}

const perfAnalyzer = new PerformanceAnalyzer();
perfAnalyzer.recordLatency('A', 10);
perfAnalyzer.recordLatency('A', 20);
perfAnalyzer.recordLatency('B', 15);
perfAnalyzer.recordLatency('B', 25);
console.log('Stats:', perfAnalyzer.getStats());

// 3. Salary Analysis
console.log('\n=== Salary Analysis ===');

class SalaryAnalyzer {
  analyzeDepartments(dept1Salaries, dept2Salaries) {
    const combinedMedian = findMedianSortedArraysBinarySearch(
      dept1Salaries.sort((a, b) => a - b),
      dept2Salaries.sort((a, b) => a - b)
    );
    
    const dept1Median = this.getMedian(dept1Salaries);
    const dept2Median = this.getMedian(dept2Salaries);
    
    return {
      combinedMedian,
      dept1Median,
      dept2Median,
      difference: Math.abs(dept1Median - dept2Median),
      recommendation: Math.abs(dept1Median - dept2Median) > 10000 ?
        'Consider salary adjustment' : 'Salaries are balanced'
    };
  }
  
  getMedian(arr) {
    const len = arr.length;
    if (len % 2 === 0) {
      return (arr[len / 2 - 1] + arr[len / 2]) / 2;
    } else {
      return arr[Math.floor(len / 2)];
    }
  }
}

const salaryAnalyzer = new SalaryAnalyzer();
console.log(salaryAnalyzer.analyzeDepartments(
  [50000, 60000, 70000],
  [55000, 65000, 75000]
));

// 4. Time Series Merge
console.log('\n=== Time Series Merge ===');

class TimeSeriesMerger {
  mergeTimeSeries(series1, series2) {
    // Each series: [{time, value}, ...]
    const times1 = series1.map(s => s.time).sort((a, b) => a - b);
    const times2 = series2.map(s => s.time).sort((a, b) => a - b);
    
    const medianTime = findMedianSortedArraysBinarySearch(times1, times2);
    
    return {
      medianTime,
      totalDataPoints: times1.length + times2.length,
      timeRange: {
        start: Math.min(times1[0], times2[0]),
        end: Math.max(times1[times1.length - 1], times2[times2.length - 1])
      }
    };
  }
}

const tsMerger = new TimeSeriesMerger();
const series1 = [{time: 1, value: 10}, {time: 3, value: 30}, {time: 5, value: 50}];
const series2 = [{time: 2, value: 20}, {time: 4, value: 40}, {time: 6, value: 60}];
console.log(tsMerger.mergeTimeSeries(series1, series2));

// 5. Load Balancing
console.log('\n=== Load Balancing ===');

class LoadBalancer {
  constructor() {
    this.server1Loads = [];
    this.server2Loads = [];
  }
  
  recordLoad(server, load) {
    if (server === 1) {
      this.server1Loads.push(load);
      this.server1Loads.sort((a, b) => a - b);
    } else {
      this.server2Loads.push(load);
      this.server2Loads.sort((a, b) => a - b);
    }
  }
  
  shouldRebalance() {
    if (this.server1Loads.length === 0 || this.server2Loads.length === 0) {
      return false;
    }
    
    const median1 = this.getMedian(this.server1Loads);
    const median2 = this.getMedian(this.server2Loads);
    
    const threshold = 20; // 20% difference
    const diff = Math.abs(median1 - median2);
    const avg = (median1 + median2) / 2;
    
    return (diff / avg * 100) > threshold;
  }
  
  getMedian(arr) {
    const len = arr.length;
    if (len % 2 === 0) {
      return (arr[len / 2 - 1] + arr[len / 2]) / 2;
    }
    return arr[Math.floor(len / 2)];
  }
}

const balancer = new LoadBalancer();
balancer.recordLoad(1, 50);
balancer.recordLoad(1, 60);
balancer.recordLoad(2, 80);
balancer.recordLoad(2, 90);
console.log('Should rebalance:', balancer.shouldRebalance());
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty arrays
console.log('Both empty:', findMedianSortedArraysBinarySearch([], [])); // Should handle
console.log('One empty:', findMedianSortedArraysBinarySearch([], [1,2,3])); // 2

// Single element
console.log('Single in each:', findMedianSortedArraysBinarySearch([1], [2])); // 1.5

// Different lengths
console.log('Different lengths:', findMedianSortedArraysBinarySearch([1], [2,3,4,5])); // 3

// All same values
console.log('All same:', findMedianSortedArraysBinarySearch([1,1,1], [1,1,1])); // 1

// Negative numbers
console.log('Negatives:', findMedianSortedArraysBinarySearch([-5,-3,-1], [-4,-2,0])); // -2.5

// Large numbers
console.log('Large:', findMedianSortedArraysBinarySearch([1000000], [1000001])); // 1000000.5

// No overlap
console.log('No overlap:', findMedianSortedArraysBinarySearch([1,2], [10,11])); // 6

// Complete overlap
console.log('Overlap:', findMedianSortedArraysBinarySearch([1,3,5], [2,4,6])); // 3.5
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Algorithm Comparison:
┌────────────────────┬─────────────────┬─────────────┬──────────────┐
│ Algorithm          │ Time            │ Space       │ Best For     │
├────────────────────┼─────────────────┼─────────────┼──────────────┤
│ Merge              │ O(m + n)        │ O(m + n)    │ Simple,      │
│                    │                 │             │ intuitive    │
├────────────────────┼─────────────────┼─────────────┼──────────────┤
│ Binary Search      │ O(log(min(m,n)))│ O(1)        │ Optimal,     │
│                    │                 │             │ most efficient│
├────────────────────┼─────────────────┼─────────────┼──────────────┤
│ K-th Element       │ O(log(m + n))   │ O(log(m+n)) │ Generalized  │
├────────────────────┼─────────────────┼─────────────┼──────────────┤
│ No Extra Space     │ O(m + n)        │ O(1)        │ Space-       │
│                    │                 │             │ constrained  │
└────────────────────┴─────────────────┴─────────────┴──────────────┘

Key Insights:
• Binary search on smaller array for O(log(min(m,n)))
• Partition both arrays so left half < right half
• Handle edge cases: empty arrays, single elements
• Median: middle element (odd) or average of two middle (even)

Partition Logic:
• partition1 + partition2 = (m + n + 1) / 2
• maxLeft1 <= minRight2 && maxLeft2 <= minRight1
• If partition valid: median = max(maxLeft1, maxLeft2) for odd
•                     or avg of max(left) and min(right) for even

Applications:
• Data stream median (two heaps)
• Performance metrics merging
• Salary/statistics analysis
• Time series data merging
• Load balancing decisions

Optimization:
• Always search on smaller array
• Use infinity for boundary conditions
• Binary search more efficient than merge for large arrays
`);
```

**Interview Tips:**
- Find median of two sorted arrays: combine and find middle element(s)
- Naive: merge O(m+n), optimal: binary search O(log(min(m,n)))
- Binary search approach: partition arrays so all left < all right
- Key insight: find correct partition point using binary search
- Partition: divide arrays such that left half has (m+n+1)/2 elements
- Valid partition: maxLeft1 ≤ minRight2 AND maxLeft2 ≤ minRight1
- Median calculation: max of left halves (odd) or average with min of right halves (even)
- Always binary search on smaller array for better complexity
- Handle edge cases: use -Infinity and Infinity for boundaries
- Time: O(log(min(m,n))), space: O(1) - optimal solution
- Alternative: merge arrays O(m+n) - simpler but not optimal
- K-th element approach: generalized solution for any k
- Applications: streaming median, performance metrics, statistical analysis
- Two heaps pattern: maintain max heap for lower half, min heap for upper half
- Edge cases: empty arrays, single elements, no overlap, all same values
- Common mistake: not handling boundary conditions with infinities
- Clarify: both arrays sorted? allow duplicates? integer or float values?
- Follow-ups: k-th element, median of k sorted arrays, streaming median
- Optimization: binary search preferred, demonstrate O(log n) understanding

</details>

112. Implement a function to serialize and deserialize a binary tree

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Pre-order Traversal with Null Markers**
```javascript
/**
 * Serialize and deserialize binary tree using pre-order
 * Time Complexity: O(n) for both operations
 * Space Complexity: O(n)
 */
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}

class Codec {
  // Serialize tree to string
  serialize(root) {
    const result = [];
    
    function preorder(node) {
      if (node === null) {
        result.push('null');
        return;
      }
      
      result.push(node.val);
      preorder(node.left);
      preorder(node.right);
    }
    
    preorder(root);
    return result.join(',');
  }
  
  // Deserialize string to tree
  deserialize(data) {
    const values = data.split(',');
    let index = 0;
    
    function buildTree() {
      if (index >= values.length || values[index] === 'null') {
        index++;
        return null;
      }
      
      const node = new TreeNode(parseInt(values[index]));
      index++;
      
      node.left = buildTree();
      node.right = buildTree();
      
      return node;
    }
    
    return buildTree();
  }
}

// Test
console.log('=== Pre-order Serialization ===');

const codec1 = new Codec();
const tree1 = new TreeNode(1,
  new TreeNode(2),
  new TreeNode(3,
    new TreeNode(4),
    new TreeNode(5)
  )
);

const serialized1 = codec1.serialize(tree1);
console.log('Serialized:', serialized1); // "1,2,null,null,3,4,null,null,5,null,null"

const deserialized1 = codec1.deserialize(serialized1);
console.log('Deserialized:', codec1.serialize(deserialized1));
```

### **Approach 2: Level-order Traversal (BFS)**
```javascript
/**
 * Serialize using level-order (breadth-first)
 * More readable format
 */
class CodecBFS {
  serialize(root) {
    if (!root) return '';
    
    const result = [];
    const queue = [root];
    
    while (queue.length > 0) {
      const node = queue.shift();
      
      if (node === null) {
        result.push('null');
        continue;
      }
      
      result.push(node.val);
      queue.push(node.left);
      queue.push(node.right);
    }
    
    // Remove trailing nulls
    while (result[result.length - 1] === 'null') {
      result.pop();
    }
    
    return result.join(',');
  }
  
  deserialize(data) {
    if (!data) return null;
    
    const values = data.split(',');
    const root = new TreeNode(parseInt(values[0]));
    const queue = [root];
    let i = 1;
    
    while (queue.length > 0 && i < values.length) {
      const node = queue.shift();
      
      // Left child
      if (i < values.length && values[i] !== 'null') {
        node.left = new TreeNode(parseInt(values[i]));
        queue.push(node.left);
      }
      i++;
      
      // Right child
      if (i < values.length && values[i] !== 'null') {
        node.right = new TreeNode(parseInt(values[i]));
        queue.push(node.right);
      }
      i++;
    }
    
    return root;
  }
}

// Test
console.log('\n=== BFS Serialization ===');

const codec2 = new CodecBFS();
const tree2 = new TreeNode(1,
  new TreeNode(2),
  new TreeNode(3,
    new TreeNode(4),
    new TreeNode(5)
  )
);

const serialized2 = codec2.serialize(tree2);
console.log('Serialized:', serialized2); // "1,2,3,null,null,4,5"

const deserialized2 = codec2.deserialize(serialized2);
console.log('Deserialized:', codec2.serialize(deserialized2));
```

### **Approach 3: JSON Format**
```javascript
/**
 * Serialize to JSON format (human-readable)
 */
class CodecJSON {
  serialize(root) {
    if (!root) return JSON.stringify(null);
    
    const obj = {
      val: root.val,
      left: root.left ? JSON.parse(this.serialize(root.left)) : null,
      right: root.right ? JSON.parse(this.serialize(root.right)) : null
    };
    
    return JSON.stringify(obj);
  }
  
  deserialize(data) {
    const obj = JSON.parse(data);
    
    if (obj === null) return null;
    
    const node = new TreeNode(obj.val);
    node.left = this.deserialize(JSON.stringify(obj.left));
    node.right = this.deserialize(JSON.stringify(obj.right));
    
    return node;
  }
  
  // Pretty print
  prettyPrint(root, indent = 0) {
    if (!root) return;
    
    console.log(' '.repeat(indent) + root.val);
    this.prettyPrint(root.left, indent + 2);
    this.prettyPrint(root.right, indent + 2);
  }
}

// Test
console.log('\n=== JSON Serialization ===');

const codec3 = new CodecJSON();
const tree3 = new TreeNode(1,
  new TreeNode(2),
  new TreeNode(3,
    new TreeNode(4),
    new TreeNode(5)
  )
);

const serialized3 = codec3.serialize(tree3);
console.log('Serialized:', serialized3);
console.log('\nPretty print:');
codec3.prettyPrint(tree3);
```

### **Approach 4: Compact Binary Format**
```javascript
/**
 * Space-efficient binary encoding
 */
class CodecBinary {
  serialize(root) {
    const result = [];
    
    function encode(node) {
      if (!node) {
        result.push('0');
        return;
      }
      
      result.push('1');
      result.push(node.val.toString());
      result.push('|');
      
      encode(node.left);
      encode(node.right);
    }
    
    encode(root);
    return result.join('');
  }
  
  deserialize(data) {
    let index = 0;
    
    function decode() {
      if (index >= data.length || data[index] === '0') {
        index++;
        return null;
      }
      
      index++; // Skip '1'
      
      let val = '';
      while (index < data.length && data[index] !== '|') {
        val += data[index];
        index++;
      }
      index++; // Skip '|'
      
      const node = new TreeNode(parseInt(val));
      node.left = decode();
      node.right = decode();
      
      return node;
    }
    
    return decode();
  }
  
  getSize(data) {
    return new Blob([data]).size;
  }
}

// Test
console.log('\n=== Binary Format ===');

const codec4 = new CodecBinary();
const tree4 = new TreeNode(1,
  new TreeNode(2),
  new TreeNode(3,
    new TreeNode(4),
    new TreeNode(5)
  )
);

const serialized4 = codec4.serialize(tree4);
console.log('Serialized:', serialized4);
console.log('Size:', codec4.getSize(serialized4), 'bytes');
```

### **Approach 5: With Metadata and Validation**
```javascript
/**
 * Include metadata and validation
 */
class CodecAdvanced {
  serialize(root, includeMetadata = true) {
    const timestamp = new Date().toISOString();
    const treeData = [];
    let nodeCount = 0;
    let nullCount = 0;
    let height = 0;
    
    function preorder(node, level = 0) {
      height = Math.max(height, level);
      
      if (node === null) {
        treeData.push('null');
        nullCount++;
        return;
      }
      
      nodeCount++;
      treeData.push(node.val);
      preorder(node.left, level + 1);
      preorder(node.right, level + 1);
    }
    
    preorder(root);
    
    if (includeMetadata) {
      return JSON.stringify({
        version: '1.0',
        timestamp,
        metadata: {
          nodeCount,
          nullCount,
          height,
          totalElements: treeData.length
        },
        data: treeData.join(',')
      });
    }
    
    return treeData.join(',');
  }
  
  deserialize(data) {
    let values;
    let metadata = null;
    
    try {
      const parsed = JSON.parse(data);
      if (parsed.data) {
        values = parsed.data.split(',');
        metadata = parsed.metadata;
      } else {
        values = data.split(',');
      }
    } catch {
      values = data.split(',');
    }
    
    let index = 0;
    
    function buildTree() {
      if (index >= values.length || values[index] === 'null') {
        index++;
        return null;
      }
      
      const node = new TreeNode(parseInt(values[index]));
      index++;
      
      node.left = buildTree();
      node.right = buildTree();
      
      return node;
    }
    
    const tree = buildTree();
    
    // Validate if metadata present
    if (metadata) {
      console.log('Validation:');
      console.log('  Expected nodes:', metadata.nodeCount);
      console.log('  Tree height:', metadata.height);
      console.log('  Timestamp:', metadata.timestamp);
    }
    
    return tree;
  }
  
  // Calculate tree metrics
  getMetrics(root) {
    let nodeCount = 0;
    let height = 0;
    let leafCount = 0;
    
    function traverse(node, level = 0) {
      if (!node) return;
      
      nodeCount++;
      height = Math.max(height, level);
      
      if (!node.left && !node.right) {
        leafCount++;
      }
      
      traverse(node.left, level + 1);
      traverse(node.right, level + 1);
    }
    
    traverse(root);
    
    return { nodeCount, height, leafCount };
  }
}

// Test
console.log('\n=== Advanced Codec ===');

const codec5 = new CodecAdvanced();
const tree5 = new TreeNode(1,
  new TreeNode(2),
  new TreeNode(3,
    new TreeNode(4),
    new TreeNode(5)
  )
);

const serialized5 = codec5.serialize(tree5, true);
console.log('Serialized with metadata:');
console.log(JSON.parse(serialized5));

const deserialized5 = codec5.deserialize(serialized5);
console.log('\nMetrics:', codec5.getMetrics(deserialized5));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. File System Serialization
console.log('\n=== File System Serialization ===');

class FileNode {
  constructor(name, isDirectory = false) {
    this.name = name;
    this.isDirectory = isDirectory;
    this.children = [];
    this.size = 0;
  }
}

class FileSystemCodec {
  serialize(root) {
    const result = [];
    
    function encode(node) {
      if (!node) {
        result.push('null');
        return;
      }
      
      result.push(JSON.stringify({
        name: node.name,
        isDirectory: node.isDirectory,
        size: node.size
      }));
      
      result.push('[');
      for (const child of node.children) {
        encode(child);
      }
      result.push(']');
    }
    
    encode(root);
    return result.join('|');
  }
  
  visualize(root, indent = 0) {
    if (!root) return;
    
    const prefix = ' '.repeat(indent);
    const icon = root.isDirectory ? '📁' : '📄';
    console.log(`${prefix}${icon} ${root.name} ${!root.isDirectory ? `(${root.size}KB)` : ''}`);
    
    for (const child of root.children) {
      this.visualize(child, indent + 2);
    }
  }
}

const fsRoot = new FileNode('root', true);
fsRoot.children.push(new FileNode('document.txt', false));
fsRoot.children[0].size = 10;

const subDir = new FileNode('images', true);
subDir.children.push(new FileNode('photo.jpg', false));
subDir.children[0].size = 500;
fsRoot.children.push(subDir);

const fsCodec = new FileSystemCodec();
console.log('File system:');
fsCodec.visualize(fsRoot);

// 2. Expression Tree Serialization
console.log('\n=== Expression Tree ===');

class ExpressionCodec {
  serialize(root) {
    if (!root) return '';
    
    if (!root.left && !root.right) {
      return root.val.toString();
    }
    
    return `(${this.serialize(root.left)}${root.val}${this.serialize(root.right)})`;
  }
  
  deserialize(expr) {
    // Simple parser for mathematical expressions
    const tokens = expr.match(/\d+|[+\-*/()]/g);
    let index = 0;
    
    const parse = () => {
      if (index >= tokens.length) return null;
      
      const token = tokens[index++];
      
      if (token === '(') {
        const left = parse();
        const op = tokens[index++];
        const right = parse();
        index++; // skip ')'
        
        const node = new TreeNode(op);
        node.left = left;
        node.right = right;
        return node;
      }
      
      return new TreeNode(parseInt(token));
    };
    
    return parse();
  }
  
  evaluate(root) {
    if (!root) return 0;
    
    if (!root.left && !root.right) {
      return parseInt(root.val);
    }
    
    const left = this.evaluate(root.left);
    const right = this.evaluate(root.right);
    
    switch (root.val) {
      case '+': return left + right;
      case '-': return left - right;
      case '*': return left * right;
      case '/': return left / right;
      default: return 0;
    }
  }
}

const exprCodec = new ExpressionCodec();
const exprTree = new TreeNode('+',
  new TreeNode('*',
    new TreeNode(2),
    new TreeNode(3)
  ),
  new TreeNode(4)
);

console.log('Expression:', exprCodec.serialize(exprTree)); // "(2*3)+4"
console.log('Result:', exprCodec.evaluate(exprTree)); // 10

// 3. DOM Tree Serialization
console.log('\n=== DOM Tree Serialization ===');

class DOMNode {
  constructor(tag, attributes = {}) {
    this.tag = tag;
    this.attributes = attributes;
    this.children = [];
    this.text = '';
  }
}

class DOMCodec {
  serialize(root) {
    if (!root) return '';
    
    let html = `<${root.tag}`;
    
    for (const [key, value] of Object.entries(root.attributes)) {
      html += ` ${key}="${value}"`;
    }
    
    html += '>';
    
    if (root.text) {
      html += root.text;
    }
    
    for (const child of root.children) {
      html += this.serialize(child);
    }
    
    html += `</${root.tag}>`;
    
    return html;
  }
  
  deserialize(html) {
    // Simplified HTML parser
    const tagRegex = /<(\w+)([^>]*)>(.*?)<\/\1>/s;
    const match = html.match(tagRegex);
    
    if (!match) return null;
    
    const tag = match[1];
    const attrsStr = match[2];
    const content = match[3];
    
    const node = new DOMNode(tag);
    
    // Parse attributes
    const attrRegex = /(\w+)="([^"]*)"/g;
    let attrMatch;
    while ((attrMatch = attrRegex.exec(attrsStr)) !== null) {
      node.attributes[attrMatch[1]] = attrMatch[2];
    }
    
    // Check if content is text or child nodes
    if (!content.includes('<')) {
      node.text = content;
    }
    
    return node;
  }
}

const domRoot = new DOMNode('div', { class: 'container' });
domRoot.text = 'Hello';

const domCodec = new DOMCodec();
console.log('HTML:', domCodec.serialize(domRoot));

// 4. Decision Tree Serialization
console.log('\n=== Decision Tree ===');

class DecisionNode {
  constructor(question, yesNode = null, noNode = null) {
    this.question = question;
    this.yesNode = yesNode;
    this.noNode = noNode;
  }
}

class DecisionTreeCodec {
  serialize(root) {
    if (!root) return 'null';
    
    return JSON.stringify({
      q: root.question,
      y: JSON.parse(this.serialize(root.yesNode)),
      n: JSON.parse(this.serialize(root.noNode))
    });
  }
  
  deserialize(data) {
    const obj = JSON.parse(data);
    if (obj === 'null') return null;
    
    const node = new DecisionNode(obj.q);
    node.yesNode = this.deserialize(JSON.stringify(obj.y));
    node.noNode = this.deserialize(JSON.stringify(obj.n));
    
    return node;
  }
  
  evaluate(root, answers) {
    if (!root) return null;
    
    if (!root.yesNode && !root.noNode) {
      return root.question; // Leaf node (answer)
    }
    
    const response = answers[root.question];
    
    if (response === true) {
      return this.evaluate(root.yesNode, answers);
    } else {
      return this.evaluate(root.noNode, answers);
    }
  }
}

const decisionTree = new DecisionNode(
  'Is it alive?',
  new DecisionNode(
    'Can it fly?',
    new DecisionNode('Bird'),
    new DecisionNode('Mammal')
  ),
  new DecisionNode('Rock')
);

const dtCodec = new DecisionTreeCodec();
console.log('Decision tree:', dtCodec.serialize(decisionTree));

const answers = { 'Is it alive?': true, 'Can it fly?': false };
console.log('Classification:', dtCodec.evaluate(decisionTree, answers)); // Mammal

// 5. State Machine Serialization
console.log('\n=== State Machine ===');

class StateNode {
  constructor(name, transitions = {}) {
    this.name = name;
    this.transitions = transitions; // { event: nextState }
    this.onEnter = null;
    this.onExit = null;
  }
}

class StateMachineCodec {
  serialize(states) {
    return JSON.stringify(
      Object.entries(states).map(([name, state]) => ({
        name,
        transitions: state.transitions
      }))
    );
  }
  
  deserialize(data) {
    const statesData = JSON.parse(data);
    const states = {};
    
    for (const stateData of statesData) {
      states[stateData.name] = new StateNode(stateData.name, stateData.transitions);
    }
    
    return states;
  }
  
  visualize(states) {
    console.log('State Machine:');
    for (const [name, state] of Object.entries(states)) {
      console.log(`  ${name}:`);
      for (const [event, nextState] of Object.entries(state.transitions)) {
        console.log(`    ${event} -> ${nextState}`);
      }
    }
  }
}

const states = {
  idle: new StateNode('idle', { start: 'running' }),
  running: new StateNode('running', { pause: 'paused', stop: 'idle' }),
  paused: new StateNode('paused', { resume: 'running', stop: 'idle' })
};

const smCodec = new StateMachineCodec();
const serializedSM = smCodec.serialize(states);
console.log('Serialized:', serializedSM);

const deserializedSM = smCodec.deserialize(serializedSM);
smCodec.visualize(deserializedSM);
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

const codecTest = new Codec();

// Empty tree
console.log('Empty:', codecTest.serialize(null)); // "null"

// Single node
const single = new TreeNode(1);
console.log('Single node:', codecTest.serialize(single)); // "1,null,null"

// Left-skewed tree
const leftSkewed = new TreeNode(1, new TreeNode(2, new TreeNode(3)));
console.log('Left-skewed:', codecTest.serialize(leftSkewed));

// Right-skewed tree
const rightSkewed = new TreeNode(1, null, new TreeNode(2, null, new TreeNode(3)));
console.log('Right-skewed:', codecTest.serialize(rightSkewed));

// Complete binary tree
const complete = new TreeNode(1,
  new TreeNode(2, new TreeNode(4), new TreeNode(5)),
  new TreeNode(3, new TreeNode(6), new TreeNode(7))
);
console.log('Complete:', codecTest.serialize(complete));

// Tree with negative values
const negative = new TreeNode(-1, new TreeNode(-2), new TreeNode(-3));
console.log('Negative values:', codecTest.serialize(negative));

// Large values
const large = new TreeNode(1000000, new TreeNode(999999));
console.log('Large values:', codecTest.serialize(large));

// Deep tree
let deepRoot = new TreeNode(1);
let current = deepRoot;
for (let i = 2; i <= 5; i++) {
  current.left = new TreeNode(i);
  current = current.left;
}
console.log('Deep tree:', codecTest.serialize(deepRoot));
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance Comparison ===');

const testTree = new TreeNode(1,
  new TreeNode(2, new TreeNode(4), new TreeNode(5)),
  new TreeNode(3, new TreeNode(6), new TreeNode(7))
);

// Size comparison
const c1 = new Codec();
const c2 = new CodecBFS();
const c3 = new CodecJSON();
const c4 = new CodecBinary();

const s1 = c1.serialize(testTree);
const s2 = c2.serialize(testTree);
const s3 = c3.serialize(testTree);
const s4 = c4.serialize(testTree);

console.log('Pre-order:', s1, `(${s1.length} chars)`);
console.log('BFS:', s2, `(${s2.length} chars)`);
console.log('JSON:', s3, `(${s3.length} chars)`);
console.log('Binary:', s4, `(${s4.length} chars)`);

console.log(`
Algorithm Comparison:
┌──────────────────┬─────────────┬─────────────┬──────────────────┐
│ Method           │ Time        │ Space       │ Best For         │
├──────────────────┼─────────────┼─────────────┼──────────────────┤
│ Pre-order        │ O(n)        │ O(n)        │ Compact,         │
│                  │             │             │ fast rebuild     │
├──────────────────┼─────────────┼─────────────┼──────────────────┤
│ Level-order (BFS)│ O(n)        │ O(n)        │ Readable,        │
│                  │             │             │ layer by layer   │
├──────────────────┼─────────────┼─────────────┼──────────────────┤
│ JSON             │ O(n)        │ O(n)        │ Human-readable,  │
│                  │             │             │ debugging        │
├──────────────────┼─────────────┼─────────────┼──────────────────┤
│ Binary           │ O(n)        │ O(n)        │ Space-efficient, │
│                  │             │             │ production       │
├──────────────────┼─────────────┼─────────────┼──────────────────┤
│ With Metadata    │ O(n)        │ O(n)        │ Validation,      │
│                  │             │             │ versioning       │
└──────────────────┴─────────────┴─────────────┴──────────────────┘

Key Insights:
• Pre-order: serialize root, then left subtree, then right subtree
• Need null markers to reconstruct structure unambiguously
• BFS: level-by-level, can omit trailing nulls
• Deserialization reverses the serialization process
• Must handle edge cases: empty tree, single node, skewed trees

Serialization Formats:
• String: "1,2,null,null,3,4,null,null,5,null,null"
• Array: [1,2,3,null,null,4,5]
• JSON: {"val":1,"left":{"val":2},"right":{"val":3}}
• Binary: "1|1|2|0|0|1|3|..."

Applications:
• File system backup/restore
• Expression tree evaluation
• DOM/HTML parsing and rendering
• Decision tree persistence
• State machine configuration
• Network transmission of tree structures
• Database tree storage

Optimization Tips:
• Use BFS for better readability
• Add metadata for validation and versioning
• Compress with gzip for network transmission
• Use binary format for space efficiency
• Include checksums for data integrity
`);
```

**Interview Tips:**
- Serialization: convert tree structure to string/bytes for storage/transmission
- Deserialization: reconstruct tree from serialized format
- Pre-order traversal: most common, serialize root → left → right with null markers
- Must include null markers to uniquely identify tree structure
- BFS (level-order): more readable, serialize layer by layer
- Deserialization reverses the process using same traversal order
- Pre-order: use recursion with index pointer to track position
- BFS: use queue for both serialization and deserialization
- Time: O(n) for both operations (visit each node once)
- Space: O(n) for storing serialized data and recursion stack
- Edge cases: empty tree (return "null"), single node, skewed trees
- Applications: save/load trees, network transmission, deep copy, persistence
- JSON format: human-readable but verbose, good for debugging
- Binary format: space-efficient for production use
- Consider adding metadata: version, timestamp, node count for validation
- Common mistakes: forgetting null markers, wrong traversal order for deserialize
- Follow-ups: serialize N-ary tree, with parent pointers, generic tree, graph
- Optimization: compress data, use delimiter efficiently, handle special characters
- Clarify: format (string/binary), traversal method, handle duplicates?, size constraints?

</details>

113. Solve the "N-Queens" problem

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Backtracking with Array**
```javascript
/**
 * Classic N-Queens backtracking solution
 * Time Complexity: O(n!)
 * Space Complexity: O(n^2)
 */
function solveNQueens(n) {
  const result = [];
  const board = Array(n).fill(null).map(() => Array(n).fill('.'));
  
  function isValid(row, col) {
    // Check column
    for (let i = 0; i < row; i++) {
      if (board[i][col] === 'Q') return false;
    }
    
    // Check upper-left diagonal
    for (let i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
      if (board[i][j] === 'Q') return false;
    }
    
    // Check upper-right diagonal
    for (let i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
      if (board[i][j] === 'Q') return false;
    }
    
    return true;
  }
  
  function backtrack(row) {
    if (row === n) {
      result.push(board.map(r => r.join('')));
      return;
    }
    
    for (let col = 0; col < n; col++) {
      if (isValid(row, col)) {
        board[row][col] = 'Q';
        backtrack(row + 1);
        board[row][col] = '.';
      }
    }
  }
  
  backtrack(0);
  return result;
}

// Test
console.log('=== N-Queens Backtracking ===');

const solutions4 = solveNQueens(4);
console.log(`4-Queens (${solutions4.length} solutions):`);
solutions4.forEach((solution, idx) => {
  console.log(`\nSolution ${idx + 1}:`);
  solution.forEach(row => console.log(row));
});

const solutions8 = solveNQueens(8);
console.log(`\n8-Queens: ${solutions8.length} solutions`);
```

### **Approach 2: Optimized with Sets**
```javascript
/**
 * Use sets to track attacked positions - O(1) validation
 * Time Complexity: O(n!)
 * Space Complexity: O(n)
 */
function solveNQueensOptimized(n) {
  const result = [];
  const cols = new Set();
  const diag1 = new Set(); // row - col
  const diag2 = new Set(); // row + col
  const board = Array(n).fill(null).map(() => Array(n).fill('.'));
  
  function backtrack(row) {
    if (row === n) {
      result.push(board.map(r => r.join('')));
      return;
    }
    
    for (let col = 0; col < n; col++) {
      const d1 = row - col;
      const d2 = row + col;
      
      if (cols.has(col) || diag1.has(d1) || diag2.has(d2)) {
        continue;
      }
      
      // Place queen
      board[row][col] = 'Q';
      cols.add(col);
      diag1.add(d1);
      diag2.add(d2);
      
      backtrack(row + 1);
      
      // Remove queen
      board[row][col] = '.';
      cols.delete(col);
      diag1.delete(d1);
      diag2.delete(d2);
    }
  }
  
  backtrack(0);
  return result;
}

// Test
console.log('\n=== Optimized with Sets ===');

const start = performance.now();
const optimized = solveNQueensOptimized(8);
const end = performance.now();

console.log(`8-Queens: ${optimized.length} solutions`);
console.log(`Time: ${(end - start).toFixed(2)}ms`);
```

### **Approach 3: Count Solutions Only**
```javascript
/**
 * Count total number of solutions without storing them
 * More memory efficient
 */
function totalNQueens(n) {
  let count = 0;
  const cols = new Set();
  const diag1 = new Set();
  const diag2 = new Set();
  
  function backtrack(row) {
    if (row === n) {
      count++;
      return;
    }
    
    for (let col = 0; col < n; col++) {
      const d1 = row - col;
      const d2 = row + col;
      
      if (!cols.has(col) && !diag1.has(d1) && !diag2.has(d2)) {
        cols.add(col);
        diag1.add(d1);
        diag2.add(d2);
        
        backtrack(row + 1);
        
        cols.delete(col);
        diag1.delete(d1);
        diag2.delete(d2);
      }
    }
  }
  
  backtrack(0);
  return count;
}

// Test
console.log('\n=== Count Solutions ===');

for (let n = 1; n <= 10; n++) {
  const count = totalNQueens(n);
  console.log(`${n}-Queens: ${count} solutions`);
}
```

### **Approach 4: Bit Manipulation**
```javascript
/**
 * Use bits to represent attacked positions
 * Most efficient representation
 */
function solveNQueensBits(n) {
  let count = 0;
  const allOnes = (1 << n) - 1; // All n bits set to 1
  
  function backtrack(cols, diag1, diag2) {
    if (cols === allOnes) {
      count++;
      return;
    }
    
    // Available positions = positions not attacked
    let available = allOnes & ~(cols | diag1 | diag2);
    
    while (available) {
      // Get rightmost bit (lowest available position)
      const position = available & -available;
      
      // Clear this bit
      available -= position;
      
      backtrack(
        cols | position,
        (diag1 | position) << 1,
        (diag2 | position) >> 1
      );
    }
  }
  
  backtrack(0, 0, 0);
  return count;
}

// Test
console.log('\n=== Bit Manipulation ===');

const startBit = performance.now();
const countBit = solveNQueensBits(12);
const endBit = performance.now();

console.log(`12-Queens: ${countBit} solutions`);
console.log(`Time: ${(endBit - startBit).toFixed(2)}ms`);
```

### **Approach 5: Visualize and Animate**
```javascript
/**
 * Visualize N-Queens solution with animations
 */
class NQueensVisualizer {
  constructor(n) {
    this.n = n;
    this.solutions = [];
    this.steps = [];
  }
  
  solve() {
    const board = Array(this.n).fill(null).map(() => Array(this.n).fill('.'));
    const cols = new Set();
    const diag1 = new Set();
    const diag2 = new Set();
    
    const backtrack = (row) => {
      if (row === this.n) {
        this.solutions.push(board.map(r => [...r]));
        this.steps.push({ type: 'solution', board: board.map(r => [...r]) });
        return;
      }
      
      for (let col = 0; col < this.n; col++) {
        const d1 = row - col;
        const d2 = row + col;
        
        if (!cols.has(col) && !diag1.has(d1) && !diag2.has(d2)) {
          // Try placing queen
          board[row][col] = 'Q';
          this.steps.push({ type: 'place', row, col, board: board.map(r => [...r]) });
          
          cols.add(col);
          diag1.add(d1);
          diag2.add(d2);
          
          backtrack(row + 1);
          
          // Backtrack
          board[row][col] = '.';
          this.steps.push({ type: 'remove', row, col, board: board.map(r => [...r]) });
          
          cols.delete(col);
          diag1.delete(d1);
          diag2.delete(d2);
        }
      }
    };
    
    backtrack(0);
    return this.solutions;
  }
  
  printBoard(board) {
    console.log('┌' + '─'.repeat(this.n * 2 + 1) + '┐');
    board.forEach(row => {
      console.log('│ ' + row.join(' ') + ' │');
    });
    console.log('└' + '─'.repeat(this.n * 2 + 1) + '┘');
  }
  
  printSolution(index) {
    if (index >= 0 && index < this.solutions.length) {
      console.log(`\nSolution ${index + 1}:`);
      this.printBoard(this.solutions[index]);
    }
  }
  
  getAllSolutions() {
    return this.solutions;
  }
  
  getStepCount() {
    return this.steps.length;
  }
  
  isValid(board, row, col) {
    // Check if placing queen at (row, col) is valid
    for (let i = 0; i < row; i++) {
      if (board[i][col] === 'Q') return false;
    }
    
    for (let i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
      if (board[i][j] === 'Q') return false;
    }
    
    for (let i = row - 1, j = col + 1; i >= 0 && j < this.n; i--, j++) {
      if (board[i][j] === 'Q') return false;
    }
    
    return true;
  }
}

// Test
console.log('\n=== Visualizer ===');

const viz = new NQueensVisualizer(4);
viz.solve();

console.log(`Total solutions: ${viz.getAllSolutions().length}`);
console.log(`Total steps: ${viz.getStepCount()}`);

viz.printSolution(0);
viz.printSolution(1);
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications of N-Queens logic
 */

// 1. Task Scheduling with Conflicts
console.log('\n=== Task Scheduling ===');

class TaskScheduler {
  constructor(tasks, conflicts) {
    this.tasks = tasks;
    this.conflicts = conflicts; // [[task1, task2], ...] cannot be together
    this.n = tasks.length;
  }
  
  findSchedules(slots) {
    const result = [];
    const schedule = Array(slots).fill(null);
    const used = new Set();
    
    const hasConflict = (taskId, slot) => {
      for (let i = 0; i < slot; i++) {
        if (schedule[i] !== null) {
          const conflict = this.conflicts.find(
            ([a, b]) => 
              (a === taskId && b === schedule[i]) ||
              (b === taskId && a === schedule[i])
          );
          if (conflict) return true;
        }
      }
      return false;
    };
    
    const backtrack = (slot) => {
      if (slot === slots) {
        result.push([...schedule]);
        return;
      }
      
      for (let taskId = 0; taskId < this.n; taskId++) {
        if (!used.has(taskId) && !hasConflict(taskId, slot)) {
          schedule[slot] = taskId;
          used.add(taskId);
          
          backtrack(slot + 1);
          
          schedule[slot] = null;
          used.delete(taskId);
        }
      }
    };
    
    backtrack(0);
    return result;
  }
}

const tasks = ['Task A', 'Task B', 'Task C', 'Task D'];
const conflicts = [[0, 1], [1, 2]]; // A conflicts with B, B conflicts with C

const scheduler = new TaskScheduler(tasks, conflicts);
const schedules = scheduler.findSchedules(4);

console.log(`Valid schedules: ${schedules.length}`);
schedules.slice(0, 2).forEach((schedule, idx) => {
  console.log(`Schedule ${idx + 1}:`, schedule.map(i => tasks[i]).join(' → '));
});

// 2. Seating Arrangement
console.log('\n=== Seating Arrangement ===');

class SeatingArranger {
  constructor(people, incompatibilities) {
    this.people = people;
    this.incompatibilities = incompatibilities; // Set of [person1, person2]
    this.n = people.length;
  }
  
  findArrangements(tables) {
    const result = [];
    const seating = Array(tables).fill(null);
    const seated = new Set();
    
    const canSitTogether = (person, table) => {
      for (let i = 0; i < table; i++) {
        if (seating[i] !== null) {
          const incompatible = this.incompatibilities.some(
            ([p1, p2]) =>
              (p1 === person && p2 === seating[i]) ||
              (p2 === person && p1 === seating[i])
          );
          if (incompatible) return false;
        }
      }
      return true;
    };
    
    const backtrack = (table) => {
      if (table === tables) {
        result.push([...seating]);
        return;
      }
      
      for (let person = 0; person < this.n; person++) {
        if (!seated.has(person) && canSitTogether(person, table)) {
          seating[table] = person;
          seated.add(person);
          
          backtrack(table + 1);
          
          seating[table] = null;
          seated.delete(person);
        }
      }
    };
    
    backtrack(0);
    return result;
  }
}

const people = ['Alice', 'Bob', 'Charlie', 'Diana'];
const incompatibilities = [[0, 1], [2, 3]]; // Alice-Bob, Charlie-Diana

const seater = new SeatingArranger(people, incompatibilities);
const arrangements = seater.findArrangements(4);

console.log(`Valid arrangements: ${arrangements.length}`);
arrangements.slice(0, 2).forEach((arr, idx) => {
  console.log(`Arrangement ${idx + 1}:`, arr.map(i => people[i]).join(', '));
});

// 3. Resource Allocation
console.log('\n=== Resource Allocation ===');

class ResourceAllocator {
  constructor(resources, projects) {
    this.resources = resources;
    this.projects = projects;
    this.n = resources.length;
  }
  
  allocate(requirements) {
    // requirements[project] = [required resource types]
    const allocation = {};
    const used = new Set();
    
    const backtrack = (projectIdx) => {
      if (projectIdx === this.projects.length) {
        return true; // Found valid allocation
      }
      
      const project = this.projects[projectIdx];
      const required = requirements[project];
      
      for (let resIdx = 0; resIdx < this.n; resIdx++) {
        if (!used.has(resIdx) && required.includes(this.resources[resIdx])) {
          allocation[project] = this.resources[resIdx];
          used.add(resIdx);
          
          if (backtrack(projectIdx + 1)) {
            return true;
          }
          
          delete allocation[project];
          used.delete(resIdx);
        }
      }
      
      return false;
    };
    
    return backtrack(0) ? allocation : null;
  }
}

const resources = ['Dev1', 'Dev2', 'Designer1', 'Designer2'];
const projects = ['ProjectA', 'ProjectB'];
const requirements = {
  'ProjectA': ['Dev1', 'Dev2'],
  'ProjectB': ['Designer1', 'Designer2']
};

const allocator = new ResourceAllocator(resources, projects);
const allocation = allocator.allocate(requirements);

console.log('Allocation:', allocation);

// 4. Exam Scheduling
console.log('\n=== Exam Scheduling ===');

class ExamScheduler {
  constructor(courses, studentEnrollments) {
    this.courses = courses;
    this.studentEnrollments = studentEnrollments; // { student: [courses] }
    this.n = courses.length;
  }
  
  scheduleExams(timeSlots) {
    const schedule = Array(timeSlots).fill(null);
    const used = new Set();
    
    const hasConflict = (course, slot) => {
      // Check if any student enrolled in this course
      // is also enrolled in already scheduled courses in this slot
      for (let i = 0; i < slot; i++) {
        if (schedule[i] !== null) {
          const scheduled = schedule[i];
          
          for (const [student, courses] of Object.entries(this.studentEnrollments)) {
            if (courses.includes(course) && courses.includes(scheduled)) {
              return true; // Same student in both courses
            }
          }
        }
      }
      return false;
    };
    
    const backtrack = (slot) => {
      if (slot === timeSlots || used.size === this.n) {
        return used.size === this.n;
      }
      
      for (let courseIdx = 0; courseIdx < this.n; courseIdx++) {
        if (!used.has(courseIdx) && !hasConflict(this.courses[courseIdx], slot)) {
          schedule[slot] = this.courses[courseIdx];
          used.add(courseIdx);
          
          if (backtrack(slot + 1)) {
            return true;
          }
          
          schedule[slot] = null;
          used.delete(courseIdx);
        }
      }
      
      return false;
    };
    
    return backtrack(0) ? schedule.filter(s => s !== null) : null;
  }
}

const courses = ['Math', 'Physics', 'Chemistry', 'Biology'];
const enrollments = {
  'Student1': ['Math', 'Physics'],
  'Student2': ['Physics', 'Chemistry'],
  'Student3': ['Math', 'Biology']
};

const examScheduler = new ExamScheduler(courses, enrollments);
const examSchedule = examScheduler.scheduleExams(4);

console.log('Exam schedule:', examSchedule);

// 5. Graph Coloring (Sudoku-like)
console.log('\n=== Graph Coloring ===');

class GraphColoring {
  constructor(graph) {
    this.graph = graph; // adjacency list
    this.n = Object.keys(graph).length;
  }
  
  color(numColors) {
    const colors = {};
    
    const isValidColor = (node, color) => {
      for (const neighbor of this.graph[node]) {
        if (colors[neighbor] === color) {
          return false;
        }
      }
      return true;
    };
    
    const backtrack = (nodeIdx) => {
      const nodes = Object.keys(this.graph);
      
      if (nodeIdx === nodes.length) {
        return true;
      }
      
      const node = nodes[nodeIdx];
      
      for (let color = 0; color < numColors; color++) {
        if (isValidColor(node, color)) {
          colors[node] = color;
          
          if (backtrack(nodeIdx + 1)) {
            return true;
          }
          
          delete colors[node];
        }
      }
      
      return false;
    };
    
    return backtrack(0) ? colors : null;
  }
}

const graph = {
  'A': ['B', 'C'],
  'B': ['A', 'C', 'D'],
  'C': ['A', 'B', 'D'],
  'D': ['B', 'C']
};

const coloring = new GraphColoring(graph);
const result = coloring.color(3);

console.log('Graph coloring:', result);
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// N = 1 (trivial)
console.log('1-Queen:', totalNQueens(1)); // 1

// N = 2 (no solution)
console.log('2-Queens:', totalNQueens(2)); // 0

// N = 3 (no solution)
console.log('3-Queens:', totalNQueens(3)); // 0

// N = 4 (2 solutions)
const sol4 = solveNQueens(4);
console.log('4-Queens:', sol4.length, 'solutions');

// Large N
const sol10 = totalNQueens(10);
console.log('10-Queens:', sol10, 'solutions');

// Verify solution validity
function isValidSolution(board) {
  const n = board.length;
  
  for (let row = 0; row < n; row++) {
    for (let col = 0; col < n; col++) {
      if (board[row][col] === 'Q') {
        // Check column
        for (let i = row + 1; i < n; i++) {
          if (board[i][col] === 'Q') return false;
        }
        
        // Check diagonals
        for (let i = row + 1, j = col + 1; i < n && j < n; i++, j++) {
          if (board[i][j] === 'Q') return false;
        }
        
        for (let i = row + 1, j = col - 1; i < n && j >= 0; i++, j--) {
          if (board[i][j] === 'Q') return false;
        }
      }
    }
  }
  
  return true;
}

const testSol = solveNQueens(8);
console.log('All 8-Queens solutions valid:', testSol.every(s => isValidSolution(s.map(row => row.split('')))));
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Algorithm Comparison:
┌──────────────────────┬─────────────┬─────────────┬──────────────┐
│ Algorithm            │ Time        │ Space       │ Best For     │
├──────────────────────┼─────────────┼─────────────┼──────────────┤
│ Backtracking (array) │ O(n!)       │ O(n²)       │ Store all    │
│                      │             │             │ solutions    │
├──────────────────────┼─────────────┼─────────────┼──────────────┤
│ Optimized (sets)     │ O(n!)       │ O(n)        │ Faster,      │
│                      │             │             │ O(1) checks  │
├──────────────────────┼─────────────┼─────────────┼──────────────┤
│ Count only           │ O(n!)       │ O(n)        │ Memory       │
│                      │             │             │ efficient    │
├──────────────────────┼─────────────┼─────────────┼──────────────┤
│ Bit manipulation     │ O(n!)       │ O(n)        │ Fastest,     │
│                      │             │             │ large N      │
└──────────────────────┴─────────────┴─────────────┴──────────────┘

Known Solutions Count:
┌─────┬───────────┐
│ N   │ Solutions │
├─────┼───────────┤
│ 1   │ 1         │
│ 2   │ 0         │
│ 3   │ 0         │
│ 4   │ 2         │
│ 5   │ 10        │
│ 6   │ 4         │
│ 7   │ 40        │
│ 8   │ 92        │
│ 9   │ 352       │
│ 10  │ 724       │
│ 11  │ 2,680     │
│ 12  │ 14,200    │
└─────┴───────────┘

Key Insights:
• Classic backtracking problem with constraint satisfaction
• Place queens row by row, trying each column
• Check three constraints: column, two diagonals
• Diagonal identification: row-col (negative diagonal), row+col (positive)
• Use sets for O(1) constraint checking instead of O(n)
• Bit manipulation: fastest for counting solutions only

Optimization Techniques:
• Sets for O(1) validation vs O(n) array checking
• Track columns and diagonals with sets
• Bit manipulation for ultimate speed
• Prune early: skip invalid positions immediately
• Symmetry: can reduce by 2x (mirror solutions)

Applications:
• Task scheduling with conflicts
• Seating arrangements with incompatibilities
• Resource allocation with constraints
• Exam scheduling avoiding conflicts
• Graph coloring problems
• Constraint satisfaction problems (CSP)

Tips:
• Always place one queen per row (reduces search space)
• Diagonal formula: main diagonal (row-col), anti-diagonal (row+col)
• Base case: all rows filled = solution found
• No solution for N=2 and N=3
• Exponential growth in solution count
`);
```

**Interview Tips:**
- N-Queens: place N queens on N×N chessboard so no two queens attack each other
- Queens attack: same row, column, or diagonal
- Classic backtracking: place queens row by row, try each column
- Constraint checking: column, negative diagonal (row-col), positive diagonal (row+col)
- Three sets optimization: track occupied columns, both diagonals for O(1) validation
- Time: O(n!) in worst case due to trying all permutations with pruning
- Space: O(n²) for board or O(n) with sets optimization
- Place one queen per row: reduces search space significantly
- Diagonal formula: row-col gives diagonal index, row+col gives anti-diagonal
- Base case: when row == n, found valid solution
- Backtracking: place queen, recurse, remove queen if no solution
- Edge cases: n=1 (trivial, 1 solution), n=2,3 (no solutions), n=4 (2 solutions)
- Bit manipulation: most efficient for large N, represent positions as bits
- Applications: scheduling, seating, resource allocation, CSP problems
- Common mistakes: checking only rows/columns, forgetting one diagonal, wrong diagonal formula
- Optimization: use sets instead of arrays for O(1) lookups, bit manipulation for speed
- Follow-ups: count solutions only, find first solution, generalized to M×N board
- Clarify: return all solutions or count? just one solution? optimize for speed or readability?
- Interview: explain backtracking clearly, show constraint checking, discuss optimizations

</details>

114. Implement a function for "Edit Distance" (Levenshtein distance)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Dynamic Programming (2D Table)**
```javascript
/**
 * Edit distance using DP table
 * Time Complexity: O(m * n)
 * Space Complexity: O(m * n)
 */
function minDistance(word1, word2) {
  const m = word1.length;
  const n = word2.length;
  
  // dp[i][j] = min operations to convert word1[0..i-1] to word2[0..j-1]
  const dp = Array(m + 1).fill(null).map(() => Array(n + 1).fill(0));
  
  // Base cases: empty string conversions
  for (let i = 0; i <= m; i++) {
    dp[i][0] = i; // Delete all characters from word1
  }
  
  for (let j = 0; j <= n; j++) {
    dp[0][j] = j; // Insert all characters to form word2
  }
  
  // Fill the DP table
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        // Characters match, no operation needed
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        // Min of: replace, delete, insert
        dp[i][j] = 1 + Math.min(
          dp[i - 1][j - 1], // Replace
          dp[i - 1][j],     // Delete from word1
          dp[i][j - 1]      // Insert into word1
        );
      }
    }
  }
  
  return dp[m][n];
}

// Test
console.log('=== Edit Distance DP ===');

console.log(minDistance('horse', 'ros')); // 3 (horse -> rorse -> rose -> ros)
console.log(minDistance('intention', 'execution')); // 5
console.log(minDistance('kitten', 'sitting')); // 3
console.log(minDistance('', 'abc')); // 3
console.log(minDistance('abc', '')); // 3
console.log(minDistance('abc', 'abc')); // 0
```

### **Approach 2: Space-Optimized DP (1D Array)**
```javascript
/**
 * Optimize space to O(n) using single array
 * Time Complexity: O(m * n)
 * Space Complexity: O(n)
 */
function minDistanceOptimized(word1, word2) {
  const m = word1.length;
  const n = word2.length;
  
  // Use only one row at a time
  let prev = Array(n + 1).fill(0);
  
  // Initialize first row
  for (let j = 0; j <= n; j++) {
    prev[j] = j;
  }
  
  for (let i = 1; i <= m; i++) {
    const curr = Array(n + 1).fill(0);
    curr[0] = i;
    
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        curr[j] = prev[j - 1];
      } else {
        curr[j] = 1 + Math.min(
          prev[j - 1], // Replace
          prev[j],     // Delete
          curr[j - 1]  // Insert
        );
      }
    }
    
    prev = curr;
  }
  
  return prev[n];
}

// Test
console.log('\n=== Space-Optimized ===');

console.log(minDistanceOptimized('horse', 'ros')); // 3
console.log(minDistanceOptimized('intention', 'execution')); // 5
console.log(minDistanceOptimized('kitten', 'sitting')); // 3
```

### **Approach 3: With Operation Tracking**
```javascript
/**
 * Track the actual operations performed
 */
function minDistanceWithOps(word1, word2) {
  const m = word1.length;
  const n = word2.length;
  
  const dp = Array(m + 1).fill(null).map(() => Array(n + 1).fill(0));
  const ops = Array(m + 1).fill(null).map(() => Array(n + 1).fill(null));
  
  // Base cases
  for (let i = 0; i <= m; i++) {
    dp[i][0] = i;
    ops[i][0] = i > 0 ? 'delete' : 'none';
  }
  
  for (let j = 0; j <= n; j++) {
    dp[0][j] = j;
    ops[0][j] = j > 0 ? 'insert' : 'none';
  }
  
  // Fill DP table
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];
        ops[i][j] = 'match';
      } else {
        const replace = dp[i - 1][j - 1];
        const deleteOp = dp[i - 1][j];
        const insert = dp[i][j - 1];
        
        const minCost = Math.min(replace, deleteOp, insert);
        dp[i][j] = 1 + minCost;
        
        if (minCost === replace) {
          ops[i][j] = 'replace';
        } else if (minCost === deleteOp) {
          ops[i][j] = 'delete';
        } else {
          ops[i][j] = 'insert';
        }
      }
    }
  }
  
  // Reconstruct operations
  const operations = [];
  let i = m, j = n;
  
  while (i > 0 || j > 0) {
    const op = ops[i][j];
    
    if (op === 'match') {
      i--;
      j--;
    } else if (op === 'replace') {
      operations.unshift(`Replace '${word1[i - 1]}' with '${word2[j - 1]}' at position ${i - 1}`);
      i--;
      j--;
    } else if (op === 'delete') {
      operations.unshift(`Delete '${word1[i - 1]}' at position ${i - 1}`);
      i--;
    } else if (op === 'insert') {
      operations.unshift(`Insert '${word2[j - 1]}' at position ${i}`);
      j--;
    } else {
      break;
    }
  }
  
  return {
    distance: dp[m][n],
    operations
  };
}

// Test
console.log('\n=== With Operations ===');

const result1 = minDistanceWithOps('horse', 'ros');
console.log(`Distance: ${result1.distance}`);
console.log('Operations:');
result1.operations.forEach(op => console.log(`  ${op}`));

console.log();

const result2 = minDistanceWithOps('kitten', 'sitting');
console.log(`Distance: ${result2.distance}`);
console.log('Operations:');
result2.operations.forEach(op => console.log(`  ${op}`));
```

### **Approach 4: Recursive with Memoization**
```javascript
/**
 * Top-down approach with memoization
 * Time Complexity: O(m * n)
 * Space Complexity: O(m * n)
 */
function minDistanceMemo(word1, word2) {
  const memo = new Map();
  
  function helper(i, j) {
    // Base cases
    if (i === 0) return j; // Insert all remaining chars from word2
    if (j === 0) return i; // Delete all remaining chars from word1
    
    const key = `${i},${j}`;
    if (memo.has(key)) return memo.get(key);
    
    let result;
    
    if (word1[i - 1] === word2[j - 1]) {
      // Characters match
      result = helper(i - 1, j - 1);
    } else {
      // Try all three operations
      result = 1 + Math.min(
        helper(i - 1, j - 1), // Replace
        helper(i - 1, j),     // Delete
        helper(i, j - 1)      // Insert
      );
    }
    
    memo.set(key, result);
    return result;
  }
  
  return helper(word1.length, word2.length);
}

// Test
console.log('\n=== Recursive with Memoization ===');

console.log(minDistanceMemo('horse', 'ros')); // 3
console.log(minDistanceMemo('intention', 'execution')); // 5
console.log(minDistanceMemo('kitten', 'sitting')); // 3
```

### **Approach 5: With Cost Customization**
```javascript
/**
 * Allow custom costs for different operations
 */
class EditDistanceCalculator {
  constructor(insertCost = 1, deleteCost = 1, replaceCost = 1) {
    this.insertCost = insertCost;
    this.deleteCost = deleteCost;
    this.replaceCost = replaceCost;
  }
  
  calculate(word1, word2) {
    const m = word1.length;
    const n = word2.length;
    
    const dp = Array(m + 1).fill(null).map(() => Array(n + 1).fill(0));
    
    // Base cases with custom costs
    for (let i = 0; i <= m; i++) {
      dp[i][0] = i * this.deleteCost;
    }
    
    for (let j = 0; j <= n; j++) {
      dp[0][j] = j * this.insertCost;
    }
    
    for (let i = 1; i <= m; i++) {
      for (let j = 1; j <= n; j++) {
        if (word1[i - 1] === word2[j - 1]) {
          dp[i][j] = dp[i - 1][j - 1];
        } else {
          dp[i][j] = Math.min(
            dp[i - 1][j - 1] + this.replaceCost,
            dp[i - 1][j] + this.deleteCost,
            dp[i][j - 1] + this.insertCost
          );
        }
      }
    }
    
    return dp[m][n];
  }
  
  // Calculate similarity percentage
  similarity(word1, word2) {
    const distance = this.calculate(word1, word2);
    const maxLen = Math.max(word1.length, word2.length);
    
    if (maxLen === 0) return 100;
    
    return ((1 - distance / maxLen) * 100).toFixed(2);
  }
}

// Test
console.log('\n=== Custom Costs ===');

const calc1 = new EditDistanceCalculator(1, 1, 1);
console.log('Equal costs:', calc1.calculate('horse', 'ros')); // 3

const calc2 = new EditDistanceCalculator(1, 2, 1);
console.log('Delete costs 2x:', calc2.calculate('horse', 'ros')); // Different

console.log('\nSimilarity:');
const calc3 = new EditDistanceCalculator();
console.log('horse vs ros:', calc3.similarity('horse', 'ros') + '%');
console.log('kitten vs sitting:', calc3.similarity('kitten', 'sitting') + '%');
console.log('hello vs hello:', calc3.similarity('hello', 'hello') + '%');
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Spell Checker
console.log('\n=== Spell Checker ===');

class SpellChecker {
  constructor(dictionary) {
    this.dictionary = dictionary;
  }
  
  suggest(word, maxDistance = 2, maxSuggestions = 5) {
    const suggestions = [];
    
    for (const dictWord of this.dictionary) {
      const distance = minDistance(word, dictWord);
      
      if (distance <= maxDistance) {
        suggestions.push({ word: dictWord, distance });
      }
    }
    
    // Sort by distance
    suggestions.sort((a, b) => a.distance - b.distance);
    
    return suggestions.slice(0, maxSuggestions);
  }
  
  correct(text) {
    const words = text.split(' ');
    const corrected = [];
    
    for (const word of words) {
      if (this.dictionary.includes(word.toLowerCase())) {
        corrected.push(word);
      } else {
        const suggestions = this.suggest(word.toLowerCase(), 2, 1);
        corrected.push(suggestions.length > 0 ? suggestions[0].word : word);
      }
    }
    
    return corrected.join(' ');
  }
}

const dictionary = ['hello', 'world', 'cat', 'dog', 'house', 'mouse'];
const spellChecker = new SpellChecker(dictionary);

console.log('Suggestions for "helo":', spellChecker.suggest('helo'));
console.log('Suggestions for "wrld":', spellChecker.suggest('wrld'));
console.log('Auto-correct:', spellChecker.correct('helo wrld'));

// 2. DNA Sequence Alignment
console.log('\n=== DNA Sequence Alignment ===');

class DNAAligner {
  constructor() {
    this.matchScore = 0;
    this.mismatchPenalty = 1;
    this.gapPenalty = 2;
  }
  
  align(seq1, seq2) {
    const m = seq1.length;
    const n = seq2.length;
    
    const dp = Array(m + 1).fill(null).map(() => Array(n + 1).fill(0));
    
    // Initialize with gap penalties
    for (let i = 0; i <= m; i++) {
      dp[i][0] = i * this.gapPenalty;
    }
    
    for (let j = 0; j <= n; j++) {
      dp[0][j] = j * this.gapPenalty;
    }
    
    // Fill DP table
    for (let i = 1; i <= m; i++) {
      for (let j = 1; j <= n; j++) {
        const match = seq1[i - 1] === seq2[j - 1];
        
        dp[i][j] = Math.min(
          dp[i - 1][j - 1] + (match ? this.matchScore : this.mismatchPenalty),
          dp[i - 1][j] + this.gapPenalty,
          dp[i][j - 1] + this.gapPenalty
        );
      }
    }
    
    return {
      alignmentScore: dp[m][n],
      similarity: ((1 - dp[m][n] / Math.max(m, n)) * 100).toFixed(2) + '%'
    };
  }
  
  findSimilarSequences(query, database, threshold = 0.7) {
    const results = [];
    
    for (const [id, sequence] of Object.entries(database)) {
      const alignment = this.align(query, sequence);
      const similarity = parseFloat(alignment.similarity) / 100;
      
      if (similarity >= threshold) {
        results.push({ id, sequence, similarity: alignment.similarity });
      }
    }
    
    return results.sort((a, b) => parseFloat(b.similarity) - parseFloat(a.similarity));
  }
}

const aligner = new DNAAligner();
console.log('Alignment:', aligner.align('ACGT', 'AGCT'));

const database = {
  'seq1': 'ACGTACGT',
  'seq2': 'ACGTACGG',
  'seq3': 'TGCATGCA'
};

console.log('Similar sequences:', aligner.findSimilarSequences('ACGTACGT', database, 0.8));

// 3. Fuzzy String Matching
console.log('\n=== Fuzzy Search ===');

class FuzzySearch {
  constructor(items) {
    this.items = items;
  }
  
  search(query, threshold = 0.5) {
    const results = [];
    
    for (const item of this.items) {
      const distance = minDistance(query.toLowerCase(), item.toLowerCase());
      const maxLen = Math.max(query.length, item.length);
      const similarity = 1 - distance / maxLen;
      
      if (similarity >= threshold) {
        results.push({
          item,
          similarity: (similarity * 100).toFixed(2) + '%',
          distance
        });
      }
    }
    
    return results.sort((a, b) => parseFloat(b.similarity) - parseFloat(a.similarity));
  }
  
  findClosest(query, count = 3) {
    const allResults = this.items.map(item => {
      const distance = minDistance(query.toLowerCase(), item.toLowerCase());
      return { item, distance };
    });
    
    allResults.sort((a, b) => a.distance - b.distance);
    return allResults.slice(0, count);
  }
}

const items = ['JavaScript', 'Python', 'Java', 'TypeScript', 'Ruby', 'Go'];
const fuzzy = new FuzzySearch(items);

console.log('Fuzzy search for "javscript":', fuzzy.search('javscript', 0.7));
console.log('Closest to "pythn":', fuzzy.findClosest('pythn', 2));

// 4. Plagiarism Detection
console.log('\n=== Plagiarism Detection ===');

class PlagiarismDetector {
  constructor(threshold = 0.8) {
    this.threshold = threshold;
  }
  
  compare(text1, text2) {
    // Split into sentences
    const sentences1 = text1.split(/[.!?]+/).filter(s => s.trim());
    const sentences2 = text2.split(/[.!?]+/).filter(s => s.trim());
    
    let totalSimilarity = 0;
    let matches = 0;
    
    for (const sent1 of sentences1) {
      for (const sent2 of sentences2) {
        const distance = minDistance(sent1.trim(), sent2.trim());
        const maxLen = Math.max(sent1.length, sent2.length);
        const similarity = 1 - distance / maxLen;
        
        if (similarity >= this.threshold) {
          matches++;
          totalSimilarity += similarity;
        }
      }
    }
    
    return {
      matches,
      averageSimilarity: matches > 0 ? (totalSimilarity / matches * 100).toFixed(2) + '%' : '0%',
      isPlagiarized: matches > 0
    };
  }
  
  detect(document, corpus) {
    const results = [];
    
    for (const [id, text] of Object.entries(corpus)) {
      const comparison = this.compare(document, text);
      
      if (comparison.isPlagiarized) {
        results.push({ id, ...comparison });
      }
    }
    
    return results;
  }
}

const detector = new PlagiarismDetector(0.8);
const doc = 'The quick brown fox jumps. Over the lazy dog.';
const corpus = {
  'doc1': 'The quick brown fox jumps. Over a lazy dog.',
  'doc2': 'A completely different text here.'
};

console.log('Plagiarism check:', detector.detect(doc, corpus));

// 5. Version Control Diff
console.log('\n=== Version Control Diff ===');

class TextDiffer {
  diff(oldText, newText) {
    const oldLines = oldText.split('\n');
    const newLines = newText.split('\n');
    
    const m = oldLines.length;
    const n = newLines.length;
    
    const dp = Array(m + 1).fill(null).map(() => Array(n + 1).fill(0));
    const ops = Array(m + 1).fill(null).map(() => Array(n + 1).fill(null));
    
    // Base cases
    for (let i = 0; i <= m; i++) {
      dp[i][0] = i;
      ops[i][0] = 'delete';
    }
    
    for (let j = 0; j <= n; j++) {
      dp[0][j] = j;
      ops[0][j] = 'insert';
    }
    
    // Fill DP
    for (let i = 1; i <= m; i++) {
      for (let j = 1; j <= n; j++) {
        if (oldLines[i - 1] === newLines[j - 1]) {
          dp[i][j] = dp[i - 1][j - 1];
          ops[i][j] = 'match';
        } else {
          const costs = [
            dp[i - 1][j - 1],
            dp[i - 1][j],
            dp[i][j - 1]
          ];
          
          const minCost = Math.min(...costs);
          dp[i][j] = 1 + minCost;
          
          if (minCost === costs[0]) ops[i][j] = 'replace';
          else if (minCost === costs[1]) ops[i][j] = 'delete';
          else ops[i][j] = 'insert';
        }
      }
    }
    
    // Reconstruct diff
    const diff = [];
    let i = m, j = n;
    
    while (i > 0 || j > 0) {
      const op = ops[i][j];
      
      if (op === 'match') {
        diff.unshift({ type: ' ', line: oldLines[i - 1] });
        i--;
        j--;
      } else if (op === 'replace') {
        diff.unshift({ type: '-', line: oldLines[i - 1] });
        diff.unshift({ type: '+', line: newLines[j - 1] });
        i--;
        j--;
      } else if (op === 'delete') {
        diff.unshift({ type: '-', line: oldLines[i - 1] });
        i--;
      } else if (op === 'insert') {
        diff.unshift({ type: '+', line: newLines[j - 1] });
        j--;
      } else {
        break;
      }
    }
    
    return diff;
  }
  
  printDiff(diff) {
    diff.forEach(({ type, line }) => {
      console.log(`${type} ${line}`);
    });
  }
}

const differ = new TextDiffer();
const oldText = 'Hello World\nThis is a test\nGoodbye';
const newText = 'Hello World\nThis is modified\nGoodbye';

const diff = differ.diff(oldText, newText);
console.log('Diff:');
differ.printDiff(diff);

// 6. Autocomplete with Typo Tolerance
console.log('\n=== Autocomplete ===');

class Autocomplete {
  constructor(dictionary) {
    this.dictionary = dictionary;
  }
  
  suggest(prefix, maxDistance = 1, maxResults = 5) {
    const suggestions = [];
    
    for (const word of this.dictionary) {
      // Check if word starts with prefix (with tolerance)
      if (word.startsWith(prefix)) {
        suggestions.push({ word, distance: 0, exact: true });
      } else {
        // Calculate distance for prefix match with typos
        const prefixDistance = minDistance(prefix, word.substring(0, prefix.length));
        
        if (prefixDistance <= maxDistance) {
          suggestions.push({ word, distance: prefixDistance, exact: false });
        }
      }
    }
    
    // Sort: exact matches first, then by distance
    suggestions.sort((a, b) => {
      if (a.exact !== b.exact) return a.exact ? -1 : 1;
      return a.distance - b.distance;
    });
    
    return suggestions.slice(0, maxResults).map(s => s.word);
  }
}

const autoDict = ['javascript', 'java', 'python', 'typescript', 'ruby'];
const autocomplete = new Autocomplete(autoDict);

console.log('Autocomplete "jav":', autocomplete.suggest('jav'));
console.log('Autocomplete "pyt" (with typo tolerance):', autocomplete.suggest('pyt', 1));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Empty strings
console.log('Empty to "abc":', minDistance('', 'abc')); // 3
console.log('"abc" to empty:', minDistance('abc', '')); // 3
console.log('Both empty:', minDistance('', '')); // 0

// Same strings
console.log('Same strings:', minDistance('hello', 'hello')); // 0

// Single character
console.log('Single char:', minDistance('a', 'b')); // 1
console.log('Single char same:', minDistance('a', 'a')); // 0

// One character off
console.log('Insert one:', minDistance('abc', 'abcd')); // 1
console.log('Delete one:', minDistance('abcd', 'abc')); // 1
console.log('Replace one:', minDistance('abc', 'abd')); // 1

// Completely different
console.log('Completely different:', minDistance('abc', 'xyz')); // 3

// Long strings
console.log('Long strings:', minDistance('abcdefghij', 'jihgfedcba')); // 10

// Special characters
console.log('Special chars:', minDistance('hello!', 'hello?')); // 1

// Case sensitivity
console.log('Case sensitive:', minDistance('Hello', 'hello')); // 1

// Reversed
console.log('Reversed:', minDistance('abc', 'cba')); // 2
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

function benchmark(fn, name, word1, word2) {
  const start = performance.now();
  const result = fn(word1, word2);
  const end = performance.now();
  console.log(`${name}: ${result} (${(end - start).toFixed(3)}ms)`);
}

const w1 = 'algorithm';
const w2 = 'altruistic';

benchmark(minDistance, '2D DP', w1, w2);
benchmark(minDistanceOptimized, '1D DP', w1, w2);
benchmark(minDistanceMemo, 'Memoization', w1, w2);

console.log(`
Algorithm Comparison:
┌────────────────────┬─────────────┬─────────────┬──────────────┐
│ Algorithm          │ Time        │ Space       │ Best For     │
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ 2D DP              │ O(m×n)      │ O(m×n)      │ Tracking ops,│
│                    │             │             │ clarity      │
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ 1D DP (optimized)  │ O(m×n)      │ O(n)        │ Space-       │
│                    │             │             │ efficient    │
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ Recursive + Memo   │ O(m×n)      │ O(m×n)      │ Top-down,    │
│                    │             │             │ intuitive    │
├────────────────────┼─────────────┼─────────────┼──────────────┤
│ Custom Costs       │ O(m×n)      │ O(m×n)      │ Weighted ops │
└────────────────────┴─────────────┴─────────────┴──────────────┘

Key Insights:
• Edit distance (Levenshtein): minimum operations to transform string A to B
• Three operations: insert, delete, replace (all cost 1)
• DP formula: 
  - If chars match: dp[i][j] = dp[i-1][j-1]
  - Else: dp[i][j] = 1 + min(replace, delete, insert)
• Base cases: dp[i][0] = i (delete all), dp[0][j] = j (insert all)
• Can optimize space from O(m×n) to O(n) using rolling array

Recurrence Relation:
dp[i][j] = {
  dp[i-1][j-1]                      if word1[i-1] == word2[j-1]
  1 + min(
    dp[i-1][j-1],  // Replace word1[i-1] with word2[j-1]
    dp[i-1][j],    // Delete word1[i-1]
    dp[i][j-1]     // Insert word2[j-1]
  )                                 otherwise
}

Real-World Applications:
• Spell checker and auto-correction
• DNA/protein sequence alignment
• Fuzzy string matching and search
• Plagiarism detection
• Version control (diff/patch)
• Autocomplete with typo tolerance
• Similar text recommendation
• Data deduplication

Optimization Tips:
• Use 1D array to save space (only need previous row)
• Early termination if distance exceeds threshold
• For large alphabets, consider preprocessing
• Can weight operations differently (custom costs)
• Parallel processing for multiple comparisons
`);
```

**Interview Tips:**
- Edit distance (Levenshtein): minimum operations to convert string A to B
- Three operations: insert character, delete character, replace character
- Classic DP problem: dp[i][j] = min operations for word1[0..i-1] → word2[0..j-1]
- Base cases: empty string conversions (dp[i][0]=i, dp[0][j]=j)
- If characters match: no operation needed, dp[i][j] = dp[i-1][j-1]
- If different: 1 + min(replace, delete from word1, insert into word1)
- Replace: dp[i-1][j-1], Delete: dp[i-1][j], Insert: dp[i][j-1]
- Time: O(m×n), Space: O(m×n) for 2D table, O(n) with optimization
- Space optimization: only need current and previous row, use rolling array
- Can track operations by storing which operation was used at each step
- Applications: spell checker, DNA alignment, fuzzy search, plagiarism, diff tools
- Variations: weighted costs, different operations, similarity percentage
- Edge cases: empty strings, same strings, single character, completely different
- Common mistakes: wrong base cases, incorrect recurrence, off-by-one indexing
- Follow-ups: find actual operations, custom costs, substring distance, multiple strings
- Optimization: early termination if threshold exceeded, parallel processing
- Similarity: (1 - distance/maxLength) × 100%
- Clarify: case-sensitive? allow which operations? return distance or operations?

</details>

115. Solve the "Maximum Subarray Sum" problem (Kadane's algorithm)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Kadane's Algorithm (Optimal)**
```javascript
/**
 * Find maximum sum of contiguous subarray
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
function maxSubArray(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];
  
  for (let i = 1; i < nums.length; i++) {
    // Either extend existing subarray or start new one
    currentSum = Math.max(nums[i], currentSum + nums[i]);
    maxSum = Math.max(maxSum, currentSum);
  }
  
  return maxSum;
}

// Test
console.log('=== Kadane\'s Algorithm ===');

console.log(maxSubArray([-2,1,-3,4,-1,2,1,-5,4])); // 6 ([4,-1,2,1])
console.log(maxSubArray([1])); // 1
console.log(maxSubArray([5,4,-1,7,8])); // 23 (entire array)
console.log(maxSubArray([-1])); // -1
console.log(maxSubArray([-2,-1])); // -1
```

### **Approach 2: With Subarray Indices**
```javascript
/**
 * Track the start and end indices of maximum subarray
 */
function maxSubArrayWithIndices(nums) {
  let maxSum = nums[0];
  let currentSum = nums[0];
  let start = 0;
  let end = 0;
  let tempStart = 0;
  
  for (let i = 1; i < nums.length; i++) {
    if (nums[i] > currentSum + nums[i]) {
      currentSum = nums[i];
      tempStart = i;
    } else {
      currentSum += nums[i];
    }
    
    if (currentSum > maxSum) {
      maxSum = currentSum;
      start = tempStart;
      end = i;
    }
  }
  
  return {
    maxSum,
    subarray: nums.slice(start, end + 1),
    indices: { start, end }
  };
}

// Test
console.log('\n=== With Indices ===');

const result1 = maxSubArrayWithIndices([-2,1,-3,4,-1,2,1,-5,4]);
console.log('Max sum:', result1.maxSum);
console.log('Subarray:', result1.subarray);
console.log('Indices:', result1.indices);

const result2 = maxSubArrayWithIndices([5,4,-1,7,8]);
console.log('\nMax sum:', result2.maxSum);
console.log('Subarray:', result2.subarray);
```

### **Approach 3: Divide and Conquer**
```javascript
/**
 * Divide and conquer approach
 * Time Complexity: O(n log n)
 * Space Complexity: O(log n) for recursion stack
 */
function maxSubArrayDC(nums) {
  function maxCrossingSum(nums, left, mid, right) {
    // Include elements on left of mid
    let leftSum = -Infinity;
    let sum = 0;
    
    for (let i = mid; i >= left; i--) {
      sum += nums[i];
      leftSum = Math.max(leftSum, sum);
    }
    
    // Include elements on right of mid
    let rightSum = -Infinity;
    sum = 0;
    
    for (let i = mid + 1; i <= right; i++) {
      sum += nums[i];
      rightSum = Math.max(rightSum, sum);
    }
    
    return leftSum + rightSum;
  }
  
  function helper(nums, left, right) {
    if (left === right) {
      return nums[left];
    }
    
    const mid = Math.floor((left + right) / 2);
    
    const leftMax = helper(nums, left, mid);
    const rightMax = helper(nums, mid + 1, right);
    const crossMax = maxCrossingSum(nums, left, mid, right);
    
    return Math.max(leftMax, rightMax, crossMax);
  }
  
  return helper(nums, 0, nums.length - 1);
}

// Test
console.log('\n=== Divide and Conquer ===');

console.log(maxSubArrayDC([-2,1,-3,4,-1,2,1,-5,4])); // 6
console.log(maxSubArrayDC([5,4,-1,7,8])); // 23
```

### **Approach 4: Dynamic Programming (Explicit)**
```javascript
/**
 * Explicit DP array showing all subproblem solutions
 */
function maxSubArrayDP(nums) {
  const n = nums.length;
  const dp = new Array(n);
  
  // dp[i] = maximum sum ending at index i
  dp[0] = nums[0];
  let maxSum = dp[0];
  
  for (let i = 1; i < n; i++) {
    dp[i] = Math.max(nums[i], dp[i - 1] + nums[i]);
    maxSum = Math.max(maxSum, dp[i]);
  }
  
  return {
    maxSum,
    dp // Shows max sum ending at each position
  };
}

// Test
console.log('\n=== Explicit DP ===');

const dpResult = maxSubArrayDP([-2,1,-3,4,-1,2,1,-5,4]);
console.log('Max sum:', dpResult.maxSum);
console.log('DP array:', dpResult.dp);
```

### **Approach 5: All Subarrays (Brute Force)**
```javascript
/**
 * Generate all possible subarrays and find maximum
 * Time Complexity: O(n^2)
 * Space Complexity: O(1)
 * Educational purpose only - not optimal
 */
function maxSubArrayBruteForce(nums) {
  let maxSum = -Infinity;
  let bestSubarray = [];
  
  for (let i = 0; i < nums.length; i++) {
    let currentSum = 0;
    
    for (let j = i; j < nums.length; j++) {
      currentSum += nums[j];
      
      if (currentSum > maxSum) {
        maxSum = currentSum;
        bestSubarray = nums.slice(i, j + 1);
      }
    }
  }
  
  return {
    maxSum,
    subarray: bestSubarray
  };
}

// Test
console.log('\n=== Brute Force ===');

const bruteResult = maxSubArrayBruteForce([-2,1,-3,4,-1,2,1,-5,4]);
console.log('Max sum:', bruteResult.maxSum);
console.log('Subarray:', bruteResult.subarray);
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Stock Trading - Best Time to Buy and Sell
console.log('\n=== Stock Trading ===');

class StockAnalyzer {
  maxProfit(prices) {
    if (prices.length < 2) return 0;
    
    // Convert to price differences
    const changes = [];
    for (let i = 1; i < prices.length; i++) {
      changes.push(prices[i] - prices[i - 1]);
    }
    
    // Find maximum subarray sum in changes
    let maxProfit = 0;
    let currentProfit = 0;
    let buyDay = 0;
    let sellDay = 0;
    let tempBuy = 0;
    
    for (let i = 0; i < changes.length; i++) {
      if (currentProfit + changes[i] < 0) {
        currentProfit = 0;
        tempBuy = i + 1;
      } else {
        currentProfit += changes[i];
      }
      
      if (currentProfit > maxProfit) {
        maxProfit = currentProfit;
        buyDay = tempBuy;
        sellDay = i + 1;
      }
    }
    
    return {
      maxProfit,
      buyDay,
      sellDay,
      buyPrice: prices[buyDay],
      sellPrice: prices[sellDay]
    };
  }
  
  findTradingWindow(prices) {
    const analysis = this.maxProfit(prices);
    
    return {
      ...analysis,
      recommendation: analysis.maxProfit > 0 ? 
        `Buy on day ${analysis.buyDay} at $${analysis.buyPrice}, sell on day ${analysis.sellDay} at $${analysis.sellPrice}` :
        'Hold - no profitable trading opportunity'
    };
  }
}

const analyzer = new StockAnalyzer();
const prices = [7, 1, 5, 3, 6, 4];
console.log('Stock analysis:', analyzer.findTradingWindow(prices));

// 2. Sensor Data Analysis
console.log('\n=== Sensor Data Analysis ===');

class SensorAnalyzer {
  findMaxAnomalyPeriod(readings) {
    // Normalize readings (subtract average)
    const avg = readings.reduce((a, b) => a + b, 0) / readings.length;
    const normalized = readings.map(r => r - avg);
    
    // Find period with maximum deviation
    const result = maxSubArrayWithIndices(normalized);
    
    return {
      period: result.indices,
      totalDeviation: result.maxSum.toFixed(2),
      readings: result.subarray.map((v, i) => ({
        index: result.indices.start + i,
        deviation: v.toFixed(2),
        actual: readings[result.indices.start + i]
      }))
    };
  }
}

const sensorReadings = [20, 22, 19, 25, 30, 35, 28, 21, 20];
const sensorAnalyzer = new SensorAnalyzer();
console.log('Max anomaly period:', sensorAnalyzer.findMaxAnomalyPeriod(sensorReadings));

// 3. Gaming - Maximum Score Streak
console.log('\n=== Gaming Score Streak ===');

class GameAnalyzer {
  findBestStreak(scoreChanges) {
    // scoreChanges: positive for wins, negative for losses
    const result = maxSubArrayWithIndices(scoreChanges);
    
    const totalGames = result.subarray.length;
    const wins = result.subarray.filter(s => s > 0).length;
    const losses = totalGames - wins;
    
    return {
      maxPoints: result.maxSum,
      streakPeriod: result.indices,
      totalGames,
      wins,
      losses,
      winRate: ((wins / totalGames) * 100).toFixed(1) + '%'
    };
  }
}

const scoreChanges = [10, -5, 20, -3, 15, 8, -10, -20, 25, 30];
const gameAnalyzer = new GameAnalyzer();
console.log('Best streak:', gameAnalyzer.findBestStreak(scoreChanges));

// 4. Network Traffic Analysis
console.log('\n=== Network Traffic ===');

class NetworkAnalyzer {
  findPeakTrafficPeriod(traffic) {
    // traffic: array of throughput changes (positive = increase, negative = decrease)
    const result = maxSubArrayWithIndices(traffic);
    
    return {
      peakIncrease: result.maxSum.toFixed(2) + ' Mbps',
      startTime: result.indices.start,
      endTime: result.indices.end,
      duration: result.indices.end - result.indices.start + 1,
      pattern: result.subarray
    };
  }
  
  identifyBottleneck(traffic, threshold) {
    const avgTraffic = traffic.reduce((a, b) => a + b, 0) / traffic.length;
    
    // Find periods exceeding threshold
    const exceedances = traffic.map(t => t > threshold ? t - threshold : 0);
    const result = maxSubArrayWithIndices(exceedances);
    
    if (result.maxSum === 0) {
      return { bottleneck: false, message: 'No bottleneck detected' };
    }
    
    return {
      bottleneck: true,
      severity: result.maxSum.toFixed(2),
      period: result.indices,
      recommendation: 'Consider scaling up during this period'
    };
  }
}

const trafficChanges = [5, -2, 10, 15, -5, 8, -3, 20];
const networkAnalyzer = new NetworkAnalyzer();
console.log('Peak traffic:', networkAnalyzer.findPeakTrafficPeriod(trafficChanges));

// 5. Financial Analysis - Cash Flow
console.log('\n=== Cash Flow Analysis ===');

class CashFlowAnalyzer {
  analyzeCashFlow(transactions) {
    // transactions: positive = income, negative = expenses
    const result = maxSubArrayWithIndices(transactions);
    
    const period = result.subarray;
    const income = period.filter(t => t > 0).reduce((a, b) => a + b, 0);
    const expenses = Math.abs(period.filter(t => t < 0).reduce((a, b) => a + b, 0));
    
    return {
      maxCashFlow: result.maxSum,
      period: result.indices,
      months: result.indices.end - result.indices.start + 1,
      income: income.toFixed(2),
      expenses: expenses.toFixed(2),
      netFlow: result.maxSum.toFixed(2)
    };
  }
  
  findBestInvestmentWindow(cashFlows, minWindow = 3) {
    let bestSum = -Infinity;
    let bestWindow = null;
    
    for (let windowSize = minWindow; windowSize <= cashFlows.length; windowSize++) {
      for (let i = 0; i <= cashFlows.length - windowSize; i++) {
        const windowSum = cashFlows.slice(i, i + windowSize).reduce((a, b) => a + b, 0);
        
        if (windowSum > bestSum) {
          bestSum = windowSum;
          bestWindow = { start: i, end: i + windowSize - 1 };
        }
      }
    }
    
    return {
      maxReturn: bestSum,
      window: bestWindow,
      duration: bestWindow.end - bestWindow.start + 1
    };
  }
}

const cashFlows = [1000, -200, 500, -100, 800, -300, 600];
const cashAnalyzer = new CashFlowAnalyzer();
console.log('Cash flow analysis:', cashAnalyzer.analyzeCashFlow(cashFlows));

// 6. Fitness Tracker - Best Performance Period
console.log('\n=== Fitness Tracker ===');

class FitnessAnalyzer {
  findBestPerformancePeriod(workoutScores) {
    const result = maxSubArrayWithIndices(workoutScores);
    
    const avgScore = result.subarray.reduce((a, b) => a + b, 0) / result.subarray.length;
    
    return {
      totalScore: result.maxSum,
      period: result.indices,
      workouts: result.subarray.length,
      averageScore: avgScore.toFixed(2),
      improvement: ((result.subarray[result.subarray.length - 1] - result.subarray[0]) / result.subarray[0] * 100).toFixed(1) + '%'
    };
  }
}

const workoutScores = [50, 55, -10, 60, 65, 70, -20, 55, 80];
const fitnessAnalyzer = new FitnessAnalyzer();
console.log('Best performance:', fitnessAnalyzer.findBestPerformancePeriod(workoutScores));
```

### **Edge Cases and Testing**
```javascript
console.log('\n=== Edge Cases ===');

// Single element
console.log('Single positive:', maxSubArray([5])); // 5
console.log('Single negative:', maxSubArray([-5])); // -5

// All positive
console.log('All positive:', maxSubArray([1, 2, 3, 4])); // 10

// All negative
console.log('All negative:', maxSubArray([-1, -2, -3, -4])); // -1

// Mixed with zeros
console.log('With zeros:', maxSubArray([0, -1, 0, 2, 0])); // 2

// Large numbers
console.log('Large numbers:', maxSubArray([1000000, -1, 1000000])); // 1999999

// Alternating
console.log('Alternating:', maxSubArray([1, -1, 1, -1, 1])); // 1

// Two elements
console.log('Two positive:', maxSubArray([1, 2])); // 3
console.log('Two negative:', maxSubArray([-1, -2])); // -1
console.log('Mixed two:', maxSubArray([5, -3])); // 5

// Empty (if allowed)
// console.log('Empty:', maxSubArray([])); // Handle based on requirements
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

const testArray = Array.from({ length: 1000 }, () => Math.floor(Math.random() * 200) - 100);

function benchmark(fn, name) {
  const start = performance.now();
  const result = fn(testArray);
  const end = performance.now();
  console.log(`${name}: ${result} (${(end - start).toFixed(3)}ms)`);
}

benchmark(maxSubArray, 'Kadane');
benchmark(maxSubArrayDC, 'Divide & Conquer');
benchmark((arr) => maxSubArrayBruteForce(arr).maxSum, 'Brute Force');

console.log(`
Algorithm Comparison:
┌──────────────────────┬─────────────┬─────────────┬──────────────┐
│ Algorithm            │ Time        │ Space       │ Best For     │
├──────────────────────┼─────────────┼─────────────┼──────────────┤
│ Kadane's Algorithm   │ O(n)        │ O(1)        │ Optimal      │
│                      │             │             │ solution     │
├──────────────────────┼─────────────┼─────────────┼──────────────┤
│ Divide & Conquer     │ O(n log n)  │ O(log n)    │ Educational, │
│                      │             │             │ parallel     │
├──────────────────────┼─────────────┼─────────────┼──────────────┤
│ Dynamic Programming  │ O(n)        │ O(n)        │ Show all     │
│ (explicit)           │             │             │ subproblems  │
├──────────────────────┼─────────────┼─────────────┼──────────────┤
│ Brute Force          │ O(n²)       │ O(1)        │ Small arrays │
│                      │             │             │ only         │
└──────────────────────┴─────────────┴─────────────┴──────────────┘

Kadane's Algorithm Explained:
1. Maintain two variables:
   - currentSum: max sum ending at current position
   - maxSum: overall maximum sum seen so far

2. For each element:
   - Either extend current subarray: currentSum + nums[i]
   - Or start new subarray: nums[i]
   - Choose whichever is larger

3. Update maxSum if currentSum is greater

Key Insight:
At each position, decide: extend previous subarray or start fresh?
If currentSum < 0, starting fresh is always better!

Recurrence:
currentSum = max(nums[i], currentSum + nums[i])
maxSum = max(maxSum, currentSum)

Applications:
• Stock trading (best buy/sell window)
• Sensor data anomaly detection
• Gaming score optimization
• Network traffic peak analysis
• Financial cash flow analysis
• Fitness performance tracking
• Any time-series max sum problem

Variations:
• Find subarray indices (track start/end)
• Find all subarrays with max sum
• Circular array version
• 2D version (maximum sum rectangle)
• K maximum subarrays
• Maximum product subarray

Tips:
• Handle all-negative arrays correctly (return largest element)
• Can be extended to track indices easily
• Works with any data that can be summed
• Foundation for many optimization problems
`);
```

**Interview Tips:**
- Maximum subarray sum: find contiguous subarray with largest sum
- Kadane's algorithm: optimal O(n) solution with O(1) space
- Key insight: at each position, either extend existing subarray or start new one
- Decision rule: extend if (currentSum + nums[i]) > nums[i], else start fresh
- Equivalently: currentSum = max(nums[i], currentSum + nums[i])
- Track maxSum separately as overall maximum seen so far
- Time: O(n) single pass, Space: O(1) only two variables needed
- Handle all-negative arrays: maxSum initializes to first element, not 0
- Can track indices: when starting fresh, update tempStart; when maxSum updates, record final start/end
- Base case: single element (return that element)
- Divide and conquer: O(n log n), less optimal but shows recursive thinking
- DP interpretation: dp[i] = max sum ending at index i
- Recurrence: dp[i] = max(nums[i], dp[i-1] + nums[i])
- Applications: stock trading profit, sensor anomalies, score streaks, traffic peaks, cash flow
- Edge cases: single element, all positive, all negative, zeros, empty array
- Follow-ups: return indices, circular array, 2D max rectangle, max product, k subarrays
- Common mistakes: initializing maxSum to 0 (fails for all-negative), not handling single element
- Variation: maximum product subarray requires tracking both max and min products
- Optimization: early exit not possible, must scan entire array
- Clarify: return sum or subarray? empty subarray allowed? handle negatives how?
- Interview: explain logic clearly, walk through example, mention O(n) is optimal

</details>

### **Real-World Scenarios**

116. Build a simple virtual DOM implementation

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Virtual DOM with Diffing**
```javascript
/**
 * Simple Virtual DOM implementation
 * Virtual nodes represent DOM structure as plain objects
 */

// Create virtual node
function h(type, props, ...children) {
  return {
    type,
    props: props || {},
    children: children.flat()
  };
}

// Create text node
function createTextNode(text) {
  return {
    type: 'TEXT',
    props: {},
    children: [],
    text
  };
}

// Render virtual node to real DOM
function createElement(vnode) {
  if (vnode.type === 'TEXT') {
    return document.createTextNode(vnode.text);
  }
  
  const el = document.createElement(vnode.type);
  
  // Set properties
  Object.entries(vnode.props).forEach(([key, value]) => {
    setAttribute(el, key, value);
  });
  
  // Append children
  vnode.children.forEach(child => {
    el.appendChild(createElement(child));
  });
  
  return el;
}

// Set attribute helper
function setAttribute(el, key, value) {
  if (key.startsWith('on')) {
    const event = key.substring(2).toLowerCase();
    el.addEventListener(event, value);
  } else if (key === 'className') {
    el.className = value;
  } else if (key === 'style' && typeof value === 'object') {
    Object.assign(el.style, value);
  } else {
    el.setAttribute(key, value);
  }
}

// Test
console.log('=== Basic Virtual DOM ===');

const vdom = h('div', { className: 'container' },
  h('h1', {}, 'Hello Virtual DOM'),
  h('p', {}, 'This is a paragraph')
);

console.log('Virtual DOM:', JSON.stringify(vdom, null, 2));

// Render to DOM (in browser environment)
if (typeof document !== 'undefined') {
  const realDOM = createElement(vdom);
  console.log('Real DOM element:', realDOM);
}
```

### **Approach 2: Diffing Algorithm**
```javascript
/**
 * Diff two virtual DOM trees and apply patches
 */

// Diff types
const PATCH_TYPES = {
  REPLACE: 'REPLACE',
  UPDATE_PROPS: 'UPDATE_PROPS',
  UPDATE_TEXT: 'UPDATE_TEXT',
  REORDER: 'REORDER',
  REMOVE: 'REMOVE'
};

// Diff two vnodes
function diff(oldVNode, newVNode) {
  // New node doesn't exist - remove
  if (!newVNode) {
    return { type: PATCH_TYPES.REMOVE };
  }
  
  // Text nodes
  if (oldVNode.type === 'TEXT' && newVNode.type === 'TEXT') {
    if (oldVNode.text !== newVNode.text) {
      return {
        type: PATCH_TYPES.UPDATE_TEXT,
        text: newVNode.text
      };
    }
    return null;
  }
  
  // Different types - replace
  if (oldVNode.type !== newVNode.type) {
    return {
      type: PATCH_TYPES.REPLACE,
      newVNode
    };
  }
  
  // Same type - check props and children
  const propsDiff = diffProps(oldVNode.props, newVNode.props);
  const childrenDiff = diffChildren(oldVNode.children, newVNode.children);
  
  if (propsDiff || childrenDiff) {
    return {
      type: PATCH_TYPES.UPDATE_PROPS,
      props: propsDiff,
      children: childrenDiff
    };
  }
  
  return null;
}

// Diff props
function diffProps(oldProps, newProps) {
  const patches = {};
  
  // Check for changed/new props
  for (const key in newProps) {
    if (oldProps[key] !== newProps[key]) {
      patches[key] = newProps[key];
    }
  }
  
  // Check for removed props
  for (const key in oldProps) {
    if (!(key in newProps)) {
      patches[key] = null;
    }
  }
  
  return Object.keys(patches).length > 0 ? patches : null;
}

// Diff children
function diffChildren(oldChildren, newChildren) {
  const patches = [];
  const length = Math.max(oldChildren.length, newChildren.length);
  
  for (let i = 0; i < length; i++) {
    patches.push(diff(oldChildren[i], newChildren[i]));
  }
  
  return patches;
}

// Apply patch to real DOM
function patch(el, patches) {
  if (!patches) return el;
  
  switch (patches.type) {
    case PATCH_TYPES.REMOVE:
      el.parentNode.removeChild(el);
      return null;
      
    case PATCH_TYPES.REPLACE:
      const newEl = createElement(patches.newVNode);
      el.parentNode.replaceChild(newEl, el);
      return newEl;
      
    case PATCH_TYPES.UPDATE_TEXT:
      el.textContent = patches.text;
      return el;
      
    case PATCH_TYPES.UPDATE_PROPS:
      if (patches.props) {
        updateProps(el, patches.props);
      }
      if (patches.children) {
        patches.children.forEach((childPatch, i) => {
          patch(el.childNodes[i], childPatch);
        });
      }
      return el;
      
    default:
      return el;
  }
}

// Update props on real DOM element
function updateProps(el, props) {
  Object.entries(props).forEach(([key, value]) => {
    if (value === null) {
      el.removeAttribute(key);
    } else {
      setAttribute(el, key, value);
    }
  });
}

// Test
console.log('\n=== Diffing Algorithm ===');

const oldVNode = h('div', { className: 'old' },
  h('h1', {}, 'Old Title'),
  h('p', {}, 'Old content')
);

const newVNode = h('div', { className: 'new' },
  h('h1', {}, 'New Title'),
  h('p', {}, 'New content'),
  h('span', {}, 'Extra element')
);

const patches = diff(oldVNode, newVNode);
console.log('Patches:', JSON.stringify(patches, null, 2));
```

### **Approach 3: Component-Based Virtual DOM**
```javascript
/**
 * Virtual DOM with component support
 */

class Component {
  constructor(props) {
    this.props = props;
    this.state = {};
  }
  
  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.update();
  }
  
  update() {
    if (this._vnode && this._el) {
      const newVNode = this.render();
      const patches = diff(this._vnode, newVNode);
      this._el = patch(this._el, patches);
      this._vnode = newVNode;
    }
  }
  
  render() {
    throw new Error('Component must implement render()');
  }
}

// Mount component to DOM
function mount(component, container) {
  const vnode = component.render();
  const el = createElement(vnode);
  
  component._vnode = vnode;
  component._el = el;
  
  container.appendChild(el);
  
  return component;
}

// Example component
class Counter extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  
  increment() {
    this.setState({ count: this.state.count + 1 });
  }
  
  render() {
    return h('div', { className: 'counter' },
      h('h2', {}, `Count: ${this.state.count}`),
      h('button', { 
        onclick: () => this.increment()
      }, 'Increment')
    );
  }
}

console.log('\n=== Component-Based ===');

const counter = new Counter({ initialCount: 0 });
console.log('Counter component:', counter);
console.log('Initial render:', JSON.stringify(counter.render(), null, 2));
```

### **Approach 4: Key-Based Reconciliation**
```javascript
/**
 * Efficient list rendering with keys
 */

function diffChildrenWithKeys(oldChildren, newChildren) {
  const oldKeyed = {};
  const newKeyed = {};
  
  // Build key maps
  oldChildren.forEach((child, i) => {
    const key = child.props?.key ?? i;
    oldKeyed[key] = child;
  });
  
  newChildren.forEach((child, i) => {
    const key = child.props?.key ?? i;
    newKeyed[key] = child;
  });
  
  const patches = [];
  const moves = [];
  
  // Find all keys
  const allKeys = new Set([
    ...Object.keys(oldKeyed),
    ...Object.keys(newKeyed)
  ]);
  
  allKeys.forEach(key => {
    const oldChild = oldKeyed[key];
    const newChild = newKeyed[key];
    
    if (!newChild) {
      // Removed
      patches.push({ type: 'remove', key });
    } else if (!oldChild) {
      // Added
      patches.push({ type: 'add', key, vnode: newChild });
    } else {
      // Updated
      const patch = diff(oldChild, newChild);
      if (patch) {
        patches.push({ type: 'update', key, patch });
      }
    }
  });
  
  // Detect moves
  const oldOrder = oldChildren.map((c, i) => c.props?.key ?? i);
  const newOrder = newChildren.map((c, i) => c.props?.key ?? i);
  
  if (JSON.stringify(oldOrder) !== JSON.stringify(newOrder)) {
    moves.push({ oldOrder, newOrder });
  }
  
  return { patches, moves };
}

// Test
console.log('\n=== Key-Based Reconciliation ===');

const oldList = [
  h('li', { key: 'a' }, 'Item A'),
  h('li', { key: 'b' }, 'Item B'),
  h('li', { key: 'c' }, 'Item C')
];

const newList = [
  h('li', { key: 'b' }, 'Item B'),
  h('li', { key: 'c' }, 'Item C (updated)'),
  h('li', { key: 'd' }, 'Item D')
];

const listDiff = diffChildrenWithKeys(oldList, newList);
console.log('List diff:', JSON.stringify(listDiff, null, 2));
```

### **Approach 5: Complete Virtual DOM Library**
```javascript
/**
 * Full-featured Virtual DOM implementation
 */

class VirtualDOM {
  constructor() {
    this.rootComponent = null;
    this.rootElement = null;
  }
  
  // Create element
  createElement(vnode) {
    if (typeof vnode === 'string' || typeof vnode === 'number') {
      return document.createTextNode(vnode);
    }
    
    if (vnode.type === 'TEXT') {
      return document.createTextNode(vnode.text);
    }
    
    const el = document.createElement(vnode.type);
    
    // Set props
    this.updateProps(el, {}, vnode.props);
    
    // Render children
    (vnode.children || []).forEach(child => {
      el.appendChild(this.createElement(child));
    });
    
    return el;
  }
  
  // Update props
  updateProps(el, oldProps, newProps) {
    // Remove old props
    Object.keys(oldProps).forEach(key => {
      if (!(key in newProps)) {
        this.removeProp(el, key, oldProps[key]);
      }
    });
    
    // Set new props
    Object.keys(newProps).forEach(key => {
      if (oldProps[key] !== newProps[key]) {
        this.setProp(el, key, newProps[key]);
      }
    });
  }
  
  // Set property
  setProp(el, key, value) {
    if (key === 'className') {
      el.className = value;
    } else if (key === 'style') {
      if (typeof value === 'object') {
        Object.assign(el.style, value);
      } else {
        el.style.cssText = value;
      }
    } else if (key.startsWith('on')) {
      const event = key.substring(2).toLowerCase();
      el.addEventListener(event, value);
    } else if (key === 'checked' || key === 'disabled') {
      el[key] = value;
    } else {
      el.setAttribute(key, value);
    }
  }
  
  // Remove property
  removeProp(el, key, value) {
    if (key.startsWith('on')) {
      const event = key.substring(2).toLowerCase();
      el.removeEventListener(event, value);
    } else if (key === 'className') {
      el.className = '';
    } else {
      el.removeAttribute(key);
    }
  }
  
  // Diff and patch
  updateElement(parent, oldVNode, newVNode, index = 0) {
    // No old node - add new
    if (!oldVNode) {
      parent.appendChild(this.createElement(newVNode));
      return;
    }
    
    const el = parent.childNodes[index];
    
    // No new node - remove old
    if (!newVNode) {
      parent.removeChild(el);
      return;
    }
    
    // Different types - replace
    if (this.hasChanged(oldVNode, newVNode)) {
      parent.replaceChild(this.createElement(newVNode), el);
      return;
    }
    
    // Same type - update props and children
    if (newVNode.type && newVNode.type !== 'TEXT') {
      this.updateProps(el, oldVNode.props, newVNode.props);
      
      const oldChildren = oldVNode.children || [];
      const newChildren = newVNode.children || [];
      const maxLength = Math.max(oldChildren.length, newChildren.length);
      
      for (let i = 0; i < maxLength; i++) {
        this.updateElement(el, oldChildren[i], newChildren[i], i);
      }
    }
  }
  
  // Check if nodes changed
  hasChanged(oldVNode, newVNode) {
    // String/number nodes
    if (typeof oldVNode !== typeof newVNode) {
      return true;
    }
    
    if (typeof oldVNode === 'string' || typeof oldVNode === 'number') {
      return oldVNode !== newVNode;
    }
    
    // Different types
    if (oldVNode.type !== newVNode.type) {
      return true;
    }
    
    // Text nodes
    if (oldVNode.type === 'TEXT') {
      return oldVNode.text !== newVNode.text;
    }
    
    return false;
  }
  
  // Render to container
  render(vnode, container) {
    if (!this.rootElement) {
      this.rootElement = this.createElement(vnode);
      container.appendChild(this.rootElement);
    } else {
      this.updateElement(container, this.rootComponent, vnode);
    }
    
    this.rootComponent = vnode;
  }
}

// Test
console.log('\n=== Complete Virtual DOM ===');

const vdom = new VirtualDOM();

// Example app
function createApp(count) {
  return h('div', { className: 'app' },
    h('h1', {}, `Count: ${count}`),
    h('ul', {},
      ...Array.from({ length: count }, (_, i) => 
        h('li', { key: i }, `Item ${i + 1}`)
      )
    )
  );
}

console.log('App with count 3:', JSON.stringify(createApp(3), null, 2));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Simple UI Framework
console.log('\n=== UI Framework ===');

class UIFramework {
  constructor() {
    this.vdom = new VirtualDOM();
    this.currentTree = null;
    this.container = null;
  }
  
  createComponent(render, initialState = {}) {
    const component = {
      state: initialState,
      
      setState(newState) {
        this.state = { ...this.state, ...newState };
        component.update();
      },
      
      update() {
        if (!component.container) return;
        
        const newTree = render(component.state);
        this.vdom.updateElement(
          component.container,
          component.currentTree,
          newTree,
          0
        );
        component.currentTree = newTree;
      },
      
      mount(container) {
        component.container = container;
        component.currentTree = render(component.state);
        container.innerHTML = '';
        this.vdom.render(component.currentTree, container);
      }
    };
    
    return component;
  }
}

const framework = new UIFramework();

// Example: Todo app component
const todoApp = framework.createComponent(
  (state) => {
    return h('div', { className: 'todo-app' },
      h('h1', {}, 'Todo List'),
      h('ul', {},
        ...state.todos.map((todo, i) =>
          h('li', { key: i }, todo)
        )
      ),
      h('p', {}, `Total: ${state.todos.length}`)
    );
  },
  { todos: ['Learn Virtual DOM', 'Build app'] }
);

console.log('Todo app created:', todoApp);

// 2. Form Management
console.log('\n=== Form Management ===');

class FormManager {
  constructor() {
    this.vdom = new VirtualDOM();
  }
  
  createForm(fields, onSubmit) {
    const state = {};
    fields.forEach(field => {
      state[field.name] = field.defaultValue || '';
    });
    
    return {
      state,
      
      handleChange(name, value) {
        state[name] = value;
        this.render();
      },
      
      handleSubmit() {
        onSubmit(state);
      },
      
      render() {
        return h('form', { 
          onsubmit: (e) => {
            e.preventDefault();
            this.handleSubmit();
          }
        },
          ...fields.map(field =>
            h('div', { className: 'form-group' },
              h('label', {}, field.label),
              h('input', {
                type: field.type || 'text',
                name: field.name,
                value: state[field.name],
                oninput: (e) => this.handleChange(field.name, e.target.value)
              })
            )
          ),
          h('button', { type: 'submit' }, 'Submit')
        );
      }
    };
  }
}

const formManager = new FormManager();
const loginForm = formManager.createForm(
  [
    { name: 'username', label: 'Username' },
    { name: 'password', label: 'Password', type: 'password' }
  ],
  (data) => console.log('Form submitted:', data)
);

console.log('Login form:', loginForm);

// 3. List Virtualization
console.log('\n=== List Virtualization ===');

class VirtualList {
  constructor(items, itemHeight, viewportHeight) {
    this.items = items;
    this.itemHeight = itemHeight;
    this.viewportHeight = viewportHeight;
    this.scrollTop = 0;
  }
  
  getVisibleRange() {
    const start = Math.floor(this.scrollTop / this.itemHeight);
    const visibleCount = Math.ceil(this.viewportHeight / this.itemHeight);
    const end = Math.min(start + visibleCount, this.items.length);
    
    return { start, end };
  }
  
  render() {
    const { start, end } = this.getVisibleRange();
    const visibleItems = this.items.slice(start, end);
    
    const offsetY = start * this.itemHeight;
    const totalHeight = this.items.length * this.itemHeight;
    
    return h('div', {
      className: 'virtual-list',
      style: {
        height: `${this.viewportHeight}px`,
        overflow: 'auto'
      },
      onscroll: (e) => {
        this.scrollTop = e.target.scrollTop;
        this.update();
      }
    },
      h('div', {
        style: {
          height: `${totalHeight}px`,
          position: 'relative'
        }
      },
        h('div', {
          style: {
            transform: `translateY(${offsetY}px)`
          }
        },
          ...visibleItems.map((item, i) =>
            h('div', {
              key: start + i,
              style: { height: `${this.itemHeight}px` }
            }, item)
          )
        )
      )
    );
  }
}

const bigList = new VirtualList(
  Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`),
  30,
  300
);

console.log('Virtual list (10000 items):', bigList.getVisibleRange());

// 4. Animation System
console.log('\n=== Animation System ===');

class AnimationManager {
  constructor(vdom) {
    this.vdom = vdom;
    this.animations = new Map();
  }
  
  animate(key, from, to, duration, easing = 'linear') {
    const startTime = Date.now();
    
    const easingFunctions = {
      linear: t => t,
      easeIn: t => t * t,
      easeOut: t => t * (2 - t),
      easeInOut: t => t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t
    };
    
    const ease = easingFunctions[easing];
    
    const update = () => {
      const elapsed = Date.now() - startTime;
      const progress = Math.min(elapsed / duration, 1);
      const eased = ease(progress);
      
      const current = from + (to - from) * eased;
      
      if (progress < 1) {
        this.animations.set(key, current);
        requestAnimationFrame(update);
      } else {
        this.animations.delete(key);
      }
      
      this.render();
    };
    
    update();
  }
  
  render() {
    return h('div', {},
      ...Array.from(this.animations.entries()).map(([key, value]) =>
        h('div', {
          key,
          style: {
            transform: `translateX(${value}px)`,
            transition: 'transform 0.016s'
          }
        }, `Animation ${key}: ${value.toFixed(0)}px`)
      )
    );
  }
}

const animManager = new AnimationManager();
console.log('Animation manager created');

// 5. Router with Virtual DOM
console.log('\n=== Router ===');

class Router {
  constructor(routes) {
    this.routes = routes;
    this.currentRoute = '/';
    
    window.addEventListener('popstate', () => {
      this.currentRoute = window.location.pathname;
      this.render();
    });
  }
  
  navigate(path) {
    this.currentRoute = path;
    window.history.pushState({}, '', path);
    this.render();
  }
  
  render() {
    const route = this.routes[this.currentRoute] || this.routes['/404'];
    
    return h('div', { className: 'app' },
      h('nav', {},
        ...Object.keys(this.routes).map(path =>
          h('a', {
            href: path,
            onclick: (e) => {
              e.preventDefault();
              this.navigate(path);
            }
          }, path)
        )
      ),
      h('main', {}, route())
    );
  }
}

const router = new Router({
  '/': () => h('h1', {}, 'Home Page'),
  '/about': () => h('h1', {}, 'About Page'),
  '/contact': () => h('h1', {}, 'Contact Page'),
  '/404': () => h('h1', {}, 'Not Found')
});

console.log('Router with 4 routes created');
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Virtual DOM Benefits:
┌────────────────────────┬──────────────────────────────────┐
│ Aspect                 │ Benefit                          │
├────────────────────────┼──────────────────────────────────┤
│ Performance            │ Batches DOM updates, minimizes   │
│                        │ reflows/repaints                 │
├────────────────────────┼──────────────────────────────────┤
│ Declarative            │ Describe what UI should look like│
│                        │ not how to change it             │
├────────────────────────┼──────────────────────────────────┤
│ Abstraction            │ Platform-agnostic (works on      │
│                        │ server, mobile, canvas)          │
├────────────────────────┼──────────────────────────────────┤
│ Testability            │ Easy to test without real DOM    │
├────────────────────────┼──────────────────────────────────┤
│ Time Travel            │ Can store/replay states          │
└────────────────────────┴──────────────────────────────────┘

Key Concepts:
• Virtual DOM: lightweight JS representation of real DOM
• Reconciliation: process of comparing old and new virtual trees
• Diffing: algorithm to find minimal set of changes
• Patching: applying changes to real DOM
• Keys: help identify which items changed/moved/removed

Diffing Algorithm Steps:
1. Compare node types
2. If different → replace entire subtree
3. If same → compare props
4. Recursively diff children
5. Generate patch operations

Common Optimizations:
• Key-based reconciliation for lists
• Shallow comparison for props
• Batching multiple updates
• Fiber architecture (incremental rendering)
• Memoization of unchanged subtrees

Real-World Uses:
• React, Preact, Vue use virtual DOM
• Server-side rendering
• Mobile apps (React Native)
• Canvas/WebGL rendering
• Testing without browser

Limitations:
• Memory overhead for virtual tree
• Not always faster for simple updates
• Learning curve for developers
• Debugging can be harder
`);
```

**Interview Tips:**
- Virtual DOM: lightweight JavaScript representation of real DOM tree
- Purpose: minimize expensive DOM operations by batching and optimizing updates
- Core concepts: virtual nodes (vnodes), diffing algorithm, patching
- VNode structure: { type, props, children } - plain JavaScript object
- Diffing: compare old and new virtual trees to find minimal changes
- Reconciliation: process of updating real DOM based on diff
- Three main operations: create (add new), update (modify), remove (delete)
- Key optimization: use "key" prop to track list items across renders
- Keys help identify which items moved, added, or removed in lists
- Diffing algorithm: O(n) with heuristics vs O(n³) naive tree diff
- Heuristics: same level comparison, different types = replace subtree
- Benefits: declarative UI, batched updates, cross-platform, testable
- Trade-offs: memory for virtual tree, not always faster for simple DOM updates
- createElement: converts vnode to real DOM element
- patch: applies diff results to real DOM
- Used by: React, Vue, Preact, Inferno
- Component pattern: encapsulate state and render logic
- Lifecycle: mount (initial render), update (re-render), unmount (cleanup)
- Follow-ups: fiber architecture, concurrent rendering, SSR, fragments
- Clarify: support JSX? handle events? component lifecycle? state management?

</details>

117. Implement a reactive state management system (like Redux)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Store with Reducers**
```javascript
/**
 * Simple Redux-like store implementation
 */

function createStore(reducer, initialState) {
  let state = initialState;
  const listeners = [];
  
  const getState = () => state;
  
  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
    return action;
  };
  
  const subscribe = (listener) => {
    listeners.push(listener);
    
    // Return unsubscribe function
    return () => {
      const index = listeners.indexOf(listener);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    };
  };
  
  // Initialize state
  dispatch({ type: '@@INIT' });
  
  return { getState, dispatch, subscribe };
}

// Test
console.log('=== Basic Store ===');

// Counter reducer
function counterReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'SET':
      return { ...state, count: action.payload };
    default:
      return state;
  }
}

const store = createStore(counterReducer);

const unsubscribe = store.subscribe(() => {
  console.log('State changed:', store.getState());
});

store.dispatch({ type: 'INCREMENT' });
store.dispatch({ type: 'INCREMENT' });
store.dispatch({ type: 'DECREMENT' });
store.dispatch({ type: 'SET', payload: 10 });

unsubscribe();
```

### **Approach 2: Middleware Support**
```javascript
/**
 * Add middleware support for async actions, logging, etc.
 */

function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, initialState) => {
    const store = createStore(reducer, initialState);
    let dispatch = store.dispatch;
    
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    };
    
    const chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);
    
    return { ...store, dispatch };
  };
}

function compose(...funcs) {
  if (funcs.length === 0) return arg => arg;
  if (funcs.length === 1) return funcs[0];
  return funcs.reduce((a, b) => (...args) => a(b(...args)));
}

// Logger middleware
const logger = (store) => (next) => (action) => {
  console.log('Dispatching:', action);
  console.log('Previous state:', store.getState());
  
  const result = next(action);
  
  console.log('Next state:', store.getState());
  return result;
};

// Thunk middleware for async actions
const thunk = (store) => (next) => (action) => {
  if (typeof action === 'function') {
    return action(store.dispatch, store.getState);
  }
  return next(action);
};

// Test
console.log('\n=== Middleware Support ===');

const enhancedCreateStore = applyMiddleware(logger, thunk)(createStore);
const storeWithMiddleware = enhancedCreateStore(counterReducer);

storeWithMiddleware.dispatch({ type: 'INCREMENT' });

// Async action
const incrementAsync = (delay) => (dispatch, getState) => {
  setTimeout(() => {
    dispatch({ type: 'INCREMENT' });
    console.log('Async increment completed');
  }, delay);
};

storeWithMiddleware.dispatch(incrementAsync(100));
```

### **Approach 3: Combine Reducers**
```javascript
/**
 * Combine multiple reducers into one
 */

function combineReducers(reducers) {
  return (state = {}, action) => {
    const nextState = {};
    let hasChanged = false;
    
    for (const key in reducers) {
      const reducer = reducers[key];
      const previousStateForKey = state[key];
      const nextStateForKey = reducer(previousStateForKey, action);
      
      nextState[key] = nextStateForKey;
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey;
    }
    
    return hasChanged ? nextState : state;
  };
}

// Test
console.log('\n=== Combine Reducers ===');

// Todo reducer
function todosReducer(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, { id: Date.now(), text: action.payload, done: false }];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.payload
          ? { ...todo, done: !todo.done }
          : todo
      );
    case 'REMOVE_TODO':
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
}

// User reducer
function userReducer(state = { name: '', isLoggedIn: false }, action) {
  switch (action.type) {
    case 'LOGIN':
      return { name: action.payload, isLoggedIn: true };
    case 'LOGOUT':
      return { name: '', isLoggedIn: false };
    default:
      return state;
  }
}

const rootReducer = combineReducers({
  counter: counterReducer,
  todos: todosReducer,
  user: userReducer
});

const appStore = createStore(rootReducer);

appStore.subscribe(() => {
  console.log('App state:', appStore.getState());
});

appStore.dispatch({ type: 'INCREMENT' });
appStore.dispatch({ type: 'ADD_TODO', payload: 'Learn Redux' });
appStore.dispatch({ type: 'LOGIN', payload: 'John' });
```

### **Approach 4: Selectors and Memoization**
```javascript
/**
 * Efficient state selection with memoization
 */

function createSelector(...inputSelectors) {
  const resultFunc = inputSelectors.pop();
  let lastArgs = null;
  let lastResult = null;
  
  return (state) => {
    const args = inputSelectors.map(selector => selector(state));
    
    if (lastArgs && args.every((arg, i) => arg === lastArgs[i])) {
      return lastResult;
    }
    
    lastArgs = args;
    lastResult = resultFunc(...args);
    return lastResult;
  };
}

// Test
console.log('\n=== Selectors ===');

// Basic selectors
const getTodos = (state) => state.todos;
const getUser = (state) => state.user;

// Memoized selectors
const getCompletedTodos = createSelector(
  getTodos,
  (todos) => {
    console.log('Computing completed todos...');
    return todos.filter(todo => todo.done);
  }
);

const getTodoStats = createSelector(
  getTodos,
  (todos) => ({
    total: todos.length,
    completed: todos.filter(t => t.done).length,
    pending: todos.filter(t => !t.done).length
  })
);

const state = {
  todos: [
    { id: 1, text: 'Task 1', done: true },
    { id: 2, text: 'Task 2', done: false },
    { id: 3, text: 'Task 3', done: true }
  ],
  user: { name: 'John', isLoggedIn: true }
};

console.log('Completed todos:', getCompletedTodos(state));
console.log('Todo stats:', getTodoStats(state));
console.log('Completed todos (cached):', getCompletedTodos(state));
```

### **Approach 5: Observable Store with RxJS-like API**
```javascript
/**
 * Reactive store using observables
 */

class Observable {
  constructor(subscribe) {
    this._subscribe = subscribe;
  }
  
  subscribe(observer) {
    return this._subscribe(observer);
  }
  
  map(fn) {
    return new Observable(observer => {
      return this.subscribe({
        next: value => observer.next(fn(value)),
        error: err => observer.error(err),
        complete: () => observer.complete()
      });
    });
  }
  
  filter(predicate) {
    return new Observable(observer => {
      return this.subscribe({
        next: value => predicate(value) && observer.next(value),
        error: err => observer.error(err),
        complete: () => observer.complete()
      });
    });
  }
}

class ObservableStore {
  constructor(reducer, initialState) {
    this.state = initialState;
    this.reducer = reducer;
    this.observers = [];
  }
  
  dispatch(action) {
    const previousState = this.state;
    this.state = this.reducer(this.state, action);
    
    if (this.state !== previousState) {
      this.observers.forEach(observer => {
        observer.next(this.state);
      });
    }
    
    return action;
  }
  
  select(selector) {
    return new Observable(observer => {
      const listener = (state) => {
        observer.next(selector(state));
      };
      
      // Emit current value
      listener(this.state);
      
      // Subscribe to changes
      this.observers.push({ next: listener });
      
      // Return unsubscribe
      return () => {
        const index = this.observers.findIndex(obs => obs.next === listener);
        if (index > -1) {
          this.observers.splice(index, 1);
        }
      };
    });
  }
  
  getState() {
    return this.state;
  }
}

// Test
console.log('\n=== Observable Store ===');

const observableStore = new ObservableStore(counterReducer, { count: 0 });

observableStore.select(state => state.count)
  .filter(count => count % 2 === 0)
  .subscribe({
    next: count => console.log('Even count:', count)
  });

observableStore.dispatch({ type: 'INCREMENT' }); // 1 (not logged)
observableStore.dispatch({ type: 'INCREMENT' }); // 2 (logged)
observableStore.dispatch({ type: 'INCREMENT' }); // 3 (not logged)
observableStore.dispatch({ type: 'INCREMENT' }); // 4 (logged)
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Shopping Cart
console.log('\n=== Shopping Cart ===');

const cartReducer = (state = { items: [], total: 0 }, action) => {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingItem = state.items.find(item => item.id === action.payload.id);
      
      if (existingItem) {
        return {
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
          total: state.total + action.payload.price
        };
      }
      
      return {
        items: [...state.items, { ...action.payload, quantity: 1 }],
        total: state.total + action.payload.price
      };
      
    case 'REMOVE_ITEM':
      const item = state.items.find(item => item.id === action.payload);
      return {
        items: state.items.filter(item => item.id !== action.payload),
        total: state.total - (item.price * item.quantity)
      };
      
    case 'CLEAR_CART':
      return { items: [], total: 0 };
      
    default:
      return state;
  }
};

const cartStore = createStore(cartReducer);

cartStore.subscribe(() => {
  const state = cartStore.getState();
  console.log(`Cart: ${state.items.length} items, Total: $${state.total.toFixed(2)}`);
});

cartStore.dispatch({ type: 'ADD_ITEM', payload: { id: 1, name: 'Book', price: 19.99 } });
cartStore.dispatch({ type: 'ADD_ITEM', payload: { id: 2, name: 'Pen', price: 2.99 } });
cartStore.dispatch({ type: 'ADD_ITEM', payload: { id: 1, name: 'Book', price: 19.99 } });

// 2. Form State Management
console.log('\n=== Form Management ===');

const formReducer = (state = { fields: {}, errors: {}, isValid: true }, action) => {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        fields: {
          ...state.fields,
          [action.payload.name]: action.payload.value
        }
      };
      
    case 'SET_ERROR':
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.payload.field]: action.payload.message
        },
        isValid: false
      };
      
    case 'CLEAR_ERRORS':
      return {
        ...state,
        errors: {},
        isValid: true
      };
      
    case 'RESET_FORM':
      return { fields: {}, errors: {}, isValid: true };
      
    default:
      return state;
  }
};

const formStore = createStore(formReducer);

formStore.dispatch({ type: 'SET_FIELD', payload: { name: 'email', value: 'user@example.com' } });
formStore.dispatch({ type: 'SET_FIELD', payload: { name: 'password', value: '12345' } });
formStore.dispatch({ type: 'SET_ERROR', payload: { field: 'password', message: 'Password too short' } });

console.log('Form state:', formStore.getState());

// 3. Undo/Redo
console.log('\n=== Undo/Redo ===');

function undoable(reducer) {
  const initialState = {
    past: [],
    present: reducer(undefined, { type: '@@INIT' }),
    future: []
  };
  
  return (state = initialState, action) => {
    const { past, present, future } = state;
    
    switch (action.type) {
      case 'UNDO':
        if (past.length === 0) return state;
        
        return {
          past: past.slice(0, -1),
          present: past[past.length - 1],
          future: [present, ...future]
        };
        
      case 'REDO':
        if (future.length === 0) return state;
        
        return {
          past: [...past, present],
          present: future[0],
          future: future.slice(1)
        };
        
      default:
        const newPresent = reducer(present, action);
        
        if (present === newPresent) return state;
        
        return {
          past: [...past, present],
          present: newPresent,
          future: []
        };
    }
  };
}

const undoableCounter = undoable(counterReducer);
const undoStore = createStore(undoableCounter);

undoStore.subscribe(() => {
  const state = undoStore.getState();
  console.log('Count:', state.present.count, `(${state.past.length} undo, ${state.future.length} redo)`);
});

undoStore.dispatch({ type: 'INCREMENT' });
undoStore.dispatch({ type: 'INCREMENT' });
undoStore.dispatch({ type: 'INCREMENT' });
undoStore.dispatch({ type: 'UNDO' });
undoStore.dispatch({ type: 'UNDO' });
undoStore.dispatch({ type: 'REDO' });

// 4. WebSocket Integration
console.log('\n=== WebSocket Integration ===');

const wsMiddleware = (url) => (store) => (next) => (action) => {
  if (action.type === 'WS_CONNECT') {
    const ws = new WebSocket(url);
    
    ws.onmessage = (event) => {
      const data = JSON.parse(event.data);
      store.dispatch({ type: 'WS_MESSAGE', payload: data });
    };
    
    ws.onopen = () => {
      store.dispatch({ type: 'WS_CONNECTED' });
    };
    
    ws.onclose = () => {
      store.dispatch({ type: 'WS_DISCONNECTED' });
    };
    
    // Store WebSocket instance
    store.ws = ws;
  }
  
  if (action.type === 'WS_SEND' && store.ws) {
    store.ws.send(JSON.stringify(action.payload));
  }
  
  return next(action);
};

// Example usage (won't actually connect)
console.log('WebSocket middleware configured');

// 5. Persistence
console.log('\n=== Persistence ===');

const persistMiddleware = (key) => (store) => (next) => (action) => {
  const result = next(action);
  
  // Save to localStorage
  try {
    const state = store.getState();
    localStorage.setItem(key, JSON.stringify(state));
  } catch (e) {
    console.warn('Failed to persist state:', e);
  }
  
  return result;
};

function loadPersistedState(key) {
  try {
    const serialized = localStorage.getItem(key);
    return serialized ? JSON.parse(serialized) : undefined;
  } catch (e) {
    console.warn('Failed to load state:', e);
    return undefined;
  }
}

// Usage example
const persistedState = loadPersistedState('app-state');
console.log('Loaded persisted state:', persistedState);
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
State Management Patterns:
┌─────────────────────┬────────────────────────────────────┐
│ Pattern             │ Characteristics                    │
├─────────────────────┼────────────────────────────────────┤
│ Redux               │ Single store, reducers, actions    │
│                     │ Predictable, time-travel debugging │
├─────────────────────┼────────────────────────────────────┤
│ MobX                │ Observable state, automatic updates│
│                     │ Less boilerplate, reactive         │
├─────────────────────┼────────────────────────────────────┤
│ Zustand             │ Hooks-based, simple API            │
│                     │ No providers, minimal boilerplate  │
├─────────────────────┼────────────────────────────────────┤
│ Recoil              │ Atomic state, derived state        │
│                     │ React-specific, fine-grained       │
└─────────────────────┴────────────────────────────────────┘

Redux Principles:
1. Single source of truth (one store)
2. State is read-only (only changed via actions)
3. Changes made with pure functions (reducers)

Core Concepts:
• Store: holds application state
• Actions: plain objects describing what happened
• Reducers: pure functions (state, action) => newState
• Dispatch: send action to store
• Subscribe: listen for state changes

Middleware:
• Sits between dispatch and reducer
• Can modify, delay, or cancel actions
• Use cases: logging, async, routing, crash reporting
• Signature: store => next => action => result

Best Practices:
• Keep reducers pure (no side effects)
• Normalize state shape (flat, not nested)
• Use selectors for derived data
• Memoize expensive selectors
• Split reducers by domain
• Use action creators for consistency
• Add TypeScript for type safety

Common Patterns:
• Async actions with thunks
• Normalized state with entities
• Optimistic updates
• Undo/redo with history
• Persistence to localStorage
• DevTools integration

Performance:
• Shallow comparison for changes
• Reselect for memoized selectors
• Immutable updates (spread, Object.assign)
• Batch updates when possible
• Normalize to avoid deep nesting
`);
```

**Interview Tips:**
- State management: centralized store for application state
- Redux pattern: single store, actions, reducers, unidirectional data flow
- Store: holds state, provides getState(), dispatch(), subscribe()
- Actions: plain objects with "type" property describing what happened
- Reducers: pure functions (state, action) => newState, no side effects
- Dispatch: send action to store, triggers reducer, notifies subscribers
- Immutability: never mutate state directly, always return new objects
- Middleware: intercept actions before they reach reducer
- Middleware signature: store => next => action => result
- Common middleware: thunk (async), logger, router, persistence
- Thunk: dispatch functions instead of objects for async operations
- CombineReducers: merge multiple reducers into single root reducer
- Selectors: functions that extract/compute derived state
- Memoization: cache selector results to avoid recomputation
- Three principles: single source of truth, read-only state, pure reducers
- Benefits: predictable state, time-travel debugging, testable, devtools
- Alternatives: MobX (observable), Zustand (hooks), Recoil (atomic), Context API
- Use cases: complex state, multiple components sharing data, state history
- Trade-offs: boilerplate code, learning curve, overkill for simple apps
- Follow-ups: async actions, normalization, performance, devtools integration
- Clarify: sync or async? middleware? persistence? multiple stores? TypeScript?

</details>

118. Create a dependency injection container

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Dependency Injection Container**
```javascript
/**
 * Simple DI container with register and resolve
 */

class DIContainer {
  constructor() {
    this.services = new Map();
  }
  
  register(name, definition, dependencies = []) {
    this.services.set(name, {
      definition,
      dependencies,
      instance: null,
      singleton: false
    });
  }
  
  registerSingleton(name, definition, dependencies = []) {
    this.services.set(name, {
      definition,
      dependencies,
      instance: null,
      singleton: true
    });
  }
  
  resolve(name) {
    const service = this.services.get(name);
    
    if (!service) {
      throw new Error(`Service '${name}' not found`);
    }
    
    // Return existing singleton instance
    if (service.singleton && service.instance) {
      return service.instance;
    }
    
    // Resolve dependencies
    const dependencies = service.dependencies.map(dep => this.resolve(dep));
    
    // Create instance
    const instance = typeof service.definition === 'function'
      ? new service.definition(...dependencies)
      : service.definition;
    
    // Store singleton instance
    if (service.singleton) {
      service.instance = instance;
    }
    
    return instance;
  }
}

// Test
console.log('=== Basic DI Container ===');

class Database {
  constructor() {
    this.connection = 'Connected to DB';
  }
  
  query(sql) {
    return `Executing: ${sql}`;
  }
}

class UserRepository {
  constructor(database) {
    this.db = database;
  }
  
  findAll() {
    return this.db.query('SELECT * FROM users');
  }
}

class UserService {
  constructor(userRepository) {
    this.repository = userRepository;
  }
  
  getUsers() {
    return this.repository.findAll();
  }
}

const container = new DIContainer();

container.registerSingleton('database', Database);
container.register('userRepository', UserRepository, ['database']);
container.register('userService', UserService, ['userRepository']);

const userService = container.resolve('userService');
console.log(userService.getUsers());

// Verify singleton
const db1 = container.resolve('database');
const db2 = container.resolve('database');
console.log('Same instance:', db1 === db2); // true
```

### **Approach 2: Factory and Value Registration**
```javascript
/**
 * Support different registration types: class, factory, value
 */

class AdvancedDIContainer {
  constructor() {
    this.services = new Map();
  }
  
  // Register a class
  registerClass(name, Class, dependencies = [], options = {}) {
    this.services.set(name, {
      type: 'class',
      definition: Class,
      dependencies,
      singleton: options.singleton || false,
      instance: null
    });
    return this;
  }
  
  // Register a factory function
  registerFactory(name, factory, dependencies = [], options = {}) {
    this.services.set(name, {
      type: 'factory',
      definition: factory,
      dependencies,
      singleton: options.singleton || false,
      instance: null
    });
    return this;
  }
  
  // Register a value
  registerValue(name, value) {
    this.services.set(name, {
      type: 'value',
      definition: value,
      dependencies: [],
      singleton: true,
      instance: value
    });
    return this;
  }
  
  resolve(name) {
    const service = this.services.get(name);
    
    if (!service) {
      throw new Error(`Service '${name}' not found`);
    }
    
    // Return cached singleton
    if (service.singleton && service.instance !== null) {
      return service.instance;
    }
    
    // Resolve dependencies
    const deps = service.dependencies.map(dep => this.resolve(dep));
    
    let instance;
    
    switch (service.type) {
      case 'class':
        instance = new service.definition(...deps);
        break;
        
      case 'factory':
        instance = service.definition(...deps);
        break;
        
      case 'value':
        instance = service.definition;
        break;
        
      default:
        throw new Error(`Unknown service type: ${service.type}`);
    }
    
    if (service.singleton) {
      service.instance = instance;
    }
    
    return instance;
  }
  
  has(name) {
    return this.services.has(name);
  }
  
  clear() {
    this.services.clear();
  }
}

// Test
console.log('\n=== Factory and Value Registration ===');

const container2 = new AdvancedDIContainer();

// Register config value
container2.registerValue('config', {
  apiUrl: 'https://api.example.com',
  timeout: 5000
});

// Register factory
container2.registerFactory(
  'httpClient',
  (config) => {
    return {
      get: (url) => `GET ${config.apiUrl}${url}`,
      post: (url, data) => `POST ${config.apiUrl}${url} with ${JSON.stringify(data)}`
    };
  },
  ['config'],
  { singleton: true }
);

// Register class
class ApiService {
  constructor(httpClient, config) {
    this.http = httpClient;
    this.config = config;
  }
  
  fetchUsers() {
    return this.http.get('/users');
  }
}

container2.registerClass('apiService', ApiService, ['httpClient', 'config']);

const apiService = container2.resolve('apiService');
console.log(apiService.fetchUsers());
```

### **Approach 3: Auto-Wiring with Decorators**
```javascript
/**
 * Automatic dependency injection using decorators
 */

// Decorator metadata storage
const metadata = new WeakMap();

// Injectable decorator
function Injectable(dependencies = []) {
  return function(target) {
    metadata.set(target, { dependencies });
    return target;
  };
}

// Inject decorator for properties
function Inject(serviceName) {
  return function(target, propertyKey) {
    if (!metadata.has(target.constructor)) {
      metadata.set(target.constructor, { dependencies: [] });
    }
    
    const meta = metadata.get(target.constructor);
    meta.propertyInjections = meta.propertyInjections || [];
    meta.propertyInjections.push({ propertyKey, serviceName });
  };
}

class AutoWiringContainer {
  constructor() {
    this.services = new Map();
    this.instances = new Map();
  }
  
  register(name, Class, options = {}) {
    const meta = metadata.get(Class) || { dependencies: [] };
    
    this.services.set(name, {
      Class,
      dependencies: meta.dependencies || [],
      propertyInjections: meta.propertyInjections || [],
      singleton: options.singleton !== false,
      instance: null
    });
    
    return this;
  }
  
  resolve(name) {
    const service = this.services.get(name);
    
    if (!service) {
      throw new Error(`Service '${name}' not found`);
    }
    
    // Return singleton
    if (service.singleton && service.instance) {
      return service.instance;
    }
    
    // Resolve constructor dependencies
    const deps = service.dependencies.map(dep => this.resolve(dep));
    
    // Create instance
    const instance = new service.Class(...deps);
    
    // Inject properties
    service.propertyInjections.forEach(({ propertyKey, serviceName }) => {
      instance[propertyKey] = this.resolve(serviceName);
    });
    
    if (service.singleton) {
      service.instance = instance;
    }
    
    return instance;
  }
}

// Test
console.log('\n=== Auto-Wiring ===');

@Injectable([])
class Logger {
  log(message) {
    console.log(`[LOG] ${message}`);
  }
}

@Injectable(['logger'])
class DataService {
  constructor(logger) {
    this.logger = logger;
    this.data = [];
  }
  
  add(item) {
    this.logger.log(`Adding item: ${item}`);
    this.data.push(item);
  }
}

const autoContainer = new AutoWiringContainer();
autoContainer.register('logger', Logger);
autoContainer.register('dataService', DataService);

const dataService = autoContainer.resolve('dataService');
dataService.add('test item');
```

### **Approach 4: Scoped Containers**
```javascript
/**
 * Support different scopes: singleton, transient, scoped
 */

const Lifetime = {
  SINGLETON: 'singleton',
  TRANSIENT: 'transient',
  SCOPED: 'scoped'
};

class ScopedContainer {
  constructor(parent = null) {
    this.parent = parent;
    this.services = new Map();
    this.instances = new Map();
    this.scopes = new Map();
  }
  
  register(name, definition, dependencies = [], lifetime = Lifetime.TRANSIENT) {
    this.services.set(name, {
      definition,
      dependencies,
      lifetime
    });
    return this;
  }
  
  resolve(name) {
    // Check scoped instances first
    if (this.instances.has(name)) {
      return this.instances.get(name);
    }
    
    const service = this.services.get(name) || 
                   (this.parent && this.parent.services.get(name));
    
    if (!service) {
      throw new Error(`Service '${name}' not found`);
    }
    
    // Check for singleton in root
    if (service.lifetime === Lifetime.SINGLETON) {
      const root = this.getRoot();
      
      if (root.instances.has(name)) {
        return root.instances.get(name);
      }
      
      const instance = this.createInstance(service);
      root.instances.set(name, instance);
      return instance;
    }
    
    // Scoped - cache in current scope
    if (service.lifetime === Lifetime.SCOPED) {
      const instance = this.createInstance(service);
      this.instances.set(name, instance);
      return instance;
    }
    
    // Transient - always create new
    return this.createInstance(service);
  }
  
  createInstance(service) {
    const deps = service.dependencies.map(dep => this.resolve(dep));
    
    return typeof service.definition === 'function'
      ? new service.definition(...deps)
      : service.definition;
  }
  
  createScope() {
    return new ScopedContainer(this);
  }
  
  getRoot() {
    let root = this;
    while (root.parent) {
      root = root.parent;
    }
    return root;
  }
  
  dispose() {
    this.instances.clear();
  }
}

// Test
console.log('\n=== Scoped Containers ===');

class RequestId {
  constructor() {
    this.id = Math.random().toString(36).substr(2, 9);
  }
}

class RequestLogger {
  constructor(requestId) {
    this.requestId = requestId;
  }
  
  log(message) {
    console.log(`[${this.requestId.id}] ${message}`);
  }
}

const rootContainer = new ScopedContainer();
rootContainer.register('requestId', RequestId, [], Lifetime.SCOPED);
rootContainer.register('logger', RequestLogger, ['requestId'], Lifetime.SCOPED);

// Simulate two requests
const scope1 = rootContainer.createScope();
const logger1 = scope1.resolve('logger');
logger1.log('Request 1 - Action 1');
logger1.log('Request 1 - Action 2');

const scope2 = rootContainer.createScope();
const logger2 = scope2.resolve('logger');
logger2.log('Request 2 - Action 1');
logger2.log('Request 2 - Action 2');

console.log('Different scopes:', logger1.requestId !== logger2.requestId);
```

### **Approach 5: Circular Dependency Detection**
```javascript
/**
 * Detect and handle circular dependencies
 */

class SafeDIContainer {
  constructor() {
    this.services = new Map();
    this.resolving = new Set();
  }
  
  register(name, definition, dependencies = []) {
    this.services.set(name, { definition, dependencies, instance: null });
    return this;
  }
  
  resolve(name) {
    // Check for circular dependency
    if (this.resolving.has(name)) {
      const chain = Array.from(this.resolving).join(' -> ');
      throw new Error(`Circular dependency detected: ${chain} -> ${name}`);
    }
    
    const service = this.services.get(name);
    
    if (!service) {
      throw new Error(`Service '${name}' not found`);
    }
    
    if (service.instance) {
      return service.instance;
    }
    
    // Mark as resolving
    this.resolving.add(name);
    
    try {
      // Resolve dependencies
      const deps = service.dependencies.map(dep => this.resolve(dep));
      
      // Create instance
      const instance = new service.definition(...deps);
      service.instance = instance;
      
      return instance;
    } finally {
      // Remove from resolving
      this.resolving.delete(name);
    }
  }
  
  // Validate all registrations
  validate() {
    const errors = [];
    
    for (const [name, service] of this.services) {
      for (const dep of service.dependencies) {
        if (!this.services.has(dep)) {
          errors.push(`Service '${name}' depends on '${dep}' which is not registered`);
        }
      }
    }
    
    // Try to resolve all to detect circular dependencies
    for (const name of this.services.keys()) {
      try {
        this.resolve(name);
      } catch (error) {
        errors.push(error.message);
      }
    }
    
    return {
      valid: errors.length === 0,
      errors
    };
  }
}

// Test
console.log('\n=== Circular Dependency Detection ===');

class ServiceA {
  constructor(serviceB) {
    this.serviceB = serviceB;
  }
}

class ServiceB {
  constructor(serviceC) {
    this.serviceC = serviceC;
  }
}

class ServiceC {
  constructor() {
    this.name = 'ServiceC';
  }
}

const safeContainer = new SafeDIContainer();
safeContainer.register('serviceA', ServiceA, ['serviceB']);
safeContainer.register('serviceB', ServiceB, ['serviceC']);
safeContainer.register('serviceC', ServiceC, []);

console.log('Valid setup:', safeContainer.validate());

// Try circular dependency
const circularContainer = new SafeDIContainer();
circularContainer.register('serviceA', ServiceA, ['serviceB']);
circularContainer.register('serviceB', ServiceB, ['serviceA']); // Circular!

try {
  circularContainer.resolve('serviceA');
} catch (error) {
  console.log('Caught circular dependency:', error.message);
}
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Express.js-like Middleware
console.log('\n=== Express Middleware ===');

class Application {
  constructor(container) {
    this.container = container;
    this.middlewares = [];
  }
  
  use(middleware) {
    this.middlewares.push(middleware);
  }
  
  handle(req, res) {
    let index = 0;
    
    const next = () => {
      if (index >= this.middlewares.length) return;
      
      const middleware = this.middlewares[index++];
      
      // Resolve dependencies for middleware
      const deps = middleware.dependencies || [];
      const resolvedDeps = deps.map(dep => this.container.resolve(dep));
      
      middleware.handler(req, res, next, ...resolvedDeps);
    };
    
    next();
  }
}

const appContainer = new AdvancedDIContainer();

appContainer.registerValue('logger', {
  log: (msg) => console.log(`[LOGGER] ${msg}`)
});

appContainer.registerValue('auth', {
  verify: (token) => token === 'valid-token'
});

const app = new Application(appContainer);

app.use({
  dependencies: ['logger'],
  handler: (req, res, next, logger) => {
    logger.log(`${req.method} ${req.url}`);
    next();
  }
});

app.use({
  dependencies: ['auth'],
  handler: (req, res, next, auth) => {
    if (!auth.verify(req.headers.token)) {
      res.status = 401;
      res.body = 'Unauthorized';
      return;
    }
    next();
  }
});

const mockReq = { method: 'GET', url: '/api/users', headers: { token: 'valid-token' } };
const mockRes = { status: 200, body: null };

app.handle(mockReq, mockRes);

// 2. Testing with Mocks
console.log('\n=== Testing with Mocks ===');

class TestContainer extends AdvancedDIContainer {
  mock(name, mockValue) {
    this.registerValue(name, mockValue);
    return this;
  }
}

// Production service
class EmailService {
  send(to, message) {
    console.log(`Sending email to ${to}: ${message}`);
    return { sent: true, id: Math.random() };
  }
}

// Service using EmailService
class NotificationService {
  constructor(emailService) {
    this.emailService = emailService;
  }
  
  notify(user, message) {
    return this.emailService.send(user.email, message);
  }
}

// Test setup
const testContainer = new TestContainer();

testContainer.mock('emailService', {
  send: (to, message) => {
    console.log(`[MOCK] Would send email to ${to}`);
    return { sent: true, id: 'mock-id' };
  }
});

testContainer.registerClass('notificationService', NotificationService, ['emailService']);

const notificationService = testContainer.resolve('notificationService');
const result = notificationService.notify({ email: 'user@test.com' }, 'Test message');

console.log('Mock result:', result);

// 3. Plugin System
console.log('\n=== Plugin System ===');

class PluginManager {
  constructor(container) {
    this.container = container;
    this.plugins = new Map();
  }
  
  registerPlugin(name, plugin) {
    this.plugins.set(name, plugin);
    
    // Register plugin's services
    if (plugin.services) {
      for (const [serviceName, service] of Object.entries(plugin.services)) {
        this.container.registerFactory(
          `${name}.${serviceName}`,
          service.factory,
          service.dependencies || []
        );
      }
    }
    
    // Initialize plugin
    if (plugin.initialize) {
      plugin.initialize(this.container);
    }
  }
  
  getPlugin(name) {
    return this.plugins.get(name);
  }
}

const pluginContainer = new AdvancedDIContainer();
const pluginManager = new PluginManager(pluginContainer);

// Example plugin
const analyticsPlugin = {
  name: 'analytics',
  
  services: {
    tracker: {
      factory: () => ({
        track: (event) => console.log(`Tracking: ${event}`)
      })
    }
  },
  
  initialize: (container) => {
    console.log('Analytics plugin initialized');
  }
};

pluginManager.registerPlugin('analytics', analyticsPlugin);

const tracker = pluginContainer.resolve('analytics.tracker');
tracker.track('page_view');

// 4. Configuration Management
console.log('\n=== Configuration Management ===');

class ConfigManager {
  constructor() {
    this.config = new Map();
  }
  
  set(key, value) {
    this.config.set(key, value);
  }
  
  get(key, defaultValue = null) {
    return this.config.has(key) ? this.config.get(key) : defaultValue;
  }
  
  getAll() {
    return Object.fromEntries(this.config);
  }
}

const appContainerWithConfig = new AdvancedDIContainer();

appContainerWithConfig.registerClass('config', ConfigManager, [], { singleton: true });

appContainerWithConfig.registerFactory(
  'database',
  (config) => {
    const dbConfig = {
      host: config.get('DB_HOST', 'localhost'),
      port: config.get('DB_PORT', 5432),
      name: config.get('DB_NAME', 'myapp')
    };
    
    return {
      connect: () => console.log('Connected to:', dbConfig),
      query: (sql) => `Executing: ${sql}`
    };
  },
  ['config'],
  { singleton: true }
);

const config = appContainerWithConfig.resolve('config');
config.set('DB_HOST', 'production-db.example.com');
config.set('DB_NAME', 'production_db');

const database = appContainerWithConfig.resolve('database');
database.connect();

// 5. Microservices Communication
console.log('\n=== Microservices ===');

class ServiceRegistry {
  constructor() {
    this.services = new Map();
  }
  
  register(name, endpoint) {
    this.services.set(name, endpoint);
  }
  
  discover(name) {
    const endpoint = this.services.get(name);
    if (!endpoint) {
      throw new Error(`Service '${name}' not found in registry`);
    }
    return endpoint;
  }
}

class ServiceClient {
  constructor(registry, serviceName) {
    this.registry = registry;
    this.serviceName = serviceName;
  }
  
  call(method, params) {
    const endpoint = this.registry.discover(this.serviceName);
    return `Calling ${endpoint}.${method}(${JSON.stringify(params)})`;
  }
}

const microserviceContainer = new AdvancedDIContainer();

microserviceContainer.registerClass('registry', ServiceRegistry, [], { singleton: true });

microserviceContainer.registerFactory(
  'userServiceClient',
  (registry) => new ServiceClient(registry, 'user-service'),
  ['registry']
);

microserviceContainer.registerFactory(
  'orderServiceClient',
  (registry) => new ServiceClient(registry, 'order-service'),
  ['registry']
);

const registry = microserviceContainer.resolve('registry');
registry.register('user-service', 'http://users-api:3000');
registry.register('order-service', 'http://orders-api:3001');

const userClient = microserviceContainer.resolve('userServiceClient');
const orderClient = microserviceContainer.resolve('orderServiceClient');

console.log(userClient.call('getUser', { id: 123 }));
console.log(orderClient.call('getOrders', { userId: 123 }));
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Dependency Injection Patterns:
┌──────────────────────┬─────────────────────────────────┐
│ Pattern              │ Characteristics                 │
├──────────────────────┼─────────────────────────────────┤
│ Constructor Injection│ Dependencies via constructor    │
│                      │ Most common, explicit deps     │
├──────────────────────┼─────────────────────────────────┤
│ Property Injection   │ Set properties after creation   │
│                      │ Optional dependencies           │
├──────────────────────┼─────────────────────────────────┤
│ Method Injection     │ Pass deps to methods            │
│                      │ Per-operation dependencies      │
└──────────────────────┴─────────────────────────────────┘

Lifetime Management:
┌──────────────┬──────────────────────────────────────┐
│ Lifetime     │ Behavior                             │
├──────────────┼──────────────────────────────────────┤
│ Singleton    │ One instance for entire app          │
│              │ Shared state, cached                 │
├──────────────┼──────────────────────────────────────┤
│ Transient    │ New instance every resolution        │
│              │ No shared state, stateless           │
├──────────────┼──────────────────────────────────────┤
│ Scoped       │ One instance per scope/request       │
│              │ Isolated per operation               │
└──────────────┴──────────────────────────────────────┘

Benefits:
• Decoupling: components don't create dependencies
• Testability: easy to inject mocks/stubs
• Flexibility: swap implementations easily
• Maintainability: centralized dependency management
• Lazy loading: create instances only when needed

Common Features:
• Registration: bind interfaces to implementations
• Resolution: create instances with dependencies
• Lifetime management: singleton, transient, scoped
• Auto-wiring: automatic dependency resolution
• Circular detection: prevent infinite loops
• Validation: check dependency graph

Best Practices:
• Prefer constructor injection (explicit)
• Use interfaces/protocols for abstractions
• Keep dependencies minimal
• Avoid service locator anti-pattern
• Validate container at startup
• Use scopes for request-based dependencies
• Register by interface, resolve by name

Use Cases:
• Large applications with many components
• Testing with mocked dependencies
• Plugin/extension systems
• Microservices communication
• Configuration management
• Middleware pipelines
`);
```

**Interview Tips:**
- Dependency Injection: design pattern where dependencies are provided from outside
- DI Container: manages object creation and lifetime, resolves dependencies
- Three injection types: constructor (common), property (optional), method (per-call)
- Registration: tell container how to create services (class, factory, value)
- Resolution: container creates instance with all dependencies
- Lifetime management: singleton (one instance), transient (always new), scoped (per scope)
- Dependencies list: what each service needs to be created
- Circular dependencies: A needs B, B needs A - must detect and prevent
- Benefits: loose coupling, testability, flexibility, maintainability
- Constructor injection preferred: explicit dependencies visible in signature
- Service locator anti-pattern: don't inject container itself, hide dependencies
- Auto-wiring: automatically resolve dependencies without explicit configuration
- Scopes: useful for web requests (new scope per request)
- Validation: check all dependencies registered before app starts
- Mocking: easy to replace real services with test doubles
- Common use: large apps (Angular, NestJS), testing, plugins, microservices
- Trade-offs: complexity overhead, learning curve, runtime errors if misconfigured
- Alternatives: simple factory pattern, direct instantiation, module pattern
- Follow-ups: async resolution, lazy loading, decorators, hierarchical containers
- Clarify: lifetime requirements? circular detection? validation? TypeScript support?

</details>

119. Implement a template engine (string interpolation)

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Simple String Interpolation**
```javascript
/**
 * Basic template engine with {{ variable }} syntax
 */

function simpleTemplate(template, data) {
  return template.replace(/\{\{(\w+)\}\}/g, (match, key) => {
    return data.hasOwnProperty(key) ? data[key] : match;
  });
}

// Test
console.log('=== Simple Template ===');

const template1 = 'Hello, {{name}}! You are {{age}} years old.';
const data1 = { name: 'John', age: 25 };

console.log(simpleTemplate(template1, data1));
// "Hello, John! You are 25 years old."

const template2 = 'Welcome to {{city}}, {{name}}!';
const data2 = { name: 'Alice', city: 'New York' };

console.log(simpleTemplate(template2, data2));
// "Welcome to New York, Alice!"
```

### **Approach 2: Nested Property Access**
```javascript
/**
 * Support nested properties with dot notation
 */

function nestedTemplate(template, data) {
  return template.replace(/\{\{([\w.]+)\}\}/g, (match, path) => {
    const keys = path.split('.');
    let value = data;
    
    for (const key of keys) {
      if (value && value.hasOwnProperty(key)) {
        value = value[key];
      } else {
        return match; // Keep original if not found
      }
    }
    
    return value !== undefined ? value : match;
  });
}

// Test
console.log('\n=== Nested Properties ===');

const template3 = 'User: {{user.name}}, Email: {{user.email}}, City: {{user.address.city}}';
const data3 = {
  user: {
    name: 'Bob',
    email: 'bob@example.com',
    address: {
      city: 'San Francisco',
      zip: '94102'
    }
  }
};

console.log(nestedTemplate(template3, data3));
```

### **Approach 3: Expressions and Helpers**
```javascript
/**
 * Support expressions and helper functions
 */

class TemplateEngine {
  constructor() {
    this.helpers = new Map();
  }
  
  registerHelper(name, fn) {
    this.helpers.set(name, fn);
  }
  
  compile(template) {
    return (data) => {
      // Handle expressions: {{ expression }}
      return template.replace(/\{\{([^}]+)\}\}/g, (match, expression) => {
        expression = expression.trim();
        
        try {
          // Create function with data and helpers in scope
          const fn = new Function(
            'data',
            'helpers',
            `with(data) { return ${expression}; }`
          );
          
          const result = fn(data, Object.fromEntries(this.helpers));
          return result !== undefined ? result : '';
        } catch (error) {
          console.error(`Error evaluating: ${expression}`, error);
          return match;
        }
      });
    };
  }
  
  render(template, data) {
    const compiled = this.compile(template);
    return compiled(data);
  }
}

// Test
console.log('\n=== Expressions and Helpers ===');

const engine = new TemplateEngine();

// Register helpers
engine.registerHelper('upper', (str) => str.toUpperCase());
engine.registerHelper('lower', (str) => str.toLowerCase());
engine.registerHelper('truncate', (str, len) => 
  str.length > len ? str.substring(0, len) + '...' : str
);

const template4 = `
Name: {{ name }}
Uppercase: {{ helpers.upper(name) }}
Age next year: {{ age + 1 }}
Is adult: {{ age >= 18 ? 'Yes' : 'No' }}
Bio: {{ helpers.truncate(bio, 20) }}
`;

const data4 = {
  name: 'Charlie',
  age: 17,
  bio: 'This is a very long biography that should be truncated'
};

console.log(engine.render(template4, data4));
```

### **Approach 4: Conditionals and Loops**
```javascript
/**
 * Support control structures: if/else, loops
 */

class AdvancedTemplateEngine {
  compile(template) {
    // Convert template to JavaScript code
    let code = 'let output = "";\n';
    let cursor = 0;
    
    // Regex patterns
    const patterns = {
      variable: /\{\{([^}]+)\}\}/g,
      ifStart: /\{%\s*if\s+([^%]+)\s*%\}/g,
      elseIf: /\{%\s*elseif\s+([^%]+)\s*%\}/g,
      else: /\{%\s*else\s*%\}/g,
      ifEnd: /\{%\s*endif\s*%\}/g,
      forStart: /\{%\s*for\s+(\w+)\s+in\s+([^%]+)\s*%\}/g,
      forEnd: /\{%\s*endfor\s*%\}/g
    };
    
    while (cursor < template.length) {
      // Check for if statement
      patterns.ifStart.lastIndex = cursor;
      let match = patterns.ifStart.exec(template);
      
      if (match && match.index === cursor) {
        code += `if (${match[1]}) {\n`;
        cursor = patterns.ifStart.lastIndex;
        continue;
      }
      
      // Check for else
      patterns.else.lastIndex = cursor;
      match = patterns.else.exec(template);
      
      if (match && match.index === cursor) {
        code += `} else {\n`;
        cursor = patterns.else.lastIndex;
        continue;
      }
      
      // Check for endif
      patterns.ifEnd.lastIndex = cursor;
      match = patterns.ifEnd.exec(template);
      
      if (match && match.index === cursor) {
        code += `}\n`;
        cursor = patterns.ifEnd.lastIndex;
        continue;
      }
      
      // Check for for loop
      patterns.forStart.lastIndex = cursor;
      match = patterns.forStart.exec(template);
      
      if (match && match.index === cursor) {
        const [, itemVar, arrayExpr] = match;
        code += `for (const ${itemVar} of ${arrayExpr}) {\n`;
        cursor = patterns.forStart.lastIndex;
        continue;
      }
      
      // Check for endfor
      patterns.forEnd.lastIndex = cursor;
      match = patterns.forEnd.exec(template);
      
      if (match && match.index === cursor) {
        code += `}\n`;
        cursor = patterns.forEnd.lastIndex;
        continue;
      }
      
      // Check for variable
      patterns.variable.lastIndex = cursor;
      match = patterns.variable.exec(template);
      
      if (match && match.index === cursor) {
        code += `output += (${match[1]}) || '';\n`;
        cursor = patterns.variable.lastIndex;
        continue;
      }
      
      // Regular text
      const nextSpecial = template.substring(cursor).search(/\{[{%]/);
      
      if (nextSpecial === -1) {
        // No more special syntax
        const text = template.substring(cursor).replace(/'/g, "\\'");
        code += `output += '${text}';\n`;
        break;
      } else {
        const text = template.substring(cursor, cursor + nextSpecial).replace(/'/g, "\\'");
        code += `output += '${text}';\n`;
        cursor += nextSpecial;
      }
    }
    
    code += 'return output;';
    
    return new Function('data', `with(data) { ${code} }`);
  }
  
  render(template, data) {
    const compiled = this.compile(template);
    return compiled(data);
  }
}

// Test
console.log('\n=== Conditionals and Loops ===');

const advEngine = new AdvancedTemplateEngine();

const template5 = `
<h1>{{title}}</h1>

{% if user.isLoggedIn %}
  <p>Welcome, {{user.name}}!</p>
{% else %}
  <p>Please log in.</p>
{% endif %}

<h2>Items:</h2>
<ul>
{% for item in items %}
  <li>{{item.name}} - ${{item.price}}</li>
{% endfor %}
</ul>

{% if items.length > 0 %}
  <p>Total items: {{items.length}}</p>
{% else %}
  <p>No items available.</p>
{% endif %}
`;

const data5 = {
  title: 'My Store',
  user: {
    isLoggedIn: true,
    name: 'David'
  },
  items: [
    { name: 'Book', price: 19.99 },
    { name: 'Pen', price: 2.99 },
    { name: 'Notebook', price: 5.99 }
  ]
};

console.log(advEngine.render(template5, data5));
```

### **Approach 5: Component-Based Templates**
```javascript
/**
 * Support reusable components and partials
 */

class ComponentTemplateEngine {
  constructor() {
    this.components = new Map();
    this.partials = new Map();
  }
  
  registerComponent(name, template) {
    this.components.set(name, template);
  }
  
  registerPartial(name, template) {
    this.partials.set(name, template);
  }
  
  render(template, data) {
    // Replace partials: {% include "partial-name" %}
    template = template.replace(
      /\{%\s*include\s+"([^"]+)"\s*%\}/g,
      (match, partialName) => {
        const partial = this.partials.get(partialName);
        return partial ? this.render(partial, data) : match;
      }
    );
    
    // Replace components: <ComponentName prop1="value1" />
    template = template.replace(
      /<(\w+)\s*([^/>]*)\s*\/>/g,
      (match, componentName, propsStr) => {
        const component = this.components.get(componentName);
        
        if (!component) return match;
        
        // Parse props
        const props = {};
        const propRegex = /(\w+)="([^"]*)"/g;
        let propMatch;
        
        while ((propMatch = propRegex.exec(propsStr)) !== null) {
          const [, key, value] = propMatch;
          // Evaluate value expression
          try {
            props[key] = new Function('data', `with(data) { return ${value}; }`)(data);
          } catch {
            props[key] = value;
          }
        }
        
        return this.render(component, { ...data, ...props });
      }
    );
    
    // Replace variables
    template = template.replace(/\{\{([^}]+)\}\}/g, (match, expr) => {
      try {
        const result = new Function('data', `with(data) { return ${expr.trim()}; }`)(data);
        return result !== undefined ? result : '';
      } catch {
        return match;
      }
    });
    
    return template;
  }
}

// Test
console.log('\n=== Component-Based ===');

const compEngine = new ComponentTemplateEngine();

// Register partials
compEngine.registerPartial('header', `
<header>
  <h1>{{siteName}}</h1>
  <nav>Home | About | Contact</nav>
</header>
`);

compEngine.registerPartial('footer', `
<footer>
  <p>&copy; {{year}} {{siteName}}</p>
</footer>
`);

// Register components
compEngine.registerComponent('UserCard', `
<div class="user-card">
  <h3>{{name}}</h3>
  <p>{{email}}</p>
  <p>Role: {{role}}</p>
</div>
`);

compEngine.registerComponent('Button', `
<button class="{{className}}">{{text}}</button>
`);

// Main template
const mainTemplate = `
{% include "header" %}

<main>
  <h2>Users</h2>
  <UserCard name="user.name" email="user.email" role="user.role" />
  
  <Button text="'Click Me'" className="'btn-primary'" />
</main>

{% include "footer" %}
`;

const mainData = {
  siteName: 'My Website',
  year: 2024,
  user: {
    name: 'Emma',
    email: 'emma@example.com',
    role: 'Admin'
  }
};

console.log(compEngine.render(mainTemplate, mainData));
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Email Templates
console.log('\n=== Email Templates ===');

class EmailTemplateEngine {
  constructor() {
    this.engine = new TemplateEngine();
    
    // Register email-specific helpers
    this.engine.registerHelper('formatDate', (date) => {
      return new Date(date).toLocaleDateString();
    });
    
    this.engine.registerHelper('currency', (amount) => {
      return `$${amount.toFixed(2)}`;
    });
  }
  
  renderWelcomeEmail(user) {
    const template = `
Hi {{user.name}},

Welcome to our platform! Your account has been created successfully.

Username: {{user.email}}
Registered: {{ helpers.formatDate(user.registeredAt) }}

Thank you for joining us!

Best regards,
The Team
    `;
    
    return this.engine.render(template, { user });
  }
  
  renderOrderConfirmation(order) {
    const template = `
Order Confirmation #{{order.id}}

Thank you for your order, {{order.customerName}}!

Order Date: {{ helpers.formatDate(order.date) }}
Total: {{ helpers.currency(order.total) }}

Items:
{{order.items.map(item => '- ' + item.name + ': ' + helpers.currency(item.price)).join('\\n')}}

We'll send you tracking information once your order ships.
    `;
    
    return this.engine.render(template, { order });
  }
}

const emailEngine = new EmailTemplateEngine();

console.log(emailEngine.renderWelcomeEmail({
  name: 'Frank',
  email: 'frank@example.com',
  registeredAt: '2024-01-15'
}));

console.log('\n');

console.log(emailEngine.renderOrderConfirmation({
  id: 'ORD-12345',
  customerName: 'Grace',
  date: '2024-01-20',
  total: 49.97,
  items: [
    { name: 'Widget A', price: 19.99 },
    { name: 'Widget B', price: 29.98 }
  ]
}));

// 2. Report Generation
console.log('\n=== Report Generation ===');

class ReportGenerator {
  generateSalesReport(data) {
    const engine = new AdvancedTemplateEngine();
    
    const template = `
# Sales Report - {{reportDate}}

## Summary
- Total Sales: ${{totalSales}}
- Total Orders: {{totalOrders}}
- Average Order: ${{(totalSales / totalOrders).toFixed(2)}}

## Top Products
{% for product in topProducts %}
{{product.rank}}. {{product.name}} - {{product.units}} units - ${{product.revenue}}
{% endfor %}

## Sales by Region
{% for region in regions %}
### {{region.name}}
- Orders: {{region.orders}}
- Revenue: ${{region.revenue}}
{% endfor %}
    `;
    
    return engine.render(template, data);
  }
}

const reportGen = new ReportGenerator();

console.log(reportGen.generateSalesReport({
  reportDate: '2024-01-20',
  totalSales: 125000,
  totalOrders: 450,
  topProducts: [
    { rank: 1, name: 'Product A', units: 120, revenue: 24000 },
    { rank: 2, name: 'Product B', units: 95, revenue: 19000 },
    { rank: 3, name: 'Product C', units: 80, revenue: 16000 }
  ],
  regions: [
    { name: 'North', orders: 150, revenue: 45000 },
    { name: 'South', orders: 180, revenue: 54000 },
    { name: 'West', orders: 120, revenue: 26000 }
  ]
}));

// 3. Static Site Generator
console.log('\n=== Static Site Generator ===');

class StaticSiteGenerator {
  constructor() {
    this.engine = new ComponentTemplateEngine();
    this.layouts = new Map();
  }
  
  registerLayout(name, template) {
    this.layouts.set(name, template);
  }
  
  renderPage(content, layout, data) {
    const layoutTemplate = this.layouts.get(layout);
    
    if (!layoutTemplate) {
      throw new Error(`Layout '${layout}' not found`);
    }
    
    return this.engine.render(layoutTemplate, { ...data, content });
  }
}

const siteGen = new StaticSiteGenerator();

siteGen.registerLayout('default', `
<!DOCTYPE html>
<html>
<head>
  <title>{{title}} - {{siteName}}</title>
</head>
<body>
  <header>
    <h1>{{siteName}}</h1>
  </header>
  
  <main>
    {{content}}
  </main>
  
  <footer>
    <p>&copy; {{year}} {{siteName}}</p>
  </footer>
</body>
</html>
`);

const pageContent = `
<h2>{{pageTitle}}</h2>
<p>{{pageDescription}}</p>
`;

const html = siteGen.renderPage(pageContent, 'default', {
  siteName: 'My Blog',
  title: 'Home',
  year: 2024,
  pageTitle: 'Welcome',
  pageDescription: 'This is the home page.'
});

console.log(html);

// 4. Code Generation
console.log('\n=== Code Generation ===');

class CodeGenerator {
  generateClass(spec) {
    const engine = new AdvancedTemplateEngine();
    
    const template = `
class {{className}} {
  constructor({{spec.constructor.params.join(', ')}}) {
    {% for param in spec.constructor.params %}
    this.{{param}} = {{param}};
    {% endfor %}
  }
  
  {% for method in spec.methods %}
  {{method.name}}({{method.params.join(', ')}}) {
    {{method.body}}
  }
  
  {% endfor %}
}
    `;
    
    return engine.render(template, { className: spec.name, spec });
  }
}

const codeGen = new CodeGenerator();

const classSpec = {
  name: 'Person',
  constructor: {
    params: ['name', 'age']
  },
  methods: [
    {
      name: 'greet',
      params: [],
      body: 'return `Hello, I am ${this.name}`;'
    },
    {
      name: 'celebrateBirthday',
      params: [],
      body: 'this.age++;'
    }
  ]
};

console.log(codeGen.generateClass(classSpec));

// 5. Internationalization (i18n)
console.log('\n=== Internationalization ===');

class I18nTemplateEngine {
  constructor() {
    this.translations = new Map();
    this.currentLocale = 'en';
  }
  
  addTranslations(locale, translations) {
    this.translations.set(locale, translations);
  }
  
  setLocale(locale) {
    this.currentLocale = locale;
  }
  
  t(key) {
    const translations = this.translations.get(this.currentLocale) || {};
    return translations[key] || key;
  }
  
  render(template, data) {
    return template.replace(/\{\{t\s+"([^"]+)"\}\}/g, (match, key) => {
      return this.t(key);
    });
  }
}

const i18nEngine = new I18nTemplateEngine();

i18nEngine.addTranslations('en', {
  'greeting': 'Hello',
  'welcome': 'Welcome to our site',
  'goodbye': 'Goodbye'
});

i18nEngine.addTranslations('es', {
  'greeting': 'Hola',
  'welcome': 'Bienvenido a nuestro sitio',
  'goodbye': 'Adiós'
});

const i18nTemplate = `
<h1>{{t "greeting"}}</h1>
<p>{{t "welcome"}}</p>
<footer>{{t "goodbye"}}</footer>
`;

i18nEngine.setLocale('en');
console.log('English:', i18nEngine.render(i18nTemplate, {}));

i18nEngine.setLocale('es');
console.log('\nSpanish:', i18nEngine.render(i18nTemplate, {}));
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Template Engine Features:
┌──────────────────────┬──────────────────────────────┐
│ Feature              │ Description                  │
├──────────────────────┼──────────────────────────────┤
│ Variable Interpolation│ {{variable}} - simple replace │
│ Nested Access        │ {{user.name}} - dot notation │
│ Expressions          │ {{age + 1}} - evaluate JS    │
│ Conditionals         │ {% if %} - control flow      │
│ Loops                │ {% for %} - iteration        │
│ Components           │ <Component /> - reusable     │
│ Helpers              │ {{helper(value)}} - functions│
│ Partials             │ {% include %} - composition  │
└──────────────────────┴──────────────────────────────┘

Compilation Strategies:
┌──────────────────┬────────────────────────────────┐
│ Strategy         │ When to Use                    │
├──────────────────┼────────────────────────────────┤
│ Runtime          │ Simple, dynamic templates      │
│                  │ Regex replacement              │
├──────────────────┼────────────────────────────────┤
│ Precompilation   │ Production, static templates   │
│                  │ Generate JS functions          │
├──────────────────┼────────────────────────────────┤
│ AST-based        │ Complex transformations        │
│                  │ Parse to tree, transform       │
└──────────────────┴────────────────────────────────┘

Common Syntax Patterns:
• Mustache: {{variable}} - simple, safe
• ERB/EJS: <% code %>, <%= output %> - Ruby/Rails style
• Jinja2/Django: {% tag %}, {{ var }} - Python style
• Handlebars: {{#helper}}, {{variable}} - extended Mustache
• JSX: <Component /> - React style

Security Considerations:
• Always escape HTML to prevent XSS
• Use separate delimiters for safe/unsafe content
• Validate/sanitize user-provided templates
• Avoid eval() when possible
• Whitelist allowed functions/helpers

Performance Optimization:
• Precompile templates in production
• Cache compiled functions
• Minimize regex operations
• Use string concatenation or array join
• Lazy load partials/components
• Stream output for large templates

Real-World Uses:
• Email templates (transactional, marketing)
• Report generation (PDF, HTML)
• Static site generators (blogs, docs)
• Code generation (scaffolding)
• i18n (multilingual content)
• Server-side rendering (SSR)
`);
```

**Interview Tips:**
- Template engine: renders dynamic content by replacing placeholders with data
- Core concept: parse template string, replace variables with actual values
- Common syntaxes: {{variable}} (Mustache), <%= %> (ERB), {% %} (Jinja)
- Variable interpolation: simplest form, regex replace {{key}} with data[key]
- Nested access: support dot notation {{user.name}} for object properties
- Expressions: evaluate JavaScript expressions {{age + 1}}, {{condition ? 'yes' : 'no'}}
- Control structures: conditionals {% if %}, loops {% for item in items %}
- Helpers/filters: custom functions {{formatDate(date)}}, {{uppercase(text)}}
- Partials/includes: reusable template fragments {% include "header" %}
- Components: composable UI elements <Button text="Click" />
- Compilation: convert template to function for performance (precompilation)
- Security: escape HTML to prevent XSS, sanitize user input
- Two strategies: runtime (flexible) vs precompiled (fast)
- Use with(data) or Function() to evaluate expressions in context
- Real-world: emails, reports, SSR, static sites, code generation, i18n
- Popular engines: Handlebars, Mustache, EJS, Pug, Nunjucks
- Trade-offs: flexibility vs security, features vs performance
- Optimization: cache compiled templates, minimize regex, stream output
- Follow-ups: HTML escaping, custom syntax, AST parsing, streaming, async
- Clarify: syntax preference? features needed? security requirements? performance?

</details>

120. Build a simple JavaScript module bundler

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Module Resolver**
```javascript
/**
 * Simple module bundler - resolve dependencies
 */

const fs = require('fs');
const path = require('path');

class BasicBundler {
  constructor(entry) {
    this.entry = entry;
    this.modules = new Map();
    this.moduleId = 0;
  }
  
  // Parse module and extract dependencies
  parseModule(filePath) {
    const content = fs.readFileSync(filePath, 'utf-8');
    const id = this.moduleId++;
    
    // Extract require() calls
    const requireRegex = /require\(['"](.+?)['"]\)/g;
    const dependencies = [];
    let match;
    
    while ((match = requireRegex.exec(content)) !== null) {
      dependencies.push(match[1]);
    }
    
    return {
      id,
      filePath,
      content,
      dependencies,
      mapping: {}
    };
  }
  
  // Resolve relative path
  resolvePath(basePath, relativePath) {
    const dir = path.dirname(basePath);
    let resolved = path.resolve(dir, relativePath);
    
    // Add .js extension if missing
    if (!resolved.endsWith('.js')) {
      resolved += '.js';
    }
    
    return resolved;
  }
  
  // Build dependency graph
  buildGraph(entryPath) {
    const queue = [entryPath];
    const entryModule = this.parseModule(entryPath);
    this.modules.set(entryPath, entryModule);
    
    while (queue.length > 0) {
      const currentPath = queue.shift();
      const currentModule = this.modules.get(currentPath);
      
      for (const dependency of currentModule.dependencies) {
        const depPath = this.resolvePath(currentPath, dependency);
        
        // Store mapping for this module
        currentModule.mapping[dependency] = depPath;
        
        // If not already processed, add to queue
        if (!this.modules.has(depPath)) {
          const depModule = this.parseModule(depPath);
          this.modules.set(depPath, depModule);
          queue.push(depPath);
        }
      }
    }
    
    return entryModule;
  }
  
  // Generate bundle code
  bundle() {
    const graph = this.buildGraph(this.entry);
    
    // Convert modules to object
    const modulesCode = Array.from(this.modules.values())
      .map(module => {
        return `
          ${module.id}: {
            factory: function(require, module, exports) {
              ${module.content}
            },
            mapping: ${JSON.stringify(module.mapping)}
          }
        `;
      })
      .join(',');
    
    // Runtime code
    const bundleCode = `
      (function(modules) {
        const moduleCache = {};
        
        function require(id) {
          if (moduleCache[id]) {
            return moduleCache[id].exports;
          }
          
          const module = { exports: {} };
          moduleCache[id] = module;
          
          const moduleInfo = modules[id];
          
          function localRequire(relativePath) {
            const resolvedId = moduleInfo.mapping[relativePath];
            // Find module id by path
            for (const [id, mod] of Object.entries(modules)) {
              if (mod.mapping === undefined) continue;
              const paths = Object.values(mod.mapping);
              if (paths.includes(resolvedId)) {
                const targetMod = Object.values(modules).find(m => 
                  Object.values(m.mapping || {}).includes(resolvedId) ||
                  m.filePath === resolvedId
                );
                if (targetMod) return require(targetMod.id);
              }
            }
            return require(resolvedId);
          }
          
          moduleInfo.factory(localRequire, module, module.exports);
          
          return module.exports;
        }
        
        require(0); // Start with entry module
      })({${modulesCode}});
    `;
    
    return bundleCode;
  }
}

// Simulated file system for demo
console.log('=== Basic Module Bundler ===');
console.log('Creates a dependency graph and bundles modules');
console.log('Entry -> Dependencies -> Bundle');
```

### **Approach 2: AST-Based Parser**
```javascript
/**
 * Parse modules using Abstract Syntax Tree
 */

// Simulated AST parser (in real world, use @babel/parser)
class ASTParser {
  parse(code) {
    // Simple AST representation
    const imports = [];
    const exports = [];
    
    // Parse import statements
    const importRegex = /import\s+(?:{([^}]+)}|\*\s+as\s+(\w+)|(\w+))\s+from\s+['"]([^'"]+)['"]/g;
    let match;
    
    while ((match = importRegex.exec(code)) !== null) {
      const [, named, namespace, defaultImport, source] = match;
      
      if (named) {
        imports.push({
          type: 'named',
          specifiers: named.split(',').map(s => s.trim()),
          source
        });
      } else if (namespace) {
        imports.push({
          type: 'namespace',
          name: namespace,
          source
        });
      } else if (defaultImport) {
        imports.push({
          type: 'default',
          name: defaultImport,
          source
        });
      }
    }
    
    // Parse export statements
    const exportRegex = /export\s+(?:default\s+)?(?:const|let|var|function|class)\s+(\w+)/g;
    
    while ((match = exportRegex.exec(code)) !== null) {
      exports.push(match[1]);
    }
    
    return {
      imports,
      exports,
      code
    };
  }
  
  transform(ast, transformers) {
    let code = ast.code;
    
    transformers.forEach(transformer => {
      code = transformer(code, ast);
    });
    
    return code;
  }
}

class ES6Bundler {
  constructor() {
    this.parser = new ASTParser();
    this.modules = new Map();
  }
  
  addModule(name, code) {
    const ast = this.parser.parse(code);
    this.modules.set(name, ast);
    return this;
  }
  
  bundle() {
    const modulesCode = [];
    
    for (const [name, ast] of this.modules) {
      // Transform imports to require-like calls
      let transformed = this.parser.transform(ast, [
        // Replace imports with internal require
        (code) => {
          return code.replace(
            /import\s+{([^}]+)}\s+from\s+['"]([^'"]+)['"]/g,
            (match, imports, source) => {
              const names = imports.split(',').map(s => s.trim());
              return `const { ${names.join(', ')} } = __require__('${source}');`;
            }
          );
        },
        // Replace default imports
        (code) => {
          return code.replace(
            /import\s+(\w+)\s+from\s+['"]([^'"]+)['"]/g,
            "const $1 = __require__('$2').default;"
          );
        },
        // Replace exports
        (code) => {
          return code.replace(
            /export\s+const\s+(\w+)\s*=/g,
            'const $1 ='
          ) + `\n__exports__.${ast.exports.join(' = ').split(' ')[0]} = ${ast.exports[0]};`;
        }
      ]);
      
      modulesCode.push(`
        '${name}': function(__require__, __exports__) {
          ${transformed}
        }
      `);
    }
    
    return `
      (function() {
        const modules = {${modulesCode.join(',')}};
        const cache = {};
        
        function require(name) {
          if (cache[name]) return cache[name];
          
          const exports = {};
          cache[name] = exports;
          
          modules[name](require, exports);
          
          return exports;
        }
        
        require('main');
      })();
    `;
  }
}

// Test
console.log('\n=== ES6 Module Bundler ===');

const es6Bundler = new ES6Bundler();

es6Bundler.addModule('math', `
  export const add = (a, b) => a + b;
  export const multiply = (a, b) => a * b;
`);

es6Bundler.addModule('main', `
  import { add, multiply } from 'math';
  
  console.log('2 + 3 =', add(2, 3));
  console.log('2 * 3 =', multiply(2, 3));
`);

console.log(es6Bundler.bundle());
```

### **Approach 3: Tree Shaking (Dead Code Elimination)**
```javascript
/**
 * Remove unused exports (tree shaking)
 */

class TreeShakingBundler {
  constructor() {
    this.modules = new Map();
    this.usedExports = new Map();
  }
  
  addModule(name, exports, code) {
    this.modules.set(name, { exports, code });
    this.usedExports.set(name, new Set());
  }
  
  // Analyze which exports are used
  analyzeUsage(entryModule) {
    const queue = [entryModule];
    const visited = new Set();
    
    while (queue.length > 0) {
      const moduleName = queue.shift();
      
      if (visited.has(moduleName)) continue;
      visited.add(moduleName);
      
      const module = this.modules.get(moduleName);
      if (!module) continue;
      
      // Find imports in this module
      const importRegex = /import\s+{([^}]+)}\s+from\s+['"]([^'"]+)['"]/g;
      let match;
      
      while ((match = importRegex.exec(module.code)) !== null) {
        const [, imports, source] = match;
        const importNames = imports.split(',').map(s => s.trim());
        
        // Mark these exports as used
        if (!this.usedExports.has(source)) {
          this.usedExports.set(source, new Set());
        }
        
        importNames.forEach(name => {
          this.usedExports.get(source).add(name);
        });
        
        queue.push(source);
      }
    }
  }
  
  bundle(entry) {
    this.analyzeUsage(entry);
    
    const bundledCode = [];
    
    for (const [moduleName, module] of this.modules) {
      const usedInModule = this.usedExports.get(moduleName);
      
      if (!usedInModule || usedInModule.size === 0) {
        console.log(`Tree shaking: removed unused module '${moduleName}'`);
        continue;
      }
      
      // Filter exports to only include used ones
      const filteredCode = this.filterUnusedExports(
        module.code,
        module.exports,
        usedInModule
      );
      
      bundledCode.push(`// Module: ${moduleName}\n${filteredCode}`);
    }
    
    return bundledCode.join('\n\n');
  }
  
  filterUnusedExports(code, allExports, usedExports) {
    // Remove unused export declarations
    allExports.forEach(exportName => {
      if (!usedExports.has(exportName)) {
        console.log(`Tree shaking: removed unused export '${exportName}'`);
        // In real implementation, use AST to remove the entire declaration
        code = code.replace(
          new RegExp(`export const ${exportName} = [^;]+;`, 'g'),
          `// Removed unused: ${exportName}`
        );
      }
    });
    
    return code;
  }
}

// Test
console.log('\n=== Tree Shaking ===');

const tsBundle = new TreeShakingBundler();

tsBundle.addModule('utils', ['add', 'subtract', 'multiply', 'divide'], `
  export const add = (a, b) => a + b;
  export const subtract = (a, b) => a - b;
  export const multiply = (a, b) => a * b;
  export const divide = (a, b) => a / b;
`);

tsBundle.addModule('main', [], `
  import { add, multiply } from 'utils';
  
  console.log(add(5, 3));
  console.log(multiply(5, 3));
`);

console.log(tsBundle.bundle('main'));
```

### **Approach 4: Code Splitting**
```javascript
/**
 * Split code into multiple chunks for lazy loading
 */

class CodeSplittingBundler {
  constructor() {
    this.modules = new Map();
    this.chunks = new Map();
    this.chunkId = 0;
  }
  
  addModule(name, code, chunk = 'main') {
    this.modules.set(name, { code, chunk });
    
    if (!this.chunks.has(chunk)) {
      this.chunks.set(chunk, []);
    }
    
    this.chunks.get(chunk).push(name);
  }
  
  // Detect dynamic imports
  analyzeDynamicImports(code) {
    const dynamicImports = [];
    const importRegex = /import\(['"]([^'"]+)['"]\)/g;
    let match;
    
    while ((match = importRegex.exec(code)) !== null) {
      dynamicImports.push(match[1]);
    }
    
    return dynamicImports;
  }
  
  // Generate chunk
  generateChunk(chunkName, modules) {
    const moduleCode = modules
      .map(moduleName => {
        const module = this.modules.get(moduleName);
        return `
          '${moduleName}': function(require, exports) {
            ${module.code}
          }
        `;
      })
      .join(',');
    
    return `
      // Chunk: ${chunkName}
      (function(chunkModules) {
        window.__registerChunk__('${chunkName}', chunkModules);
      })({${moduleCode}});
    `;
  }
  
  // Generate runtime
  generateRuntime() {
    return `
      (function() {
        const chunks = {};
        const moduleCache = {};
        const chunkCache = {};
        
        window.__registerChunk__ = function(chunkName, modules) {
          chunks[chunkName] = modules;
          
          // Execute pending imports
          if (chunkCache[chunkName]) {
            chunkCache[chunkName].forEach(resolve => resolve());
            delete chunkCache[chunkName];
          }
        };
        
        function require(moduleName) {
          if (moduleCache[moduleName]) {
            return moduleCache[moduleName];
          }
          
          const exports = {};
          moduleCache[moduleName] = exports;
          
          // Find module in chunks
          for (const chunkModules of Object.values(chunks)) {
            if (chunkModules[moduleName]) {
              chunkModules[moduleName](require, exports);
              return exports;
            }
          }
          
          throw new Error(\`Module '\${moduleName}' not found\`);
        }
        
        window.__import__ = function(chunkName) {
          return new Promise((resolve, reject) => {
            if (chunks[chunkName]) {
              resolve();
              return;
            }
            
            // Register pending import
            if (!chunkCache[chunkName]) {
              chunkCache[chunkName] = [];
            }
            chunkCache[chunkName].push(resolve);
            
            // Load chunk script
            const script = document.createElement('script');
            script.src = \`/chunks/\${chunkName}.js\`;
            script.onload = () => resolve();
            script.onerror = () => reject(new Error(\`Failed to load chunk '\${chunkName}'\`));
            document.head.appendChild(script);
          });
        };
        
        window.__require__ = require;
      })();
    `;
  }
  
  bundle() {
    const output = {
      runtime: this.generateRuntime(),
      chunks: {}
    };
    
    for (const [chunkName, modules] of this.chunks) {
      output.chunks[chunkName] = this.generateChunk(chunkName, modules);
    }
    
    return output;
  }
}

// Test
console.log('\n=== Code Splitting ===');

const csBundler = new CodeSplittingBundler();

csBundler.addModule('main', `
  console.log('Main module loaded');
  
  document.getElementById('loadBtn').addEventListener('click', async () => {
    await window.__import__('feature');
    const feature = window.__require__('feature');
    feature.init();
  });
`, 'main');

csBundler.addModule('feature', `
  export function init() {
    console.log('Feature loaded dynamically!');
  }
`, 'feature');

const result = csBundler.bundle();

console.log('Runtime:', result.runtime);
console.log('\nMain chunk:', result.chunks.main);
console.log('\nFeature chunk:', result.chunks.feature);
```

### **Approach 5: Plugin System**
```javascript
/**
 * Extensible bundler with plugin support
 */

class PluginBundler {
  constructor() {
    this.plugins = [];
    this.hooks = {
      beforeBundle: [],
      afterParse: [],
      beforeEmit: [],
      afterBundle: []
    };
  }
  
  use(plugin) {
    this.plugins.push(plugin);
    
    // Register plugin hooks
    if (plugin.apply) {
      plugin.apply(this);
    }
    
    return this;
  }
  
  tap(hookName, callback) {
    if (this.hooks[hookName]) {
      this.hooks[hookName].push(callback);
    }
  }
  
  async callHook(hookName, context) {
    const hooks = this.hooks[hookName] || [];
    
    for (const hook of hooks) {
      context = await hook(context) || context;
    }
    
    return context;
  }
  
  async bundle(entry, modules) {
    let context = {
      entry,
      modules,
      output: ''
    };
    
    // Call beforeBundle hooks
    context = await this.callHook('beforeBundle', context);
    
    // Parse modules
    const parsedModules = new Map();
    
    for (const [name, code] of Object.entries(modules)) {
      let moduleContext = { name, code };
      moduleContext = await this.callHook('afterParse', moduleContext);
      parsedModules.set(name, moduleContext.code);
    }
    
    context.modules = parsedModules;
    
    // Generate output
    let output = this.generateBundle(context);
    context.output = output;
    
    // Call beforeEmit hooks
    context = await this.callHook('beforeEmit', context);
    
    // Call afterBundle hooks
    context = await this.callHook('afterBundle', context);
    
    return context.output;
  }
  
  generateBundle(context) {
    const moduleCode = Array.from(context.modules.entries())
      .map(([name, code]) => `'${name}': function() { ${code} }`)
      .join(',');
    
    return `(function() {
      const modules = {${moduleCode}};
      modules['${context.entry}']();
    })();`;
  }
}

// Example plugins
const MinifyPlugin = {
  apply(bundler) {
    bundler.tap('beforeEmit', (context) => {
      console.log('[MinifyPlugin] Minifying output...');
      
      // Simple minification: remove extra whitespace
      context.output = context.output
        .replace(/\s+/g, ' ')
        .replace(/\n/g, '');
      
      return context;
    });
  }
};

const AnalyzerPlugin = {
  apply(bundler) {
    bundler.tap('afterBundle', (context) => {
      console.log('[AnalyzerPlugin] Bundle analysis:');
      console.log(`  Entry: ${context.entry}`);
      console.log(`  Modules: ${context.modules.size}`);
      console.log(`  Output size: ${context.output.length} bytes`);
      
      return context;
    });
  }
};

const TransformPlugin = {
  apply(bundler) {
    bundler.tap('afterParse', (moduleContext) => {
      console.log(`[TransformPlugin] Transforming ${moduleContext.name}...`);
      
      // Example: transform console.log to custom logger
      moduleContext.code = moduleContext.code.replace(
        /console\.log/g,
        'logger.info'
      );
      
      return moduleContext;
    });
  }
};

// Test
console.log('\n=== Plugin System ===');

const pluginBundler = new PluginBundler();

pluginBundler
  .use(TransformPlugin)
  .use(MinifyPlugin)
  .use(AnalyzerPlugin);

pluginBundler.bundle('main', {
  main: `
    console.log('Hello from main');
    console.log('Bundle created!');
  `
}).then(output => {
  console.log('\nFinal output:', output);
});
```

### **Real-World Use Cases**
```javascript
/**
 * Practical bundler applications
 */

// 1. Development Server with HMR (Hot Module Replacement)
console.log('\n=== Development Server ===');

class DevServer {
  constructor(bundler) {
    this.bundler = bundler;
    this.watchers = new Map();
    this.clients = new Set();
  }
  
  watch(files) {
    files.forEach(file => {
      console.log(`Watching: ${file}`);
      
      // Simulate file watcher
      this.watchers.set(file, {
        onChange: () => this.onFileChange(file)
      });
    });
  }
  
  onFileChange(file) {
    console.log(`File changed: ${file}`);
    
    // Rebuild
    const newBundle = this.bundler.bundle();
    
    // Notify clients
    this.notifyClients({
      type: 'update',
      file,
      bundle: newBundle
    });
  }
  
  notifyClients(message) {
    console.log(`Notifying ${this.clients.size} client(s):`, message.type);
    
    this.clients.forEach(client => {
      // In real implementation, use WebSocket
      console.log(`  -> Client received update for ${message.file}`);
    });
  }
  
  addClient(client) {
    this.clients.add(client);
  }
}

const devBundler = new PluginBundler();
const devServer = new DevServer(devBundler);

devServer.watch(['src/main.js', 'src/utils.js']);
devServer.addClient({ id: 'client-1' });

// Simulate file change
setTimeout(() => {
  const watcher = devServer.watchers.get('src/main.js');
  watcher.onChange();
}, 100);

// 2. Asset Pipeline (Images, CSS)
console.log('\n=== Asset Pipeline ===');

class AssetBundler {
  constructor() {
    this.assets = new Map();
    this.loaders = new Map();
  }
  
  registerLoader(extension, loader) {
    this.loaders.set(extension, loader);
  }
  
  async processAsset(filePath, content) {
    const ext = filePath.split('.').pop();
    const loader = this.loaders.get(ext);
    
    if (!loader) {
      throw new Error(`No loader for .${ext} files`);
    }
    
    return await loader(content);
  }
  
  async bundle(assets) {
    const processed = new Map();
    
    for (const [path, content] of Object.entries(assets)) {
      const result = await this.processAsset(path, content);
      processed.set(path, result);
    }
    
    return this.generateBundle(processed);
  }
  
  generateBundle(assets) {
    const assetMap = {};
    
    for (const [path, content] of assets) {
      assetMap[path] = content;
    }
    
    return `
      window.__assets__ = ${JSON.stringify(assetMap)};
      
      function loadAsset(path) {
        return window.__assets__[path];
      }
    `;
  }
}

const assetBundler = new AssetBundler();

// Register loaders
assetBundler.registerLoader('css', (content) => {
  console.log('Processing CSS...');
  return {
    type: 'css',
    content: content.replace(/\s+/g, ' ') // Minify
  };
});

assetBundler.registerLoader('png', (content) => {
  console.log('Processing image...');
  return {
    type: 'image',
    url: `data:image/png;base64,${Buffer.from(content).toString('base64')}`
  };
});

assetBundler.bundle({
  'styles.css': 'body { margin: 0; padding: 0; }',
  'logo.png': 'fake-image-data'
}).then(bundle => {
  console.log('Asset bundle created');
});

// 3. Multi-Target Bundler (Browser, Node, Worker)
console.log('\n=== Multi-Target Bundler ===');

class MultiTargetBundler {
  constructor() {
    this.targets = {
      browser: {
        globals: ['window', 'document'],
        wrapper: (code) => `(function(window, document) { ${code} })(window, document);`
      },
      node: {
        globals: ['process', 'require'],
        wrapper: (code) => `(function(process, require) { ${code} })(process, require);`
      },
      worker: {
        globals: ['self', 'postMessage'],
        wrapper: (code) => `(function(self, postMessage) { ${code} })(self, postMessage);`
      }
    };
  }
  
  bundle(code, target) {
    const config = this.targets[target];
    
    if (!config) {
      throw new Error(`Unknown target: ${target}`);
    }
    
    console.log(`Building for ${target}...`);
    console.log(`Available globals: ${config.globals.join(', ')}`);
    
    return config.wrapper(code);
  }
}

const multiTargetBundler = new MultiTargetBundler();

const sourceCode = `
  console.log('Universal module');
`;

console.log('Browser bundle:', multiTargetBundler.bundle(sourceCode, 'browser'));
console.log('Node bundle:', multiTargetBundler.bundle(sourceCode, 'node'));
console.log('Worker bundle:', multiTargetBundler.bundle(sourceCode, 'worker'));

// 4. Dependency Graph Visualization
console.log('\n=== Dependency Graph ===');

class DependencyAnalyzer {
  constructor() {
    this.graph = new Map();
  }
  
  addModule(name, dependencies) {
    this.graph.set(name, dependencies);
  }
  
  visualize() {
    console.log('Dependency Graph:');
    console.log('─────────────────');
    
    for (const [module, deps] of this.graph) {
      console.log(`${module}`);
      
      deps.forEach((dep, index) => {
        const isLast = index === deps.length - 1;
        const prefix = isLast ? '└─' : '├─';
        console.log(`  ${prefix} ${dep}`);
      });
      
      console.log('');
    }
  }
  
  findCircular() {
    const visited = new Set();
    const recursionStack = new Set();
    const cycles = [];
    
    const dfs = (module, path = []) => {
      if (recursionStack.has(module)) {
        const cycleStart = path.indexOf(module);
        cycles.push([...path.slice(cycleStart), module]);
        return;
      }
      
      if (visited.has(module)) return;
      
      visited.add(module);
      recursionStack.add(module);
      path.push(module);
      
      const deps = this.graph.get(module) || [];
      
      for (const dep of deps) {
        dfs(dep, [...path]);
      }
      
      recursionStack.delete(module);
    };
    
    for (const module of this.graph.keys()) {
      dfs(module);
    }
    
    return cycles;
  }
}

const analyzer = new DependencyAnalyzer();

analyzer.addModule('main', ['utils', 'api']);
analyzer.addModule('utils', ['helpers']);
analyzer.addModule('api', ['http', 'utils']);
analyzer.addModule('http', []);
analyzer.addModule('helpers', []);

analyzer.visualize();

const cycles = analyzer.findCircular();
if (cycles.length > 0) {
  console.log('Circular dependencies found:');
  cycles.forEach(cycle => {
    console.log(`  ${cycle.join(' -> ')}`);
  });
} else {
  console.log('No circular dependencies detected');
}

// 5. Build Optimization Pipeline
console.log('\n=== Build Optimization ===');

class OptimizationPipeline {
  constructor() {
    this.optimizers = [];
  }
  
  addOptimizer(name, fn) {
    this.optimizers.push({ name, fn });
    return this;
  }
  
  async optimize(code) {
    let optimized = code;
    const stats = {
      original: code.length,
      steps: []
    };
    
    for (const optimizer of this.optimizers) {
      const before = optimized.length;
      optimized = await optimizer.fn(optimized);
      const after = optimized.length;
      
      stats.steps.push({
        name: optimizer.name,
        before,
        after,
        saved: before - after,
        percentage: ((before - after) / before * 100).toFixed(2)
      });
    }
    
    stats.final = optimized.length;
    stats.totalSaved = stats.original - stats.final;
    stats.totalPercentage = ((stats.totalSaved / stats.original) * 100).toFixed(2);
    
    return { code: optimized, stats };
  }
  
  printStats(stats) {
    console.log('\nOptimization Report:');
    console.log(`Original size: ${stats.original} bytes`);
    console.log('');
    
    stats.steps.forEach(step => {
      console.log(`${step.name}:`);
      console.log(`  Before: ${step.before} bytes`);
      console.log(`  After:  ${step.after} bytes`);
      console.log(`  Saved:  ${step.saved} bytes (${step.percentage}%)`);
      console.log('');
    });
    
    console.log(`Final size: ${stats.final} bytes`);
    console.log(`Total saved: ${stats.totalSaved} bytes (${stats.totalPercentage}%)`);
  }
}

const pipeline = new OptimizationPipeline();

pipeline
  .addOptimizer('Remove Comments', (code) => {
    return code.replace(/\/\*[\s\S]*?\*\//g, '').replace(/\/\/.*/g, '');
  })
  .addOptimizer('Remove Whitespace', (code) => {
    return code.replace(/\s+/g, ' ').trim();
  })
  .addOptimizer('Shorten Variables', (code) => {
    // Simple example: just demonstrate the concept
    return code
      .replace(/veryLongVariableName/g, 'a')
      .replace(/anotherLongName/g, 'b');
  });

const sampleCode = `
  // This is a comment
  const veryLongVariableName = 42;
  const anotherLongName = 'hello';
  
  /* Multi-line
     comment */
  function test() {
    console.log(veryLongVariableName, anotherLongName);
  }
`;

pipeline.optimize(sampleCode).then(result => {
  pipeline.printStats(result.stats);
  console.log('\nOptimized code:', result.code);
});
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Module Bundler Concepts:
┌──────────────────────┬────────────────────────────────┐
│ Concept              │ Description                    │
├──────────────────────┼────────────────────────────────┤
│ Entry Point          │ Starting module for bundling   │
│ Dependency Graph     │ Tree of module dependencies    │
│ Module Resolution    │ Find modules by path/name      │
│ Parsing              │ Extract imports/exports (AST)  │
│ Transformation       │ Convert module formats         │
│ Code Generation      │ Produce final bundle           │
│ Tree Shaking         │ Remove unused code             │
│ Code Splitting       │ Divide into multiple chunks    │
│ Hot Module Replace   │ Update modules without reload  │
└──────────────────────┴────────────────────────────────┘

Module Formats:
┌──────────────┬────────────────────────────────────┐
│ Format       │ Syntax                             │
├──────────────┼────────────────────────────────────┤
│ CommonJS     │ require() / module.exports         │
│              │ Node.js default                    │
├──────────────┼────────────────────────────────────┤
│ ES Modules   │ import/export                      │
│              │ Browser & modern Node              │
├──────────────┼────────────────────────────────────┤
│ AMD          │ define() / require()               │
│              │ RequireJS, legacy browser          │
├──────────────┼────────────────────────────────────┤
│ UMD          │ Universal (works everywhere)       │
│              │ Combines CJS, AMD, globals         │
└──────────────┴────────────────────────────────────┘

Build Phases:
1. Parse: Read source files, build AST
2. Resolve: Find all dependencies
3. Transform: Convert syntax (Babel, TypeScript)
4. Optimize: Minify, tree shake, dedupe
5. Emit: Write output files

Optimization Techniques:
• Tree shaking: remove unused exports
• Dead code elimination: remove unreachable code
• Minification: remove whitespace, shorten names
• Code splitting: lazy load chunks
• Scope hoisting: flatten module scopes
• Deduplication: merge identical modules
• Compression: gzip/brotli output

Popular Bundlers:
• Webpack: full-featured, plugin ecosystem
• Rollup: ES modules, excellent tree shaking
• Parcel: zero-config, fast builds
• esbuild: extremely fast, Go-based
• Vite: fast dev server, Rollup for prod

Performance Factors:
• Dependency resolution: O(n) modules
• AST parsing: most expensive phase
• Transformation: depends on plugins
• Bundle size: affects load time
• Build time: dev experience

Best Practices:
• Use code splitting for large apps
• Enable tree shaking (ES modules)
• Minimize dependencies
• Use production builds
• Enable caching
• Lazy load routes/features
• Analyze bundle size regularly
• Use source maps for debugging

Real-World Features:
• Development server with HMR
• Asset pipeline (CSS, images)
• Multiple output targets
• Plugin system for extensibility
• Source maps for debugging
• Watch mode for development
• Optimization pipeline
• Dependency graph analysis
`);
```

**Interview Tips:**
- Module bundler: combines multiple files into one or more bundles for deployment
- Core process: parse modules → resolve dependencies → transform → bundle
- Entry point: starting module, builds dependency graph from there
- Dependency graph: tree structure showing which modules import which
- Module resolution: convert import paths to actual file paths
- AST (Abstract Syntax Tree): parsed representation of code, enables transformations
- CommonJS vs ES modules: require/exports vs import/export
- Tree shaking: remove unused exports, requires static analysis (ES modules)
- Code splitting: divide bundle into chunks loaded on-demand
- Lazy loading: load chunks when needed (reduces initial load time)
- HMR (Hot Module Replacement): update modules without full page reload
- Scope hoisting: flatten module wrappers, reduces overhead
- Minification: remove whitespace, shorten names, optimize code
- Source maps: map bundled code back to original for debugging
- Plugin system: extensibility (loaders, transforms, optimizations)
- Webpack: most popular, full-featured, complex config
- Rollup: excellent tree shaking, ES module focus
- Vite/esbuild: modern, extremely fast builds
- Build phases: parse → resolve → transform → optimize → emit
- Performance: AST parsing is expensive, caching helps
- Real-world: dev server, asset pipeline, multi-target, optimization
- Trade-offs: build time vs bundle size vs features
- Follow-ups: circular deps, async imports, chunking strategy, caching
- Clarify: target environment? module format? optimization level? plugin needs?

</details>

121. Implement a custom pipe for filtering data

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Angular Pipe**
```typescript
/**
 * Simple filter pipe for Angular
 */

import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'filter'
})
export class FilterPipe implements PipeTransform {
  transform(items: any[], searchTerm: string, property?: string): any[] {
    if (!items || !searchTerm) {
      return items;
    }
    
    searchTerm = searchTerm.toLowerCase();
    
    return items.filter(item => {
      if (property) {
        // Filter by specific property
        const value = this.getNestedProperty(item, property);
        return value && value.toString().toLowerCase().includes(searchTerm);
      } else {
        // Filter by any property
        return Object.values(item).some(value => 
          value && value.toString().toLowerCase().includes(searchTerm)
        );
      }
    });
  }
  
  private getNestedProperty(obj: any, path: string): any {
    return path.split('.').reduce((current, prop) => 
      current ? current[prop] : null, obj
    );
  }
}

// Usage in component:
/*
@Component({
  selector: 'app-users',
  template: `
    <input [(ngModel)]="searchTerm" placeholder="Search...">
    
    <div *ngFor="let user of users | filter:searchTerm:'name'">
      {{ user.name }} - {{ user.email }}
    </div>
  `
})
export class UsersComponent {
  searchTerm = '';
  users = [
    { name: 'Alice', email: 'alice@example.com', age: 25 },
    { name: 'Bob', email: 'bob@example.com', age: 30 },
    { name: 'Charlie', email: 'charlie@example.com', age: 35 }
  ];
}
*/

console.log('=== Basic Angular Filter Pipe ===');
console.log('Filters array by search term');
console.log('Usage: items | filter:searchTerm:property');
```

### **Approach 2: Multi-Criteria Filter Pipe**
```typescript
/**
 * Advanced filter with multiple criteria
 */

import { Pipe, PipeTransform } from '@angular/core';

interface FilterCriteria {
  property: string;
  value: any;
  operator?: 'equals' | 'contains' | 'startsWith' | 'endsWith' | 'gt' | 'lt' | 'gte' | 'lte';
}

@Pipe({
  name: 'advancedFilter'
})
export class AdvancedFilterPipe implements PipeTransform {
  transform(items: any[], criteria: FilterCriteria | FilterCriteria[]): any[] {
    if (!items || !criteria) {
      return items;
    }
    
    const criteriaArray = Array.isArray(criteria) ? criteria : [criteria];
    
    return items.filter(item => {
      return criteriaArray.every(criterion => 
        this.matchesCriterion(item, criterion)
      );
    });
  }
  
  private matchesCriterion(item: any, criterion: FilterCriteria): boolean {
    const value = this.getNestedProperty(item, criterion.property);
    const filterValue = criterion.value;
    const operator = criterion.operator || 'equals';
    
    if (value === null || value === undefined) {
      return false;
    }
    
    switch (operator) {
      case 'equals':
        return value === filterValue;
        
      case 'contains':
        return value.toString().toLowerCase()
          .includes(filterValue.toString().toLowerCase());
        
      case 'startsWith':
        return value.toString().toLowerCase()
          .startsWith(filterValue.toString().toLowerCase());
        
      case 'endsWith':
        return value.toString().toLowerCase()
          .endsWith(filterValue.toString().toLowerCase());
        
      case 'gt':
        return value > filterValue;
        
      case 'lt':
        return value < filterValue;
        
      case 'gte':
        return value >= filterValue;
        
      case 'lte':
        return value <= filterValue;
        
      default:
        return false;
    }
  }
  
  private getNestedProperty(obj: any, path: string): any {
    return path.split('.').reduce((current, prop) => 
      current ? current[prop] : null, obj
    );
  }
}

// Usage:
/*
@Component({
  template: `
    <!-- Single criterion -->
    <div *ngFor="let user of users | advancedFilter:{property: 'age', value: 30, operator: 'gt'}">
      {{ user.name }} ({{ user.age }})
    </div>
    
    <!-- Multiple criteria -->
    <div *ngFor="let user of users | advancedFilter:[
      {property: 'age', value: 25, operator: 'gte'},
      {property: 'name', value: 'a', operator: 'contains'}
    ]">
      {{ user.name }}
    </div>
  `
})
*/

console.log('\n=== Advanced Filter Pipe ===');
console.log('Supports multiple operators: equals, contains, gt, lt, etc.');
console.log('Can chain multiple criteria');
```

### **Approach 3: Pure vs Impure Pipes**
```typescript
/**
 * Demonstrate pure vs impure pipes
 */

// Pure pipe (default) - only runs when input reference changes
@Pipe({
  name: 'pureFilter',
  pure: true // Default
})
export class PureFilterPipe implements PipeTransform {
  transform(items: any[], searchTerm: string): any[] {
    console.log('PureFilter transform called');
    
    if (!items || !searchTerm) {
      return items;
    }
    
    return items.filter(item => 
      JSON.stringify(item).toLowerCase().includes(searchTerm.toLowerCase())
    );
  }
}

// Impure pipe - runs on every change detection cycle
@Pipe({
  name: 'impureFilter',
  pure: false
})
export class ImpureFilterPipe implements PipeTransform {
  transform(items: any[], searchTerm: string): any[] {
    console.log('ImpureFilter transform called');
    
    if (!items || !searchTerm) {
      return items;
    }
    
    return items.filter(item => 
      JSON.stringify(item).toLowerCase().includes(searchTerm.toLowerCase())
    );
  }
}

// Usage comparison:
/*
@Component({
  template: `
    <input [(ngModel)]="searchTerm">
    
    <!-- Pure pipe: only re-runs when items reference or searchTerm changes -->
    <div *ngFor="let item of items | pureFilter:searchTerm">
      {{ item.name }}
    </div>
    
    <!-- Impure pipe: re-runs on every change detection -->
    <div *ngFor="let item of items | impureFilter:searchTerm">
      {{ item.name }}
    </div>
    
    <button (click)="addItem()">Add Item</button>
  `
})
export class PipeComparisonComponent {
  searchTerm = '';
  items = [{ name: 'Item 1' }];
  
  addItem() {
    // Pure pipe won't detect this change (same reference)
    this.items.push({ name: 'Item ' + (this.items.length + 1) });
    
    // To trigger pure pipe, create new array:
    // this.items = [...this.items, { name: 'Item ' + (this.items.length + 1) }];
  }
}
*/

console.log('\n=== Pure vs Impure Pipes ===');
console.log('Pure: Only runs on reference change (performance)');
console.log('Impure: Runs every change detection (flexible but slower)');
```

### **Approach 4: Memoized Filter Pipe**
```typescript
/**
 * Cache results for better performance
 */

import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'memoizedFilter'
})
export class MemoizedFilterPipe implements PipeTransform {
  private cache = new Map<string, any[]>();
  
  transform(items: any[], searchTerm: string, property?: string): any[] {
    if (!items || !searchTerm) {
      return items;
    }
    
    // Create cache key
    const cacheKey = this.getCacheKey(items, searchTerm, property);
    
    // Return cached result if available
    if (this.cache.has(cacheKey)) {
      console.log('Returning cached result');
      return this.cache.get(cacheKey)!;
    }
    
    console.log('Computing new result');
    
    // Compute result
    searchTerm = searchTerm.toLowerCase();
    
    const result = items.filter(item => {
      if (property) {
        const value = item[property];
        return value && value.toString().toLowerCase().includes(searchTerm);
      } else {
        return Object.values(item).some(value => 
          value && value.toString().toLowerCase().includes(searchTerm)
        );
      }
    });
    
    // Cache result (limit cache size)
    if (this.cache.size > 100) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    this.cache.set(cacheKey, result);
    
    return result;
  }
  
  private getCacheKey(items: any[], searchTerm: string, property?: string): string {
    // Simple hash based on array length, search term, and property
    return `${items.length}-${searchTerm}-${property || 'all'}`;
  }
}

console.log('\n=== Memoized Filter Pipe ===');
console.log('Caches results to avoid recomputation');
console.log('Good for expensive filters on large datasets');
```

### **Approach 5: Async Filter Pipe**
```typescript
/**
 * Filter with async operations (API search)
 */

import { Pipe, PipeTransform } from '@angular/core';
import { Observable, of, BehaviorSubject } from 'rxjs';
import { debounceTime, distinctUntilChanged, switchMap, catchError } from 'rxjs/operators';

@Pipe({
  name: 'asyncFilter'
})
export class AsyncFilterPipe implements PipeTransform {
  private searchSubject = new BehaviorSubject<string>('');
  
  transform(items: any[], searchTerm: string): Observable<any[]> {
    if (!searchTerm) {
      return of(items);
    }
    
    this.searchSubject.next(searchTerm);
    
    return this.searchSubject.pipe(
      debounceTime(300), // Wait 300ms after last keystroke
      distinctUntilChanged(), // Only if changed
      switchMap(term => this.performSearch(items, term)),
      catchError(error => {
        console.error('Search error:', error);
        return of(items);
      })
    );
  }
  
  private performSearch(items: any[], searchTerm: string): Observable<any[]> {
    // Simulate async operation (e.g., API call)
    return new Observable(observer => {
      setTimeout(() => {
        const filtered = items.filter(item => 
          JSON.stringify(item).toLowerCase().includes(searchTerm.toLowerCase())
        );
        observer.next(filtered);
        observer.complete();
      }, 100);
    });
  }
}

// Usage with async pipe:
/*
@Component({
  template: `
    <input [(ngModel)]="searchTerm">
    
    <div *ngFor="let item of (items | asyncFilter:searchTerm | async)">
      {{ item.name }}
    </div>
  `
})
*/

console.log('\n=== Async Filter Pipe ===');
console.log('Supports debouncing and async operations');
console.log('Good for API-based searches');
```

### **Real-World Use Cases**
```typescript
/**
 * Practical pipe implementations
 */

// 1. Table Filter Pipe
console.log('\n=== Table Filter Pipe ===');

@Pipe({
  name: 'tableFilter'
})
export class TableFilterPipe implements PipeTransform {
  transform(
    items: any[], 
    filters: { [key: string]: any },
    globalSearch?: string
  ): any[] {
    if (!items) return [];
    
    let filtered = items;
    
    // Apply column-specific filters
    if (filters) {
      filtered = filtered.filter(item => {
        return Object.keys(filters).every(key => {
          const filterValue = filters[key];
          
          if (!filterValue) return true;
          
          const itemValue = item[key];
          
          if (itemValue === null || itemValue === undefined) {
            return false;
          }
          
          // Handle different types
          if (typeof filterValue === 'boolean') {
            return itemValue === filterValue;
          }
          
          if (typeof filterValue === 'number') {
            return itemValue === filterValue;
          }
          
          if (Array.isArray(filterValue)) {
            return filterValue.includes(itemValue);
          }
          
          return itemValue.toString().toLowerCase()
            .includes(filterValue.toString().toLowerCase());
        });
      });
    }
    
    // Apply global search
    if (globalSearch) {
      const search = globalSearch.toLowerCase();
      filtered = filtered.filter(item => {
        return Object.values(item).some(value => 
          value && value.toString().toLowerCase().includes(search)
        );
      });
    }
    
    return filtered;
  }
}

/*
Usage:
<input [(ngModel)]="globalSearch" placeholder="Search all...">

<table>
  <thead>
    <tr>
      <th>Name <input [(ngModel)]="filters.name"></th>
      <th>Status <select [(ngModel)]="filters.status">...</select></th>
      <th>Age <input [(ngModel)]="filters.age" type="number"></th>
    </tr>
  </thead>
  <tbody>
    <tr *ngFor="let user of users | tableFilter:filters:globalSearch">
      <td>{{ user.name }}</td>
      <td>{{ user.status }}</td>
      <td>{{ user.age }}</td>
    </tr>
  </tbody>
</table>
*/

// 2. Date Range Filter Pipe
console.log('\n=== Date Range Filter ===');

interface DateRange {
  start?: Date;
  end?: Date;
}

@Pipe({
  name: 'dateRangeFilter'
})
export class DateRangeFilterPipe implements PipeTransform {
  transform(items: any[], dateProperty: string, range: DateRange): any[] {
    if (!items || !range || (!range.start && !range.end)) {
      return items;
    }
    
    return items.filter(item => {
      const date = new Date(item[dateProperty]);
      
      if (isNaN(date.getTime())) {
        return false;
      }
      
      if (range.start && date < range.start) {
        return false;
      }
      
      if (range.end && date > range.end) {
        return false;
      }
      
      return true;
    });
  }
}

/*
Usage:
<input type="date" [(ngModel)]="dateRange.start">
<input type="date" [(ngModel)]="dateRange.end">

<div *ngFor="let order of orders | dateRangeFilter:'orderDate':dateRange">
  {{ order.id }} - {{ order.orderDate | date }}
</div>
*/

// 3. Fuzzy Search Pipe
console.log('\n=== Fuzzy Search Pipe ===');

@Pipe({
  name: 'fuzzySearch'
})
export class FuzzySearchPipe implements PipeTransform {
  transform(items: any[], searchTerm: string, properties: string[]): any[] {
    if (!items || !searchTerm) {
      return items;
    }
    
    const pattern = searchTerm.toLowerCase().split('').join('.*');
    const regex = new RegExp(pattern);
    
    return items.filter(item => {
      return properties.some(prop => {
        const value = item[prop];
        if (!value) return false;
        
        return regex.test(value.toString().toLowerCase());
      });
    }).sort((a, b) => {
      // Sort by relevance (exact match first)
      const aScore = this.getRelevanceScore(a, searchTerm, properties);
      const bScore = this.getRelevanceScore(b, searchTerm, properties);
      return bScore - aScore;
    });
  }
  
  private getRelevanceScore(item: any, searchTerm: string, properties: string[]): number {
    let score = 0;
    
    properties.forEach(prop => {
      const value = item[prop]?.toString().toLowerCase() || '';
      const term = searchTerm.toLowerCase();
      
      if (value === term) {
        score += 100; // Exact match
      } else if (value.startsWith(term)) {
        score += 50; // Starts with
      } else if (value.includes(term)) {
        score += 25; // Contains
      } else {
        score += 1; // Fuzzy match
      }
    });
    
    return score;
  }
}

/*
Usage:
<input [(ngModel)]="searchTerm" placeholder="Fuzzy search...">

<div *ngFor="let user of users | fuzzySearch:searchTerm:['name', 'email']">
  {{ user.name }} - {{ user.email }}
</div>

Example: searching "jn" will match "John", "Jane", "Jonathan"
*/

// 4. Category Filter Pipe
console.log('\n=== Category Filter ===');

@Pipe({
  name: 'categoryFilter'
})
export class CategoryFilterPipe implements PipeTransform {
  transform(
    items: any[], 
    selectedCategories: string[], 
    categoryProperty: string = 'category'
  ): any[] {
    if (!items || !selectedCategories || selectedCategories.length === 0) {
      return items;
    }
    
    return items.filter(item => {
      const itemCategory = item[categoryProperty];
      
      // Handle single category
      if (typeof itemCategory === 'string') {
        return selectedCategories.includes(itemCategory);
      }
      
      // Handle multiple categories (array)
      if (Array.isArray(itemCategory)) {
        return itemCategory.some(cat => selectedCategories.includes(cat));
      }
      
      return false;
    });
  }
}

/*
Usage:
<div class="filters">
  <label *ngFor="let category of categories">
    <input 
      type="checkbox" 
      [value]="category"
      (change)="toggleCategory(category)"
      [checked]="selectedCategories.includes(category)">
    {{ category }}
  </label>
</div>

<div *ngFor="let product of products | categoryFilter:selectedCategories">
  {{ product.name }} - {{ product.category }}
</div>
*/

// 5. Price Range Filter Pipe
console.log('\n=== Price Range Filter ===');

interface PriceRange {
  min?: number;
  max?: number;
}

@Pipe({
  name: 'priceRangeFilter'
})
export class PriceRangeFilterPipe implements PipeTransform {
  transform(
    items: any[], 
    range: PriceRange, 
    priceProperty: string = 'price'
  ): any[] {
    if (!items || !range) {
      return items;
    }
    
    return items.filter(item => {
      const price = parseFloat(item[priceProperty]);
      
      if (isNaN(price)) {
        return false;
      }
      
      if (range.min !== undefined && price < range.min) {
        return false;
      }
      
      if (range.max !== undefined && price > range.max) {
        return false;
      }
      
      return true;
    });
  }
}

/*
Usage:
<div class="price-filter">
  <label>
    Min: <input type="number" [(ngModel)]="priceRange.min">
  </label>
  <label>
    Max: <input type="number" [(ngModel)]="priceRange.max">
  </label>
</div>

<div *ngFor="let product of products | priceRangeFilter:priceRange">
  {{ product.name }} - ${{ product.price }}
</div>
*/

// 6. Search Highlight Pipe
console.log('\n=== Search Highlight ===');

@Pipe({
  name: 'highlight'
})
export class HighlightPipe implements PipeTransform {
  transform(text: string, search: string): string {
    if (!search || !text) {
      return text;
    }
    
    const pattern = search.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    const regex = new RegExp(pattern, 'gi');
    
    return text.replace(regex, match => `<mark>${match}</mark>`);
  }
}

/*
Usage:
<input [(ngModel)]="searchTerm">

<div *ngFor="let user of users | filter:searchTerm">
  <div [innerHTML]="user.name | highlight:searchTerm"></div>
  <div [innerHTML]="user.email | highlight:searchTerm"></div>
</div>

CSS:
mark {
  background-color: yellow;
  font-weight: bold;
}
*/

// 7. Sort and Filter Combined
console.log('\n=== Sort and Filter ===');

interface SortConfig {
  property: string;
  direction: 'asc' | 'desc';
}

@Pipe({
  name: 'sortAndFilter'
})
export class SortAndFilterPipe implements PipeTransform {
  transform(
    items: any[], 
    searchTerm: string, 
    sortConfig?: SortConfig
  ): any[] {
    if (!items) return [];
    
    let result = [...items];
    
    // Filter
    if (searchTerm) {
      const search = searchTerm.toLowerCase();
      result = result.filter(item => 
        Object.values(item).some(value => 
          value && value.toString().toLowerCase().includes(search)
        )
      );
    }
    
    // Sort
    if (sortConfig) {
      result.sort((a, b) => {
        const aValue = a[sortConfig.property];
        const bValue = b[sortConfig.property];
        
        let comparison = 0;
        
        if (aValue > bValue) {
          comparison = 1;
        } else if (aValue < bValue) {
          comparison = -1;
        }
        
        return sortConfig.direction === 'asc' ? comparison : -comparison;
      });
    }
    
    return result;
  }
}

/*
Usage:
<input [(ngModel)]="searchTerm" placeholder="Search...">

<button (click)="setSortConfig('name', 'asc')">Name ↑</button>
<button (click)="setSortConfig('name', 'desc')">Name ↓</button>
<button (click)="setSortConfig('age', 'asc')">Age ↑</button>

<div *ngFor="let user of users | sortAndFilter:searchTerm:sortConfig">
  {{ user.name }} - {{ user.age }}
</div>
*/
```

### **Performance Comparison**
```typescript
console.log('\n=== Performance ===');

console.log(`
Angular Pipe Concepts:
┌──────────────────────┬────────────────────────────────┐
│ Concept              │ Description                    │
├──────────────────────┼────────────────────────────────┤
│ Pure Pipe            │ Only runs on input reference   │
│ (default)            │ change. Better performance     │
├──────────────────────┼────────────────────────────────┤
│ Impure Pipe          │ Runs on every change detection │
│ (pure: false)        │ More flexible, slower          │
├──────────────────────┼────────────────────────────────┤
│ Memoization          │ Cache results to avoid recomp  │
│                      │ Good for expensive operations  │
├──────────────────────┼────────────────────────────────┤
│ Async Pipe           │ Subscribe to Observable/Promise│
│                      │ Auto unsubscribe, clean        │
└──────────────────────┴────────────────────────────────┘

Pipe Best Practices:
• Default to pure pipes (performance)
• Keep transform logic simple
• Don't mutate input arrays
• Use memoization for expensive operations
• Consider async operations separately
• Avoid side effects in transform
• Return new array, don't modify original
• Use type safety with TypeScript
• Document complex filter logic
• Test with various data sizes

Performance Tips:
• Pure pipes: only run on reference change
  - Use immutable operations (map, filter, etc.)
  - Create new array: [...items, newItem]
  - Triggers pipe: this.items = [...this.items]
  
• Impure pipes: run every change detection
  - Use sparingly, only when necessary
  - Can cause performance issues
  - Good for: date/time, random values
  
• Caching: store results for repeated inputs
  - Use WeakMap for object keys
  - Limit cache size
  - Clear cache periodically

Common Filter Operations:
• Text search: includes, startsWith, endsWith
• Numeric range: min/max filtering
• Date range: between two dates
• Category: filter by selected categories
• Multi-criteria: combine multiple filters
• Fuzzy search: approximate matching
• Regex: pattern-based filtering

Real-World Patterns:
• Table filtering (column + global search)
• E-commerce (price, category, rating)
• Date pickers (date range filtering)
• Search with highlighting
• Sort + filter combined
• Faceted search (multiple dimensions)
• Auto-complete with debouncing

Testing Pipes:
• Unit test transform method
• Test edge cases (null, undefined, empty)
• Test different input types
• Test performance with large datasets
• Mock dependencies (services)
• Test pure vs impure behavior

Common Pitfalls:
• Forgetting to return items if no filter
• Mutating input array (breaks change detection)
• Expensive operations in impure pipes
• Not handling null/undefined
• Memory leaks with impure pipes
• Over-filtering (removing too much)
`);
```

**Interview Tips:**
- Pipe: transform data in Angular templates using | syntax
- PipeTransform interface: implement transform() method
- Pure pipe (default): only runs when input reference changes (performance)
- Impure pipe (pure: false): runs every change detection cycle (flexible but slower)
- Use @Pipe decorator with name property
- Transform method: receives value and optional parameters
- Pure pipes require immutable operations: [...array] instead of array.push()
- Memoization: cache results to avoid recomputation
- Multiple parameters: pipe:param1:param2:param3
- Return items if no filter (important for null/undefined)
- Don't mutate input array, return new array
- Async pipe: subscribe to Observable/Promise, auto unsubscribe
- Chaining pipes: value | pipe1 | pipe2 | pipe3
- Common use cases: filter, sort, search, date range, price range
- Performance: pure pipes are fast, impure can cause issues with large datasets
- Testing: unit test transform method with various inputs
- Best practice: keep logic simple, avoid side effects
- Real-world: table filters, e-commerce filters, search with highlight
- Trade-offs: pure (fast, limited) vs impure (flexible, slow)
- Follow-ups: async operations, complex criteria, performance optimization
- Clarify: pure or impure? caching needed? multiple parameters? async?

</details>

122. Create a custom directive for input validation

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Validation Directive**
```typescript
/**
 * Simple email validation directive
 */

import { Directive, Input } from '@angular/core';
import { NG_VALIDATORS, Validator, AbstractControl, ValidationErrors } from '@angular/forms';

@Directive({
  selector: '[appEmailValidator]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: EmailValidatorDirective,
    multi: true
  }]
})
export class EmailValidatorDirective implements Validator {
  validate(control: AbstractControl): ValidationErrors | null {
    if (!control.value) {
      return null; // Don't validate empty values
    }
    
    const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    const valid = emailRegex.test(control.value);
    
    return valid ? null : { email: { value: control.value } };
  }
}

// Usage in template:
/*
<form #form="ngForm">
  <input 
    type="email" 
    name="email" 
    [(ngModel)]="user.email"
    appEmailValidator
    #emailField="ngModel">
  
  <div *ngIf="emailField.invalid && emailField.touched">
    <span *ngIf="emailField.errors?.['email']">
      Please enter a valid email address
    </span>
  </div>
</form>
*/

console.log('=== Basic Email Validation Directive ===');
console.log('Validates email format using regex');
console.log('Integrates with Angular forms (template-driven)');
```

### **Approach 2: Parameterized Validation Directive**
```typescript
/**
 * Min/Max length validator with custom parameters
 */

import { Directive, Input } from '@angular/core';
import { NG_VALIDATORS, Validator, AbstractControl, ValidationErrors } from '@angular/forms';

@Directive({
  selector: '[appMinMax]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: MinMaxValidatorDirective,
    multi: true
  }]
})
export class MinMaxValidatorDirective implements Validator {
  @Input('appMinMax') minMax?: { min?: number; max?: number };
  
  validate(control: AbstractControl): ValidationErrors | null {
    if (!control.value || !this.minMax) {
      return null;
    }
    
    const length = control.value.toString().length;
    const errors: ValidationErrors = {};
    
    if (this.minMax.min !== undefined && length < this.minMax.min) {
      errors['minLength'] = {
        requiredLength: this.minMax.min,
        actualLength: length
      };
    }
    
    if (this.minMax.max !== undefined && length > this.minMax.max) {
      errors['maxLength'] = {
        requiredLength: this.minMax.max,
        actualLength: length
      };
    }
    
    return Object.keys(errors).length > 0 ? errors : null;
  }
}

// Usage:
/*
<input 
  [(ngModel)]="username"
  name="username"
  [appMinMax]="{min: 3, max: 20}"
  #usernameField="ngModel">

<div *ngIf="usernameField.errors?.['minLength']">
  Minimum {{ usernameField.errors?.['minLength'].requiredLength }} characters
</div>
<div *ngIf="usernameField.errors?.['maxLength']">
  Maximum {{ usernameField.errors?.['maxLength'].requiredLength }} characters
</div>
*/

console.log('\n=== Parameterized Min/Max Directive ===');
console.log('Accepts min/max parameters via @Input');
console.log('Provides detailed error information');
```

### **Approach 3: Pattern Matching Directive**
```typescript
/**
 * Custom pattern validator with common presets
 */

import { Directive, Input } from '@angular/core';
import { NG_VALIDATORS, Validator, AbstractControl, ValidationErrors } from '@angular/forms';

type PatternType = 'phone' | 'zipCode' | 'creditCard' | 'url' | 'alphanumeric' | 'custom';

@Directive({
  selector: '[appPattern]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: PatternValidatorDirective,
    multi: true
  }]
})
export class PatternValidatorDirective implements Validator {
  @Input('appPattern') patternType?: PatternType;
  @Input('appPatternCustom') customPattern?: string;
  
  private patterns: Record<PatternType, RegExp> = {
    phone: /^\+?1?\d{9,15}$/,
    zipCode: /^\d{5}(-\d{4})?$/,
    creditCard: /^\d{4}\s?\d{4}\s?\d{4}\s?\d{4}$/,
    url: /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/,
    alphanumeric: /^[a-zA-Z0-9]+$/,
    custom: /.*/
  };
  
  validate(control: AbstractControl): ValidationErrors | null {
    if (!control.value || !this.patternType) {
      return null;
    }
    
    let pattern: RegExp;
    
    if (this.patternType === 'custom' && this.customPattern) {
      pattern = new RegExp(this.customPattern);
    } else {
      pattern = this.patterns[this.patternType];
    }
    
    const valid = pattern.test(control.value);
    
    return valid ? null : {
      pattern: {
        type: this.patternType,
        value: control.value,
        requiredPattern: pattern.toString()
      }
    };
  }
}

// Usage:
/*
<!-- Phone validation -->
<input 
  [(ngModel)]="phone"
  name="phone"
  appPattern="phone"
  #phoneField="ngModel">

<!-- Custom pattern -->
<input 
  [(ngModel)]="code"
  name="code"
  appPattern="custom"
  appPatternCustom="^[A-Z]{3}\d{3}$"
  #codeField="ngModel">

<div *ngIf="phoneField.errors?.['pattern']">
  Invalid {{ phoneField.errors?.['pattern'].type }} format
</div>
*/

console.log('\n=== Pattern Matching Directive ===');
console.log('Supports common patterns: phone, zipCode, creditCard, url');
console.log('Also supports custom regex patterns');
```

### **Approach 4: Async Validation Directive**
```typescript
/**
 * Async validator (check username availability)
 */

import { Directive, Input } from '@angular/core';
import { NG_ASYNC_VALIDATORS, AsyncValidator, AbstractControl, ValidationErrors } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { map, debounceTime, switchMap, catchError } from 'rxjs/operators';

@Directive({
  selector: '[appUniqueUsername]',
  providers: [{
    provide: NG_ASYNC_VALIDATORS,
    useExisting: UniqueUsernameValidatorDirective,
    multi: true
  }]
})
export class UniqueUsernameValidatorDirective implements AsyncValidator {
  @Input('appUniqueUsername') existingUsernames: string[] = [];
  
  validate(control: AbstractControl): Observable<ValidationErrors | null> {
    if (!control.value) {
      return of(null);
    }
    
    // Simulate API call to check username
    return of(control.value).pipe(
      debounceTime(500), // Wait 500ms after user stops typing
      switchMap(username => this.checkUsername(username)),
      map(isTaken => isTaken ? { usernameTaken: { value: control.value } } : null),
      catchError(() => of(null))
    );
  }
  
  private checkUsername(username: string): Observable<boolean> {
    // Simulate API call
    return new Observable(observer => {
      setTimeout(() => {
        const taken = this.existingUsernames.includes(username.toLowerCase());
        observer.next(taken);
        observer.complete();
      }, 300);
    });
  }
}

// Usage:
/*
<input 
  [(ngModel)]="username"
  name="username"
  [appUniqueUsername]="['admin', 'user', 'test']"
  #usernameField="ngModel">

<div *ngIf="usernameField.pending">
  Checking username...
</div>
<div *ngIf="usernameField.errors?.['usernameTaken']">
  Username is already taken
</div>
*/

console.log('\n=== Async Validation Directive ===');
console.log('Validates against API (simulated)');
console.log('Uses debounceTime to reduce API calls');
```

### **Approach 5: Cross-Field Validation Directive**
```typescript
/**
 * Compare two fields (e.g., password confirmation)
 */

import { Directive, Input } from '@angular/core';
import { NG_VALIDATORS, Validator, AbstractControl, ValidationErrors, FormGroup } from '@angular/forms';

@Directive({
  selector: '[appMatchFields]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: MatchFieldsValidatorDirective,
    multi: true
  }]
})
export class MatchFieldsValidatorDirective implements Validator {
  @Input('appMatchFields') fieldToMatch?: string;
  
  validate(control: AbstractControl): ValidationErrors | null {
    if (!control.value || !this.fieldToMatch) {
      return null;
    }
    
    const parent = control.parent as FormGroup;
    
    if (!parent) {
      return null;
    }
    
    const matchControl = parent.get(this.fieldToMatch);
    
    if (!matchControl) {
      return null;
    }
    
    const match = control.value === matchControl.value;
    
    return match ? null : {
      fieldMismatch: {
        field: this.fieldToMatch,
        value: control.value
      }
    };
  }
}

// Usage:
/*
<form #form="ngForm">
  <input 
    type="password"
    [(ngModel)]="password"
    name="password"
    #passwordField="ngModel">
  
  <input 
    type="password"
    [(ngModel)]="confirmPassword"
    name="confirmPassword"
    appMatchFields="password"
    #confirmField="ngModel">
  
  <div *ngIf="confirmField.errors?.['fieldMismatch'] && confirmField.touched">
    Passwords do not match
  </div>
</form>
*/

console.log('\n=== Cross-Field Validation ===');
console.log('Compares two fields (e.g., password confirmation)');
console.log('Accesses parent form to get other field');
```

### **Real-World Use Cases**
```typescript
/**
 * Practical validation directives
 */

// 1. Credit Card Validator with Luhn Algorithm
console.log('\n=== Credit Card Validator ===');

@Directive({
  selector: '[appCreditCard]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: CreditCardValidatorDirective,
    multi: true
  }]
})
export class CreditCardValidatorDirective implements Validator {
  validate(control: AbstractControl): ValidationErrors | null {
    if (!control.value) {
      return null;
    }
    
    const cardNumber = control.value.replace(/\s/g, '');
    
    // Check if only digits
    if (!/^\d+$/.test(cardNumber)) {
      return { creditCard: { message: 'Must contain only digits' } };
    }
    
    // Check length (13-19 digits)
    if (cardNumber.length < 13 || cardNumber.length > 19) {
      return { creditCard: { message: 'Invalid length' } };
    }
    
    // Luhn algorithm
    if (!this.luhnCheck(cardNumber)) {
      return { creditCard: { message: 'Invalid card number' } };
    }
    
    return null;
  }
  
  private luhnCheck(cardNumber: string): boolean {
    let sum = 0;
    let isEven = false;
    
    // Loop through values from right to left
    for (let i = cardNumber.length - 1; i >= 0; i--) {
      let digit = parseInt(cardNumber[i], 10);
      
      if (isEven) {
        digit *= 2;
        if (digit > 9) {
          digit -= 9;
        }
      }
      
      sum += digit;
      isEven = !isEven;
    }
    
    return sum % 10 === 0;
  }
}

/*
Usage:
<input 
  [(ngModel)]="cardNumber"
  name="cardNumber"
  appCreditCard
  #cardField="ngModel">

<div *ngIf="cardField.errors?.['creditCard']">
  {{ cardField.errors?.['creditCard'].message }}
</div>
*/

// 2. Password Strength Validator
console.log('\n=== Password Strength Validator ===');

interface PasswordStrength {
  hasMinLength: boolean;
  hasUpperCase: boolean;
  hasLowerCase: boolean;
  hasNumber: boolean;
  hasSpecialChar: boolean;
  strength: 'weak' | 'medium' | 'strong';
}

@Directive({
  selector: '[appPasswordStrength]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: PasswordStrengthValidatorDirective,
    multi: true
  }]
})
export class PasswordStrengthValidatorDirective implements Validator {
  @Input('appPasswordStrength') minStrength: 'weak' | 'medium' | 'strong' = 'medium';
  
  validate(control: AbstractControl): ValidationErrors | null {
    if (!control.value) {
      return null;
    }
    
    const strength = this.calculateStrength(control.value);
    
    const strengthLevels = { weak: 1, medium: 2, strong: 3 };
    
    if (strengthLevels[strength.strength] < strengthLevels[this.minStrength]) {
      return {
        passwordStrength: {
          required: this.minStrength,
          actual: strength.strength,
          details: strength
        }
      };
    }
    
    return null;
  }
  
  private calculateStrength(password: string): PasswordStrength {
    const strength: PasswordStrength = {
      hasMinLength: password.length >= 8,
      hasUpperCase: /[A-Z]/.test(password),
      hasLowerCase: /[a-z]/.test(password),
      hasNumber: /\d/.test(password),
      hasSpecialChar: /[!@#$%^&*(),.?":{}|<>]/.test(password),
      strength: 'weak'
    };
    
    const score = Object.values(strength).filter(v => v === true).length;
    
    if (score >= 5) {
      strength.strength = 'strong';
    } else if (score >= 3) {
      strength.strength = 'medium';
    }
    
    return strength;
  }
}

/*
Usage:
<input 
  type="password"
  [(ngModel)]="password"
  name="password"
  appPasswordStrength="strong"
  #passwordField="ngModel">

<div *ngIf="passwordField.errors?.['passwordStrength']">
  <p>Password must be {{ passwordField.errors?.['passwordStrength'].required }}</p>
  <ul>
    <li [class.valid]="passwordField.errors?.['passwordStrength'].details.hasMinLength">
      At least 8 characters
    </li>
    <li [class.valid]="passwordField.errors?.['passwordStrength'].details.hasUpperCase">
      Contains uppercase letter
    </li>
    <li [class.valid]="passwordField.errors?.['passwordStrength'].details.hasLowerCase">
      Contains lowercase letter
    </li>
    <li [class.valid]="passwordField.errors?.['passwordStrength'].details.hasNumber">
      Contains number
    </li>
    <li [class.valid]="passwordField.errors?.['passwordStrength'].details.hasSpecialChar">
      Contains special character
    </li>
  </ul>
</div>
*/

// 3. Date Range Validator
console.log('\n=== Date Range Validator ===');

@Directive({
  selector: '[appDateRange]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: DateRangeValidatorDirective,
    multi: true
  }]
})
export class DateRangeValidatorDirective implements Validator {
  @Input('appDateRange') range?: { min?: Date; max?: Date };
  
  validate(control: AbstractControl): ValidationErrors | null {
    if (!control.value || !this.range) {
      return null;
    }
    
    const date = new Date(control.value);
    
    if (isNaN(date.getTime())) {
      return { dateRange: { message: 'Invalid date' } };
    }
    
    if (this.range.min && date < this.range.min) {
      return {
        dateRange: {
          message: 'Date is before minimum',
          min: this.range.min,
          actual: date
        }
      };
    }
    
    if (this.range.max && date > this.range.max) {
      return {
        dateRange: {
          message: 'Date is after maximum',
          max: this.range.max,
          actual: date
        }
      };
    }
    
    return null;
  }
}

/*
Usage:
<input 
  type="date"
  [(ngModel)]="birthDate"
  name="birthDate"
  [appDateRange]="{min: minDate, max: maxDate}"
  #dateField="ngModel">

Component:
minDate = new Date(1900, 0, 1);
maxDate = new Date();
*/

// 4. File Upload Validator
console.log('\n=== File Upload Validator ===');

@Directive({
  selector: '[appFileValidation]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: FileValidationDirective,
    multi: true
  }]
})
export class FileValidationDirective implements Validator {
  @Input('appFileValidation') config?: {
    maxSize?: number; // in bytes
    allowedTypes?: string[]; // ['image/png', 'image/jpeg']
  };
  
  validate(control: AbstractControl): ValidationErrors | null {
    if (!control.value || !this.config) {
      return null;
    }
    
    const file = control.value as File;
    
    // Check file size
    if (this.config.maxSize && file.size > this.config.maxSize) {
      return {
        fileValidation: {
          message: 'File too large',
          maxSize: this.config.maxSize,
          actualSize: file.size
        }
      };
    }
    
    // Check file type
    if (this.config.allowedTypes && !this.config.allowedTypes.includes(file.type)) {
      return {
        fileValidation: {
          message: 'Invalid file type',
          allowedTypes: this.config.allowedTypes,
          actualType: file.type
        }
      };
    }
    
    return null;
  }
}

/*
Usage:
<input 
  type="file"
  (change)="onFileSelected($event)"
  [appFileValidation]="{
    maxSize: 5242880,
    allowedTypes: ['image/png', 'image/jpeg', 'image/gif']
  }">

Component:
onFileSelected(event: Event) {
  const input = event.target as HTMLInputElement;
  if (input.files && input.files[0]) {
    this.selectedFile = input.files[0];
  }
}
*/

// 5. Conditional Validation Directive
console.log('\n=== Conditional Validator ===');

@Directive({
  selector: '[appConditionalValidator]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: ConditionalValidatorDirective,
    multi: true
  }]
})
export class ConditionalValidatorDirective implements Validator {
  @Input('appConditionalValidator') condition?: boolean;
  @Input('appConditionalValidatorType') validatorType?: 'required' | 'email' | 'minLength';
  @Input('appConditionalValidatorParam') param?: any;
  
  validate(control: AbstractControl): ValidationErrors | null {
    // Only validate if condition is true
    if (!this.condition) {
      return null;
    }
    
    if (!control.value) {
      return { required: true };
    }
    
    switch (this.validatorType) {
      case 'required':
        return control.value ? null : { required: true };
        
      case 'email':
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(control.value) ? null : { email: true };
        
      case 'minLength':
        const length = control.value.length;
        return length >= this.param ? null : {
          minLength: { requiredLength: this.param, actualLength: length }
        };
        
      default:
        return null;
    }
  }
}

/*
Usage:
<form #form="ngForm">
  <label>
    <input 
      type="checkbox" 
      [(ngModel)]="hasCompany"
      name="hasCompany">
    I work for a company
  </label>
  
  <input 
    [(ngModel)]="companyName"
    name="companyName"
    [appConditionalValidator]="hasCompany"
    appConditionalValidatorType="required"
    #companyField="ngModel">
  
  <div *ngIf="companyField.invalid && hasCompany">
    Company name is required
  </div>
</form>
*/

// 6. Custom Error Display Directive
console.log('\n=== Error Display Directive ===');

@Directive({
  selector: '[appShowErrors]'
})
export class ShowErrorsDirective {
  @Input('appShowErrors') control?: AbstractControl;
  
  constructor(private el: ElementRef, private renderer: Renderer2) {}
  
  ngOnInit() {
    if (!this.control) return;
    
    this.control.statusChanges?.subscribe(() => {
      this.updateErrors();
    });
    
    this.updateErrors();
  }
  
  private updateErrors() {
    if (!this.control) return;
    
    const errors = this.control.errors;
    const touched = this.control.touched;
    
    // Clear previous errors
    this.renderer.setProperty(this.el.nativeElement, 'innerHTML', '');
    
    if (errors && touched) {
      const errorMessages = this.getErrorMessages(errors);
      
      errorMessages.forEach(message => {
        const div = this.renderer.createElement('div');
        const text = this.renderer.createText(message);
        
        this.renderer.addClass(div, 'error-message');
        this.renderer.appendChild(div, text);
        this.renderer.appendChild(this.el.nativeElement, div);
      });
    }
  }
  
  private getErrorMessages(errors: ValidationErrors): string[] {
    const messages: string[] = [];
    
    if (errors['required']) {
      messages.push('This field is required');
    }
    
    if (errors['email']) {
      messages.push('Please enter a valid email');
    }
    
    if (errors['minLength']) {
      messages.push(`Minimum ${errors['minLength'].requiredLength} characters required`);
    }
    
    if (errors['maxLength']) {
      messages.push(`Maximum ${errors['maxLength'].requiredLength} characters allowed`);
    }
    
    if (errors['pattern']) {
      messages.push(`Invalid ${errors['pattern'].type} format`);
    }
    
    return messages;
  }
}

/*
Usage:
<input 
  [(ngModel)]="email"
  name="email"
  appEmailValidator
  #emailField="ngModel">

<div [appShowErrors]="emailField.control"></div>

CSS:
.error-message {
  color: red;
  font-size: 12px;
  margin-top: 4px;
}
*/

// 7. Debounced Validation Directive
console.log('\n=== Debounced Validator ===');

@Directive({
  selector: '[appDebouncedValidator]'
})
export class DebouncedValidatorDirective implements OnInit {
  @Input('appDebouncedValidator') debounceTime: number = 500;
  
  constructor(@Self() private ngControl: NgControl) {}
  
  ngOnInit() {
    if (!this.ngControl.control) return;
    
    // Store original value changes
    const originalValueChanges = this.ngControl.control.valueChanges;
    
    // Replace with debounced version
    this.ngControl.control.valueChanges = originalValueChanges?.pipe(
      debounceTime(this.debounceTime),
      distinctUntilChanged()
    );
  }
}

/*
Usage:
<input 
  [(ngModel)]="searchTerm"
  name="search"
  appDebouncedValidator
  [appDebouncedValidator]="800"
  #searchField="ngModel">
*/
```

### **Performance Comparison**
```typescript
console.log('\n=== Performance ===');

console.log(`
Angular Validation Concepts:
┌──────────────────────┬────────────────────────────────┐
│ Concept              │ Description                    │
├──────────────────────┼────────────────────────────────┤
│ Validator            │ Implements validate() method   │
│                      │ Returns ValidationErrors or null│
├──────────────────────┼────────────────────────────────┤
│ AsyncValidator       │ Returns Observable<Errors>     │
│                      │ Used for API calls             │
├──────────────────────┼────────────────────────────────┤
│ NG_VALIDATORS        │ Token for sync validators      │
│                      │ Register in providers array    │
├──────────────────────┼────────────────────────────────┤
│ NG_ASYNC_VALIDATORS  │ Token for async validators     │
│                      │ Separate from sync validators  │
├──────────────────────┼────────────────────────────────┤
│ ValidationErrors     │ Object with error details      │
│                      │ { errorKey: { ...details } }   │
└──────────────────────┴────────────────────────────────┘

Validation Best Practices:
• Return null for valid values
• Return ValidationErrors object for invalid
• Don't validate empty values (use required separately)
• Provide detailed error information
• Use multi: true in providers
• Debounce async validators
• Handle edge cases (null, undefined, empty string)
• Make directives reusable
• Type-safe with TypeScript
• Test thoroughly

Common Validation Types:
┌──────────────────┬────────────────────────────────┐
│ Type             │ Use Case                       │
├──────────────────┼────────────────────────────────┤
│ Format           │ Email, phone, URL, pattern     │
│ Length           │ Min/max characters             │
│ Range            │ Min/max numbers, dates         │
│ Comparison       │ Match fields (password confirm)│
│ Uniqueness       │ Check against API/database     │
│ Custom Logic     │ Business rules, complex checks │
│ File             │ Size, type, dimensions         │
└──────────────────┴────────────────────────────────┘

Directive Lifecycle:
1. Directive created and attached to element
2. @Input values bound
3. validate() called initially
4. validate() called on value changes
5. Errors propagated to form control
6. Template updated based on control state

Form Control States:
• valid/invalid: validation status
• pristine/dirty: user interaction
• touched/untouched: focus/blur events
• pending: async validation in progress
• disabled/enabled: control state

Error Display Strategies:
• Show on touch: touched && invalid
• Show on submit: submitted && invalid
• Show immediately: always show if invalid
• Show with delay: debounce error display
• Custom directive: automate error display

Integration Points:
• Template-driven forms: NG_VALIDATORS
• Reactive forms: add to FormControl/FormGroup
• Custom validators: implement Validator interface
• Async validators: implement AsyncValidator
• Cross-field: access parent FormGroup

Performance Optimization:
• Debounce async validators (reduce API calls)
• Cache validation results when possible
• Use pure functions for validation logic
• Avoid expensive operations in validate()
• Unsubscribe from observables properly
• Limit validator execution frequency

Real-World Patterns:
• Email format validation
• Password strength checking
• Credit card with Luhn algorithm
• Async username availability
• Cross-field password confirmation
• Conditional validation (show/hide)
• File upload restrictions
• Date range validation
• Custom pattern matching
• Auto-display errors

Testing Directives:
• Unit test validate() method
• Test with various input values
• Test edge cases (null, empty, invalid)
• Mock async operations
• Test error message content
• Test integration with forms
`);
```

**Interview Tips:**
- Validation directive: Angular directive that validates form input
- Implements Validator or AsyncValidator interface
- Register with NG_VALIDATORS or NG_ASYNC_VALIDATORS token
- multi: true allows multiple validators on same element
- validate() method: receives AbstractControl, returns ValidationErrors or null
- Return null for valid, { errorKey: details } for invalid
- ValidationErrors: object with error key and details
- Don't validate empty values (null check first)
- @Input for parameters: [appMinMax]="{min: 3, max: 20}"
- Async validators: return Observable<ValidationErrors | null>
- Use debounceTime for async validators (reduce API calls)
- Cross-field validation: access control.parent to get other fields
- Form states: valid/invalid, pristine/dirty, touched/untouched, pending
- Error display: check touched && invalid before showing
- Template-driven forms: attach directive to input element
- Reactive forms: add validator to FormControl
- Common validators: email, pattern, min/max length, required, custom logic
- Real-world: password strength, credit card (Luhn), async username check
- Performance: debounce, cache results, avoid expensive operations
- Trade-offs: sync (fast) vs async (flexible but slower)
- Follow-ups: async validation, cross-field, conditional, file upload
- Clarify: sync or async? parameters? error messages? integration with forms?

</details>

123. Build a custom async validator for forms

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Simple Username Availability Validator**
```typescript
/**
 * Check if username is available via API
 */

import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { map, catchError, debounceTime, switchMap } from 'rxjs/operators';
import { HttpClient } from '@angular/common/http';

export function usernameAvailabilityValidator(http: HttpClient): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }
    
    return of(control.value).pipe(
      debounceTime(500), // Wait 500ms after user stops typing
      switchMap(username => 
        http.get<{ available: boolean }>(`/api/check-username/${username}`)
      ),
      map(response => response.available ? null : { usernameTaken: true }),
      catchError(() => of(null)) // Handle errors gracefully
    );
  };
}

// Usage in component:
/*
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

export class SignupComponent {
  signupForm: FormGroup;
  
  constructor(
    private fb: FormBuilder,
    private http: HttpClient
  ) {
    this.signupForm = this.fb.group({
      username: ['', 
        [Validators.required, Validators.minLength(3)],
        [usernameAvailabilityValidator(this.http)]  // Async validator
      ]
    });
  }
}

// Template:
<form [formGroup]="signupForm">
  <input formControlName="username">
  
  <div *ngIf="signupForm.get('username')?.pending">
    Checking username...
  </div>
  
  <div *ngIf="signupForm.get('username')?.hasError('usernameTaken')">
    Username is already taken
  </div>
</form>
*/

console.log('=== Simple Username Availability Validator ===');
console.log('Checks API with debouncing');
console.log('Shows pending state while validating');
```

### **Approach 2: Email Domain Validator**
```typescript
/**
 * Validate email domain against whitelist/blacklist
 */

import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { map, delay } from 'rxjs/operators';

export function emailDomainValidator(
  allowedDomains?: string[],
  blockedDomains?: string[]
): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }
    
    const email = control.value;
    const domain = email.split('@')[1]?.toLowerCase();
    
    if (!domain) {
      return of({ invalidEmail: true });
    }
    
    // Simulate API call
    return of(domain).pipe(
      delay(300), // Simulate network delay
      map(dom => {
        // Check blocked domains
        if (blockedDomains && blockedDomains.includes(dom)) {
          return { domainBlocked: { domain: dom } };
        }
        
        // Check allowed domains
        if (allowedDomains && allowedDomains.length > 0) {
          if (!allowedDomains.includes(dom)) {
            return { domainNotAllowed: { domain: dom, allowed: allowedDomains } };
          }
        }
        
        return null;
      })
    );
  };
}

// Usage:
/*
this.form = this.fb.group({
  email: ['',
    [Validators.required, Validators.email],
    [emailDomainValidator(['company.com', 'example.com'], ['spam.com'])]
  ]
});

// Template:
<input formControlName="email">

<div *ngIf="form.get('email')?.hasError('domainBlocked')">
  This email domain is not allowed
</div>

<div *ngIf="form.get('email')?.hasError('domainNotAllowed')">
  Please use: {{ form.get('email')?.errors?.['domainNotAllowed'].allowed.join(', ') }}
</div>
*/

console.log('\n=== Email Domain Validator ===');
console.log('Validates against whitelist/blacklist');
console.log('Simulates async API check');
```

### **Approach 3: Password Breach Checker**
```typescript
/**
 * Check if password has been compromised (Have I Been Pwned API)
 */

import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { map, catchError, debounceTime, switchMap, distinctUntilChanged } from 'rxjs/operators';
import { HttpClient } from '@angular/common/http';
import * as crypto from 'crypto';

export function passwordBreachValidator(http: HttpClient): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value || control.value.length < 8) {
      return of(null); // Skip validation for short passwords
    }
    
    return of(control.value).pipe(
      debounceTime(1000), // Wait longer for password typing
      distinctUntilChanged(),
      switchMap(password => this.checkPasswordBreach(http, password)),
      map(breached => breached ? { passwordBreached: true } : null),
      catchError(error => {
        console.error('Error checking password breach:', error);
        return of(null); // Don't block form on error
      })
    );
  };
  
  function checkPasswordBreach(http: HttpClient, password: string): Observable<boolean> {
    // Hash password with SHA-1
    const hash = crypto.createHash('sha1').update(password).digest('hex').toUpperCase();
    const prefix = hash.substring(0, 5);
    const suffix = hash.substring(5);
    
    // Check against Have I Been Pwned API (k-anonymity model)
    return http.get(`https://api.pwnedpasswords.com/range/${prefix}`, {
      responseType: 'text'
    }).pipe(
      map(response => {
        // Check if hash suffix appears in response
        const hashes = response.split('\n');
        return hashes.some(line => line.startsWith(suffix));
      })
    );
  }
}

/*
Usage:
this.form = this.fb.group({
  password: ['',
    [Validators.required, Validators.minLength(8)],
    [passwordBreachValidator(this.http)]
  ]
});

<input type="password" formControlName="password">

<div *ngIf="form.get('password')?.pending">
  Checking password security...
</div>

<div *ngIf="form.get('password')?.hasError('passwordBreached')">
  This password has been compromised in a data breach. Please choose a different one.
</div>
*/

console.log('\n=== Password Breach Validator ===');
console.log('Checks Have I Been Pwned API');
console.log('Uses k-anonymity for privacy');
```

### **Approach 4: Debounced API Validator with Caching**
```typescript
/**
 * Generic async validator with caching
 */

import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { map, catchError, debounceTime, switchMap, tap } from 'rxjs/operators';

export class CachedAsyncValidator {
  private cache = new Map<string, ValidationErrors | null>();
  
  createValidator(
    validationFn: (value: any) => Observable<boolean>,
    errorKey: string,
    debounceMs: number = 500
  ): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      if (!control.value) {
        return of(null);
      }
      
      const cacheKey = `${errorKey}-${control.value}`;
      
      // Check cache
      if (this.cache.has(cacheKey)) {
        console.log('Returning cached result for:', control.value);
        return of(this.cache.get(cacheKey)!);
      }
      
      console.log('Validating:', control.value);
      
      return of(control.value).pipe(
        debounceTime(debounceMs),
        switchMap(value => validationFn(value)),
        map(isValid => {
          const error = isValid ? null : { [errorKey]: true };
          
          // Cache result
          this.cache.set(cacheKey, error);
          
          // Limit cache size
          if (this.cache.size > 100) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
          }
          
          return error;
        }),
        catchError(() => of(null))
      );
    };
  }
  
  clearCache() {
    this.cache.clear();
  }
}

// Usage:
/*
export class MyComponent {
  cachedValidator = new CachedAsyncValidator();
  
  form = this.fb.group({
    email: ['',
      [Validators.required, Validators.email],
      [this.cachedValidator.createValidator(
        (email) => this.http.get<{exists: boolean}>(`/api/check-email/${email}`)
          .pipe(map(res => !res.exists)),
        'emailTaken',
        800
      )]
    ],
    
    username: ['',
      [Validators.required],
      [this.cachedValidator.createValidator(
        (username) => this.http.get<{available: boolean}>(`/api/check-username/${username}`)
          .pipe(map(res => res.available)),
        'usernameTaken',
        500
      )]
    ]
  });
}
*/

console.log('\n=== Cached Async Validator ===');
console.log('Generic validator with caching');
console.log('Reuses results for same input');
```

### **Approach 5: Multi-Field Async Validator**
```typescript
/**
 * Async validator that checks multiple fields
 */

import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { Observable, of, combineLatest } from 'rxjs';
import { map, debounceTime, switchMap, catchError } from 'rxjs/operators';
import { HttpClient } from '@angular/common/http';

export function addressValidator(http: HttpClient): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    const street = control.get('street')?.value;
    const city = control.get('city')?.value;
    const zipCode = control.get('zipCode')?.value;
    
    if (!street || !city || !zipCode) {
      return of(null); // Wait for all fields to be filled
    }
    
    return of({ street, city, zipCode }).pipe(
      debounceTime(1000),
      switchMap(address => 
        http.post<{ valid: boolean; suggestions?: any[] }>('/api/validate-address', address)
      ),
      map(response => {
        if (response.valid) {
          return null;
        }
        
        return {
          invalidAddress: {
            message: 'Address could not be verified',
            suggestions: response.suggestions || []
          }
        };
      }),
      catchError(() => of(null))
    );
  };
}

// Usage:
/*
this.form = this.fb.group({
  address: this.fb.group({
    street: ['', Validators.required],
    city: ['', Validators.required],
    zipCode: ['', Validators.required]
  }, {
    asyncValidators: [addressValidator(this.http)]  // Group-level validator
  })
});

// Template:
<div formGroupName="address">
  <input formControlName="street" placeholder="Street">
  <input formControlName="city" placeholder="City">
  <input formControlName="zipCode" placeholder="Zip Code">
  
  <div *ngIf="form.get('address')?.pending">
    Validating address...
  </div>
  
  <div *ngIf="form.get('address')?.hasError('invalidAddress')">
    {{ form.get('address')?.errors?.['invalidAddress'].message }}
    
    <div *ngIf="form.get('address')?.errors?.['invalidAddress'].suggestions?.length">
      Did you mean:
      <ul>
        <li *ngFor="let suggestion of form.get('address')?.errors?.['invalidAddress'].suggestions">
          {{ suggestion }}
        </li>
      </ul>
    </div>
  </div>
</div>
*/

console.log('\n=== Multi-Field Async Validator ===');
console.log('Validates entire address group');
console.log('Returns suggestions for corrections');
```

### **Real-World Use Cases**
```typescript
/**
 * Practical async validators
 */

// 1. Slug/URL Validator
console.log('\n=== Slug Validator ===');

export function slugValidator(http: HttpClient, resourceType: string): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }
    
    const slug = control.value.toLowerCase().trim();
    
    // Check format first
    if (!/^[a-z0-9-]+$/.test(slug)) {
      return of({ invalidSlugFormat: true });
    }
    
    // Check availability
    return of(slug).pipe(
      debounceTime(500),
      switchMap(s => 
        http.get<{ available: boolean }>(`/api/${resourceType}/check-slug/${s}`)
      ),
      map(response => response.available ? null : { slugTaken: { slug } }),
      catchError(() => of(null))
    );
  };
}

/*
Usage for blog posts:
this.form = this.fb.group({
  title: ['', Validators.required],
  slug: ['',
    [Validators.required],
    [slugValidator(this.http, 'posts')]
  ]
});

// Auto-generate slug from title
this.form.get('title')?.valueChanges.subscribe(title => {
  const slug = title.toLowerCase()
    .replace(/[^\w\s-]/g, '')
    .replace(/\s+/g, '-');
  this.form.get('slug')?.setValue(slug);
});
*/

// 2. Credit Card Validator with BIN Lookup
console.log('\n=== Credit Card BIN Validator ===');

interface CardInfo {
  type: string;
  brand: string;
  valid: boolean;
}

export function creditCardValidator(http: HttpClient): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }
    
    const cardNumber = control.value.replace(/\s/g, '');
    
    // Basic validation first
    if (!/^\d{13,19}$/.test(cardNumber)) {
      return of({ invalidCardFormat: true });
    }
    
    // Get BIN (first 6 digits)
    const bin = cardNumber.substring(0, 6);
    
    return of(bin).pipe(
      debounceTime(800),
      switchMap(b => 
        http.get<CardInfo>(`/api/card/bin-lookup/${b}`)
      ),
      map(cardInfo => {
        if (!cardInfo.valid) {
          return { invalidCard: { message: 'Card not supported' } };
        }
        
        // Also set card type for display
        control.root.get('cardType')?.setValue(cardInfo.brand, { emitEvent: false });
        
        return null;
      }),
      catchError(() => of({ cardValidationError: true }))
    );
  };
}

/*
Usage:
this.form = this.fb.group({
  cardNumber: ['',
    [Validators.required],
    [creditCardValidator(this.http)]
  ],
  cardType: [''] // Auto-populated by validator
});
*/

// 3. Promo Code Validator
console.log('\n=== Promo Code Validator ===');

interface PromoCodeInfo {
  valid: boolean;
  discount?: number;
  expiresAt?: Date;
  message?: string;
}

export function promoCodeValidator(http: HttpClient): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }
    
    const code = control.value.toUpperCase();
    
    return of(code).pipe(
      debounceTime(500),
      switchMap(c => 
        http.post<PromoCodeInfo>('/api/validate-promo-code', {
          code: c,
          cartTotal: control.root.get('cartTotal')?.value
        })
      ),
      map(info => {
        if (!info.valid) {
          return {
            invalidPromoCode: {
              message: info.message || 'Invalid promo code'
            }
          };
        }
        
        // Store discount info for later use
        control.root.get('discount')?.setValue(info.discount, { emitEvent: false });
        
        return null;
      }),
      catchError(() => of({ promoCodeError: true }))
    );
  };
}

/*
Usage:
this.form = this.fb.group({
  promoCode: ['',
    [],
    [promoCodeValidator(this.http)]
  ],
  cartTotal: [100],
  discount: [0]
});

// Template shows discount when code is valid
<input formControlName="promoCode" placeholder="Enter promo code">

<div *ngIf="form.get('promoCode')?.pending">
  Validating code...
</div>

<div *ngIf="form.get('promoCode')?.valid && form.get('discount')?.value">
  Discount applied: ${{ form.get('discount')?.value }}
</div>
*/

// 4. Social Media Handle Validator
console.log('\n=== Social Media Validator ===');

export function socialHandleValidator(
  http: HttpClient,
  platform: 'twitter' | 'instagram' | 'tiktok'
): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }
    
    const handle = control.value.replace('@', '').toLowerCase();
    
    // Platform-specific format validation
    const patterns = {
      twitter: /^[A-Za-z0-9_]{1,15}$/,
      instagram: /^[A-Za-z0-9._]{1,30}$/,
      tiktok: /^[A-Za-z0-9._]{2,24}$/
    };
    
    if (!patterns[platform].test(handle)) {
      return of({ invalidHandleFormat: { platform } });
    }
    
    // Check if handle exists
    return of(handle).pipe(
      debounceTime(700),
      switchMap(h => 
        http.get<{ exists: boolean; verified?: boolean }>
          (`/api/social/${platform}/check/${h}`)
      ),
      map(response => {
        if (!response.exists) {
          return { handleNotFound: { platform, handle } };
        }
        
        // Store verification status
        if (response.verified) {
          control.root.get(`${platform}Verified`)?.setValue(true, { emitEvent: false });
        }
        
        return null;
      }),
      catchError(() => of(null))
    );
  };
}

/*
Usage:
this.form = this.fb.group({
  twitterHandle: ['',
    [],
    [socialHandleValidator(this.http, 'twitter')]
  ],
  twitterVerified: [false]
});
*/

// 5. File Upload Validator (Async)
console.log('\n=== Async File Validator ===');

interface FileValidationResult {
  valid: boolean;
  errors?: string[];
  virusScan?: boolean;
  metadata?: any;
}

export function fileUploadValidator(http: HttpClient): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    const file = control.value as File;
    
    if (!file) {
      return of(null);
    }
    
    // Convert file to base64 or FormData
    const formData = new FormData();
    formData.append('file', file);
    
    return http.post<FileValidationResult>('/api/validate-file', formData).pipe(
      map(result => {
        if (!result.valid) {
          return {
            fileValidation: {
              errors: result.errors || ['File validation failed']
            }
          };
        }
        
        // Store metadata
        if (result.metadata) {
          control.root.get('fileMetadata')?.setValue(result.metadata, { emitEvent: false });
        }
        
        return null;
      }),
      catchError(error => {
        return of({
          fileUploadError: {
            message: 'Error uploading file for validation'
          }
        });
      })
    );
  };
}

/*
Usage:
this.form = this.fb.group({
  file: [null,
    [],
    [fileUploadValidator(this.http)]
  ],
  fileMetadata: [null]
});

onFileSelected(event: Event) {
  const input = event.target as HTMLInputElement;
  if (input.files && input.files[0]) {
    this.form.get('file')?.setValue(input.files[0]);
  }
}

<input type="file" (change)="onFileSelected($event)">

<div *ngIf="form.get('file')?.pending">
  Scanning file...
</div>
*/

// 6. Coordinated Validators (With Retry)
console.log('\n=== Validator with Retry ===');

export function validatorWithRetry(
  http: HttpClient,
  url: string,
  maxRetries: number = 3
): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }
    
    return of(control.value).pipe(
      debounceTime(500),
      switchMap(value => this.validateWithRetry(http, url, value, maxRetries)),
      catchError(() => of({ validationError: true }))
    );
  };
  
  function validateWithRetry(
    http: HttpClient,
    url: string,
    value: any,
    retriesLeft: number
  ): Observable<ValidationErrors | null> {
    return http.post<{ valid: boolean }>(url, { value }).pipe(
      map(response => response.valid ? null : { invalid: true }),
      catchError(error => {
        if (retriesLeft > 0) {
          console.log(`Retrying... (${retriesLeft} attempts left)`);
          return validateWithRetry(http, url, value, retriesLeft - 1);
        }
        throw error;
      })
    );
  }
}

// 7. Conditional Async Validator
console.log('\n=== Conditional Async Validator ===');

export function conditionalAsyncValidator(
  condition: () => boolean,
  validator: AsyncValidatorFn
): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!condition()) {
      return of(null); // Skip validation if condition not met
    }
    
    return validator(control);
  };
}

/*
Usage:
this.form = this.fb.group({
  requiresVerification: [false],
  email: ['',
    [Validators.email],
    [conditionalAsyncValidator(
      () => this.form?.get('requiresVerification')?.value === true,
      emailVerificationValidator(this.http)
    )]
  ]
});
*/
```

### **Performance Comparison**
```typescript
console.log('\n=== Performance ===');

console.log(`
Async Validator Best Practices:
┌──────────────────────┬────────────────────────────────┐
│ Practice             │ Reason                         │
├──────────────────────┼────────────────────────────────┤
│ Debouncing           │ Reduce API calls, wait for     │
│ (debounceTime)       │ user to finish typing          │
├──────────────────────┼────────────────────────────────┤
│ Caching              │ Reuse results for same input   │
│                      │ Improves performance           │
├──────────────────────┼────────────────────────────────┤
│ Error Handling       │ Use catchError to prevent form │
│ (catchError)         │ from being blocked on errors   │
├──────────────────────┼────────────────────────────────┤
│ distinctUntilChanged │ Only validate when value       │
│                      │ actually changes               │
├──────────────────────┼────────────────────────────────┤
│ switchMap            │ Cancel previous requests when  │
│                      │ new value comes in             │
└──────────────────────┴────────────────────────────────┘

Async Validation Flow:
1. User types in input
2. Value changes detected
3. Debounce timer starts
4. After debounce, API call made
5. Control marked as "pending"
6. Response received
7. Validation result applied
8. Control marked as "valid" or "invalid"

RxJS Operators for Validators:
• debounceTime(ms): wait before validating
• distinctUntilChanged(): skip if same value
• switchMap(): cancel previous requests
• map(): transform response to errors
• catchError(): handle failures gracefully
• tap(): side effects (logging, caching)
• retry(n): retry failed requests
• timeout(ms): prevent hanging requests

Performance Optimization:
• Debounce: 300-1000ms depending on operation
• Cache: store results, limit size (100-1000 items)
• Cancel: use switchMap to abort old requests
• Conditional: skip validation when not needed
• Batch: combine multiple checks in one request
• Timeout: fail fast on slow requests
• Offline: handle network errors

Common Patterns:
┌──────────────────────┬────────────────────────────────┐
│ Pattern              │ Use Case                       │
├──────────────────────┼────────────────────────────────┤
│ Username/Email Check │ Uniqueness validation          │
│ Domain Whitelist     │ Corporate email validation     │
│ Password Breach      │ Security (HIBP API)            │
│ Address Validation   │ Shipping address verification  │
│ Promo Code           │ Discount code validation       │
│ Slug/URL             │ SEO-friendly URL checking      │
│ Credit Card BIN      │ Card type detection            │
│ Social Handle        │ Profile linking validation     │
│ File Upload          │ Virus scan, metadata check     │
└──────────────────────┴────────────────────────────────┘

Testing Async Validators:
• Use fakeAsync and tick() for timing
• Mock HTTP responses with HttpTestingController
• Test debounce behavior
• Test error handling
• Test caching logic
• Test pending state
• Verify API calls are made correctly

Error Handling Strategies:
• Return null on error (don't block form)
• Log errors for debugging
• Show generic message to user
• Implement retry logic
• Use timeout to prevent hanging
• Provide fallback validation

Integration Tips:
• Add to FormControl as third parameter
• Can combine with sync validators
• Use at field or group level
• Handle pending state in template
• Show loading indicator
• Disable submit while validating
• Clear errors on value change

Real-World Considerations:
• Rate limiting: respect API limits
• Network failures: handle gracefully
• Privacy: don't leak sensitive data
• UX: clear pending indicators
• Performance: minimize API calls
• Security: validate server-side too
• Accessibility: announce validation status
`);
```

**Interview Tips:**
- Async validator: validates against external source (API, database)
- Returns Observable<ValidationErrors | null>
- Third parameter in FormControl: FormControl('', syncValidators, asyncValidators)
- Use debounceTime to wait after user stops typing (reduce API calls)
- switchMap cancels previous requests when new value comes in
- Control has "pending" state while async validation runs
- Always use catchError to handle failures gracefully (return of(null))
- distinctUntilChanged skips validation if value hasn't changed
- Caching: store results to avoid repeated API calls for same value
- Common use cases: username availability, email uniqueness, password breach check
- Multi-field validators: apply at FormGroup level, access child controls
- Show loading state in template: *ngIf="control.pending"
- Performance: debounce (300-1000ms), cache, cancel old requests
- Error handling: don't block form on network errors
- Security: always validate server-side too (client-side can be bypassed)
- Testing: use fakeAsync, tick(), HttpTestingController for mocking
- RxJS operators: debounceTime, switchMap, map, catchError, tap
- Best practices: debounce, cache, handle errors, show pending state
- Trade-offs: better validation vs increased latency and API load
- Follow-ups: caching strategy, retry logic, timeout, batching, offline handling
- Clarify: API endpoint? debounce time? caching? error handling? retry logic?

</details>

124. Implement a custom RxJS operator

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Custom Operator**
```typescript
/**
 * Simple operator that logs values passing through
 */

import { Observable } from 'rxjs';

// Custom operator using pipe
function debug<T>(tag: string) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      console.log(`[${tag}] Subscribed`);
      
      return source.subscribe({
        next: (value) => {
          console.log(`[${tag}] Next:`, value);
          observer.next(value);
        },
        error: (error) => {
          console.log(`[${tag}] Error:`, error);
          observer.error(error);
        },
        complete: () => {
          console.log(`[${tag}] Complete`);
          observer.complete();
        }
      });
    });
  };
}

// Usage:
/*
import { of } from 'rxjs';

of(1, 2, 3)
  .pipe(
    debug('Stream 1'),
    map(x => x * 2),
    debug('After map')
  )
  .subscribe(value => console.log('Final:', value));

// Output:
// [Stream 1] Subscribed
// [Stream 1] Next: 1
// [After map] Subscribed
// [After map] Next: 2
// Final: 2
// ...
*/

console.log('=== Basic Debug Operator ===');
console.log('Logs all events passing through stream');
console.log('Useful for debugging RxJS pipelines');
```

### **Approach 2: Operator with Parameters**
```typescript
/**
 * Filter by predicate with optional default value
 */

import { Observable } from 'rxjs';

function filterWithDefault<T>(
  predicate: (value: T) => boolean,
  defaultValue: T
) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      let hasEmitted = false;
      
      return source.subscribe({
        next: (value) => {
          if (predicate(value)) {
            hasEmitted = true;
            observer.next(value);
          }
        },
        error: (error) => observer.error(error),
        complete: () => {
          if (!hasEmitted) {
            observer.next(defaultValue);
          }
          observer.complete();
        }
      });
    });
  };
}

// Usage:
/*
import { of } from 'rxjs';

of(1, 2, 3, 4, 5)
  .pipe(
    filterWithDefault(x => x > 10, 0)
  )
  .subscribe(console.log);
// Output: 0 (default, since no values > 10)

of(1, 2, 15, 4, 5)
  .pipe(
    filterWithDefault(x => x > 10, 0)
  )
  .subscribe(console.log);
// Output: 15
*/

console.log('\n=== Filter with Default ===');
console.log('Filters values, emits default if none match');
```

### **Approach 3: Stateful Operator**
```typescript
/**
 * Accumulate values and emit when threshold reached
 */

import { Observable } from 'rxjs';

function bufferByCount<T>(count: number) {
  return (source: Observable<T>) => {
    return new Observable<T[]>(observer => {
      let buffer: T[] = [];
      
      return source.subscribe({
        next: (value) => {
          buffer.push(value);
          
          if (buffer.length >= count) {
            observer.next([...buffer]);
            buffer = [];
          }
        },
        error: (error) => observer.error(error),
        complete: () => {
          if (buffer.length > 0) {
            observer.next(buffer);
          }
          observer.complete();
        }
      });
    });
  };
}

// Usage:
/*
import { interval } from 'rxjs';
import { take } from 'rxjs/operators';

interval(100)
  .pipe(
    take(10),
    bufferByCount(3)
  )
  .subscribe(batch => console.log('Batch:', batch));

// Output:
// Batch: [0, 1, 2]
// Batch: [3, 4, 5]
// Batch: [6, 7, 8]
// Batch: [9]
*/

console.log('\n=== Buffer by Count ===');
console.log('Collects values and emits in batches');
console.log('Stateful operator with internal buffer');
```

### **Approach 4: Operator with Side Effects**
```typescript
/**
 * Tap into stream for side effects at different points
 */

import { Observable } from 'rxjs';

interface TapConfig<T> {
  onNext?: (value: T) => void;
  onError?: (error: any) => void;
  onComplete?: () => void;
  onSubscribe?: () => void;
  onUnsubscribe?: () => void;
}

function tapAdvanced<T>(config: TapConfig<T>) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      config.onSubscribe?.();
      
      const subscription = source.subscribe({
        next: (value) => {
          config.onNext?.(value);
          observer.next(value);
        },
        error: (error) => {
          config.onError?.(error);
          observer.error(error);
        },
        complete: () => {
          config.onComplete?.();
          observer.complete();
        }
      });
      
      return () => {
        config.onUnsubscribe?.();
        subscription.unsubscribe();
      };
    });
  };
}

// Usage:
/*
import { interval } from 'rxjs';
import { take } from 'rxjs/operators';

const subscription = interval(100)
  .pipe(
    take(5),
    tapAdvanced({
      onSubscribe: () => console.log('Subscribed!'),
      onNext: (value) => console.log('Processing:', value),
      onComplete: () => console.log('Stream complete'),
      onUnsubscribe: () => console.log('Unsubscribed')
    })
  )
  .subscribe();

setTimeout(() => subscription.unsubscribe(), 350);
*/

console.log('\n=== Advanced Tap ===');
console.log('Side effects at all lifecycle points');
console.log('Includes subscribe/unsubscribe hooks');
```

### **Approach 5: Transformation Operator**
```typescript
/**
 * Rate limiting operator
 */

import { Observable } from 'rxjs';

function rateLimit<T>(maxEmissions: number, timeWindow: number) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      let emissions: number[] = [];
      
      return source.subscribe({
        next: (value) => {
          const now = Date.now();
          
          // Remove old emissions outside time window
          emissions = emissions.filter(time => now - time < timeWindow);
          
          if (emissions.length < maxEmissions) {
            emissions.push(now);
            observer.next(value);
          } else {
            console.log('Rate limit exceeded, dropping value:', value);
          }
        },
        error: (error) => observer.error(error),
        complete: () => observer.complete()
      });
    });
  };
}

// Usage:
/*
import { interval } from 'rxjs';
import { take } from 'rxjs/operators';

// Allow max 3 emissions per 1000ms
interval(100)
  .pipe(
    take(10),
    rateLimit(3, 1000)
  )
  .subscribe(value => console.log('Emitted:', value));

// Output: Emits first 3 values, then rate-limits remaining
*/

console.log('\n=== Rate Limit Operator ===');
console.log('Limits emissions per time window');
console.log('Drops excess values');
```

### **Real-World Use Cases**
```typescript
/**
 * Practical custom operators
 */

// 1. Retry with Exponential Backoff
console.log('\n=== Retry with Backoff ===');

import { Observable, throwError, timer } from 'rxjs';
import { mergeMap } from 'rxjs/operators';

function retryWithBackoff<T>(
  maxRetries: number,
  initialDelay: number = 1000,
  backoffMultiplier: number = 2
) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      let retryCount = 0;
      
      const attempt = () => {
        source.subscribe({
          next: (value) => observer.next(value),
          error: (error) => {
            if (retryCount >= maxRetries) {
              console.log('Max retries reached');
              observer.error(error);
              return;
            }
            
            const delay = initialDelay * Math.pow(backoffMultiplier, retryCount);
            console.log(`Retry ${retryCount + 1}/${maxRetries} after ${delay}ms`);
            retryCount++;
            
            setTimeout(attempt, delay);
          },
          complete: () => observer.complete()
        });
      };
      
      attempt();
    });
  };
}

/*
Usage:
fetchData()
  .pipe(
    retryWithBackoff(3, 1000, 2)
  )
  .subscribe({
    next: data => console.log('Success:', data),
    error: err => console.log('Failed after retries:', err)
  });

// Retry schedule: 1s, 2s, 4s
*/

// 2. Cache and Replay
console.log('\n=== Cache Operator ===');

function cache<T>(ttl: number = Infinity) {
  return (source: Observable<T>) => {
    let cachedValue: T | undefined;
    let cacheTime: number | undefined;
    let inProgress: Observable<T> | undefined;
    
    return new Observable<T>(observer => {
      const now = Date.now();
      
      // Return cached value if valid
      if (cachedValue !== undefined && 
          cacheTime !== undefined && 
          (ttl === Infinity || now - cacheTime < ttl)) {
        console.log('Returning cached value');
        observer.next(cachedValue);
        observer.complete();
        return;
      }
      
      // Wait for in-progress request
      if (inProgress) {
        console.log('Waiting for in-progress request');
        return inProgress.subscribe(observer);
      }
      
      // Make new request
      console.log('Fetching new value');
      inProgress = new Observable<T>(obs => {
        return source.subscribe({
          next: (value) => {
            cachedValue = value;
            cacheTime = Date.now();
            obs.next(value);
            observer.next(value);
          },
          error: (error) => {
            inProgress = undefined;
            obs.error(error);
            observer.error(error);
          },
          complete: () => {
            inProgress = undefined;
            obs.complete();
            observer.complete();
          }
        });
      });
      
      return inProgress.subscribe(observer);
    });
  };
}

/*
Usage:
const apiCall$ = http.get('/api/data').pipe(
  cache(5000) // Cache for 5 seconds
);

// First call: makes API request
apiCall$.subscribe(data => console.log('Call 1:', data));

// Second call (within 5s): returns cached
setTimeout(() => {
  apiCall$.subscribe(data => console.log('Call 2:', data));
}, 1000);
*/

// 3. Debounce with Immediate
console.log('\n=== Debounce with Immediate ===');

function debounceWithImmediate<T>(duration: number, immediate: boolean = true) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      let timeout: any;
      let isFirstEmission = true;
      
      return source.subscribe({
        next: (value) => {
          const shouldEmitImmediately = immediate && isFirstEmission;
          
          if (shouldEmitImmediately) {
            observer.next(value);
            isFirstEmission = false;
          }
          
          clearTimeout(timeout);
          
          timeout = setTimeout(() => {
            if (!shouldEmitImmediately) {
              observer.next(value);
            }
            isFirstEmission = true;
          }, duration);
        },
        error: (error) => {
          clearTimeout(timeout);
          observer.error(error);
        },
        complete: () => {
          clearTimeout(timeout);
          observer.complete();
        }
      });
    });
  };
}

/*
Usage: Search with immediate first result
searchInput$
  .pipe(
    debounceWithImmediate(500, true)
  )
  .subscribe(term => performSearch(term));

// First keystroke: searches immediately
// Subsequent keystrokes: debounced 500ms
*/

// 4. Conditional Operator
console.log('\n=== Conditional Operator ===');

function switchWhen<T>(
  condition: (value: T) => boolean,
  trueOperator: (source: Observable<T>) => Observable<T>,
  falseOperator?: (source: Observable<T>) => Observable<T>
) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      return source.subscribe({
        next: (value) => {
          const result = condition(value);
          
          const operator = result ? trueOperator : falseOperator;
          
          if (operator) {
            operator(new Observable(obs => {
              obs.next(value);
              obs.complete();
            })).subscribe({
              next: (v) => observer.next(v),
              error: (e) => observer.error(e)
            });
          } else {
            observer.next(value);
          }
        },
        error: (error) => observer.error(error),
        complete: () => observer.complete()
      });
    });
  };
}

/*
Usage:
numbers$
  .pipe(
    switchWhen(
      x => x > 10,
      source => source.pipe(map(x => x * 2)),  // If > 10
      source => source.pipe(map(x => x + 1))   // If <= 10
    )
  )
  .subscribe(console.log);
*/

// 5. Timeout with Fallback
console.log('\n=== Timeout with Fallback ===');

function timeoutWithFallback<T>(
  duration: number,
  fallback: Observable<T>
) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      let completed = false;
      
      const timeout = setTimeout(() => {
        if (!completed) {
          console.log('Timeout! Using fallback');
          fallback.subscribe({
            next: (value) => observer.next(value),
            error: (error) => observer.error(error),
            complete: () => observer.complete()
          });
        }
      }, duration);
      
      const subscription = source.subscribe({
        next: (value) => {
          clearTimeout(timeout);
          completed = true;
          observer.next(value);
        },
        error: (error) => {
          clearTimeout(timeout);
          completed = true;
          observer.error(error);
        },
        complete: () => {
          clearTimeout(timeout);
          completed = true;
          observer.complete();
        }
      });
      
      return () => {
        clearTimeout(timeout);
        subscription.unsubscribe();
      };
    });
  };
}

/*
Usage:
slowApiCall$
  .pipe(
    timeoutWithFallback(3000, of({ cached: true, data: [] }))
  )
  .subscribe(data => console.log(data));

// If API takes > 3s, uses fallback
*/

// 6. Distinct by Property
console.log('\n=== Distinct by Property ===');

function distinctByProperty<T>(propertySelector: (value: T) => any) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      const seen = new Set();
      
      return source.subscribe({
        next: (value) => {
          const key = propertySelector(value);
          
          if (!seen.has(key)) {
            seen.add(key);
            observer.next(value);
          }
        },
        error: (error) => observer.error(error),
        complete: () => observer.complete()
      });
    });
  };
}

/*
Usage:
interface User {
  id: number;
  name: string;
}

users$
  .pipe(
    distinctByProperty(user => user.id)
  )
  .subscribe(user => console.log(user));

// Only emits users with unique IDs
*/

// 7. Batch Requests
console.log('\n=== Batch Requests ===');

function batchRequests<T, R>(
  batchSize: number,
  batchDelay: number,
  requestFn: (batch: T[]) => Observable<R[]>
) {
  return (source: Observable<T>) => {
    return new Observable<R>(observer => {
      let batch: T[] = [];
      let timeout: any;
      
      const processBatch = () => {
        if (batch.length === 0) return;
        
        const currentBatch = [...batch];
        batch = [];
        
        requestFn(currentBatch).subscribe({
          next: (results) => {
            results.forEach(result => observer.next(result));
          },
          error: (error) => observer.error(error)
        });
      };
      
      const subscription = source.subscribe({
        next: (value) => {
          batch.push(value);
          
          if (batch.length >= batchSize) {
            clearTimeout(timeout);
            processBatch();
          } else {
            clearTimeout(timeout);
            timeout = setTimeout(processBatch, batchDelay);
          }
        },
        error: (error) => {
          clearTimeout(timeout);
          processBatch();
          observer.error(error);
        },
        complete: () => {
          clearTimeout(timeout);
          processBatch();
          observer.complete();
        }
      });
      
      return () => {
        clearTimeout(timeout);
        subscription.unsubscribe();
      };
    });
  };
}

/*
Usage:
userIds$
  .pipe(
    batchRequests(
      10,              // Batch size
      500,             // Wait 500ms for more
      (ids) => http.post('/api/users/batch', { ids })
    )
  )
  .subscribe(users => console.log(users));

// Batches user ID requests efficiently
*/
```

### **Performance Comparison**
```typescript
console.log('\n=== Performance ===');

console.log(`
Custom RxJS Operator Patterns:
┌──────────────────────┬────────────────────────────────┐
│ Pattern              │ Use Case                       │
├──────────────────────┼────────────────────────────────┤
│ Transformation       │ Modify values (map, filter)    │
│ Filtering            │ Conditional emission           │
│ Combination          │ Merge multiple streams         │
│ Error Handling       │ Retry, fallback, catch         │
│ Utility              │ Tap, debug, log                │
│ Buffering            │ Collect and batch values       │
│ Time-based           │ Debounce, throttle, delay      │
│ Stateful             │ Remember previous values       │
└──────────────────────┴────────────────────────────────┘

Operator Structure:
function myOperator<T>(params) {
  return (source: Observable<T>) => {
    return new Observable<T>(observer => {
      return source.subscribe({
        next: (value) => observer.next(value),
        error: (error) => observer.error(error),
        complete: () => observer.complete()
      });
    });
  };
}

Key Concepts:
• Operator is a function that returns a function
• Takes source Observable, returns new Observable
• Subscribe to source, forward events to observer
• Can transform, filter, buffer, or delay values
• Must handle next, error, and complete
• Return unsubscribe function for cleanup

Best Practices:
• Always forward errors and completion
• Clean up resources (timers, subscriptions)
• Handle unsubscribe properly
• Don't swallow errors silently
• Make operators pure (no side effects unless intended)
• Use TypeScript generics for type safety
• Test with marble diagrams
• Document behavior clearly

Common Use Cases:
• Debug: log values at specific points
• Retry: with exponential backoff
• Cache: store and reuse results
• Rate limit: control emission rate
• Batch: collect values for bulk operations
• Timeout: with fallback values
• Conditional: different paths based on value
• Distinct: by property or custom logic

Testing Operators:
• Use TestScheduler for marble testing
• Test next, error, complete paths
• Test cleanup/unsubscribe
• Test with multiple subscribers
• Test timing behavior
• Mock dependencies

Performance Tips:
• Minimize subscriptions
• Unsubscribe to prevent leaks
• Use shareReplay for expensive operations
• Batch API requests when possible
• Debounce user input
• Cache frequently accessed data
• Use switchMap to cancel old requests

Real-World Applications:
• API retry with backoff
• Search debouncing with immediate first
• Request caching with TTL
• Rate limiting API calls
• Batching database operations
• Timeout with fallback data
• Conditional transformations
• Distinct by complex keys
`);
```

**Interview Tips:**
- Custom operator: function that takes Observable and returns Observable
- Pattern: (source: Observable<T>) => Observable<R>
- Must subscribe to source and forward to observer
- Handle all three events: next, error, complete
- Return unsubscribe function for cleanup
- Operators are composable: pipe(op1, op2, op3)
- Higher-order function: returns function that returns Observable
- Pure vs impure: pure has no side effects, impure may (like tap)
- Stateful operators: maintain internal state between emissions
- Common patterns: transform, filter, buffer, delay, retry
- Must handle unsubscribe to prevent memory leaks
- Use TypeScript generics for type safety: <T, R>
- Testing: use marble diagrams and TestScheduler
- Real-world examples: retry with backoff, cache with TTL, rate limiting
- Difference from pipeable operator: custom creates new Observable
- Built-in operators: map, filter, debounceTime, switchMap, etc.
- Side effects: use tap or custom operator with onNext callback
- Error handling: always forward errors, don't swallow
- Performance: minimize subscriptions, clean up resources
- Trade-offs: flexibility vs complexity, reusability vs specificity
- Follow-ups: stateful operators, error recovery, cancellation, testing
- Clarify: transformation needed? stateful? error handling? cleanup required?

</details>



<details>
<summary><b>125. Create a reusable form component with ControlValueAccessor</b></summary>

### **Approach 1: Basic ControlValueAccessor**
```typescript
/**
 * Simple custom input component
 */

import { Component, forwardRef } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Component({
  selector: 'app-custom-input',
  template: `
    <div class="custom-input">
      <label>{{ label }}</label>
      <input
        [type]="type"
        [value]="value"
        (input)="onInput($event)"
        (blur)="onTouched()"
        [disabled]="disabled">
    </div>
  `,
  styles: [`
    .custom-input {
      margin: 10px 0;
    }
    label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    input {
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
      width: 100%;
    }
    input:disabled {
      background-color: #f0f0f0;
    }
  `],
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => CustomInputComponent),
    multi: true
  }]
})
export class CustomInputComponent implements ControlValueAccessor {
  @Input() label = '';
  @Input() type = 'text';
  
  value: any = '';
  disabled = false;
  
  onChange: any = () => {};
  onTouched: any = () => {};
  
  // Called by Angular when value changes from outside
  writeValue(value: any): void {
    this.value = value || '';
  }
  
  // Register callback for value changes
  registerOnChange(fn: any): void {
    this.onChange = fn;
  }
  
  // Register callback for touch events
  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
  
  // Called when form control is disabled/enabled
  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
  }
  
  // Handle input event
  onInput(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.value = input.value;
    this.onChange(this.value);
  }
}

// Usage:
/*
// Template:
<form [formGroup]="form">
  <app-custom-input 
    formControlName="username"
    label="Username">
  </app-custom-input>
</form>

// Component:
this.form = this.fb.group({
  username: ['', Validators.required]
});
*/

console.log('=== Basic ControlValueAccessor ===');
console.log('Implements ControlValueAccessor interface');
console.log('Works with formControlName and ngModel');
```

### **Approach 2: Rating Component**
```typescript
/**
 * Star rating component
 */

import { Component, forwardRef, Input } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Component({
  selector: 'app-rating',
  template: `
    <div class="rating">
      <span
        *ngFor="let star of stars; let i = index"
        class="star"
        [class.filled]="i < value"
        [class.disabled]="disabled"
        (click)="rate(i + 1)"
        (mouseenter)="hoverValue = i + 1"
        (mouseleave)="hoverValue = 0">
        {{ (hoverValue > 0 && i < hoverValue) || i < value ? '★' : '☆' }}
      </span>
      <span class="rating-text" *ngIf="showText">
        {{ value }}/{{ maxRating }}
      </span>
    </div>
  `,
  styles: [`
    .rating {
      display: inline-flex;
      align-items: center;
      gap: 5px;
    }
    .star {
      font-size: 24px;
      cursor: pointer;
      color: #ffd700;
      user-select: none;
    }
    .star.disabled {
      cursor: not-allowed;
      opacity: 0.5;
    }
    .rating-text {
      margin-left: 10px;
      font-size: 14px;
      color: #666;
    }
  `],
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => RatingComponent),
    multi: true
  }]
})
export class RatingComponent implements ControlValueAccessor {
  @Input() maxRating = 5;
  @Input() showText = true;
  
  value = 0;
  hoverValue = 0;
  disabled = false;
  stars: number[] = [];
  
  onChange: any = () => {};
  onTouched: any = () => {};
  
  ngOnInit() {
    this.stars = Array(this.maxRating).fill(0);
  }
  
  writeValue(value: number): void {
    this.value = value || 0;
  }
  
  registerOnChange(fn: any): void {
    this.onChange = fn;
  }
  
  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
  
  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
  }
  
  rate(rating: number): void {
    if (this.disabled) return;
    
    this.value = rating;
    this.onChange(this.value);
    this.onTouched();
  }
}

/*
Usage:
<form [formGroup]="form">
  <app-rating 
    formControlName="rating"
    [maxRating]="5"
    [showText]="true">
  </app-rating>
</form>

this.form = this.fb.group({
  rating: [0, [Validators.min(1), Validators.max(5)]]
});
*/

console.log('\n=== Rating Component ===');
console.log('Interactive star rating with hover');
console.log('Fully integrated with Angular forms');
```

### **Approach 3: Tag Input Component**
```typescript
/**
 * Tag/chip input component
 */

import { Component, forwardRef, Input } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Component({
  selector: 'app-tag-input',
  template: `
    <div class="tag-input" [class.disabled]="disabled">
      <div class="tags">
        <span 
          *ngFor="let tag of tags; let i = index" 
          class="tag">
          {{ tag }}
          <button 
            type="button"
            (click)="removeTag(i)"
            [disabled]="disabled">
            ×
          </button>
        </span>
      </div>
      <input
        #input
        type="text"
        [placeholder]="placeholder"
        (keydown.enter)="addTag($event)"
        (keydown.backspace)="onBackspace($event)"
        (blur)="onTouched()"
        [disabled]="disabled">
    </div>
  `,
  styles: [`
    .tag-input {
      display: flex;
      flex-wrap: wrap;
      gap: 5px;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
      min-height: 40px;
    }
    .tag-input.disabled {
      background-color: #f0f0f0;
    }
    .tags {
      display: flex;
      flex-wrap: wrap;
      gap: 5px;
    }
    .tag {
      display: inline-flex;
      align-items: center;
      gap: 5px;
      padding: 4px 8px;
      background-color: #007bff;
      color: white;
      border-radius: 3px;
      font-size: 14px;
    }
    .tag button {
      background: none;
      border: none;
      color: white;
      cursor: pointer;
      font-size: 18px;
      padding: 0;
      line-height: 1;
    }
    input {
      flex: 1;
      border: none;
      outline: none;
      min-width: 100px;
    }
  `],
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => TagInputComponent),
    multi: true
  }]
})
export class TagInputComponent implements ControlValueAccessor {
  @Input() placeholder = 'Add tag...';
  @Input() maxTags?: number;
  
  tags: string[] = [];
  disabled = false;
  
  onChange: any = () => {};
  onTouched: any = () => {};
  
  writeValue(value: string[]): void {
    this.tags = value || [];
  }
  
  registerOnChange(fn: any): void {
    this.onChange = fn;
  }
  
  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
  
  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
  }
  
  addTag(event: Event): void {
    event.preventDefault();
    
    const input = event.target as HTMLInputElement;
    const value = input.value.trim();
    
    if (value && !this.tags.includes(value)) {
      if (!this.maxTags || this.tags.length < this.maxTags) {
        this.tags = [...this.tags, value];
        this.onChange(this.tags);
        input.value = '';
      }
    }
  }
  
  removeTag(index: number): void {
    this.tags = this.tags.filter((_, i) => i !== index);
    this.onChange(this.tags);
  }
  
  onBackspace(event: KeyboardEvent): void {
    const input = event.target as HTMLInputElement;
    
    if (input.value === '' && this.tags.length > 0) {
      this.removeTag(this.tags.length - 1);
    }
  }
}

/*
Usage:
<form [formGroup]="form">
  <app-tag-input 
    formControlName="tags"
    placeholder="Add skills..."
    [maxTags]="10">
  </app-tag-input>
</form>

this.form = this.fb.group({
  tags: [['JavaScript', 'TypeScript']]
});
*/

console.log('\n=== Tag Input Component ===');
console.log('Add/remove tags with keyboard');
console.log('Value is array of strings');
```

### **Approach 4: Date Range Picker**
```typescript
/**
 * Date range component
 */

import { Component, forwardRef } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

interface DateRange {
  start: Date | null;
  end: Date | null;
}

@Component({
  selector: 'app-date-range',
  template: `
    <div class="date-range">
      <div class="date-input">
        <label>Start Date</label>
        <input
          type="date"
          [value]="startDateString"
          (change)="onStartChange($event)"
          [disabled]="disabled">
      </div>
      
      <div class="date-input">
        <label>End Date</label>
        <input
          type="date"
          [value]="endDateString"
          (change)="onEndChange($event)"
          [min]="startDateString"
          [disabled]="disabled">
      </div>
    </div>
    
    <div class="error" *ngIf="hasError">
      End date must be after start date
    </div>
  `,
  styles: [`
    .date-range {
      display: flex;
      gap: 20px;
    }
    .date-input {
      flex: 1;
    }
    .date-input label {
      display: block;
      margin-bottom: 5px;
      font-weight: bold;
    }
    .date-input input {
      width: 100%;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    .error {
      color: red;
      font-size: 12px;
      margin-top: 5px;
    }
  `],
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => DateRangeComponent),
    multi: true
  }]
})
export class DateRangeComponent implements ControlValueAccessor {
  value: DateRange = { start: null, end: null };
  disabled = false;
  hasError = false;
  
  onChange: any = () => {};
  onTouched: any = () => {};
  
  get startDateString(): string {
    return this.value.start ? this.formatDate(this.value.start) : '';
  }
  
  get endDateString(): string {
    return this.value.end ? this.formatDate(this.value.end) : '';
  }
  
  writeValue(value: DateRange): void {
    this.value = value || { start: null, end: null };
  }
  
  registerOnChange(fn: any): void {
    this.onChange = fn;
  }
  
  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
  
  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
  }
  
  onStartChange(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.value.start = input.value ? new Date(input.value) : null;
    this.validate();
    this.onChange(this.value);
    this.onTouched();
  }
  
  onEndChange(event: Event): void {
    const input = event.target as HTMLInputElement;
    this.value.end = input.value ? new Date(input.value) : null;
    this.validate();
    this.onChange(this.value);
    this.onTouched();
  }
  
  validate(): void {
    if (this.value.start && this.value.end) {
      this.hasError = this.value.end < this.value.start;
    } else {
      this.hasError = false;
    }
  }
  
  formatDate(date: Date): string {
    return date.toISOString().split('T')[0];
  }
}

/*
Usage:
<form [formGroup]="form">
  <app-date-range formControlName="dateRange"></app-date-range>
</form>

this.form = this.fb.group({
  dateRange: [{ start: new Date(), end: null }]
});
*/

console.log('\n=== Date Range Component ===');
console.log('Two date pickers with validation');
console.log('Value is {start, end} object');
```

### **Approach 5: Rich Text Editor (Simplified)**
```typescript
/**
 * Simple rich text editor
 */

import { Component, forwardRef, ElementRef, ViewChild } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Component({
  selector: 'app-rich-editor',
  template: `
    <div class="rich-editor">
      <div class="toolbar">
        <button 
          type="button"
          (click)="execCommand('bold')"
          [disabled]="disabled">
          <strong>B</strong>
        </button>
        <button 
          type="button"
          (click)="execCommand('italic')"
          [disabled]="disabled">
          <em>I</em>
        </button>
        <button 
          type="button"
          (click)="execCommand('underline')"
          [disabled]="disabled">
          <u>U</u>
        </button>
        <button 
          type="button"
          (click)="execCommand('insertUnorderedList')"
          [disabled]="disabled">
          List
        </button>
      </div>
      
      <div
        #editor
        class="editor-content"
        contenteditable="true"
        (input)="onInput()"
        (blur)="onTouched()"
        [attr.contenteditable]="!disabled">
      </div>
    </div>
  `,
  styles: [`
    .rich-editor {
      border: 1px solid #ccc;
      border-radius: 4px;
    }
    .toolbar {
      display: flex;
      gap: 5px;
      padding: 8px;
      background-color: #f5f5f5;
      border-bottom: 1px solid #ccc;
    }
    .toolbar button {
      padding: 5px 10px;
      border: 1px solid #ccc;
      background: white;
      cursor: pointer;
      border-radius: 3px;
    }
    .toolbar button:hover {
      background-color: #e0e0e0;
    }
    .toolbar button:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }
    .editor-content {
      min-height: 150px;
      padding: 10px;
      outline: none;
    }
    .editor-content[contenteditable="false"] {
      background-color: #f0f0f0;
    }
  `],
  providers: [{
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => RichEditorComponent),
    multi: true
  }]
})
export class RichEditorComponent implements ControlValueAccessor {
  @ViewChild('editor', { static: true }) editorRef!: ElementRef;
  
  disabled = false;
  
  onChange: any = () => {};
  onTouched: any = () => {};
  
  writeValue(value: string): void {
    if (this.editorRef) {
      this.editorRef.nativeElement.innerHTML = value || '';
    }
  }
  
  registerOnChange(fn: any): void {
    this.onChange = fn;
  }
  
  registerOnTouched(fn: any): void {
    this.onTouched = fn;
  }
  
  setDisabledState(isDisabled: boolean): void {
    this.disabled = isDisabled;
  }
  
  onInput(): void {
    const html = this.editorRef.nativeElement.innerHTML;
    this.onChange(html);
  }
  
  execCommand(command: string): void {
    document.execCommand(command, false);
    this.editorRef.nativeElement.focus();
    this.onInput();
  }
}

/*
Usage:
<form [formGroup]="form">
  <app-rich-editor formControlName="content"></app-rich-editor>
</form>

this.form = this.fb.group({
  content: ['<p>Initial content</p>']
});
*/

console.log('\n=== Rich Text Editor ===');
console.log('Contenteditable with formatting toolbar');
console.log('Value is HTML string');
```

### **Real-World Use Cases**
```typescript
/**
 * Advanced custom form controls
 */

// Complete implementation in separate files would include:

console.log('\n=== Real-World Components ===');

console.log(`
1. Autocomplete Component:
   - Search as you type
   - Debounced API calls
   - Keyboard navigation
   - Highlight matching text
   - Custom templates for options

2. File Upload Component:
   - Drag & drop support
   - Preview thumbnails
   - Progress indicator
   - File validation
   - Multiple files support

3. Color Picker Component:
   - Visual color selection
   - RGB/HSL/HEX formats
   - Predefined palettes
   - Opacity control
   - Recent colors

4. Slider Component:
   - Range selection
   - Step values
   - Custom labels
   - Vertical/horizontal
   - Tooltips with value

5. Phone Input Component:
   - Country code selector
   - Format as you type
   - Validation
   - Flag icons
   - International formats

6. Address Input Component:
   - Google Maps integration
   - Autocomplete suggestions
   - Structured output
   - Validation
   - Custom formatting

7. WYSIWYG Editor Component:
   - Rich text formatting
   - Image upload
   - Tables, links
   - Markdown support
   - Custom plugins
`);
```

### **Performance Comparison**
```typescript
console.log('\n=== Performance ===');

console.log(`
ControlValueAccessor Interface:
┌──────────────────────┬────────────────────────────────┐
│ Method               │ Purpose                        │
├──────────────────────┼────────────────────────────────┤
│ writeValue()         │ Write value from form to view  │
│                      │ Called when form value changes │
├──────────────────────┼────────────────────────────────┤
│ registerOnChange()   │ Register callback for changes  │
│                      │ Call when component value changes│
├──────────────────────┼────────────────────────────────┤
│ registerOnTouched()  │ Register callback for blur     │
│                      │ Call when component is touched │
├──────────────────────┼────────────────────────────────┤
│ setDisabledState()   │ Set disabled state             │
│ (optional)           │ Handle form disable/enable     │
└──────────────────────┴────────────────────────────────┘

Provider Configuration:
providers: [{
  provide: NG_VALUE_ACCESSOR,
  useExisting: forwardRef(() => MyComponent),
  multi: true
}]

• NG_VALUE_ACCESSOR: token for form controls
• useExisting: use component class
• forwardRef: resolve forward references
• multi: allow multiple providers

Best Practices:
• Always implement all 4 methods
• Call onChange when value changes internally
• Call onTouched on blur events
• Handle null/undefined in writeValue
• Don't call onChange in writeValue (loop!)
• Clean up subscriptions in ngOnDestroy
• Use forwardRef for provider
• Type value appropriately (string, object, etc.)
• Support disabled state
• Emit value on every change

Common Patterns:
• Input components: text, number, select
• Complex inputs: date range, tags, rating
• Rich editors: WYSIWYG, markdown
• File uploads: with preview, progress
• Autocomplete: with async search
• Custom selects: with search, multi-select
• Color pickers: visual selection
• Sliders: range, single value

Value Types:
• Primitive: string, number, boolean
• Object: {start: Date, end: Date}
• Array: string[], File[]
• Complex: nested objects, custom types

Integration:
• Template-driven: works with ngModel
• Reactive: works with formControlName
• Validation: combine with validators
• Async validators: supported
• Form array: can be used in arrays
• Form group: can contain other controls

Testing:
• Test writeValue updates view
• Test user interaction calls onChange
• Test onTouched on blur
• Test disabled state
• Test with FormControl
• Test validation integration

Advantages:
• Seamless Angular forms integration
• Reusable across projects
• Consistent API with built-in controls
• Works with validation
• Type-safe with TypeScript
• Encapsulates complex logic

Common Mistakes:
• Calling onChange in writeValue
• Not handling null values
• Forgetting onTouched
• Missing forwardRef in provider
• Not cleaning up resources
• Mutating input values
`);
```

**Interview Tips:**
- ControlValueAccessor: interface to integrate custom components with Angular forms
- Four methods: writeValue, registerOnChange, registerOnTouched, setDisabledState
- writeValue: called when form sets value, updates component view
- registerOnChange: registers callback, call it when component value changes
- registerOnTouched: registers callback, call it on blur (touched)
- setDisabledState: optional, handle disabled state
- Provider: NG_VALUE_ACCESSOR token with multi: true
- Use forwardRef to resolve circular dependency
- Works with both template-driven (ngModel) and reactive (formControlName)
- Don't call onChange inside writeValue (creates loop!)
- Handle null/undefined in writeValue
- Common uses: rating stars, tag input, date range, rich editor, file upload
- Value can be any type: string, object, array, complex
- Integrates with validators: sync and async
- Call onChange on every internal value change
- Call onTouched on blur events
- Best practice: encapsulate complex input logic
- Testing: test all four methods, user interaction, form integration
- Real-world: autocomplete, color picker, phone input, address input
- Trade-offs: reusability vs complexity, flexibility vs simplicity
- Follow-ups: validation, async updates, complex values, accessibility
- Clarify: value type? validation needed? async operations? accessibility?

</details>

### **Performance & Optimization**

126. Implement lazy loading for images

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Intersection Observer API**
```javascript
/**
 * Modern lazy loading with Intersection Observer
 */

class LazyImageLoader {
  constructor(options = {}) {
    this.options = {
      root: null,
      rootMargin: '50px',
      threshold: 0.01,
      ...options
    };
    
    this.observer = null;
    this.init();
  }
  
  init() {
    if (!('IntersectionObserver' in window)) {
      console.warn('IntersectionObserver not supported, loading all images');
      this.loadAllImages();
      return;
    }
    
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersection(entries),
      this.options
    );
    
    this.observeImages();
  }
  
  observeImages() {
    const images = document.querySelectorAll('img[data-src]');
    
    images.forEach(img => {
      this.observer.observe(img);
    });
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadImage(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }
  
  loadImage(img) {
    const src = img.dataset.src;
    const srcset = img.dataset.srcset;
    
    if (!src) return;
    
    // Show loading state
    img.classList.add('loading');
    
    // Create new image to preload
    const tempImg = new Image();
    
    tempImg.onload = () => {
      img.src = src;
      if (srcset) {
        img.srcset = srcset;
      }
      img.classList.remove('loading');
      img.classList.add('loaded');
    };
    
    tempImg.onerror = () => {
      img.classList.remove('loading');
      img.classList.add('error');
      console.error('Failed to load image:', src);
    };
    
    tempImg.src = src;
  }
  
  loadAllImages() {
    const images = document.querySelectorAll('img[data-src]');
    images.forEach(img => this.loadImage(img));
  }
  
  disconnect() {
    if (this.observer) {
      this.observer.disconnect();
    }
  }
}

// Usage:
/*
HTML:
<img 
  data-src="image.jpg" 
  data-srcset="image-480w.jpg 480w, image-800w.jpg 800w"
  alt="Description"
  class="lazy">

CSS:
.lazy {
  background-color: #f0f0f0;
  min-height: 200px;
}

.lazy.loading {
  opacity: 0.5;
}

.lazy.loaded {
  animation: fadeIn 0.3s;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

JavaScript:
const lazyLoader = new LazyImageLoader({
  rootMargin: '100px',  // Load 100px before entering viewport
  threshold: 0.01
});
*/

console.log('=== Intersection Observer Lazy Loading ===');
console.log('Modern, performant approach');
console.log('Loads images as they enter viewport');
```

### **Approach 2: Scroll Event with Debouncing**
```javascript
/**
 * Fallback using scroll events
 */

class ScrollBasedLazyLoader {
  constructor(options = {}) {
    this.options = {
      threshold: 200,  // Load when 200px from viewport
      debounceDelay: 100,
      ...options
    };
    
    this.images = [];
    this.scrollTimeout = null;
    
    this.init();
  }
  
  init() {
    this.images = Array.from(document.querySelectorAll('img[data-src]'));
    
    if (this.images.length === 0) return;
    
    // Initial check
    this.checkImages();
    
    // Listen to scroll events
    window.addEventListener('scroll', this.handleScroll.bind(this), { passive: true });
    window.addEventListener('resize', this.handleScroll.bind(this), { passive: true });
  }
  
  handleScroll() {
    if (this.scrollTimeout) {
      clearTimeout(this.scrollTimeout);
    }
    
    this.scrollTimeout = setTimeout(() => {
      this.checkImages();
    }, this.options.debounceDelay);
  }
  
  checkImages() {
    this.images = this.images.filter(img => {
      if (this.isInViewport(img)) {
        this.loadImage(img);
        return false; // Remove from array
      }
      return true; // Keep in array
    });
    
    // Remove listeners if all images loaded
    if (this.images.length === 0) {
      this.cleanup();
    }
  }
  
  isInViewport(element) {
    const rect = element.getBoundingClientRect();
    const windowHeight = window.innerHeight || document.documentElement.clientHeight;
    
    return (
      rect.top <= windowHeight + this.options.threshold &&
      rect.bottom >= -this.options.threshold
    );
  }
  
  loadImage(img) {
    const src = img.dataset.src;
    
    if (!src) return;
    
    img.classList.add('loading');
    
    const tempImg = new Image();
    
    tempImg.onload = () => {
      img.src = src;
      img.classList.remove('loading');
      img.classList.add('loaded');
    };
    
    tempImg.onerror = () => {
      img.classList.remove('loading');
      img.classList.add('error');
    };
    
    tempImg.src = src;
  }
  
  cleanup() {
    window.removeEventListener('scroll', this.handleScroll.bind(this));
    window.removeEventListener('resize', this.handleScroll.bind(this));
    
    if (this.scrollTimeout) {
      clearTimeout(this.scrollTimeout);
    }
  }
}

// Usage:
/*
const lazyLoader = new ScrollBasedLazyLoader({
  threshold: 300,
  debounceDelay: 150
});
*/

console.log('\n=== Scroll-Based Lazy Loading ===');
console.log('Fallback for older browsers');
console.log('Uses debounced scroll events');
```

### **Approach 3: Progressive Image Loading**
```javascript
/**
 * Load low-quality placeholder first, then high-quality
 */

class ProgressiveImageLoader {
  constructor() {
    this.observer = null;
    this.init();
  }
  
  init() {
    if ('IntersectionObserver' in window) {
      this.observer = new IntersectionObserver(
        (entries) => this.handleIntersection(entries),
        { rootMargin: '50px' }
      );
      
      this.observeImages();
    } else {
      this.loadAllImages();
    }
  }
  
  observeImages() {
    const images = document.querySelectorAll('.progressive-image[data-src]');
    images.forEach(img => this.observer.observe(img));
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadProgressiveImage(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }
  
  loadProgressiveImage(img) {
    const placeholder = img.dataset.placeholder;  // Low-quality image
    const src = img.dataset.src;                  // High-quality image
    
    // Load placeholder first
    if (placeholder) {
      this.loadImage(img, placeholder, 'placeholder-loaded');
    }
    
    // Then load full image
    this.loadImage(img, src, 'full-loaded');
  }
  
  loadImage(img, src, className) {
    const tempImg = new Image();
    
    tempImg.onload = () => {
      img.src = src;
      img.classList.add(className);
    };
    
    tempImg.src = src;
  }
  
  loadAllImages() {
    const images = document.querySelectorAll('.progressive-image[data-src]');
    images.forEach(img => this.loadProgressiveImage(img));
  }
}

// Usage:
/*
HTML:
<img 
  class="progressive-image"
  data-placeholder="image-thumb.jpg"
  data-src="image-full.jpg"
  alt="Description">

CSS:
.progressive-image {
  filter: blur(10px);
  transition: filter 0.3s;
}

.progressive-image.placeholder-loaded {
  filter: blur(5px);
}

.progressive-image.full-loaded {
  filter: blur(0);
}

JavaScript:
const progressiveLoader = new ProgressiveImageLoader();
*/

console.log('\n=== Progressive Image Loading ===');
console.log('Loads low-quality placeholder first');
console.log('Smooth transition to high-quality');
```

### **Approach 4: Background Image Lazy Loading**
```javascript
/**
 * Lazy load background images
 */

class BackgroundLazyLoader {
  constructor(options = {}) {
    this.options = {
      selector: '[data-bg]',
      rootMargin: '50px',
      ...options
    };
    
    this.observer = null;
    this.init();
  }
  
  init() {
    if ('IntersectionObserver' in window) {
      this.observer = new IntersectionObserver(
        (entries) => this.handleIntersection(entries),
        { rootMargin: this.options.rootMargin }
      );
      
      this.observeElements();
    } else {
      this.loadAllBackgrounds();
    }
  }
  
  observeElements() {
    const elements = document.querySelectorAll(this.options.selector);
    elements.forEach(el => this.observer.observe(el));
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadBackground(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }
  
  loadBackground(element) {
    const bg = element.dataset.bg;
    const bgSet = element.dataset.bgSet; // Multiple backgrounds
    
    if (!bg && !bgSet) return;
    
    element.classList.add('bg-loading');
    
    // Preload image
    const img = new Image();
    
    img.onload = () => {
      if (bg) {
        element.style.backgroundImage = `url('${bg}')`;
      }
      
      if (bgSet) {
        // Handle responsive backgrounds
        const sources = bgSet.split(',').map(s => s.trim());
        element.style.backgroundImage = `url('${sources[0].split(' ')[0]}')`;
      }
      
      element.classList.remove('bg-loading');
      element.classList.add('bg-loaded');
    };
    
    img.onerror = () => {
      element.classList.remove('bg-loading');
      element.classList.add('bg-error');
    };
    
    img.src = bg || bgSet.split(',')[0].split(' ')[0];
  }
  
  loadAllBackgrounds() {
    const elements = document.querySelectorAll(this.options.selector);
    elements.forEach(el => this.loadBackground(el));
  }
}

// Usage:
/*
HTML:
<div 
  data-bg="hero-image.jpg"
  data-bg-set="hero-small.jpg 480w, hero-large.jpg 1024w"
  class="hero">
  <h1>Hero Content</h1>
</div>

CSS:
[data-bg] {
  background-size: cover;
  background-position: center;
  background-color: #f0f0f0;
}

[data-bg].bg-loading {
  opacity: 0.5;
}

[data-bg].bg-loaded {
  animation: fadeIn 0.5s;
}

JavaScript:
const bgLoader = new BackgroundLazyLoader({
  selector: '[data-bg]',
  rootMargin: '100px'
});
*/

console.log('\n=== Background Image Lazy Loading ===');
console.log('For CSS background images');
console.log('Supports responsive backgrounds');
```

### **Approach 5: Native Loading Attribute**
```javascript
/**
 * Use native lazy loading with fallback
 */

class NativeLazyLoader {
  constructor() {
    this.init();
  }
  
  init() {
    if (this.supportsNativeLazyLoading()) {
      console.log('Using native lazy loading');
      this.setupNativeLazyLoading();
    } else {
      console.log('Native lazy loading not supported, using fallback');
      this.setupFallback();
    }
  }
  
  supportsNativeLazyLoading() {
    return 'loading' in HTMLImageElement.prototype;
  }
  
  setupNativeLazyLoading() {
    const images = document.querySelectorAll('img[data-src]');
    
    images.forEach(img => {
      img.src = img.dataset.src;
      img.loading = 'lazy';
      
      if (img.dataset.srcset) {
        img.srcset = img.dataset.srcset;
      }
    });
  }
  
  setupFallback() {
    // Use Intersection Observer as fallback
    const observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            const img = entry.target;
            img.src = img.dataset.src;
            
            if (img.dataset.srcset) {
              img.srcset = img.dataset.srcset;
            }
            
            observer.unobserve(img);
          }
        });
      },
      { rootMargin: '50px' }
    );
    
    const images = document.querySelectorAll('img[data-src]');
    images.forEach(img => observer.observe(img));
  }
}

// Usage:
/*
HTML:
<img 
  data-src="image.jpg"
  data-srcset="image-480w.jpg 480w, image-800w.jpg 800w"
  alt="Description"
  loading="lazy">  <!-- Native attribute -->

JavaScript:
const nativeLoader = new NativeLazyLoader();
*/

console.log('\n=== Native Lazy Loading ===');
console.log('Uses loading="lazy" attribute');
console.log('Fallback to Intersection Observer');
```

### **Real-World Use Cases**
```javascript
/**
 * Practical implementations
 */

// 1. Image Gallery with Lazy Loading
console.log('\n=== Image Gallery ===');

class LazyGallery {
  constructor(containerSelector) {
    this.container = document.querySelector(containerSelector);
    this.images = [];
    this.observer = null;
    
    this.init();
  }
  
  init() {
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersection(entries),
      { rootMargin: '100px' }
    );
    
    this.loadInitialImages();
  }
  
  loadInitialImages() {
    // Load first few images immediately
    const allImages = this.container.querySelectorAll('img[data-src]');
    const visibleCount = 6;
    
    allImages.forEach((img, index) => {
      if (index < visibleCount) {
        this.loadImage(img);
      } else {
        this.observer.observe(img);
      }
    });
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadImage(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }
  
  loadImage(img) {
    const src = img.dataset.src;
    const thumb = img.dataset.thumb;
    
    // Load thumbnail first
    if (thumb) {
      img.src = thumb;
      img.classList.add('thumb-loaded');
    }
    
    // Then load full image
    const fullImg = new Image();
    
    fullImg.onload = () => {
      img.src = src;
      img.classList.remove('thumb-loaded');
      img.classList.add('full-loaded');
    };
    
    fullImg.src = src;
  }
  
  addImage(imageSrc, thumbSrc) {
    const img = document.createElement('img');
    img.dataset.src = imageSrc;
    img.dataset.thumb = thumbSrc;
    img.alt = 'Gallery image';
    
    this.container.appendChild(img);
    this.observer.observe(img);
  }
}

/*
Usage:
<div id="gallery" class="image-gallery">
  <img data-src="img1-full.jpg" data-thumb="img1-thumb.jpg">
  <img data-src="img2-full.jpg" data-thumb="img2-thumb.jpg">
  <!-- More images -->
</div>

const gallery = new LazyGallery('#gallery');
*/

// 2. Infinite Scroll with Lazy Loading
console.log('\n=== Infinite Scroll ===');

class InfiniteScrollGallery {
  constructor(containerSelector, apiEndpoint) {
    this.container = document.querySelector(containerSelector);
    this.apiEndpoint = apiEndpoint;
    this.page = 1;
    this.loading = false;
    this.hasMore = true;
    
    this.observer = null;
    this.sentinel = null;
    
    this.init();
  }
  
  init() {
    this.createSentinel();
    this.setupObserver();
    this.loadMore();
  }
  
  createSentinel() {
    this.sentinel = document.createElement('div');
    this.sentinel.className = 'sentinel';
    this.container.appendChild(this.sentinel);
  }
  
  setupObserver() {
    this.observer = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting && !this.loading && this.hasMore) {
            this.loadMore();
          }
        });
      },
      { rootMargin: '200px' }
    );
    
    this.observer.observe(this.sentinel);
  }
  
  async loadMore() {
    this.loading = true;
    
    try {
      const response = await fetch(`${this.apiEndpoint}?page=${this.page}`);
      const data = await response.json();
      
      if (data.images.length === 0) {
        this.hasMore = false;
        return;
      }
      
      this.renderImages(data.images);
      this.page++;
    } catch (error) {
      console.error('Failed to load images:', error);
    } finally {
      this.loading = false;
    }
  }
  
  renderImages(images) {
    images.forEach(imageData => {
      const img = document.createElement('img');
      img.dataset.src = imageData.url;
      img.alt = imageData.alt || '';
      img.loading = 'lazy';
      
      this.container.insertBefore(img, this.sentinel);
      
      // Trigger lazy loading
      img.src = imageData.url;
    });
  }
}

/*
Usage:
<div id="infinite-gallery"></div>

const infiniteGallery = new InfiniteScrollGallery(
  '#infinite-gallery',
  '/api/images'
);
*/

// 3. Responsive Image Lazy Loading
console.log('\n=== Responsive Images ===');

class ResponsiveLazyLoader {
  constructor() {
    this.observer = null;
    this.init();
  }
  
  init() {
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersection(entries),
      { rootMargin: '50px' }
    );
    
    this.observeImages();
  }
  
  observeImages() {
    const images = document.querySelectorAll('img[data-src]');
    images.forEach(img => this.observer.observe(img));
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadResponsiveImage(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }
  
  loadResponsiveImage(img) {
    const src = img.dataset.src;
    const srcset = img.dataset.srcset;
    const sizes = img.dataset.sizes;
    
    // Determine best image based on viewport
    const selectedSrc = this.selectBestSource(srcset, sizes);
    
    const tempImg = new Image();
    
    tempImg.onload = () => {
      img.src = selectedSrc || src;
      
      if (srcset) {
        img.srcset = srcset;
      }
      
      if (sizes) {
        img.sizes = sizes;
      }
      
      img.classList.add('loaded');
    };
    
    tempImg.src = selectedSrc || src;
  }
  
  selectBestSource(srcset, sizes) {
    if (!srcset) return null;
    
    const sources = srcset.split(',').map(s => {
      const [url, descriptor] = s.trim().split(' ');
      const width = parseInt(descriptor);
      return { url, width };
    });
    
    const viewportWidth = window.innerWidth;
    const dpr = window.devicePixelRatio || 1;
    const targetWidth = viewportWidth * dpr;
    
    // Find closest match
    const sorted = sources.sort((a, b) => a.width - b.width);
    const selected = sorted.find(s => s.width >= targetWidth) || sorted[sorted.length - 1];
    
    return selected.url;
  }
}

/*
Usage:
<img 
  data-src="image.jpg"
  data-srcset="image-320w.jpg 320w, image-640w.jpg 640w, image-1024w.jpg 1024w"
  data-sizes="(max-width: 600px) 320px, (max-width: 1024px) 640px, 1024px"
  alt="Responsive image">

const responsiveLoader = new ResponsiveLazyLoader();
*/

// 4. Lazy Loading with Retry
console.log('\n=== With Retry Logic ===');

class LazyLoaderWithRetry {
  constructor(maxRetries = 3) {
    this.maxRetries = maxRetries;
    this.observer = null;
    this.init();
  }
  
  init() {
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersection(entries),
      { rootMargin: '50px' }
    );
    
    this.observeImages();
  }
  
  observeImages() {
    const images = document.querySelectorAll('img[data-src]');
    images.forEach(img => {
      img.dataset.retries = '0';
      this.observer.observe(img);
    });
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadImageWithRetry(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }
  
  loadImageWithRetry(img, retryCount = 0) {
    const src = img.dataset.src;
    
    img.classList.add('loading');
    
    const tempImg = new Image();
    
    tempImg.onload = () => {
      img.src = src;
      img.classList.remove('loading');
      img.classList.add('loaded');
    };
    
    tempImg.onerror = () => {
      if (retryCount < this.maxRetries) {
        console.log(`Retry ${retryCount + 1}/${this.maxRetries} for:`, src);
        
        setTimeout(() => {
          this.loadImageWithRetry(img, retryCount + 1);
        }, Math.pow(2, retryCount) * 1000); // Exponential backoff
      } else {
        console.error('Failed to load after retries:', src);
        img.classList.remove('loading');
        img.classList.add('error');
        
        // Show fallback image
        img.src = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" width="100" height="100"><rect fill="%23ddd"/><text x="50%" y="50%" text-anchor="middle">Error</text></svg>';
      }
    };
    
    tempImg.src = src;
  }
}

/*
Usage:
const lazyLoader = new LazyLoaderWithRetry(3);
*/

// 5. Performance Monitoring
console.log('\n=== With Performance Monitoring ===');

class MonitoredLazyLoader {
  constructor() {
    this.metrics = {
      totalImages: 0,
      loadedImages: 0,
      failedImages: 0,
      totalLoadTime: 0,
      averageLoadTime: 0
    };
    
    this.observer = null;
    this.init();
  }
  
  init() {
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersection(entries),
      { rootMargin: '50px' }
    );
    
    const images = document.querySelectorAll('img[data-src]');
    this.metrics.totalImages = images.length;
    
    images.forEach(img => this.observer.observe(img));
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadImage(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }
  
  loadImage(img) {
    const src = img.dataset.src;
    const startTime = performance.now();
    
    const tempImg = new Image();
    
    tempImg.onload = () => {
      const loadTime = performance.now() - startTime;
      
      img.src = src;
      img.classList.add('loaded');
      
      this.metrics.loadedImages++;
      this.metrics.totalLoadTime += loadTime;
      this.metrics.averageLoadTime = 
        this.metrics.totalLoadTime / this.metrics.loadedImages;
      
      this.logMetrics();
    };
    
    tempImg.onerror = () => {
      this.metrics.failedImages++;
      img.classList.add('error');
      
      this.logMetrics();
    };
    
    tempImg.src = src;
  }
  
  logMetrics() {
    console.log('Lazy Loading Metrics:', {
      total: this.metrics.totalImages,
      loaded: this.metrics.loadedImages,
      failed: this.metrics.failedImages,
      avgLoadTime: `${this.metrics.averageLoadTime.toFixed(2)}ms`,
      progress: `${((this.metrics.loadedImages / this.metrics.totalImages) * 100).toFixed(1)}%`
    });
  }
  
  getMetrics() {
    return { ...this.metrics };
  }
}

/*
Usage:
const monitoredLoader = new MonitoredLazyLoader();

// Get metrics
setTimeout(() => {
  const metrics = monitoredLoader.getMetrics();
  console.log('Final metrics:', metrics);
}, 5000);
*/
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Lazy Loading Strategies:
┌──────────────────────┬────────────────────────────────┐
│ Strategy             │ Use Case                       │
├──────────────────────┼────────────────────────────────┤
│ Intersection Observer│ Modern, performant, recommended│
│                      │ No scroll event listeners      │
├──────────────────────┼────────────────────────────────┤
│ Scroll Events        │ Fallback for old browsers      │
│                      │ Requires debouncing            │
├──────────────────────┼────────────────────────────────┤
│ Native loading="lazy"│ Simplest, browser-native       │
│                      │ Limited browser support        │
├──────────────────────┼────────────────────────────────┤
│ Progressive Loading  │ Better UX, load placeholder    │
│                      │ Then full image                │
└──────────────────────┴────────────────────────────────┘

Benefits:
• Faster initial page load
• Reduced bandwidth usage
• Better Core Web Vitals (LCP, CLS)
• Improved mobile experience
• Less memory consumption
• Reduced server load

Best Practices:
• Set width/height to prevent layout shift
• Use placeholder or blur effect
• Load images slightly before viewport (rootMargin)
• Provide alt text for accessibility
• Add fallback for unsupported browsers
• Handle loading states visually
• Include retry logic for failures
• Monitor performance metrics

Optimization Tips:
• Use srcset for responsive images
• Compress images appropriately
• Use modern formats (WebP, AVIF)
• Implement progressive loading
• Cache loaded images
• Prioritize above-the-fold images
• Lazy load background images too
• Consider image CDN

Performance Metrics:
• First Contentful Paint (FCP)
• Largest Contentful Paint (LCP)
• Cumulative Layout Shift (CLS)
• Total Blocking Time (TBT)
• Bandwidth saved
• Load time reduction

Common Pitfalls:
• Not setting image dimensions (CLS)
• Loading too aggressively (rootMargin too large)
• Not handling errors
• Forgetting SEO considerations
• Breaking browser back button
• Ignoring accessibility

Browser Support:
• Intersection Observer: 95%+
• Native lazy loading: 75%+
• Scroll events: 100%
• Progressive enhancement recommended

Real-World Impact:
• 50-70% reduction in initial page weight
• 2-3x faster initial load time
• Improved mobile data usage
• Better user experience
• Higher conversion rates
• Lower bounce rates
`);
```

**Interview Tips:**
- Lazy loading: defer loading images until needed (viewport proximity)
- Intersection Observer: modern API, monitors element visibility, no scroll listeners
- Native loading="lazy": browser-native, simplest but limited support
- Scroll events: fallback for older browsers, requires debouncing
- Progressive loading: show low-quality placeholder, then high-quality
- Benefits: faster initial load, reduced bandwidth, better Core Web Vitals
- rootMargin: load images before entering viewport (e.g., '50px')
- Prevent layout shift: set width/height attributes on images
- data-src attribute: stores actual image URL
- Preload with new Image(): ensures image loads before displaying
- IntersectionObserver callback: called when element intersects viewport
- unobserve(): stop watching element after loading
- Handle errors: show fallback image, implement retry logic
- Responsive images: use srcset and sizes attributes
- Background images: use data-bg attribute, similar technique
- Performance: significant reduction in initial page weight (50-70%)
- SEO: search engines can handle lazy loading with proper implementation
- Accessibility: always include alt text, ensure keyboard navigation works
- Real-world: image galleries, infinite scroll, product listings
- Trade-offs: complexity vs performance gain, UX vs implementation
- Follow-ups: SEO impact, CLS prevention, error handling, retry logic
- Clarify: browser support needed? progressive loading? responsive images?

</details>

127. Create a web worker for heavy computations

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Web Worker**
```javascript
/**
 * Simple worker for computation
 */

// Main thread (main.js)
class WorkerManager {
  constructor(workerScript) {
    this.worker = new Worker(workerScript);
    this.setupListeners();
  }
  
  setupListeners() {
    this.worker.onmessage = (event) => {
      console.log('Result from worker:', event.data);
    };
    
    this.worker.onerror = (error) => {
      console.error('Worker error:', error.message);
    };
  }
  
  compute(data) {
    this.worker.postMessage(data);
  }
  
  terminate() {
    this.worker.terminate();
  }
}

// Worker file (worker.js)
/*
self.onmessage = function(event) {
  const data = event.data;
  
  // Heavy computation
  const result = performComputation(data);
  
  // Send result back
  self.postMessage(result);
};

function performComputation(data) {
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += Math.sqrt(i) * data;
  }
  return result;
}
*/

// Usage:
/*
const manager = new WorkerManager('worker.js');
manager.compute(42);
*/

console.log('=== Basic Web Worker ===');
console.log('Offloads heavy computation to separate thread');
console.log('Prevents UI blocking');
```

### **Approach 2: Promise-Based Worker**
```javascript
/**
 * Worker with Promise interface
 */

class PromiseWorker {
  constructor(workerScript) {
    this.worker = new Worker(workerScript);
    this.taskId = 0;
    this.pendingTasks = new Map();
    
    this.worker.onmessage = (event) => {
      const { id, result, error } = event.data;
      
      const task = this.pendingTasks.get(id);
      
      if (task) {
        if (error) {
          task.reject(new Error(error));
        } else {
          task.resolve(result);
        }
        
        this.pendingTasks.delete(id);
      }
    };
    
    this.worker.onerror = (error) => {
      console.error('Worker error:', error);
    };
  }
  
  execute(action, data) {
    return new Promise((resolve, reject) => {
      const id = this.taskId++;
      
      this.pendingTasks.set(id, { resolve, reject });
      
      this.worker.postMessage({ id, action, data });
    });
  }
  
  terminate() {
    this.pendingTasks.forEach(task => {
      task.reject(new Error('Worker terminated'));
    });
    
    this.pendingTasks.clear();
    this.worker.terminate();
  }
}

// Worker file (promise-worker.js)
/*
self.onmessage = function(event) {
  const { id, action, data } = event.data;
  
  try {
    let result;
    
    switch (action) {
      case 'fibonacci':
        result = fibonacci(data);
        break;
      case 'factorial':
        result = factorial(data);
        break;
      case 'sort':
        result = data.sort((a, b) => a - b);
        break;
      default:
        throw new Error('Unknown action: ' + action);
    }
    
    self.postMessage({ id, result });
  } catch (error) {
    self.postMessage({ id, error: error.message });
  }
};

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}
*/

// Usage:
/*
const worker = new PromiseWorker('promise-worker.js');

worker.execute('fibonacci', 40)
  .then(result => console.log('Fibonacci:', result))
  .catch(error => console.error('Error:', error));

worker.execute('factorial', 20)
  .then(result => console.log('Factorial:', result));
*/

console.log('\n=== Promise-Based Worker ===');
console.log('Returns promises for async operations');
console.log('Supports multiple concurrent tasks');
```

### **Approach 3: Worker Pool**
```javascript
/**
 * Pool of workers for parallel processing
 */

class WorkerPool {
  constructor(workerScript, poolSize = navigator.hardwareConcurrency || 4) {
    this.workerScript = workerScript;
    this.poolSize = poolSize;
    this.workers = [];
    this.taskQueue = [];
    this.busyWorkers = new Set();
    
    this.init();
  }
  
  init() {
    for (let i = 0; i < this.poolSize; i++) {
      const worker = new Worker(this.workerScript);
      worker.id = i;
      
      worker.onmessage = (event) => {
        this.handleWorkerMessage(worker, event);
      };
      
      worker.onerror = (error) => {
        console.error(`Worker ${worker.id} error:`, error);
      };
      
      this.workers.push(worker);
    }
  }
  
  execute(data) {
    return new Promise((resolve, reject) => {
      const task = { data, resolve, reject };
      
      const availableWorker = this.getAvailableWorker();
      
      if (availableWorker) {
        this.assignTask(availableWorker, task);
      } else {
        this.taskQueue.push(task);
      }
    });
  }
  
  getAvailableWorker() {
    return this.workers.find(w => !this.busyWorkers.has(w.id));
  }
  
  assignTask(worker, task) {
    this.busyWorkers.add(worker.id);
    
    worker.currentTask = task;
    worker.postMessage(task.data);
  }
  
  handleWorkerMessage(worker, event) {
    const task = worker.currentTask;
    
    if (task) {
      task.resolve(event.data);
      worker.currentTask = null;
    }
    
    this.busyWorkers.delete(worker.id);
    
    // Process next task if any
    if (this.taskQueue.length > 0) {
      const nextTask = this.taskQueue.shift();
      this.assignTask(worker, nextTask);
    }
  }
  
  terminate() {
    this.workers.forEach(worker => worker.terminate());
    this.workers = [];
    this.taskQueue = [];
    this.busyWorkers.clear();
  }
  
  getStatus() {
    return {
      poolSize: this.poolSize,
      busyWorkers: this.busyWorkers.size,
      queuedTasks: this.taskQueue.length,
      availableWorkers: this.poolSize - this.busyWorkers.size
    };
  }
}

// Worker file (pool-worker.js)
/*
self.onmessage = function(event) {
  const data = event.data;
  
  // Simulate heavy computation
  const result = heavyComputation(data);
  
  self.postMessage(result);
};

function heavyComputation(data) {
  // Heavy processing
  let result = 0;
  for (let i = 0; i < 10000000; i++) {
    result += Math.sqrt(i) * data;
  }
  return result;
}
*/

// Usage:
/*
const pool = new WorkerPool('pool-worker.js', 4);

// Execute multiple tasks in parallel
const tasks = Array.from({ length: 10 }, (_, i) => i + 1);

Promise.all(tasks.map(task => pool.execute(task)))
  .then(results => {
    console.log('All results:', results);
    console.log('Pool status:', pool.getStatus());
  });
*/

console.log('\n=== Worker Pool ===');
console.log('Manages multiple workers for parallel tasks');
console.log('Automatically distributes work');
```

### **Approach 4: Inline Worker**
```javascript
/**
 * Create worker from inline code (no separate file)
 */

class InlineWorker {
  constructor(workerFunction) {
    const code = `(${workerFunction.toString()})()`;
    const blob = new Blob([code], { type: 'application/javascript' });
    const workerUrl = URL.createObjectURL(blob);
    
    this.worker = new Worker(workerUrl);
    this.workerUrl = workerUrl;
  }
  
  postMessage(data) {
    this.worker.postMessage(data);
  }
  
  onMessage(callback) {
    this.worker.onmessage = (event) => callback(event.data);
  }
  
  onError(callback) {
    this.worker.onerror = (error) => callback(error);
  }
  
  terminate() {
    this.worker.terminate();
    URL.revokeObjectURL(this.workerUrl);
  }
}

// Usage:
/*
const worker = new InlineWorker(function() {
  self.onmessage = function(event) {
    const { numbers } = event.data;
    
    // Sort large array
    const sorted = numbers.sort((a, b) => a - b);
    
    self.postMessage({ sorted });
  };
});

worker.onMessage((data) => {
  console.log('Sorted:', data.sorted);
});

worker.onError((error) => {
  console.error('Error:', error);
});

// Send data to worker
const largeArray = Array.from({ length: 1000000 }, () => Math.random());
worker.postMessage({ numbers: largeArray });
*/

console.log('\n=== Inline Worker ===');
console.log('Creates worker from inline code');
console.log('No separate file needed');
```

### **Approach 5: Transferable Objects**
```javascript
/**
 * Use transferable objects for better performance
 */

class TransferableWorker {
  constructor(workerScript) {
    this.worker = new Worker(workerScript);
    this.setupListeners();
  }
  
  setupListeners() {
    this.worker.onmessage = (event) => {
      console.log('Result received:', event.data);
    };
  }
  
  processArrayBuffer(buffer) {
    // Transfer ownership of ArrayBuffer to worker
    // Main thread can no longer access it
    this.worker.postMessage({ buffer }, [buffer]);
    
    console.log('Buffer transferred, main thread cannot access it anymore');
  }
  
  processImageData(imageData) {
    // Transfer ImageData
    this.worker.postMessage({ imageData }, [imageData.data.buffer]);
  }
  
  terminate() {
    this.worker.terminate();
  }
}

// Worker file (transferable-worker.js)
/*
self.onmessage = function(event) {
  const { buffer, imageData } = event.data;
  
  if (buffer) {
    // Process ArrayBuffer
    const view = new Uint8Array(buffer);
    
    // Modify data
    for (let i = 0; i < view.length; i++) {
      view[i] = view[i] * 2;
    }
    
    // Transfer back
    self.postMessage({ buffer }, [buffer]);
  }
  
  if (imageData) {
    // Process image data
    const data = imageData.data;
    
    // Invert colors
    for (let i = 0; i < data.length; i += 4) {
      data[i] = 255 - data[i];       // R
      data[i + 1] = 255 - data[i + 1]; // G
      data[i + 2] = 255 - data[i + 2]; // B
      // data[i + 3] is alpha, keep as is
    }
    
    self.postMessage({ imageData }, [data.buffer]);
  }
};
*/

// Usage:
/*
const worker = new TransferableWorker('transferable-worker.js');

// Create large ArrayBuffer
const buffer = new ArrayBuffer(1024 * 1024 * 10); // 10MB
const view = new Uint8Array(buffer);
view.fill(42);

// Transfer to worker (zero-copy)
worker.processArrayBuffer(buffer);

// buffer is now unusable in main thread
console.log('Buffer byteLength:', buffer.byteLength); // 0
*/

console.log('\n=== Transferable Objects ===');
console.log('Zero-copy transfer of ArrayBuffers');
console.log('Much faster for large data');
```

### **Real-World Use Cases**
```javascript
/**
 * Practical applications
 */

// 1. Image Processing
console.log('\n=== Image Processing ===');

class ImageProcessor {
  constructor() {
    this.worker = new InlineWorker(function() {
      self.onmessage = function(event) {
        const { imageData, filter } = event.data;
        const data = imageData.data;
        
        switch (filter) {
          case 'grayscale':
            for (let i = 0; i < data.length; i += 4) {
              const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
              data[i] = data[i + 1] = data[i + 2] = avg;
            }
            break;
            
          case 'sepia':
            for (let i = 0; i < data.length; i += 4) {
              const r = data[i];
              const g = data[i + 1];
              const b = data[i + 2];
              
              data[i] = Math.min(255, r * 0.393 + g * 0.769 + b * 0.189);
              data[i + 1] = Math.min(255, r * 0.349 + g * 0.686 + b * 0.168);
              data[i + 2] = Math.min(255, r * 0.272 + g * 0.534 + b * 0.131);
            }
            break;
            
          case 'invert':
            for (let i = 0; i < data.length; i += 4) {
              data[i] = 255 - data[i];
              data[i + 1] = 255 - data[i + 1];
              data[i + 2] = 255 - data[i + 2];
            }
            break;
        }
        
        self.postMessage({ imageData }, [imageData.data.buffer]);
      };
    });
  }
  
  applyFilter(canvas, filter) {
    return new Promise((resolve) => {
      const ctx = canvas.getContext('2d');
      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      
      this.worker.onMessage((data) => {
        ctx.putImageData(data.imageData, 0, 0);
        resolve();
      });
      
      this.worker.postMessage({ imageData, filter }, [imageData.data.buffer]);
    });
  }
}

/*
Usage:
const processor = new ImageProcessor();
const canvas = document.getElementById('myCanvas');

processor.applyFilter(canvas, 'grayscale')
  .then(() => console.log('Filter applied'));
*/

// 2. Data Processing/Sorting
console.log('\n=== Large Dataset Processing ===');

class DataProcessor {
  constructor() {
    this.pool = new WorkerPool('data-worker.js', 4);
  }
  
  async sortLargeDataset(data, chunkSize = 10000) {
    // Split data into chunks
    const chunks = [];
    for (let i = 0; i < data.length; i += chunkSize) {
      chunks.push(data.slice(i, i + chunkSize));
    }
    
    // Sort chunks in parallel
    const sortedChunks = await Promise.all(
      chunks.map(chunk => this.pool.execute({ action: 'sort', data: chunk }))
    );
    
    // Merge sorted chunks (in main thread for simplicity)
    return this.mergeSortedChunks(sortedChunks);
  }
  
  mergeSortedChunks(chunks) {
    // Simple merge sort of sorted arrays
    while (chunks.length > 1) {
      const merged = [];
      
      for (let i = 0; i < chunks.length; i += 2) {
        if (i + 1 < chunks.length) {
          merged.push(this.mergeTwoArrays(chunks[i], chunks[i + 1]));
        } else {
          merged.push(chunks[i]);
        }
      }
      
      chunks = merged;
    }
    
    return chunks[0];
  }
  
  mergeTwoArrays(arr1, arr2) {
    const result = [];
    let i = 0, j = 0;
    
    while (i < arr1.length && j < arr2.length) {
      if (arr1[i] < arr2[j]) {
        result.push(arr1[i++]);
      } else {
        result.push(arr2[j++]);
      }
    }
    
    return result.concat(arr1.slice(i), arr2.slice(j));
  }
}

/*
Usage:
const processor = new DataProcessor();
const largeArray = Array.from({ length: 1000000 }, () => Math.random());

processor.sortLargeDataset(largeArray)
  .then(sorted => console.log('Sorted', sorted.length, 'items'));
*/

// 3. Cryptography
console.log('\n=== Cryptographic Operations ===');

class CryptoWorker {
  constructor() {
    this.worker = new PromiseWorker('crypto-worker.js');
  }
  
  async hashPassword(password) {
    return this.worker.execute('hash', { password });
  }
  
  async encrypt(text, key) {
    return this.worker.execute('encrypt', { text, key });
  }
  
  async decrypt(encrypted, key) {
    return this.worker.execute('decrypt', { encrypted, key });
  }
}

/*
Worker file (crypto-worker.js):
self.onmessage = async function(event) {
  const { id, action, data } = event.data;
  
  try {
    let result;
    
    switch (action) {
      case 'hash':
        result = await hashPassword(data.password);
        break;
      case 'encrypt':
        result = await encrypt(data.text, data.key);
        break;
      case 'decrypt':
        result = await decrypt(data.encrypted, data.key);
        break;
    }
    
    self.postMessage({ id, result });
  } catch (error) {
    self.postMessage({ id, error: error.message });
  }
};

async function hashPassword(password) {
  const encoder = new TextEncoder();
  const data = encoder.encode(password);
  const hash = await crypto.subtle.digest('SHA-256', data);
  return Array.from(new Uint8Array(hash))
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
}

Usage:
const cryptoWorker = new CryptoWorker();

cryptoWorker.hashPassword('mypassword')
  .then(hash => console.log('Hash:', hash));
*/

// 4. Real-time Analytics
console.log('\n=== Real-time Analytics ===');

class AnalyticsWorker {
  constructor() {
    this.worker = new InlineWorker(function() {
      let dataPoints = [];
      
      self.onmessage = function(event) {
        const { action, data } = event.data;
        
        switch (action) {
          case 'add':
            dataPoints.push(data);
            break;
            
          case 'calculate':
            const stats = {
              count: dataPoints.length,
              sum: dataPoints.reduce((a, b) => a + b, 0),
              avg: dataPoints.reduce((a, b) => a + b, 0) / dataPoints.length,
              min: Math.min(...dataPoints),
              max: Math.max(...dataPoints)
            };
            
            self.postMessage({ stats });
            break;
            
          case 'clear':
            dataPoints = [];
            break;
        }
      };
    });
  }
  
  addDataPoint(value) {
    this.worker.postMessage({ action: 'add', data: value });
  }
  
  calculateStats() {
    return new Promise((resolve) => {
      this.worker.onMessage((data) => {
        if (data.stats) {
          resolve(data.stats);
        }
      });
      
      this.worker.postMessage({ action: 'calculate' });
    });
  }
  
  clear() {
    this.worker.postMessage({ action: 'clear' });
  }
}

/*
Usage:
const analytics = new AnalyticsWorker();

// Add data points
for (let i = 0; i < 10000; i++) {
  analytics.addDataPoint(Math.random() * 100);
}

// Calculate stats
analytics.calculateStats()
  .then(stats => console.log('Statistics:', stats));
*/

// 5. Progress Reporting
console.log('\n=== With Progress Reporting ===');

class ProgressWorker {
  constructor(workerScript) {
    this.worker = new Worker(workerScript);
    this.onProgress = null;
    
    this.worker.onmessage = (event) => {
      const { type, progress, result } = event.data;
      
      if (type === 'progress' && this.onProgress) {
        this.onProgress(progress);
      } else if (type === 'complete' && this.onComplete) {
        this.onComplete(result);
      }
    };
  }
  
  execute(data, onProgress, onComplete) {
    this.onProgress = onProgress;
    this.onComplete = onComplete;
    
    this.worker.postMessage(data);
  }
}

/*
Worker file (progress-worker.js):
self.onmessage = function(event) {
  const data = event.data;
  const totalSteps = 100;
  
  for (let i = 0; i <= totalSteps; i++) {
    // Heavy computation
    performStep(i, data);
    
    // Report progress
    if (i % 10 === 0) {
      self.postMessage({
        type: 'progress',
        progress: (i / totalSteps) * 100
      });
    }
  }
  
  self.postMessage({
    type: 'complete',
    result: 'Done!'
  });
};

Usage:
const worker = new ProgressWorker('progress-worker.js');

worker.execute(
  { data: 'something' },
  (progress) => console.log(`Progress: ${progress}%`),
  (result) => console.log('Complete:', result)
);
*/
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Web Worker Concepts:
┌──────────────────────┬────────────────────────────────┐
│ Concept              │ Description                    │
├──────────────────────┼────────────────────────────────┤
│ Separate Thread      │ Runs in background, no UI block│
│ Message Passing      │ postMessage/onmessage          │
│ No DOM Access        │ Cannot manipulate DOM          │
│ Transferable Objects │ Zero-copy data transfer        │
│ Worker Pool          │ Manage multiple workers        │
│ Shared Workers       │ Shared across browser contexts │
└──────────────────────┴────────────────────────────────┘

When to Use Web Workers:
• Heavy computations (sorting, filtering)
• Image/video processing
• Cryptographic operations
• Data parsing (JSON, CSV)
• Complex algorithms
• Real-time data processing
• Background sync

Communication:
• postMessage(data, [transferables])
• onmessage = (event) => { event.data }
• onerror = (error) => { error.message }
• Structured clone algorithm (no functions)

Transferable Objects:
• ArrayBuffer, MessagePort, ImageBitmap
• Zero-copy transfer (much faster)
• Original becomes unusable
• Syntax: postMessage(data, [buffer])

Limitations:
• No DOM access
• No window object
• No document object
• Limited APIs available
• Cannot manipulate UI directly
• Communication overhead

Available APIs in Workers:
• navigator, location (limited)
• XMLHttpRequest, fetch
• WebSockets
• IndexedDB
• setTimeout, setInterval
• crypto (Web Crypto API)
• postMessage, close

Best Practices:
• Use for CPU-intensive tasks
• Minimize data transfer
• Use transferable objects for large data
• Implement worker pool for parallel tasks
• Handle errors gracefully
• Terminate workers when done
• Use inline workers for simple cases
• Report progress for long operations

Performance Tips:
• Pool size = CPU cores (navigator.hardwareConcurrency)
• Batch processing for multiple tasks
• Use transferable objects (ArrayBuffer)
• Avoid excessive message passing
• Cache workers when possible
• Use SharedArrayBuffer for shared memory

Common Patterns:
• Promise wrapper for clean API
• Worker pool for parallel processing
• Progress reporting for long tasks
• Retry logic for failures
• Queue management for tasks
• Graceful degradation (fallback)

Real-World Applications:
• Image filters (Instagram-like effects)
• Large dataset sorting/filtering
• PDF generation
• Excel parsing
• Compression/decompression
• Hash calculation
• Machine learning inference
• Audio processing

Testing:
• Mock postMessage for unit tests
• Test with different data sizes
• Test error handling
• Test termination
• Benchmark performance gain
• Test browser compatibility

Browser Support:
• Web Workers: 97%+
• Transferable Objects: 95%+
• SharedArrayBuffer: 92%+ (with headers)
• Good fallback: run in main thread
`);
```

**Interview Tips:**
- Web Worker: separate thread for JavaScript, doesn't block UI
- Communication: postMessage() to send, onmessage to receive
- No DOM access: workers can't manipulate UI
- Use cases: heavy computations, image processing, data parsing, crypto
- Structured clone: data is copied, not shared (no functions)
- Transferable objects: ArrayBuffer zero-copy transfer (much faster)
- Worker pool: manage multiple workers for parallel tasks
- Inline worker: create from string using Blob URL
- Promise wrapper: cleaner API than callbacks
- Pool size: use navigator.hardwareConcurrency (CPU cores)
- Limitations: no window, document, or DOM access
- Available APIs: fetch, WebSockets, IndexedDB, crypto, timers
- Error handling: onerror event for worker errors
- Terminate: worker.terminate() to stop worker
- Performance: significant for CPU-bound tasks (50-80% faster)
- Overhead: message passing has cost, not for trivial tasks
- Best for: operations taking >50ms on main thread
- Progress reporting: send periodic progress messages
- Real-world: image filters, sorting large arrays, cryptography
- Trade-offs: complexity vs performance gain, memory overhead
- Follow-ups: SharedArrayBuffer, service workers, transferable objects
- Clarify: task complexity? data size? parallel processing? browser support?

</details>

128. Implement request batching/deduplication

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Request Deduplication**
```javascript
/**
 * Deduplicate identical concurrent requests
 */

class RequestDeduplicator {
  constructor() {
    this.pendingRequests = new Map();
  }
  
  async fetch(url, options = {}) {
    const key = this.generateKey(url, options);
    
    // Check if request is already pending
    if (this.pendingRequests.has(key)) {
      console.log('Deduplicating request:', url);
      return this.pendingRequests.get(key);
    }
    
    // Create new request
    const promise = fetch(url, options)
      .then(response => response.json())
      .finally(() => {
        // Remove from pending after completion
        this.pendingRequests.delete(key);
      });
    
    // Store pending request
    this.pendingRequests.set(key, promise);
    
    return promise;
  }
  
  generateKey(url, options) {
    // Simple key generation based on URL and method
    const method = options.method || 'GET';
    const body = options.body ? JSON.stringify(options.body) : '';
    return `${method}:${url}:${body}`;
  }
  
  clear() {
    this.pendingRequests.clear();
  }
}

// Usage:
const deduplicator = new RequestDeduplicator();

// These will result in only ONE actual HTTP request
Promise.all([
  deduplicator.fetch('https://api.example.com/users/1'),
  deduplicator.fetch('https://api.example.com/users/1'),
  deduplicator.fetch('https://api.example.com/users/1')
]).then(results => {
  console.log('All results:', results);
  // All three promises get the same result
});

console.log('=== Basic Request Deduplication ===');
console.log('Prevents duplicate concurrent requests');
console.log('Returns same promise for identical requests');
```

### **Approach 2: Request Batching with Time Window**
```javascript
/**
 * Batch multiple requests within a time window
 */

class RequestBatcher {
  constructor(options = {}) {
    this.batchWindow = options.batchWindow || 50; // ms
    this.maxBatchSize = options.maxBatchSize || 10;
    
    this.pendingRequests = [];
    this.batchTimer = null;
  }
  
  async request(endpoint, params = {}) {
    return new Promise((resolve, reject) => {
      // Add request to pending batch
      this.pendingRequests.push({
        endpoint,
        params,
        resolve,
        reject
      });
      
      // Schedule batch execution
      this.scheduleBatch();
    });
  }
  
  scheduleBatch() {
    // Clear existing timer
    if (this.batchTimer) {
      clearTimeout(this.batchTimer);
    }
    
    // Execute immediately if batch is full
    if (this.pendingRequests.length >= this.maxBatchSize) {
      this.executeBatch();
      return;
    }
    
    // Otherwise wait for more requests
    this.batchTimer = setTimeout(() => {
      this.executeBatch();
    }, this.batchWindow);
  }
  
  async executeBatch() {
    if (this.pendingRequests.length === 0) return;
    
    const batch = this.pendingRequests.splice(0);
    
    console.log(`Executing batch of ${batch.length} requests`);
    
    try {
      // Send batch request to server
      const response = await fetch('/api/batch', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          requests: batch.map(req => ({
            endpoint: req.endpoint,
            params: req.params
          }))
        })
      });
      
      const results = await response.json();
      
      // Resolve individual promises
      batch.forEach((req, index) => {
        if (results[index].error) {
          req.reject(new Error(results[index].error));
        } else {
          req.resolve(results[index].data);
        }
      });
    } catch (error) {
      // Reject all promises on batch failure
      batch.forEach(req => req.reject(error));
    }
  }
}

// Usage:
const batcher = new RequestBatcher({
  batchWindow: 100,
  maxBatchSize: 20
});

// These requests will be batched together
async function loadUserData() {
  const [user1, user2, user3] = await Promise.all([
    batcher.request('/users', { id: 1 }),
    batcher.request('/users', { id: 2 }),
    batcher.request('/users', { id: 3 })
  ]);
  
  console.log('Users:', user1, user2, user3);
}

console.log('\n=== Request Batching ===');
console.log('Combines multiple requests into one');
console.log('Reduces network overhead');
```

### **Approach 3: DataLoader Pattern (Facebook)**
```javascript
/**
 * Advanced batching and caching (like GraphQL DataLoader)
 */

class DataLoader {
  constructor(batchLoadFn, options = {}) {
    this.batchLoadFn = batchLoadFn;
    this.maxBatchSize = options.maxBatchSize || 100;
    this.batchScheduleFn = options.batchScheduleFn || this.defaultBatchSchedule;
    
    this.cache = new Map();
    this.batch = [];
    this.scheduled = false;
  }
  
  load(key) {
    // Check cache first
    if (this.cache.has(key)) {
      return Promise.resolve(this.cache.get(key));
    }
    
    // Create promise for this key
    return new Promise((resolve, reject) => {
      this.batch.push({ key, resolve, reject });
      
      // Schedule batch if not already scheduled
      if (!this.scheduled) {
        this.scheduled = true;
        this.batchScheduleFn(() => this.dispatchBatch());
      }
    });
  }
  
  loadMany(keys) {
    return Promise.all(keys.map(key => this.load(key)));
  }
  
  defaultBatchSchedule(callback) {
    // Use process.nextTick in Node.js, setTimeout in browser
    if (typeof process !== 'undefined' && process.nextTick) {
      process.nextTick(callback);
    } else {
      setTimeout(callback, 0);
    }
  }
  
  async dispatchBatch() {
    this.scheduled = false;
    
    const batch = this.batch.splice(0);
    if (batch.length === 0) return;
    
    // Extract unique keys
    const keys = batch.map(item => item.key);
    const uniqueKeys = [...new Set(keys)];
    
    try {
      // Call batch load function
      const values = await this.batchLoadFn(uniqueKeys);
      
      if (values.length !== uniqueKeys.length) {
        throw new Error('BatchLoadFn must return same number of values as keys');
      }
      
      // Create key-value map
      const valueMap = new Map();
      uniqueKeys.forEach((key, index) => {
        const value = values[index];
        valueMap.set(key, value);
        
        // Cache the result
        this.cache.set(key, value);
      });
      
      // Resolve all promises
      batch.forEach(item => {
        const value = valueMap.get(item.key);
        item.resolve(value);
      });
    } catch (error) {
      // Reject all promises
      batch.forEach(item => item.reject(error));
    }
  }
  
  clear(key) {
    if (key) {
      this.cache.delete(key);
    } else {
      this.cache.clear();
    }
  }
  
  prime(key, value) {
    this.cache.set(key, value);
  }
}

// Usage:
async function batchLoadUsers(userIds) {
  console.log('Batch loading users:', userIds);
  
  const response = await fetch('/api/users/batch', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ ids: userIds })
  });
  
  return response.json();
}

const userLoader = new DataLoader(batchLoadUsers, {
  maxBatchSize: 50
});

// These will be batched automatically
async function loadData() {
  const user1 = await userLoader.load(1);
  const user2 = await userLoader.load(2);
  const user3 = await userLoader.load(1); // Cached, no request
  
  console.log('Users:', user1, user2, user3);
}

console.log('\n=== DataLoader Pattern ===');
console.log('Batching + caching combined');
console.log('Used by GraphQL servers');
```

### **Approach 4: Request Queue with Prioritization**
```javascript
/**
 * Queue requests with priority and deduplication
 */

class PriorityRequestQueue {
  constructor(options = {}) {
    this.concurrency = options.concurrency || 5;
    this.running = 0;
    
    this.queues = {
      high: [],
      normal: [],
      low: []
    };
    
    this.pendingRequests = new Map();
  }
  
  async request(url, options = {}, priority = 'normal') {
    const key = this.generateKey(url, options);
    
    // Deduplicate
    if (this.pendingRequests.has(key)) {
      console.log('Deduplicating:', url);
      return this.pendingRequests.get(key);
    }
    
    // Create promise
    const promise = new Promise((resolve, reject) => {
      this.queues[priority].push({
        url,
        options,
        key,
        resolve,
        reject
      });
    });
    
    this.pendingRequests.set(key, promise);
    
    // Process queue
    this.processQueue();
    
    return promise;
  }
  
  async processQueue() {
    if (this.running >= this.concurrency) return;
    
    // Get next request by priority
    const request = this.getNextRequest();
    if (!request) return;
    
    this.running++;
    
    try {
      const response = await fetch(request.url, request.options);
      const data = await response.json();
      
      request.resolve(data);
    } catch (error) {
      request.reject(error);
    } finally {
      this.running--;
      this.pendingRequests.delete(request.key);
      
      // Process next request
      this.processQueue();
    }
  }
  
  getNextRequest() {
    // Check high priority first
    if (this.queues.high.length > 0) {
      return this.queues.high.shift();
    }
    
    // Then normal
    if (this.queues.normal.length > 0) {
      return this.queues.normal.shift();
    }
    
    // Finally low
    if (this.queues.low.length > 0) {
      return this.queues.low.shift();
    }
    
    return null;
  }
  
  generateKey(url, options) {
    const method = options.method || 'GET';
    const body = options.body ? JSON.stringify(options.body) : '';
    return `${method}:${url}:${body}`;
  }
  
  getQueueStatus() {
    return {
      running: this.running,
      high: this.queues.high.length,
      normal: this.queues.normal.length,
      low: this.queues.low.length
    };
  }
}

// Usage:
const queue = new PriorityRequestQueue({ concurrency: 3 });

// High priority (user interaction)
queue.request('/api/user/profile', {}, 'high')
  .then(data => console.log('Profile:', data));

// Normal priority (content)
queue.request('/api/posts', {}, 'normal')
  .then(data => console.log('Posts:', data));

// Low priority (analytics)
queue.request('/api/analytics', {}, 'low')
  .then(data => console.log('Analytics:', data));

console.log('\n=== Priority Request Queue ===');
console.log('Prioritizes important requests');
console.log('Limits concurrent requests');
```

### **Approach 5: Debounced Request Manager**
```javascript
/**
 * Debounce and batch requests with caching
 */

class DebouncedRequestManager {
  constructor(options = {}) {
    this.debounceTime = options.debounceTime || 300;
    this.cacheTime = options.cacheTime || 60000; // 1 minute
    
    this.cache = new Map();
    this.pendingRequests = new Map();
    this.debounceTimers = new Map();
  }
  
  async request(url, options = {}) {
    const key = this.generateKey(url, options);
    
    // Check cache
    const cached = this.getFromCache(key);
    if (cached) {
      console.log('Returning cached result:', url);
      return Promise.resolve(cached);
    }
    
    // Check if request is pending
    if (this.pendingRequests.has(key)) {
      console.log('Request already pending:', url);
      return this.pendingRequests.get(key);
    }
    
    // Create debounced request
    return new Promise((resolve, reject) => {
      // Clear existing timer
      if (this.debounceTimers.has(key)) {
        clearTimeout(this.debounceTimers.get(key));
      }
      
      // Set new timer
      const timer = setTimeout(async () => {
        this.debounceTimers.delete(key);
        
        try {
          const response = await fetch(url, options);
          const data = await response.json();
          
          // Cache result
          this.setCache(key, data);
          
          this.pendingRequests.delete(key);
          resolve(data);
        } catch (error) {
          this.pendingRequests.delete(key);
          reject(error);
        }
      }, this.debounceTime);
      
      this.debounceTimers.set(key, timer);
    });
  }
  
  generateKey(url, options) {
    const method = options.method || 'GET';
    const body = options.body ? JSON.stringify(options.body) : '';
    return `${method}:${url}:${body}`;
  }
  
  getFromCache(key) {
    const cached = this.cache.get(key);
    
    if (!cached) return null;
    
    // Check if expired
    if (Date.now() - cached.timestamp > this.cacheTime) {
      this.cache.delete(key);
      return null;
    }
    
    return cached.data;
  }
  
  setCache(key, data) {
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
  }
  
  clearCache(key) {
    if (key) {
      const fullKey = typeof key === 'string' ? key : this.generateKey(key.url, key.options);
      this.cache.delete(fullKey);
    } else {
      this.cache.clear();
    }
  }
}

// Usage:
const manager = new DebouncedRequestManager({
  debounceTime: 500,
  cacheTime: 30000
});

// Rapid calls - only last one executes after 500ms
for (let i = 0; i < 10; i++) {
  manager.request('/api/search', {
    method: 'POST',
    body: JSON.stringify({ query: 'test' })
  }).then(data => console.log('Search result:', data));
}

console.log('\n=== Debounced Request Manager ===');
console.log('Debounces rapid requests');
console.log('Caches results with TTL');
```

### **Real-World Use Cases**
```javascript
/**
 * Practical implementations
 */

// 1. GraphQL-like Batch Loader
console.log('\n=== GraphQL Batch Loading ===');

class GraphQLBatcher {
  constructor(endpoint) {
    this.endpoint = endpoint;
    this.loaders = new Map();
  }
  
  getLoader(type, batchFn) {
    if (!this.loaders.has(type)) {
      this.loaders.set(type, new DataLoader(batchFn));
    }
    return this.loaders.get(type);
  }
  
  async loadUser(userId) {
    const loader = this.getLoader('user', async (ids) => {
      console.log('Batch loading users:', ids);
      
      const response = await fetch(this.endpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          query: `
            query BatchUsers($ids: [ID!]!) {
              users(ids: $ids) {
                id
                name
                email
              }
            }
          `,
          variables: { ids }
        })
      });
      
      const result = await response.json();
      return result.data.users;
    });
    
    return loader.load(userId);
  }
  
  async loadPost(postId) {
    const loader = this.getLoader('post', async (ids) => {
      console.log('Batch loading posts:', ids);
      
      const response = await fetch(this.endpoint, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          query: `
            query BatchPosts($ids: [ID!]!) {
              posts(ids: $ids) {
                id
                title
                content
              }
            }
          `,
          variables: { ids }
        })
      });
      
      const result = await response.json();
      return result.data.posts;
    });
    
    return loader.load(postId);
  }
  
  clearCache() {
    this.loaders.forEach(loader => loader.clear());
  }
}

/*
Usage:
const batcher = new GraphQLBatcher('/graphql');

async function loadData() {
  // These will be batched into two requests
  const [user1, user2, post1, post2] = await Promise.all([
    batcher.loadUser(1),
    batcher.loadUser(2),
    batcher.loadPost(10),
    batcher.loadPost(20)
  ]);
  
  console.log('Loaded:', user1, user2, post1, post2);
}
*/

// 2. Search Request Deduplication
console.log('\n=== Search Request Deduplication ===');

class SearchManager {
  constructor(apiEndpoint) {
    this.apiEndpoint = apiEndpoint;
    this.pendingSearches = new Map();
    this.searchCache = new Map();
    this.debounceTimers = new Map();
  }
  
  search(query, options = {}) {
    return new Promise((resolve, reject) => {
      // Clear previous debounce timer for this query type
      const timerKey = options.category || 'default';
      if (this.debounceTimers.has(timerKey)) {
        clearTimeout(this.debounceTimers.get(timerKey));
      }
      
      // Debounce search
      const timer = setTimeout(async () => {
        this.debounceTimers.delete(timerKey);
        
        const cacheKey = `${query}:${JSON.stringify(options)}`;
        
        // Check cache
        if (this.searchCache.has(cacheKey)) {
          console.log('Returning cached search:', query);
          resolve(this.searchCache.get(cacheKey));
          return;
        }
        
        // Deduplicate
        if (this.pendingSearches.has(cacheKey)) {
          console.log('Deduplicating search:', query);
          const pending = await this.pendingSearches.get(cacheKey);
          resolve(pending);
          return;
        }
        
        // Execute search
        const searchPromise = this.executeSearch(query, options);
        this.pendingSearches.set(cacheKey, searchPromise);
        
        try {
          const results = await searchPromise;
          this.searchCache.set(cacheKey, results);
          resolve(results);
        } catch (error) {
          reject(error);
        } finally {
          this.pendingSearches.delete(cacheKey);
        }
      }, 300);
      
      this.debounceTimers.set(timerKey, timer);
    });
  }
  
  async executeSearch(query, options) {
    console.log('Executing search:', query);
    
    const response = await fetch(`${this.apiEndpoint}/search`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ query, ...options })
    });
    
    return response.json();
  }
  
  clearCache() {
    this.searchCache.clear();
  }
}

/*
Usage:
const searchManager = new SearchManager('/api');

// User typing rapidly
searchManager.search('java')
  .then(results => console.log('Results:', results));

searchManager.search('javasc')
  .then(results => console.log('Results:', results));

searchManager.search('javascript')
  .then(results => console.log('Results:', results));

// Only last search executes
*/

// 3. API Rate Limiter with Batching
console.log('\n=== Rate-Limited Batch Requests ===');

class RateLimitedBatcher {
  constructor(options = {}) {
    this.requestsPerSecond = options.requestsPerSecond || 10;
    this.batchSize = options.batchSize || 5;
    
    this.queue = [];
    this.requestsInWindow = 0;
    this.windowStart = Date.now();
  }
  
  async request(url, options = {}) {
    return new Promise((resolve, reject) => {
      this.queue.push({ url, options, resolve, reject });
      this.processQueue();
    });
  }
  
  async processQueue() {
    // Reset window if needed
    const now = Date.now();
    if (now - this.windowStart >= 1000) {
      this.requestsInWindow = 0;
      this.windowStart = now;
    }
    
    // Check rate limit
    if (this.requestsInWindow >= this.requestsPerSecond) {
      // Wait for next window
      const waitTime = 1000 - (now - this.windowStart);
      setTimeout(() => this.processQueue(), waitTime);
      return;
    }
    
    // Get batch
    const batchSize = Math.min(
      this.batchSize,
      this.requestsPerSecond - this.requestsInWindow,
      this.queue.length
    );
    
    if (batchSize === 0) return;
    
    const batch = this.queue.splice(0, batchSize);
    this.requestsInWindow += batch.length;
    
    console.log(`Processing batch of ${batch.length} requests`);
    
    // Execute batch
    const promises = batch.map(async (item) => {
      try {
        const response = await fetch(item.url, item.options);
        const data = await response.json();
        item.resolve(data);
      } catch (error) {
        item.reject(error);
      }
    });
    
    await Promise.all(promises);
    
    // Process remaining queue
    if (this.queue.length > 0) {
      setTimeout(() => this.processQueue(), 0);
    }
  }
}

/*
Usage:
const rateLimiter = new RateLimitedBatcher({
  requestsPerSecond: 5,
  batchSize: 3
});

// Make many requests
for (let i = 0; i < 20; i++) {
  rateLimiter.request(`/api/data/${i}`)
    .then(data => console.log(`Data ${i}:`, data));
}
*/

// 4. Multi-Source Data Aggregator
console.log('\n=== Multi-Source Aggregator ===');

class DataAggregator {
  constructor() {
    this.loaders = {
      users: new DataLoader(this.batchLoadUsers.bind(this)),
      posts: new DataLoader(this.batchLoadPosts.bind(this)),
      comments: new DataLoader(this.batchLoadComments.bind(this))
    };
  }
  
  async batchLoadUsers(userIds) {
    console.log('Batch loading users:', userIds);
    const response = await fetch('/api/users/batch', {
      method: 'POST',
      body: JSON.stringify({ ids: userIds })
    });
    return response.json();
  }
  
  async batchLoadPosts(postIds) {
    console.log('Batch loading posts:', postIds);
    const response = await fetch('/api/posts/batch', {
      method: 'POST',
      body: JSON.stringify({ ids: postIds })
    });
    return response.json();
  }
  
  async batchLoadComments(commentIds) {
    console.log('Batch loading comments:', commentIds);
    const response = await fetch('/api/comments/batch', {
      method: 'POST',
      body: JSON.stringify({ ids: commentIds })
    });
    return response.json();
  }
  
  async loadPostWithAuthor(postId) {
    const post = await this.loaders.posts.load(postId);
    const author = await this.loaders.users.load(post.authorId);
    
    return { ...post, author };
  }
  
  async loadPostWithComments(postId) {
    const post = await this.loaders.posts.load(postId);
    const comments = await this.loaders.comments.loadMany(post.commentIds);
    
    // Load comment authors
    const authorIds = [...new Set(comments.map(c => c.authorId))];
    const authors = await this.loaders.users.loadMany(authorIds);
    
    const authorMap = new Map(authors.map(a => [a.id, a]));
    const commentsWithAuthors = comments.map(c => ({
      ...c,
      author: authorMap.get(c.authorId)
    }));
    
    return { ...post, comments: commentsWithAuthors };
  }
  
  clearCache() {
    Object.values(this.loaders).forEach(loader => loader.clear());
  }
}

/*
Usage:
const aggregator = new DataAggregator();

// Load multiple posts with authors and comments
const posts = await Promise.all([
  aggregator.loadPostWithComments(1),
  aggregator.loadPostWithComments(2),
  aggregator.loadPostWithComments(3)
]);

// All data loaded with minimal requests due to batching
*/

// 5. Performance Monitoring
console.log('\n=== With Performance Monitoring ===');

class MonitoredBatcher {
  constructor(batchFn, options = {}) {
    this.dataLoader = new DataLoader(batchFn, options);
    
    this.metrics = {
      totalRequests: 0,
      batchedRequests: 0,
      cacheHits: 0,
      totalBatches: 0,
      averageBatchSize: 0
    };
  }
  
  async load(key) {
    this.metrics.totalRequests++;
    
    // Check if in cache
    if (this.dataLoader.cache.has(key)) {
      this.metrics.cacheHits++;
    }
    
    const result = await this.dataLoader.load(key);
    
    this.updateMetrics();
    
    return result;
  }
  
  updateMetrics() {
    const batchCount = this.metrics.totalRequests - this.metrics.cacheHits;
    this.metrics.batchedRequests = batchCount;
    
    if (batchCount > 0) {
      this.metrics.averageBatchSize = 
        this.metrics.totalRequests / batchCount;
    }
  }
  
  getMetrics() {
    return {
      ...this.metrics,
      cacheHitRate: this.metrics.cacheHits / this.metrics.totalRequests,
      batchEfficiency: 1 - (this.metrics.batchedRequests / this.metrics.totalRequests)
    };
  }
  
  resetMetrics() {
    this.metrics = {
      totalRequests: 0,
      batchedRequests: 0,
      cacheHits: 0,
      totalBatches: 0,
      averageBatchSize: 0
    };
  }
}

/*
Usage:
const batcher = new MonitoredBatcher(
  async (ids) => {
    const response = await fetch('/api/batch', {
      method: 'POST',
      body: JSON.stringify({ ids })
    });
    return response.json();
  }
);

// Make requests
await Promise.all([
  batcher.load(1),
  batcher.load(2),
  batcher.load(1), // cache hit
  batcher.load(3)
]);

console.log('Metrics:', batcher.getMetrics());
// {
//   totalRequests: 4,
//   batchedRequests: 3,
//   cacheHits: 1,
//   cacheHitRate: 0.25,
//   batchEfficiency: 0.25
// }
*/
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Request Optimization Strategies:
┌──────────────────────┬────────────────────────────────┐
│ Strategy             │ Use Case                       │
├──────────────────────┼────────────────────────────────┤
│ Deduplication        │ Identical concurrent requests  │
│                      │ Prevents duplicate work        │
├──────────────────────┼────────────────────────────────┤
│ Batching             │ Multiple similar requests      │
│                      │ Combines into single request   │
├──────────────────────┼────────────────────────────────┤
│ DataLoader           │ GraphQL-style batch + cache    │
│                      │ Per-request caching            │
├──────────────────────┼────────────────────────────────┤
│ Debouncing           │ Rapid repeated requests        │
│                      │ Search, autocomplete           │
├──────────────────────┼────────────────────────────────┤
│ Priority Queue       │ Different importance levels    │
│                      │ Rate limiting                  │
└──────────────────────┴────────────────────────────────┘

Benefits:
• Reduced network requests (50-90%)
• Lower server load
• Faster response times
• Better user experience
• Reduced API costs
• Less bandwidth usage
• Improved scalability

Deduplication:
• Same request already pending
• Return same promise to all callers
• Clear after completion
• Key: method + URL + body
• No additional server load

Batching:
• Combine N requests into 1
• Time window or size threshold
• Server must support batch endpoint
• Reduces HTTP overhead
• GraphQL uses this heavily

DataLoader Pattern:
• Batching per event loop tick
• Built-in caching (per request)
• Prevents N+1 query problem
• Facebook's solution for GraphQL
• Automatic deduplication

Best Practices:
• Use deduplication for identical requests
• Batch similar requests together
• Cache results appropriately
• Set reasonable batch windows (10-50ms)
• Implement retry logic
• Monitor batch sizes
• Handle errors gracefully
• Clear cache strategically

Common Patterns:
• Search debouncing (300-500ms)
• DataLoader for GraphQL
• Priority queue for API limits
• Request pooling for HTTP/2
• Prefetching for predictable patterns

Server-Side Considerations:
• Support batch endpoints
• Return results in same order
• Handle partial failures
• Set reasonable limits
• Use proper status codes
• Document batch format

Performance Gains:
• 50-90% reduction in requests
• 30-70% faster page loads
• Significantly lower server load
• Better mobile experience
• Reduced API quota usage

Real-World Examples:
• GraphQL DataLoader
• Facebook's Relay
• Apollo Client
• React Query batching
• REST batch APIs

Trade-offs:
• Complexity vs performance
• Batch window vs latency
• Cache size vs memory
• Server support required
• Error handling complexity

When NOT to Use:
• Single, isolated requests
• Real-time critical updates
• Different authentication per request
• Requests with side effects
• Server doesn't support batching

Testing:
• Mock network layer
• Test deduplication logic
• Verify batch formation
• Test cache behavior
• Test error scenarios
• Measure performance improvement

Metrics to Track:
• Request count before/after
• Average batch size
• Cache hit rate
• Response time improvement
• Server load reduction
• Error rate

Browser Support:
• Map/Set: 97%+
• Promise: 98%+
• fetch API: 98%+
• setTimeout: 100%
• All major browsers supported
`);
```

**Interview Tips:**
- Request deduplication: prevent duplicate concurrent requests, return same promise
- Batching: combine multiple requests into one, reduces network overhead
- DataLoader pattern: Facebook's solution, batching + caching per request
- Time window: collect requests for N milliseconds, then execute batch
- Key generation: method + URL + body uniquely identifies request
- Benefits: 50-90% reduction in network requests, faster performance
- Cache: store results with TTL to avoid repeated requests
- Debouncing: delay request until user stops typing (search, autocomplete)
- Priority queue: handle important requests first, rate limiting
- GraphQL: heavily uses batching to solve N+1 problem
- N+1 problem: loading author for each post = N+1 queries, batching solves this
- Server support: needs batch endpoint that accepts multiple requests
- Batch size: balance between efficiency and latency (typically 10-100 items)
- Max batch size: prevent oversized requests, split if needed
- Error handling: partial failures should still return successes
- process.nextTick: batches requests in same event loop tick
- Real-world: DataLoader in GraphQL servers, search deduplication, API optimization
- Trade-offs: complexity vs performance, memory for cache
- When not to use: single requests, real-time updates, side effects
- Follow-ups: cache invalidation, batch endpoint design, error handling
- Clarify: server support? batch size? cache strategy? priority needs?

</details>

129. Build a simple service worker for caching

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Service Worker with Cache First Strategy**
```javascript
/**
 * Simple service worker for static asset caching
 */

// service-worker.js
const CACHE_NAME = 'my-app-cache-v1';
const urlsToCache = [
  '/',
  '/index.html',
  '/styles.css',
  '/script.js',
  '/logo.png'
];

// Install event - cache static assets
self.addEventListener('install', (event) => {
  console.log('[Service Worker] Installing...');
  
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        console.log('[Service Worker] Caching static assets');
        return cache.addAll(urlsToCache);
      })
      .then(() => {
        console.log('[Service Worker] Installation complete');
        return self.skipWaiting(); // Activate immediately
      })
  );
});

// Activate event - clean up old caches
self.addEventListener('activate', (event) => {
  console.log('[Service Worker] Activating...');
  
  event.waitUntil(
    caches.keys()
      .then((cacheNames) => {
        return Promise.all(
          cacheNames.map((cacheName) => {
            if (cacheName !== CACHE_NAME) {
              console.log('[Service Worker] Deleting old cache:', cacheName);
              return caches.delete(cacheName);
            }
          })
        );
      })
      .then(() => {
        console.log('[Service Worker] Activation complete');
        return self.clients.claim(); // Take control immediately
      })
  );
});

// Fetch event - serve from cache first
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((cachedResponse) => {
        if (cachedResponse) {
          console.log('[Service Worker] Serving from cache:', event.request.url);
          return cachedResponse;
        }
        
        console.log('[Service Worker] Fetching from network:', event.request.url);
        return fetch(event.request)
          .then((response) => {
            // Cache successful responses
            if (response.status === 200) {
              const responseToCache = response.clone();
              
              caches.open(CACHE_NAME)
                .then((cache) => {
                  cache.put(event.request, responseToCache);
                });
            }
            
            return response;
          });
      })
      .catch((error) => {
        console.error('[Service Worker] Fetch failed:', error);
        
        // Return offline page if available
        return caches.match('/offline.html');
      })
  );
});

// Register service worker (in main.js)
/*
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-worker.js')
      .then((registration) => {
        console.log('Service Worker registered:', registration.scope);
      })
      .catch((error) => {
        console.error('Service Worker registration failed:', error);
      });
  });
}
*/

console.log('=== Basic Service Worker ===');
console.log('Cache-first strategy');
console.log('Offline support');
```

### **Approach 2: Network First with Cache Fallback**
```javascript
/**
 * Network first strategy for dynamic content
 */

// service-worker.js
const CACHE_NAME = 'network-first-v1';
const STATIC_CACHE = 'static-v1';
const DYNAMIC_CACHE = 'dynamic-v1';

const staticAssets = [
  '/',
  '/index.html',
  '/offline.html',
  '/styles.css'
];

// Install
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(STATIC_CACHE)
      .then((cache) => cache.addAll(staticAssets))
      .then(() => self.skipWaiting())
  );
});

// Activate
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys()
      .then((keys) => {
        return Promise.all(
          keys
            .filter((key) => key !== STATIC_CACHE && key !== DYNAMIC_CACHE)
            .map((key) => caches.delete(key))
        );
      })
      .then(() => self.clients.claim())
  );
});

// Fetch - Network first strategy
self.addEventListener('fetch', (event) => {
  const { request } = event;
  
  // Different strategies for different request types
  if (request.url.includes('/api/')) {
    // Network first for API calls
    event.respondWith(networkFirst(request));
  } else if (request.destination === 'image') {
    // Cache first for images
    event.respondWith(cacheFirst(request));
  } else {
    // Stale while revalidate for pages
    event.respondWith(staleWhileRevalidate(request));
  }
});

// Network first strategy
async function networkFirst(request) {
  try {
    const response = await fetch(request);
    
    // Cache successful API responses
    if (response.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, response.clone());
    }
    
    return response;
  } catch (error) {
    // Fallback to cache
    const cached = await caches.match(request);
    
    if (cached) {
      return cached;
    }
    
    // Return error response
    return new Response(JSON.stringify({ error: 'Network failed' }), {
      status: 503,
      headers: { 'Content-Type': 'application/json' }
    });
  }
}

// Cache first strategy
async function cacheFirst(request) {
  const cached = await caches.match(request);
  
  if (cached) {
    return cached;
  }
  
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, response.clone());
    }
    
    return response;
  } catch (error) {
    // Return placeholder image
    return caches.match('/placeholder.png');
  }
}

// Stale while revalidate
async function staleWhileRevalidate(request) {
  const cached = await caches.match(request);
  
  const fetchPromise = fetch(request)
    .then((response) => {
      if (response.ok) {
        const cache = caches.open(STATIC_CACHE);
        cache.then((c) => c.put(request, response.clone()));
      }
      return response;
    })
    .catch(() => cached);
  
  return cached || fetchPromise;
}

console.log('\n=== Network First Strategy ===');
console.log('Always tries network first');
console.log('Falls back to cache if offline');
```

### **Approach 3: Cache Strategies with Expiration**
```javascript
/**
 * Multiple caching strategies with TTL
 */

const CACHE_VERSION = 'v2';
const CACHES = {
  static: `static-${CACHE_VERSION}`,
  dynamic: `dynamic-${CACHE_VERSION}`,
  images: `images-${CACHE_VERSION}`
};

const CACHE_EXPIRATION = {
  static: Infinity,
  dynamic: 3600000,  // 1 hour
  images: 86400000   // 24 hours
};

const MAX_CACHE_SIZE = {
  dynamic: 50,
  images: 100
};

// Install
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHES.static)
      .then((cache) => {
        return cache.addAll([
          '/',
          '/index.html',
          '/app.css',
          '/app.js',
          '/offline.html'
        ]);
      })
      .then(() => self.skipWaiting())
  );
});

// Activate
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys()
      .then((keys) => {
        return Promise.all(
          keys
            .filter((key) => !Object.values(CACHES).includes(key))
            .map((key) => caches.delete(key))
        );
      })
      .then(() => self.clients.claim())
  );
});

// Fetch
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // Skip non-GET requests
  if (request.method !== 'GET') {
    return;
  }
  
  // Route to appropriate strategy
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirstWithExpiration(request, CACHES.dynamic));
  } else if (request.destination === 'image') {
    event.respondWith(cacheFirstWithExpiration(request, CACHES.images));
  } else {
    event.respondWith(cacheFirstWithExpiration(request, CACHES.static));
  }
});

// Network first with cache expiration
async function networkFirstWithExpiration(request, cacheName) {
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cache = await caches.open(cacheName);
      
      // Add timestamp to cached response
      const clonedResponse = response.clone();
      const body = await clonedResponse.blob();
      
      const headers = new Headers(clonedResponse.headers);
      headers.set('sw-cache-time', Date.now().toString());
      
      const cachedResponse = new Response(body, {
        status: clonedResponse.status,
        statusText: clonedResponse.statusText,
        headers: headers
      });
      
      await cache.put(request, cachedResponse);
      
      // Limit cache size
      await limitCacheSize(cacheName, MAX_CACHE_SIZE.dynamic);
    }
    
    return response;
  } catch (error) {
    const cached = await caches.match(request);
    
    if (cached) {
      // Check if expired
      const cacheTime = cached.headers.get('sw-cache-time');
      const expiration = CACHE_EXPIRATION.dynamic;
      
      if (cacheTime && Date.now() - parseInt(cacheTime) > expiration) {
        console.log('[SW] Cache expired:', request.url);
        // Could delete and return error, or still return stale data
      }
      
      return cached;
    }
    
    return new Response('Offline', { status: 503 });
  }
}

// Cache first with expiration
async function cacheFirstWithExpiration(request, cacheName) {
  const cached = await caches.match(request);
  
  if (cached) {
    // Check expiration
    const cacheTime = cached.headers.get('sw-cache-time');
    const expiration = CACHE_EXPIRATION[cacheName.split('-')[0]];
    
    if (cacheTime && Date.now() - parseInt(cacheTime) < expiration) {
      // Update in background
      updateCache(request, cacheName);
      return cached;
    }
  }
  
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cache = await caches.open(cacheName);
      
      const body = await response.clone().blob();
      const headers = new Headers(response.headers);
      headers.set('sw-cache-time', Date.now().toString());
      
      const cachedResponse = new Response(body, {
        status: response.status,
        statusText: response.statusText,
        headers: headers
      });
      
      await cache.put(request, cachedResponse);
      
      const maxSize = MAX_CACHE_SIZE[cacheName.split('-')[0]];
      if (maxSize) {
        await limitCacheSize(cacheName, maxSize);
      }
    }
    
    return response;
  } catch (error) {
    return cached || new Response('Offline', { status: 503 });
  }
}

// Update cache in background
async function updateCache(request, cacheName) {
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cache = await caches.open(cacheName);
      
      const body = await response.clone().blob();
      const headers = new Headers(response.headers);
      headers.set('sw-cache-time', Date.now().toString());
      
      const cachedResponse = new Response(body, {
        status: response.status,
        statusText: response.statusText,
        headers: headers
      });
      
      await cache.put(request, cachedResponse);
    }
  } catch (error) {
    // Silently fail
  }
}

// Limit cache size
async function limitCacheSize(cacheName, maxSize) {
  const cache = await caches.open(cacheName);
  const keys = await cache.keys();
  
  if (keys.length > maxSize) {
    // Delete oldest entries
    const toDelete = keys.length - maxSize;
    
    for (let i = 0; i < toDelete; i++) {
      await cache.delete(keys[i]);
    }
  }
}

console.log('\n=== Cache with Expiration ===');
console.log('TTL-based cache invalidation');
console.log('Maximum cache size limits');
```

### **Approach 4: Background Sync**
```javascript
/**
 * Service worker with background sync
 */

const CACHE_NAME = 'sync-v1';
const QUEUE_NAME = 'sync-queue';

// Install
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => cache.addAll(['/']))
      .then(() => self.skipWaiting())
  );
});

// Activate
self.addEventListener('activate', (event) => {
  event.waitUntil(self.clients.claim());
});

// Fetch
self.addEventListener('fetch', (event) => {
  const { request } = event;
  
  // Handle POST requests with background sync
  if (request.method === 'POST') {
    event.respondWith(handlePostRequest(request));
    return;
  }
  
  // Regular GET requests
  event.respondWith(
    caches.match(request)
      .then((cached) => cached || fetch(request))
  );
});

// Handle POST requests
async function handlePostRequest(request) {
  try {
    const response = await fetch(request.clone());
    return response;
  } catch (error) {
    // Queue for background sync
    const requestData = {
      url: request.url,
      method: request.method,
      headers: Array.from(request.headers.entries()),
      body: await request.clone().text()
    };
    
    // Store in IndexedDB or cache
    await queueRequest(requestData);
    
    // Register sync
    await self.registration.sync.register(QUEUE_NAME);
    
    return new Response(
      JSON.stringify({ queued: true, message: 'Request queued for sync' }),
      { status: 202, headers: { 'Content-Type': 'application/json' } }
    );
  }
}

// Queue request
async function queueRequest(requestData) {
  const cache = await caches.open(QUEUE_NAME);
  const queuedRequests = await getQueuedRequests(cache);
  
  queuedRequests.push(requestData);
  
  await cache.put(
    '/queue',
    new Response(JSON.stringify(queuedRequests))
  );
}

// Get queued requests
async function getQueuedRequests(cache) {
  const response = await cache.match('/queue');
  
  if (!response) {
    return [];
  }
  
  return response.json();
}

// Background sync event
self.addEventListener('sync', (event) => {
  if (event.tag === QUEUE_NAME) {
    event.waitUntil(syncQueuedRequests());
  }
});

// Sync queued requests
async function syncQueuedRequests() {
  const cache = await caches.open(QUEUE_NAME);
  const queuedRequests = await getQueuedRequests(cache);
  
  if (queuedRequests.length === 0) {
    return;
  }
  
  console.log('[SW] Syncing queued requests:', queuedRequests.length);
  
  const results = await Promise.allSettled(
    queuedRequests.map(async (reqData) => {
      const response = await fetch(reqData.url, {
        method: reqData.method,
        headers: new Headers(reqData.headers),
        body: reqData.body
      });
      
      return response;
    })
  );
  
  // Remove successfully synced requests
  const remainingRequests = [];
  
  results.forEach((result, index) => {
    if (result.status === 'rejected') {
      remainingRequests.push(queuedRequests[index]);
    }
  });
  
  // Update queue
  if (remainingRequests.length > 0) {
    await cache.put(
      '/queue',
      new Response(JSON.stringify(remainingRequests))
    );
  } else {
    await cache.delete('/queue');
  }
  
  // Notify clients
  const clients = await self.clients.matchAll();
  clients.forEach((client) => {
    client.postMessage({
      type: 'sync-complete',
      synced: results.filter((r) => r.status === 'fulfilled').length,
      failed: remainingRequests.length
    });
  });
}

console.log('\n=== Background Sync ===');
console.log('Queues failed requests');
console.log('Syncs when connection restored');
```

### **Approach 5: Advanced Service Worker with Push Notifications**
```javascript
/**
 * Full-featured service worker
 */

const VERSION = 'v1';
const CACHES = {
  static: `static-${VERSION}`,
  dynamic: `dynamic-${VERSION}`,
  images: `images-${VERSION}`
};

// Install
self.addEventListener('install', (event) => {
  console.log('[SW] Installing version:', VERSION);
  
  event.waitUntil(
    Promise.all([
      caches.open(CACHES.static).then((cache) => {
        return cache.addAll([
          '/',
          '/index.html',
          '/styles.css',
          '/app.js',
          '/offline.html',
          '/manifest.json'
        ]);
      }),
      // Precache critical resources
      self.skipWaiting()
    ])
  );
});

// Activate
self.addEventListener('activate', (event) => {
  console.log('[SW] Activating version:', VERSION);
  
  event.waitUntil(
    Promise.all([
      // Clean old caches
      caches.keys().then((keys) => {
        return Promise.all(
          keys
            .filter((key) => !Object.values(CACHES).includes(key))
            .map((key) => {
              console.log('[SW] Deleting old cache:', key);
              return caches.delete(key);
            })
        );
      }),
      // Take control
      self.clients.claim()
    ])
  );
});

// Fetch
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // Skip non-GET requests
  if (request.method !== 'GET') {
    return;
  }
  
  // Skip cross-origin requests (except images)
  if (url.origin !== location.origin && request.destination !== 'image') {
    return;
  }
  
  // Route by request type
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(networkFirst(request, CACHES.dynamic));
  } else if (request.destination === 'image') {
    event.respondWith(cacheFirst(request, CACHES.images));
  } else {
    event.respondWith(staleWhileRevalidate(request, CACHES.static));
  }
});

// Cache strategies
async function networkFirst(request, cacheName) {
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cache = await caches.open(cacheName);
      cache.put(request, response.clone());
    }
    
    return response;
  } catch (error) {
    return (await caches.match(request)) || 
      new Response('Offline', { status: 503 });
  }
}

async function cacheFirst(request, cacheName) {
  const cached = await caches.match(request);
  if (cached) return cached;
  
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cache = await caches.open(cacheName);
      cache.put(request, response.clone());
    }
    
    return response;
  } catch (error) {
    return new Response('Offline', { status: 503 });
  }
}

async function staleWhileRevalidate(request, cacheName) {
  const cached = await caches.match(request);
  
  const fetchPromise = fetch(request)
    .then((response) => {
      if (response.ok) {
        caches.open(cacheName).then((cache) => {
          cache.put(request, response.clone());
        });
      }
      return response;
    })
    .catch(() => cached);
  
  return cached || fetchPromise;
}

// Push notification
self.addEventListener('push', (event) => {
  const data = event.data ? event.data.json() : {};
  
  const title = data.title || 'Notification';
  const options = {
    body: data.body || 'You have a new notification',
    icon: data.icon || '/icon.png',
    badge: data.badge || '/badge.png',
    data: data.data || {},
    actions: data.actions || []
  };
  
  event.waitUntil(
    self.registration.showNotification(title, options)
  );
});

// Notification click
self.addEventListener('notificationclick', (event) => {
  event.notification.close();
  
  const url = event.notification.data.url || '/';
  
  event.waitUntil(
    clients.matchAll({ type: 'window' })
      .then((clientList) => {
        // Focus existing window if available
        for (const client of clientList) {
          if (client.url === url && 'focus' in client) {
            return client.focus();
          }
        }
        
        // Open new window
        if (clients.openWindow) {
          return clients.openWindow(url);
        }
      })
  );
});

// Message from client
self.addEventListener('message', (event) => {
  if (event.data.action === 'skipWaiting') {
    self.skipWaiting();
  } else if (event.data.action === 'clearCache') {
    event.waitUntil(
      caches.keys().then((keys) => {
        return Promise.all(keys.map((key) => caches.delete(key)));
      })
    );
  }
});

console.log('\n=== Advanced Service Worker ===');
console.log('Multiple strategies');
console.log('Push notifications');
console.log('Client communication');
```

### **Real-World Use Cases**
```javascript
/**
 * Practical service worker implementations
 */

// 1. Progressive Web App (PWA)
console.log('\n=== PWA Service Worker ===');

/*
// Main app (app.js)
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js')
      .then((registration) => {
        console.log('SW registered:', registration.scope);
        
        // Check for updates
        registration.addEventListener('updatefound', () => {
          const newWorker = registration.installing;
          
          newWorker.addEventListener('statechange', () => {
            if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
              // New version available
              showUpdateNotification();
            }
          });
        });
      })
      .catch((error) => {
        console.error('SW registration failed:', error);
      });
  });
  
  // Listen for messages from SW
  navigator.serviceWorker.addEventListener('message', (event) => {
    if (event.data.type === 'cache-updated') {
      console.log('Cache updated:', event.data.url);
    }
  });
}

function showUpdateNotification() {
  const notification = document.createElement('div');
  notification.innerHTML = `
    <p>New version available!</p>
    <button onclick="updateServiceWorker()">Update</button>
  `;
  document.body.appendChild(notification);
}

function updateServiceWorker() {
  navigator.serviceWorker.getRegistration().then((reg) => {
    reg.waiting.postMessage({ action: 'skipWaiting' });
  });
  
  window.location.reload();
}
*/

// 2. Offline-First App
console.log('\n=== Offline-First App ===');

/*
// sw.js
const CACHE_NAME = 'offline-first-v1';

self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll([
        '/',
        '/index.html',
        '/styles.css',
        '/app.js',
        '/offline.html',
        '/data/initial.json'
      ]);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then((cached) => {
        // Return cached immediately
        const fetchPromise = fetch(event.request)
          .then((response) => {
            // Update cache in background
            if (response.ok) {
              caches.open(CACHE_NAME).then((cache) => {
                cache.put(event.request, response.clone());
              });
            }
            return response;
          })
          .catch(() => cached);
        
        return cached || fetchPromise;
      })
  );
});
*/

// 3. API Response Caching with Refresh
console.log('\n=== API Caching with Refresh ===');

/*
// sw.js
const API_CACHE = 'api-cache-v1';
const API_CACHE_TIME = 5 * 60 * 1000; // 5 minutes

self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);
  
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(handleApiRequest(event.request));
  }
});

async function handleApiRequest(request) {
  const cache = await caches.open(API_CACHE);
  const cached = await cache.match(request);
  
  if (cached) {
    const cacheTime = cached.headers.get('sw-cache-time');
    
    if (cacheTime && Date.now() - parseInt(cacheTime) < API_CACHE_TIME) {
      // Serve from cache and update in background
      updateApiCache(request, cache);
      return cached;
    }
  }
  
  // Fetch fresh data
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cloned = response.clone();
      const body = await cloned.blob();
      
      const headers = new Headers(cloned.headers);
      headers.set('sw-cache-time', Date.now().toString());
      
      const cachedResponse = new Response(body, {
        status: cloned.status,
        statusText: cloned.statusText,
        headers: headers
      });
      
      await cache.put(request, cachedResponse);
    }
    
    return response;
  } catch (error) {
    // Return stale cache if available
    return cached || new Response('Offline', { status: 503 });
  }
}

async function updateApiCache(request, cache) {
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const body = await response.clone().blob();
      const headers = new Headers(response.headers);
      headers.set('sw-cache-time', Date.now().toString());
      
      const cachedResponse = new Response(body, {
        status: response.status,
        statusText: response.statusText,
        headers: headers
      });
      
      await cache.put(request, cachedResponse);
      
      // Notify clients about update
      const clients = await self.clients.matchAll();
      clients.forEach((client) => {
        client.postMessage({
          type: 'api-updated',
          url: request.url
        });
      });
    }
  } catch (error) {
    // Silently fail
  }
}
*/

// 4. Analytics Queueing
console.log('\n=== Analytics Queue ===');

/*
// sw.js
const ANALYTICS_QUEUE = 'analytics-queue-v1';

self.addEventListener('fetch', (event) => {
  const url = new URL(event.request.url);
  
  if (url.pathname.startsWith('/analytics')) {
    event.respondWith(handleAnalytics(event.request));
  }
});

async function handleAnalytics(request) {
  try {
    return await fetch(request);
  } catch (error) {
    // Queue analytics for later
    const data = await request.clone().text();
    await queueAnalytics(request.url, data);
    
    return new Response('Queued', { status: 202 });
  }
}

async function queueAnalytics(url, data) {
  const cache = await caches.open(ANALYTICS_QUEUE);
  const queue = await getQueue(cache);
  
  queue.push({ url, data, timestamp: Date.now() });
  
  await cache.put(
    '/queue',
    new Response(JSON.stringify(queue))
  );
  
  // Register background sync
  await self.registration.sync.register('analytics-sync');
}

async function getQueue(cache) {
  const response = await cache.match('/queue');
  return response ? response.json() : [];
}

self.addEventListener('sync', (event) => {
  if (event.tag === 'analytics-sync') {
    event.waitUntil(syncAnalytics());
  }
});

async function syncAnalytics() {
  const cache = await caches.open(ANALYTICS_QUEUE);
  const queue = await getQueue(cache);
  
  if (queue.length === 0) return;
  
  const results = await Promise.allSettled(
    queue.map((item) => {
      return fetch(item.url, {
        method: 'POST',
        body: item.data
      });
    })
  );
  
  // Remove successfully synced items
  const remaining = queue.filter((_, i) => results[i].status === 'rejected');
  
  if (remaining.length > 0) {
    await cache.put('/queue', new Response(JSON.stringify(remaining)));
  } else {
    await cache.delete('/queue');
  }
}
*/
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Service Worker Caching Strategies:
┌──────────────────────┬────────────────────────────────┐
│ Strategy             │ Use Case                       │
├──────────────────────┼────────────────────────────────┤
│ Cache First          │ Static assets (CSS, JS, images)│
│                      │ Fastest, best for immutables   │
├──────────────────────┼────────────────────────────────┤
│ Network First        │ Dynamic content, API responses │
│                      │ Fresh data, cache as fallback  │
├──────────────────────┼────────────────────────────────┤
│ Stale-While-         │ User-generated content         │
│ Revalidate           │ Instant load, update background│
├──────────────────────┼────────────────────────────────┤
│ Network Only         │ Real-time data, POST requests  │
│                      │ Never cached                   │
├──────────────────────┼────────────────────────────────┤
│ Cache Only           │ Pre-cached during install      │
│                      │ App shell pattern              │
└──────────────────────┴────────────────────────────────┘

Key Concepts:
• Install: One-time setup, precache assets
• Activate: Clean old caches, take control
• Fetch: Intercept network requests
• Cache API: Store responses by request
• Scope: SW controls URLs within its scope
• Update: Install new SW, activate on next load
• skipWaiting(): Activate immediately
• clients.claim(): Take control of pages

Benefits:
• Offline functionality
• Faster load times (cache)
• Reduced bandwidth
• Better reliability
• Progressive enhancement
• Background sync
• Push notifications

Best Practices:
• Version your caches
• Clean old caches in activate
• Don't cache authentication
• Set reasonable cache sizes
• Use appropriate strategy per resource
• Handle errors gracefully
• Provide offline fallback
• Test offline scenarios

Cache Versioning:
• Include version in cache name
• Delete old versions on activate
• Prevents stale data
• Example: 'my-app-v1', 'my-app-v2'

Lifecycle:
1. Register: navigator.serviceWorker.register()
2. Install: Download, cache assets
3. Activate: Clean up, take control
4. Fetch: Intercept requests
5. Update: New version available

Background Sync:
• Queue failed requests
• Sync when connection restored
• registration.sync.register()
• sync event listener
• Reliable data submission

Push Notifications:
• Engage users even when app closed
• push event listener
• showNotification()
• notificationclick event
• Requires user permission

Common Patterns:
• App Shell: Cache UI, network for content
• Offline Page: Fallback when offline
• Stale While Revalidate: Instant + fresh
• Cache Then Network: Best of both
• Network Then Cache: Fresh with fallback

Performance Metrics:
• First Load: Slower (SW registration)
• Subsequent Loads: 50-90% faster
• Offline Support: 100% functional
• Data Usage: 60-80% reduction
• Time to Interactive: Significantly faster

Debugging:
• Chrome DevTools > Application > Service Workers
• View cache contents
• Unregister for testing
• Update on reload option
• Console logging

Limitations:
• HTTPS only (or localhost)
• Same-origin policy
• No DOM access
• Async only (no sync XHR)
• Browser support (95%+)

Security:
• HTTPS required (except localhost)
• Cannot access cross-origin without CORS
• Content Security Policy applies
• Careful with cache poisoning

Testing:
• Test offline mode
• Test cache updates
• Test different strategies
• Test error scenarios
• Test on real devices

Real-World Impact:
• Twitter: 65% pages from cache
• Pinterest: 40% weekly users start offline
• Flipkart: 3x time on site
• Housing.com: 38% lower bounce rate
• Google I/O: Works offline completely

Browser Support:
• Chrome: Yes (since 2014)
• Firefox: Yes
• Safari: Yes (limited)
• Edge: Yes
• Mobile: 95%+ support
`);
```

**Interview Tips:**
- Service Worker: programmable network proxy, runs on separate thread
- Lifecycle: install → activate → fetch (then idle/terminated/fetch again)
- Install event: precache static assets, waitUntil ensures completion
- Activate event: clean old caches, clients.claim() takes control
- Fetch event: intercept requests, return cached or fetch from network
- Cache-first: check cache, fallback to network (static assets)
- Network-first: try network, fallback to cache (dynamic content)
- Stale-while-revalidate: serve cached, update in background
- HTTPS required: security requirement (except localhost)
- Scope: controls all pages under registration path
- skipWaiting(): activate new SW immediately, skip waiting phase
- clients.claim(): control pages immediately without reload
- Cache versioning: include version in cache name, delete old on activate
- caches.open(): access/create cache storage
- cache.addAll(): cache multiple URLs at once
- caches.match(): find cached response for request
- Background sync: queue failed requests, sync when online
- Push notifications: engage users when app closed
- event.waitUntil(): extend event lifetime, wait for async operations
- response.clone(): clone response to cache and return
- No DOM access: separate thread, can't manipulate UI
- Real-world: PWAs (Twitter, Pinterest), offline functionality, faster loads
- Benefits: offline support, 50-90% faster subsequent loads, reduced bandwidth
- Trade-offs: complexity, debugging difficulty, cache management
- Follow-ups: cache strategies, versioning, background sync, push notifications
- Clarify: offline requirements? cache strategy? update frequency?

</details>

130. Implement code splitting with dynamic imports

<details>
<summary><b>Solution</b></summary>

### **Approach 1: Basic Dynamic Import**
```javascript
/**
 * Simple dynamic import for code splitting
 */

// Traditional static import (bundles everything)
// import { heavyFunction } from './heavy-module.js';

// Dynamic import (loads on demand)
async function loadHeavyModule() {
  try {
    const module = await import('./heavy-module.js');
    
    console.log('Module loaded:', module);
    
    // Use the module
    const result = module.heavyFunction();
    console.log('Result:', result);
    
    return module;
  } catch (error) {
    console.error('Failed to load module:', error);
  }
}

// Load module when needed
document.getElementById('btn').addEventListener('click', async () => {
  const module = await loadHeavyModule();
  module.default(); // Default export
});

// Example heavy-module.js
/*
export function heavyFunction() {
  console.log('Heavy computation...');
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += Math.sqrt(i);
  }
  return result;
}

export default function() {
  console.log('Default export called');
}
*/

console.log('=== Basic Dynamic Import ===');
console.log('Loads code on demand');
console.log('Returns a promise');
```

### **Approach 2: Route-Based Code Splitting**
```javascript
/**
 * Split code by routes for SPA
 */

class Router {
  constructor() {
    this.routes = new Map();
    this.currentRoute = null;
    
    window.addEventListener('popstate', () => this.handleRoute());
  }
  
  // Register route with lazy-loaded component
  addRoute(path, componentLoader) {
    this.routes.set(path, {
      loader: componentLoader,
      component: null
    });
  }
  
  async navigate(path) {
    // Update browser history
    window.history.pushState({}, '', path);
    await this.handleRoute();
  }
  
  async handleRoute() {
    const path = window.location.pathname;
    const route = this.routes.get(path);
    
    if (!route) {
      console.error('Route not found:', path);
      return;
    }
    
    // Load component if not already loaded
    if (!route.component) {
      console.log('Loading component for:', path);
      
      try {
        const module = await route.loader();
        route.component = module.default;
      } catch (error) {
        console.error('Failed to load route:', error);
        return;
      }
    }
    
    // Render component
    this.currentRoute = path;
    this.render(route.component);
  }
  
  render(Component) {
    const app = document.getElementById('app');
    
    if (!app) return;
    
    // Clear previous content
    app.innerHTML = '';
    
    // Render new component
    if (typeof Component === 'function') {
      const content = Component();
      app.innerHTML = content;
    } else {
      app.innerHTML = Component;
    }
  }
  
  start() {
    this.handleRoute();
  }
}

// Usage:
const router = new Router();

// Register routes with dynamic imports
router.addRoute('/', () => import('./pages/Home.js'));
router.addRoute('/about', () => import('./pages/About.js'));
router.addRoute('/products', () => import('./pages/Products.js'));
router.addRoute('/contact', () => import('./pages/Contact.js'));

// Start router
router.start();

// Navigation
/*
<nav>
  <a href="/" onclick="router.navigate('/'); return false;">Home</a>
  <a href="/about" onclick="router.navigate('/about'); return false;">About</a>
  <a href="/products" onclick="router.navigate('/products'); return false;">Products</a>
</nav>
*/

// Example page component (pages/Home.js)
/*
export default function Home() {
  return `
    <div class="home">
      <h1>Home Page</h1>
      <p>Welcome to our website!</p>
    </div>
  `;
}
*/

console.log('\n=== Route-Based Code Splitting ===');
console.log('Load pages only when navigated to');
console.log('Reduces initial bundle size');
```

### **Approach 3: Component-Based Lazy Loading**
```javascript
/**
 * Lazy load components with loading states
 */

class LazyComponent {
  constructor(loader) {
    this.loader = loader;
    this.component = null;
    this.loading = false;
    this.error = null;
  }
  
  async load() {
    if (this.component) {
      return this.component;
    }
    
    if (this.loading) {
      // Wait for existing load
      return new Promise((resolve) => {
        const checkInterval = setInterval(() => {
          if (!this.loading) {
            clearInterval(checkInterval);
            resolve(this.component);
          }
        }, 100);
      });
    }
    
    this.loading = true;
    this.error = null;
    
    try {
      const module = await this.loader();
      this.component = module.default;
      return this.component;
    } catch (error) {
      this.error = error;
      throw error;
    } finally {
      this.loading = false;
    }
  }
  
  async render(container) {
    // Show loading state
    container.innerHTML = '<div class="loading">Loading...</div>';
    
    try {
      const Component = await this.load();
      
      // Render component
      if (typeof Component === 'function') {
        container.innerHTML = Component();
      } else {
        container.innerHTML = Component;
      }
    } catch (error) {
      // Show error state
      container.innerHTML = `
        <div class="error">
          <p>Failed to load component</p>
          <button onclick="this.parentElement.retry()">Retry</button>
        </div>
      `;
      
      container.querySelector('.error').retry = () => this.render(container);
    }
  }
}

// Usage:
const HeavyChart = new LazyComponent(() => import('./components/HeavyChart.js'));
const ImageGallery = new LazyComponent(() => import('./components/ImageGallery.js'));
const VideoPlayer = new LazyComponent(() => import('./components/VideoPlayer.js'));

// Load component when needed
document.getElementById('show-chart').addEventListener('click', () => {
  const container = document.getElementById('chart-container');
  HeavyChart.render(container);
});

document.getElementById('show-gallery').addEventListener('click', () => {
  const container = document.getElementById('gallery-container');
  ImageGallery.render(container);
});

console.log('\n=== Component-Based Lazy Loading ===');
console.log('Load components on demand');
console.log('With loading and error states');
```

### **Approach 4: Conditional Loading Based on Features**
```javascript
/**
 * Load features conditionally
 */

class FeatureLoader {
  constructor() {
    this.features = new Map();
    this.loaded = new Set();
  }
  
  register(featureName, loader, condition = () => true) {
    this.features.set(featureName, {
      loader,
      condition,
      module: null
    });
  }
  
  async load(featureName) {
    if (this.loaded.has(featureName)) {
      return this.features.get(featureName).module;
    }
    
    const feature = this.features.get(featureName);
    
    if (!feature) {
      throw new Error(`Feature not found: ${featureName}`);
    }
    
    // Check condition
    if (!feature.condition()) {
      console.log(`Feature ${featureName} condition not met`);
      return null;
    }
    
    console.log(`Loading feature: ${featureName}`);
    
    try {
      const module = await feature.loader();
      feature.module = module.default || module;
      this.loaded.add(featureName);
      
      return feature.module;
    } catch (error) {
      console.error(`Failed to load feature ${featureName}:`, error);
      throw error;
    }
  }
  
  async loadMultiple(featureNames) {
    return Promise.all(
      featureNames.map(name => this.load(name))
    );
  }
  
  isLoaded(featureName) {
    return this.loaded.has(featureName);
  }
}

// Usage:
const featureLoader = new FeatureLoader();

// Register features with conditions
featureLoader.register(
  'admin-panel',
  () => import('./features/AdminPanel.js'),
  () => user.isAdmin
);

featureLoader.register(
  'analytics',
  () => import('./features/Analytics.js'),
  () => user.hasPermission('analytics')
);

featureLoader.register(
  'advanced-editor',
  () => import('./features/AdvancedEditor.js'),
  () => user.isPremium
);

featureLoader.register(
  'mobile-features',
  () => import('./features/MobileFeatures.js'),
  () => window.innerWidth < 768
);

// Load feature when needed
async function showAdminPanel() {
  try {
    const AdminPanel = await featureLoader.load('admin-panel');
    
    if (AdminPanel) {
      AdminPanel.render();
    } else {
      alert('You do not have access to this feature');
    }
  } catch (error) {
    console.error('Failed to load admin panel:', error);
  }
}

// Preload features on idle
if ('requestIdleCallback' in window) {
  requestIdleCallback(() => {
    featureLoader.load('analytics');
  });
}

console.log('\n=== Conditional Feature Loading ===');
console.log('Load features based on conditions');
console.log('Role-based, device-based, etc.');
```

### **Approach 5: Prefetching and Preloading**
```javascript
/**
 * Smart prefetching for better performance
 */

class SmartLoader {
  constructor() {
    this.modules = new Map();
    this.prefetching = new Set();
    this.loaded = new Set();
  }
  
  register(moduleName, loader, options = {}) {
    this.modules.set(moduleName, {
      loader,
      priority: options.priority || 'low',
      prefetch: options.prefetch !== false,
      preload: options.preload || false
    });
  }
  
  async load(moduleName) {
    if (this.loaded.has(moduleName)) {
      return this.modules.get(moduleName).module;
    }
    
    const moduleConfig = this.modules.get(moduleName);
    
    if (!moduleConfig) {
      throw new Error(`Module not found: ${moduleName}`);
    }
    
    console.log(`Loading module: ${moduleName}`);
    
    try {
      const module = await moduleConfig.loader();
      moduleConfig.module = module.default || module;
      this.loaded.add(moduleName);
      
      return moduleConfig.module;
    } catch (error) {
      console.error(`Failed to load module ${moduleName}:`, error);
      throw error;
    }
  }
  
  prefetch(moduleName) {
    if (this.loaded.has(moduleName) || this.prefetching.has(moduleName)) {
      return;
    }
    
    this.prefetching.add(moduleName);
    
    const moduleConfig = this.modules.get(moduleName);
    
    if (!moduleConfig || !moduleConfig.prefetch) {
      return;
    }
    
    console.log(`Prefetching module: ${moduleName}`);
    
    // Use link rel="prefetch" for browser optimization
    const link = document.createElement('link');
    link.rel = 'prefetch';
    link.as = 'script';
    link.href = this.getModulePath(moduleName);
    
    document.head.appendChild(link);
    
    // Actually load the module
    if ('requestIdleCallback' in window) {
      requestIdleCallback(() => {
        this.load(moduleName);
      });
    } else {
      setTimeout(() => {
        this.load(moduleName);
      }, 100);
    }
  }
  
  preload(moduleName) {
    if (this.loaded.has(moduleName)) {
      return;
    }
    
    const moduleConfig = this.modules.get(moduleName);
    
    if (!moduleConfig) {
      return;
    }
    
    console.log(`Preloading module: ${moduleName}`);
    
    // Use link rel="preload" for critical resources
    const link = document.createElement('link');
    link.rel = 'preload';
    link.as = 'script';
    link.href = this.getModulePath(moduleName);
    
    document.head.appendChild(link);
    
    // Load immediately
    this.load(moduleName);
  }
  
  prefetchOnHover(moduleName, element) {
    let hoverTimer;
    
    element.addEventListener('mouseenter', () => {
      hoverTimer = setTimeout(() => {
        this.prefetch(moduleName);
      }, 100);
    });
    
    element.addEventListener('mouseleave', () => {
      clearTimeout(hoverTimer);
    });
  }
  
  prefetchOnVisible(moduleName, element) {
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          this.prefetch(moduleName);
          observer.unobserve(element);
        }
      });
    }, { rootMargin: '50px' });
    
    observer.observe(element);
  }
  
  getModulePath(moduleName) {
    // This would need to match your build configuration
    return `/chunks/${moduleName}.js`;
  }
}

// Usage:
const smartLoader = new SmartLoader();

// Register modules
smartLoader.register('home', () => import('./pages/Home.js'), {
  preload: true,
  priority: 'high'
});

smartLoader.register('products', () => import('./pages/Products.js'), {
  prefetch: true,
  priority: 'medium'
});

smartLoader.register('checkout', () => import('./pages/Checkout.js'), {
  prefetch: true,
  priority: 'high'
});

smartLoader.register('admin', () => import('./pages/Admin.js'), {
  prefetch: false,
  priority: 'low'
});

// Preload critical module
smartLoader.preload('home');

// Prefetch on hover
const productsLink = document.querySelector('a[href="/products"]');
smartLoader.prefetchOnHover('products', productsLink);

// Prefetch when visible
const checkoutButton = document.querySelector('.checkout-button');
smartLoader.prefetchOnVisible('checkout', checkoutButton);

// Prefetch on idle
if ('requestIdleCallback' in window) {
  requestIdleCallback(() => {
    smartLoader.prefetch('products');
    smartLoader.prefetch('checkout');
  });
}

console.log('\n=== Smart Prefetching ===');
console.log('Prefetch on hover, visible, idle');
console.log('Optimize loading performance');
```

### **Real-World Use Cases**
```javascript
/**
 * Practical code splitting implementations
 */

// 1. Modal Dialog Lazy Loading
console.log('\n=== Modal Dialog Lazy Loading ===');

class ModalManager {
  constructor() {
    this.modals = new Map();
  }
  
  register(modalName, loader) {
    this.modals.set(modalName, {
      loader,
      component: null,
      loading: false
    });
  }
  
  async show(modalName, props = {}) {
    const modal = this.modals.get(modalName);
    
    if (!modal) {
      throw new Error(`Modal not found: ${modalName}`);
    }
    
    // Load component if not loaded
    if (!modal.component && !modal.loading) {
      modal.loading = true;
      
      try {
        const module = await modal.loader();
        modal.component = module.default;
      } catch (error) {
        modal.loading = false;
        throw error;
      }
      
      modal.loading = false;
    }
    
    // Wait if loading
    while (modal.loading) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
    
    // Show modal
    if (modal.component) {
      modal.component.show(props);
    }
  }
}

/*
// Usage:
const modalManager = new ModalManager();

modalManager.register('login', () => import('./modals/LoginModal.js'));
modalManager.register('settings', () => import('./modals/SettingsModal.js'));
modalManager.register('confirm', () => import('./modals/ConfirmModal.js'));

// Show modal
document.getElementById('login-btn').addEventListener('click', () => {
  modalManager.show('login');
});

document.getElementById('settings-btn').addEventListener('click', () => {
  modalManager.show('settings');
});
*/

// 2. Tab-Based Content Loading
console.log('\n=== Tab-Based Loading ===');

class TabManager {
  constructor(containerSelector) {
    this.container = document.querySelector(containerSelector);
    this.tabs = new Map();
    this.activeTab = null;
  }
  
  addTab(tabId, label, contentLoader) {
    this.tabs.set(tabId, {
      label,
      loader: contentLoader,
      content: null,
      loaded: false
    });
  }
  
  async showTab(tabId) {
    const tab = this.tabs.get(tabId);
    
    if (!tab) return;
    
    // Load content if not loaded
    if (!tab.loaded) {
      this.showLoading();
      
      try {
        const module = await tab.loader();
        tab.content = module.default;
        tab.loaded = true;
      } catch (error) {
        this.showError(error);
        return;
      }
    }
    
    // Render content
    this.activeTab = tabId;
    this.render(tab.content);
  }
  
  showLoading() {
    this.container.innerHTML = '<div class="loading">Loading...</div>';
  }
  
  showError(error) {
    this.container.innerHTML = `<div class="error">Error: ${error.message}</div>`;
  }
  
  render(content) {
    if (typeof content === 'function') {
      this.container.innerHTML = content();
    } else {
      this.container.innerHTML = content;
    }
  }
  
  renderTabs() {
    const tabsHtml = Array.from(this.tabs.entries())
      .map(([id, tab]) => {
        const active = id === this.activeTab ? 'active' : '';
        return `<button class="tab ${active}" data-tab="${id}">${tab.label}</button>`;
      })
      .join('');
    
    const tabBar = document.createElement('div');
    tabBar.className = 'tab-bar';
    tabBar.innerHTML = tabsHtml;
    
    tabBar.addEventListener('click', (e) => {
      if (e.target.classList.contains('tab')) {
        this.showTab(e.target.dataset.tab);
      }
    });
    
    this.container.parentElement.insertBefore(tabBar, this.container);
  }
}

/*
// Usage:
const tabManager = new TabManager('#tab-content');

tabManager.addTab('overview', 'Overview', () => import('./tabs/Overview.js'));
tabManager.addTab('analytics', 'Analytics', () => import('./tabs/Analytics.js'));
tabManager.addTab('settings', 'Settings', () => import('./tabs/Settings.js'));

tabManager.renderTabs();
tabManager.showTab('overview');
*/

// 3. Viewport-Based Loading
console.log('\n=== Viewport-Based Loading ===');

class ViewportLoader {
  constructor() {
    this.components = new Map();
    this.observer = new IntersectionObserver(
      (entries) => this.handleIntersection(entries),
      { rootMargin: '100px' }
    );
  }
  
  register(element, loader) {
    this.components.set(element, {
      loader,
      loaded: false,
      loading: false
    });
    
    this.observer.observe(element);
  }
  
  handleIntersection(entries) {
    entries.forEach(async (entry) => {
      if (entry.isIntersecting) {
        const element = entry.target;
        const component = this.components.get(element);
        
        if (!component || component.loaded || component.loading) {
          return;
        }
        
        component.loading = true;
        element.innerHTML = '<div class="loading">Loading...</div>';
        
        try {
          const module = await component.loader();
          const content = module.default;
          
          element.innerHTML = typeof content === 'function' ? content() : content;
          
          component.loaded = true;
          this.observer.unobserve(element);
        } catch (error) {
          element.innerHTML = '<div class="error">Failed to load</div>';
        } finally {
          component.loading = false;
        }
      }
    });
  }
}

/*
// Usage:
const viewportLoader = new ViewportLoader();

// Register components to load when visible
const heavyCharts = document.querySelectorAll('.lazy-chart');
heavyCharts.forEach((chart) => {
  viewportLoader.register(chart, () => import('./components/Chart.js'));
});

const galleries = document.querySelectorAll('.lazy-gallery');
galleries.forEach((gallery) => {
  viewportLoader.register(gallery, () => import('./components/Gallery.js'));
});
*/

// 4. Bundle Analyzer / Size Tracker
console.log('\n=== Bundle Size Tracking ===');

class BundleTracker {
  constructor() {
    this.bundles = new Map();
    this.totalSize = 0;
    
    if (performance.getEntriesByType) {
      this.trackInitialBundles();
    }
  }
  
  trackInitialBundles() {
    const resources = performance.getEntriesByType('resource');
    
    resources
      .filter(r => r.initiatorType === 'script')
      .forEach(r => {
        this.recordBundle(r.name, r.transferSize);
      });
  }
  
  recordBundle(name, size) {
    this.bundles.set(name, {
      size,
      loadTime: performance.now()
    });
    
    this.totalSize += size;
    
    console.log(`Bundle loaded: ${name} (${this.formatSize(size)})`);
  }
  
  async trackDynamicImport(importFn) {
    const startTime = performance.now();
    
    const module = await importFn();
    
    const endTime = performance.now();
    const loadTime = endTime - startTime;
    
    // Try to find the resource
    const resources = performance.getEntriesByType('resource');
    const recentScript = resources
      .filter(r => r.initiatorType === 'script')
      .filter(r => r.startTime >= startTime)
      .pop();
    
    if (recentScript) {
      this.recordBundle(recentScript.name, recentScript.transferSize);
    }
    
    console.log(`Dynamic import completed in ${loadTime.toFixed(2)}ms`);
    
    return module;
  }
  
  formatSize(bytes) {
    if (bytes < 1024) return bytes + ' B';
    if (bytes < 1024 * 1024) return (bytes / 1024).toFixed(2) + ' KB';
    return (bytes / (1024 * 1024)).toFixed(2) + ' MB';
  }
  
  getReport() {
    return {
      totalBundles: this.bundles.size,
      totalSize: this.formatSize(this.totalSize),
      bundles: Array.from(this.bundles.entries()).map(([name, data]) => ({
        name,
        size: this.formatSize(data.size),
        loadTime: data.loadTime.toFixed(2) + 'ms'
      }))
    };
  }
  
  printReport() {
    const report = this.getReport();
    
    console.log('\n=== Bundle Report ===');
    console.log(`Total Bundles: ${report.totalBundles}`);
    console.log(`Total Size: ${report.totalSize}`);
    console.log('\nBundles:');
    
    report.bundles.forEach(bundle => {
      console.log(`  ${bundle.name}: ${bundle.size} (loaded at ${bundle.loadTime})`);
    });
  }
}

/*
// Usage:
const bundleTracker = new BundleTracker();

// Track dynamic imports
const module = await bundleTracker.trackDynamicImport(
  () => import('./heavy-module.js')
);

// Print report
setTimeout(() => {
  bundleTracker.printReport();
}, 5000);
*/

// 5. Progressive Enhancement with Code Splitting
console.log('\n=== Progressive Enhancement ===');

class ProgressiveApp {
  constructor() {
    this.features = {
      core: null,
      enhanced: null,
      advanced: null
    };
  }
  
  async init() {
    // Always load core features
    await this.loadCore();
    
    // Load enhanced features on capable devices
    if (this.isCapableDevice()) {
      await this.loadEnhanced();
    }
    
    // Load advanced features on high-end devices
    if (this.isHighEndDevice()) {
      await this.loadAdvanced();
    }
  }
  
  async loadCore() {
    console.log('Loading core features...');
    const module = await import('./features/core.js');
    this.features.core = module.default;
    this.features.core.init();
  }
  
  async loadEnhanced() {
    console.log('Loading enhanced features...');
    const module = await import('./features/enhanced.js');
    this.features.enhanced = module.default;
    this.features.enhanced.init();
  }
  
  async loadAdvanced() {
    console.log('Loading advanced features...');
    const module = await import('./features/advanced.js');
    this.features.advanced = module.default;
    this.features.advanced.init();
  }
  
  isCapableDevice() {
    return (
      'IntersectionObserver' in window &&
      'fetch' in window &&
      navigator.hardwareConcurrency >= 4
    );
  }
  
  isHighEndDevice() {
    return (
      navigator.hardwareConcurrency >= 8 &&
      navigator.deviceMemory >= 4
    );
  }
}

/*
// Usage:
const app = new ProgressiveApp();
app.init();
*/
```

### **Performance Comparison**
```javascript
console.log('\n=== Performance ===');

console.log(`
Code Splitting Strategies:
┌──────────────────────┬────────────────────────────────┐
│ Strategy             │ Use Case                       │
├──────────────────────┼────────────────────────────────┤
│ Route-Based          │ Load page components per route │
│                      │ Best for SPAs                  │
├──────────────────────┼────────────────────────────────┤
│ Component-Based      │ Load heavy components on demand│
│                      │ Modals, charts, editors        │
├──────────────────────┼────────────────────────────────┤
│ Feature-Based        │ Load features conditionally    │
│                      │ Admin panels, premium features │
├──────────────────────┼────────────────────────────────┤
│ Vendor Splitting     │ Separate vendor/library code   │
│                      │ Better caching                 │
├──────────────────────┼────────────────────────────────┤
│ Dynamic Imports      │ Load anything on demand        │
│                      │ Returns Promise<module>        │
└──────────────────────┴────────────────────────────────┘

Key Concepts:
• Dynamic import(): ES module syntax, returns Promise
• Code splitting: Break bundle into smaller chunks
• Lazy loading: Load code only when needed
• Webpack/Vite: Automatic chunk creation
• import() creates separate bundle chunk
• Tree shaking: Remove unused code

Benefits:
• Faster initial load (50-80% reduction)
• Smaller bundle size
• Better caching (vendor chunks)
• Improved TTI (Time to Interactive)
• Pay-for-what-you-use
• Better mobile experience

Dynamic Import Syntax:
• import('./module.js') - Returns Promise
• await import('./module.js') - Async/await
• .then(module => module.default)
• .catch(error => handle error)
• Named exports: module.namedExport
• Default export: module.default

Webpack Magic Comments:
• /* webpackChunkName: "my-chunk" */
• /* webpackPrefetch: true */
• /* webpackPreload: true */
• /* webpackMode: "lazy" */
• Example: import(/* webpackChunkName: "lodash" */ 'lodash')

Prefetch vs Preload:
• Prefetch: Low priority, load when idle
• Preload: High priority, load immediately
• <link rel="prefetch"> for future navigation
• <link rel="preload"> for current page

Best Practices:
• Split by routes (page level)
• Split heavy components (charts, editors)
• Keep common code in vendor bundle
• Use magic comments for chunk names
• Prefetch likely-needed modules
• Show loading states
• Handle errors gracefully
• Monitor bundle sizes

Route-Based Splitting:
• Each route = separate bundle
• Loaded when navigating
• Best initial load improvement
• Example: Home, About, Products pages

Component-Based Splitting:
• Heavy components loaded on demand
• Modals, dialogs, tooltips
• Charts, graphs, visualizations
• Video players, editors

Conditional Loading:
• Load based on user role
• Load based on device capabilities
• Load based on feature flags
• Progressive enhancement

Bundle Size Impact:
• Initial bundle: 70-90% smaller
• Total bandwidth: Same or slightly more
• User experience: Much better
• Time to Interactive: 50-80% faster

Common Patterns:
• React.lazy() + Suspense
• Vue async components
• Angular lazy modules
• Webpack splitChunks
• Dynamic import() everywhere

Loading States:
• Show spinner while loading
• Skeleton screens
• Progressive content reveal
• Error boundaries for failures

Error Handling:
• Try/catch for async imports
• Fallback UI for failures
• Retry logic
• Report errors to monitoring

Performance Metrics:
• Initial bundle size
• Time to Interactive (TTI)
• First Contentful Paint (FCP)
• Largest Contentful Paint (LCP)
• Total Blocking Time (TBT)

Build Configuration:
• Webpack: splitChunks optimization
• Vite: manualChunks configuration
• Rollup: output.manualChunks
• Configure chunk size limits

Caching Strategy:
• Vendor chunks: Long cache time
• App chunks: Version in filename
• Content hashing: [contenthash]
• Immutable chunks for better caching

Real-World Examples:
• Twitter: Route-based splitting
• Facebook: Aggressive code splitting
• Netflix: Component lazy loading
• Amazon: Feature-based splitting
• Google: Dynamic feature loading

Testing:
• Test with slow network
• Test bundle sizes
• Test loading states
• Test error scenarios
• Lighthouse audit

Common Pitfalls:
• Too many chunks (overhead)
• Not showing loading states
• Not handling errors
• Splitting too aggressively
• Forgetting to prefetch

Browser Support:
• Dynamic import(): 95%+
• Modern browsers only
• Fallback: bundle everything
• Babel transforms for older browsers

Optimization Tips:
• Group related code together
• Use vendor chunks for libraries
• Prefetch on hover/visible
• Lazy load below fold content
• Monitor with analytics

Tools:
• Webpack Bundle Analyzer
• Vite Bundle Visualizer
• Chrome Coverage tool
• Lighthouse performance audit
• bundlephobia.com for package sizes
`);
```

**Interview Tips:**
- Code splitting: break bundle into smaller chunks loaded on demand
- Dynamic import(): ES module syntax, import('./module.js') returns Promise
- Benefits: 50-80% faster initial load, smaller bundle size, better mobile experience
- Route-based: split by pages/routes, best for SPAs
- Component-based: lazy load heavy components (modals, charts, editors)
- Feature-based: conditional loading based on user role, device, permissions
- Vendor splitting: separate vendor/library code, better caching
- import() creates separate bundle chunk automatically (Webpack/Vite)
- await import('./module.js'): use with async/await
- module.default: access default export from dynamic import
- module.namedExport: access named exports
- Loading states: show spinner/skeleton while module loads
- Error handling: try/catch for failed imports, retry logic
- Webpack magic comments: /* webpackChunkName: "name" */
- Prefetch: load on idle (link rel="prefetch")
- Preload: load immediately (link rel="preload")
- Prefetch on hover: start loading when user hovers over link
- IntersectionObserver: load when component enters viewport
- React.lazy() + Suspense: React's built-in code splitting
- splitChunks: Webpack optimization for automatic splitting
- Content hashing: [contenthash] in filename for cache busting
- Bundle analyzer: visualize and optimize chunk sizes
- Real-world: Twitter routes, Facebook components, Netflix lazy loading
- Performance: measure with Lighthouse, Core Web Vitals, bundle size
- Trade-offs: more chunks = more HTTP requests, need balance
- Best practice: split by routes + heavy components + vendor
- Follow-ups: bundle optimization, caching strategy, error handling
- Clarify: framework used? build tool? performance targets?

</details>

---

## **Question Categories Overview**

### **By Topic:**
- **Arrays & Strings**: Questions 1-10, 31-50
- **Basic Algorithms**: Questions 11-20
- **Objects & Data Structures**: Questions 21-30, 61-65, 96-105
- **DOM & Browser**: Questions 51-60
- **Asynchronous Programming**: Questions 76-85
- **Advanced Functions**: Questions 86-95
- **Graph & Tree Algorithms**: Questions 96-109
- **Problem Solving**: Questions 106-115
- **Real-World Scenarios**: Questions 116-130

### **By Difficulty Distribution:**
- **Easy**: 30 questions (foundational concepts)
- **Medium**: 35 questions (practical applications)
- **Hard**: 30 questions (advanced concepts)
- **Extra Hard**: 25 questions (expert level)
- **Bonus**: 10 questions (framework-specific)

**Total: 130 Coding Questions**

---
## **Common Patterns to Master**

- **Two Pointers**: Array/string problems
- **Sliding Window**: Substring/subarray problems
- **Fast & Slow Pointers**: Linked list cycle detection
- **Recursion & Backtracking**: Permutations, combinations
- **Dynamic Programming**: Optimization problems
- **Binary Search**: Sorted array problems
- **BFS/DFS**: Tree and graph traversal
- **Hash Tables**: Frequency counting, lookups
- **Stack/Queue**: Parentheses matching, level-order traversal
- **Divide & Conquer**: Sorting, searching

---

## **Resources for Practice**

- **LeetCode**: leetcode.com (Focus on Easy & Medium)
- **HackerRank**: hackerrank.com/domains/javascript
- **CodeWars**: codewars.com
- **Exercism**: exercism.io/tracks/javascript
- **JavaScript30**: javascript30.com (Practical projects)
- **Frontend Mentor**: frontendmentor.io (UI challenges)

---

