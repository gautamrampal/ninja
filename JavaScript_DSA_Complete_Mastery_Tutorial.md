# JavaScript Data Structures & Algorithms - Complete Mastery Tutorial

## Table of Contents
- [Part 1: Fundamentals & Beginner Level](#part-1-fundamentals--beginner-level)
- [Part 2: Intermediate Data Structures](#part-2-intermediate-data-structures)
- [Part 3: Advanced Data Structures](#part-3-advanced-data-structures)
- [Part 4: Algorithm Design Paradigms](#part-4-algorithm-design-paradigms)
- [Part 5: Advanced Algorithms](#part-5-advanced-algorithms)
- [Part 6: Real-World Applications](#part-6-real-world-applications)

---

## Part 1: Fundamentals & Beginner Level

### 1.1 JavaScript Language Essentials for DSA

#### 1.1.1 Time and Space Complexity Analysis

**Big O Notation Basics:**

Big O notation describes the performance or complexity of an algorithm. It specifically describes the worst-case scenario and helps us understand how the runtime or space requirements grow as the input size increases.

```javascript
// O(1) - Constant Time
// Description: No matter how large the array is, accessing the first element
// always takes the same amount of time. This is the most efficient complexity.
function getFirstElement(arr) {
    return arr[0];  // Direct array access is O(1)
}

// O(n) - Linear Time
// Description: In the worst case, we need to check every element in the array.
// If the array has n elements, we perform n operations. The runtime grows
// linearly with input size.
function findElement(arr, target) {
    // Loop through each element - worst case: n iterations
    for (let i = 0; i < arr.length; i++) {
        if (arr[i] === target) return i;  // Best case: element found at index 0
    }
    return -1;  // Element not found after checking all n elements
}

// O(n²) - Quadratic Time
// Description: Nested loops cause quadratic complexity. For an array of size n,
// the outer loop runs n times and for each iteration, the inner loop runs
// up to n times, resulting in n × n = n² operations.
function bubbleSort(arr) {
    const n = arr.length;
    // Outer loop: runs n times
    for (let i = 0; i < n; i++) {
        // Inner loop: runs (n - i - 1) times for each outer iteration
        // Total operations: n × n = n²
        for (let j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                // Swap adjacent elements if they're in wrong order
                [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
            }
        }
    }
    return arr;
}

// O(log n) - Logarithmic Time
// Description: Each iteration eliminates half of the remaining elements.
// For an array of size n, we need at most log₂(n) iterations.
// Example: array of 1000 elements needs only ~10 iterations (2^10 = 1024)
function binarySearch(arr, target) {
    let left = 0, right = arr.length - 1;
    
    // Continue while search space is valid
    while (left <= right) {
        // Find middle element (avoids overflow compared to (left + right) / 2)
        const mid = Math.floor((left + right) / 2);
        if (arr[mid] === target) return mid;  // Found!
        if (arr[mid] < target) left = mid + 1;  // Search right half
        else right = mid - 1;  // Search left half
        // Each iteration cuts search space in half: n, n/2, n/4, n/8, ...
    }
    return -1;  // Not found
}

// O(n log n) - Linearithmic Time
// Description: Divide and conquer algorithm. We divide the array log(n) times,
// and at each level, we do O(n) work to merge. Total: O(n) × O(log n) = O(n log n)
function mergeSort(arr) {
    // Base case: array with 0 or 1 element is already sorted
    if (arr.length <= 1) return arr;
    
    // Divide: split array into two halves
    const mid = Math.floor(arr.length / 2);
    const left = mergeSort(arr.slice(0, mid));   // Recursively sort left half
    const right = mergeSort(arr.slice(mid));     // Recursively sort right half
    
    // Conquer: merge the sorted halves
    return merge(left, right);
}

function merge(left, right) {
    const result = [];
    let i = 0, j = 0;
    
    // Merge two sorted arrays into one sorted array
    // This takes O(n) time where n = left.length + right.length
    while (i < left.length && j < right.length) {
        if (left[i] <= right[j]) {
            result.push(left[i++]);  // Take from left
        } else {
            result.push(right[j++]); // Take from right
        }
    }
    
    // Append remaining elements (if any)
    return result.concat(left.slice(i)).concat(right.slice(j));
}
```

**Space Complexity Examples:**

Space complexity measures the amount of memory an algorithm uses relative to input size.

```javascript
// O(1) Space - Constant
// Description: Uses only a fixed amount of extra memory (the 'sum' variable)
// regardless of input array size. This is optimal space usage.
function sumArray(arr) {
    let sum = 0;  // Only one variable - constant space O(1)
    for (let num of arr) {
        sum += num;  // Reusing the same variable
    }
    return sum;
}

// O(n) Space - Linear
// Description: Creates a new array of the same size as input.
// If input has n elements, we use n extra spaces.
function reverseArray(arr) {
    const reversed = [];  // New array that will grow to size n
    for (let i = arr.length - 1; i >= 0; i--) {
        reversed.push(arr[i]);  // Adding n elements to new array
    }
    return reversed;  // Space used = O(n)
}

// O(n) Space - Recursive call stack
// Description: Each recursive call adds a frame to the call stack.
// For factorial(n), we have n recursive calls before reaching base case.
// Stack depth = n, therefore space complexity = O(n)
function factorial(n) {
    if (n <= 1) return 1;  // Base case - stops recursion
    return n * factorial(n - 1);  // Each call adds to stack
    // Call stack: factorial(5) -> factorial(4) -> factorial(3) -> factorial(2) -> factorial(1)
    // Maximum depth = n calls on the stack simultaneously
}
```

#### 1.1.2 JavaScript Memory Management

**Understanding Stack vs Heap:**
- **Stack**: Fast access, fixed size, stores primitive values and references
- **Heap**: Slower access, dynamic size, stores objects and arrays

```javascript
// Primitive types (stored in stack)
// Description: Primitives are stored directly in the stack memory.
// They are copied by value when assigned to another variable.
let num = 42;           // Number primitive
let str = "hello";      // String primitive
let bool = true;        // Boolean primitive

// Reference types (stored in heap)
// Description: Objects and arrays are stored in heap memory.
// Variables hold references (memory addresses) to the actual data.
let obj = { name: "Alice" };  // Object stored in heap, reference in stack
let arr = [1, 2, 3];          // Array stored in heap, reference in stack

// Shallow vs Deep Copy
const original = { a: 1, b: { c: 2 } };

// Shallow copy - only copies first level
// Description: Spread operator creates a new object but nested objects
// are still referenced, not copied.
const shallow = { ...original };
shallow.b.c = 3; // ⚠️ Affects original! Because 'b' is still a reference
// shallow.b and original.b point to the same object in memory

// Deep copy - copies all levels recursively
// Description: JSON method creates completely independent copy.
// Limitation: Doesn't work with functions, undefined, Date objects, etc.
const deep = JSON.parse(JSON.stringify(original));
deep.b.c = 4; // ✅ Doesn't affect original - completely separate object

// Better deep copy for complex objects
// Description: Recursive function that handles various data types properly
// including nested objects, arrays, and Date objects.
function deepClone(obj) {
    // Base case: primitive values and null
    if (obj === null || typeof obj !== 'object') return obj;
    
    // Handle Date objects specially
    if (obj instanceof Date) return new Date(obj);
    
    // Handle arrays recursively
    if (obj instanceof Array) return obj.map(item => deepClone(item));
    
    // Handle plain objects
    const cloned = {};
    for (let key in obj) {
        // Only clone own properties (not inherited)
        if (obj.hasOwnProperty(key)) {
            cloned[key] = deepClone(obj[key]);  // Recursive call for nested objects
        }
    }
    return cloned;
}
```

### 1.2 Arrays and Strings

#### 1.2.1 Array Operations

**Core Concept**: Arrays are contiguous memory blocks with O(1) index access but O(n) insertion/deletion (except at end).

```javascript
class ArrayOperations {
    // Insert at position
    // Time: O(n) - need to shift all elements after insertion point
    // Space: O(1) - in-place modification
    static insertAt(arr, index, element) {
        // splice modifies array in-place, shifting elements right
        arr.splice(index, 0, element);
        return arr;
    }
    
    // Delete at position
    // Time: O(n) - need to shift all elements after deletion point left
    // Space: O(1) - in-place modification
    static deleteAt(arr, index) {
        // Remove 1 element at index, shifts remaining elements left
        arr.splice(index, 1);
        return arr;
    }
    
    // Find element
    // Time: O(n) - worst case: check every element
    // Space: O(1) - only use index variable
    static find(arr, target) {
        // indexOf internally loops through array
        return arr.indexOf(target);  // Returns -1 if not found
    }
    
    // Rotate array k positions to right
    // Time: O(n) - three O(n) reversals
    // Space: O(1) - in-place rotation using reversal algorithm
    // Example: [1,2,3,4,5], k=2 -> [4,5,1,2,3]
    static rotateRight(arr, k) {
        k = k % arr.length;  // Handle k > array length
        // Algorithm: reverse entire array, then reverse first k, then reverse rest
        // [1,2,3,4,5] -> [5,4,3,2,1] -> [4,5,3,2,1] -> [4,5,1,2,3]
        this.reverse(arr, 0, arr.length - 1);  // Reverse all
        this.reverse(arr, 0, k - 1);           // Reverse first k
        this.reverse(arr, k, arr.length - 1);  // Reverse remaining
        return arr;
    }
    
    // Helper: reverse portion of array in-place
    static reverse(arr, start, end) {
        while (start < end) {
            // Swap elements from both ends moving toward center
            [arr[start], arr[end]] = [arr[end], arr[start]];
            start++;
            end--;
        }
    }
    
    // Remove duplicates from sorted array
    // Time: O(n) - single pass through array
    // Space: O(1) - two-pointer technique, in-place
    // Returns new length of array without duplicates
    static removeDuplicates(arr) {
        if (arr.length === 0) return 0;
        
        // Two-pointer technique: i tracks unique elements position
        let i = 0;
        for (let j = 1; j < arr.length; j++) {
            // When we find a new unique element
            if (arr[j] !== arr[i]) {
                i++;              // Move unique pointer forward
                arr[i] = arr[j];  // Place unique element at position i
            }
            // Skip duplicates (when arr[j] === arr[i])
        }
        return i + 1;  // Length of array with unique elements
    }
    
    // Find maximum subarray sum (Kadane's Algorithm)
    // Time: O(n) - single pass
    // Space: O(1) - only two variables
    // Classic DP problem: finds contiguous subarray with largest sum
    static maxSubarraySum(arr) {
        let maxSum = arr[0];      // Best sum found so far
        let currentSum = arr[0];  // Sum of current subarray
        
        for (let i = 1; i < arr.length; i++) {
            // Key decision: extend current subarray or start new one?
            // If current sum is negative, it's better to start fresh
            currentSum = Math.max(arr[i], currentSum + arr[i]);
            maxSum = Math.max(maxSum, currentSum);  // Track maximum
        }
        
        return maxSum;
    }
    
    // Two Sum Problem
    // Time: O(n) - single pass with hash map
    // Space: O(n) - hash map stores at most n elements
    // Given array and target, find two numbers that add up to target
    static twoSum(arr, target) {
        const map = new Map();  // Store number -> index mapping
        
        for (let i = 0; i < arr.length; i++) {
            const complement = target - arr[i];  // What we need to find
            
            // Check if complement exists in our map (seen before)
            if (map.has(complement)) {
                return [map.get(complement), i];  // Found pair!
            }
            // Store current number for future lookups
            map.set(arr[i], i);
        }
        
        return null;  // No pair found
    }
    
    // Three Sum Problem
    // Time: O(n²) - O(n log n) for sort + O(n²) for two-pointer search
    // Space: O(1) - excluding output array
    // Find all unique triplets that sum to zero
    static threeSum(arr) {
        arr.sort((a, b) => a - b);  // Sort first: O(n log n)
        const result = [];
        
        for (let i = 0; i < arr.length - 2; i++) {
            // Skip duplicate values for first element
            if (i > 0 && arr[i] === arr[i - 1]) continue;
            
            // Two-pointer approach for remaining two elements
            let left = i + 1;
            let right = arr.length - 1;
            
            while (left < right) {
                const sum = arr[i] + arr[left] + arr[right];
                
                if (sum === 0) {
                    result.push([arr[i], arr[left], arr[right]]);
                    // Skip duplicates for second element
                    while (left < right && arr[left] === arr[left + 1]) left++;
                    // Skip duplicates for third element
                    while (left < right && arr[right] === arr[right - 1]) right--;
                    left++;
                    right--;
                } else if (sum < 0) {
                    left++;   // Need larger sum
                } else {
                    right--;  // Need smaller sum
                }
            }
        }
        
        return result;
    }
    
    // Container with most water
    static maxArea(heights) {
        // Time: O(n), Space: O(1)
        let maxArea = 0;
        let left = 0;
        let right = heights.length - 1;
        
        while (left < right) {
            const width = right - left;
            const height = Math.min(heights[left], heights[right]);
            maxArea = Math.max(maxArea, width * height);
            
            if (heights[left] < heights[right]) {
                left++;
            } else {
                right--;
            }
        }
        
        return maxArea;
    }
    
    // Product of array except self
    static productExceptSelf(nums) {
        // Time: O(n), Space: O(1) excluding output
        const result = new Array(nums.length);
        
        // Left pass
        result[0] = 1;
        for (let i = 1; i < nums.length; i++) {
            result[i] = result[i - 1] * nums[i - 1];
        }
        
        // Right pass
        let right = 1;
        for (let i = nums.length - 1; i >= 0; i--) {
            result[i] *= right;
            right *= nums[i];
        }
        
        return result;
    }
}

// Testing
console.log(ArrayOperations.maxSubarraySum([-2, 1, -3, 4, -1, 2, 1, -5, 4])); // 6
console.log(ArrayOperations.twoSum([2, 7, 11, 15], 9)); // [0, 1]
console.log(ArrayOperations.threeSum([-1, 0, 1, 2, -1, -4])); // [[-1, -1, 2], [-1, 0, 1]]
```

#### 1.2.2 String Algorithms

**Core Concept**: Strings in JavaScript are immutable. Most string operations create new strings.

```javascript
class StringAlgorithms {
    // Check if string is palindrome
    // Time: O(n) - compare n/2 pairs of characters
    // Space: O(1) - only use two pointers
    // Palindrome: reads same forwards and backwards (e.g., "racecar")
    static isPalindrome(s) {
        let left = 0;              // Start from beginning
        let right = s.length - 1;  // Start from end
        
        // Compare characters from both ends moving toward center
        while (left < right) {
            if (s[left] !== s[right]) return false;  // Mismatch found
            left++;
            right--;
        }
        
        return true;  // All pairs matched
    }
    
    // Longest palindromic substring
    // Time: O(n²) - for each center, expand takes O(n)
    // Space: O(1) - only store indices
    // Approach: Expand around each possible center (2n-1 centers)
    static longestPalindrome(s) {
        if (s.length < 2) return s;
        
        let start = 0;    // Start index of longest palindrome found
        let maxLen = 0;   // Length of longest palindrome
        
        // Helper: expand around center and return length
        const expandAroundCenter = (left, right) => {
            // Keep expanding while characters match and in bounds
            while (left >= 0 && right < s.length && s[left] === s[right]) {
                left--;
                right++;
            }
            return right - left - 1;  // Length of palindrome
        };
        
        for (let i = 0; i < s.length; i++) {
            // Check for odd-length palindrome (single character center)
            const len1 = expandAroundCenter(i, i);
            // Check for even-length palindrome (between two characters)
            const len2 = expandAroundCenter(i, i + 1);
            const len = Math.max(len1, len2);
            
            if (len > maxLen) {
                maxLen = len;
                // Calculate start position from center and length
                start = i - Math.floor((len - 1) / 2);
            }
        }
        
        return s.substring(start, start + maxLen);
    }
    
    // Anagram check
    static isAnagram(s, t) {
        // Time: O(n), Space: O(1) - limited to 26 characters
        if (s.length !== t.length) return false;
        
        const count = new Array(26).fill(0);
        
        for (let i = 0; i < s.length; i++) {
            count[s.charCodeAt(i) - 97]++;
            count[t.charCodeAt(i) - 97]--;
        }
        
        return count.every(c => c === 0);
    }
    
    // Group anagrams
    static groupAnagrams(strs) {
        // Time: O(n * k), Space: O(n * k)
        const map = new Map();
        
        for (let str of strs) {
            const count = new Array(26).fill(0);
            for (let char of str) {
                count[char.charCodeAt(0) - 97]++;
            }
            const key = count.join('#');
            
            if (!map.has(key)) {
                map.set(key, []);
            }
            map.get(key).push(str);
        }
        
        return Array.from(map.values());
    }
    
    // Longest substring without repeating characters
    static lengthOfLongestSubstring(s) {
        // Time: O(n), Space: O(min(n, m)) where m is charset size
        const map = new Map();
        let maxLen = 0;
        let left = 0;
        
        for (let right = 0; right < s.length; right++) {
            if (map.has(s[right])) {
                left = Math.max(left, map.get(s[right]) + 1);
            }
            map.set(s[right], right);
            maxLen = Math.max(maxLen, right - left + 1);
        }
        
        return maxLen;
    }
    
    // KMP Pattern Matching
    // Time: O(n + m) where n=text length, m=pattern length
    // Space: O(m) for LPS array
    // KMP avoids re-checking characters we already know match
    static kmpSearch(text, pattern) {
        if (pattern.length === 0) return 0;
        
        const lps = this.computeLPS(pattern);  // Longest Proper Prefix-Suffix
        const matches = [];
        let i = 0; // text index
        let j = 0; // pattern index
        
        while (i < text.length) {
            if (text[i] === pattern[j]) {
                i++;
                j++;
            }
            
            if (j === pattern.length) {
                // Found complete match!
                matches.push(i - j);
                j = lps[j - 1];  // Continue searching with prefix knowledge
            } else if (i < text.length && text[i] !== pattern[j]) {
                // Mismatch: use LPS to skip redundant comparisons
                if (j !== 0) {
                    j = lps[j - 1];  // Jump based on longest prefix-suffix
                } else {
                    i++;  // No prefix to use, move to next character
                }
            }
        }
        
        return matches;
    }
    
    // Compute Longest Proper Prefix which is also Suffix array
    // This preprocessing enables efficient pattern matching
    static computeLPS(pattern) {
        const lps = new Array(pattern.length).fill(0);
        let len = 0;  // Length of previous longest prefix suffix
        let i = 1;
        
        while (i < pattern.length) {
            if (pattern[i] === pattern[len]) {
                len++;
                lps[i] = len;
                i++;
            } else {
                if (len !== 0) {
                    len = lps[len - 1];  // Fallback to previous prefix
                } else {
                    lps[i] = 0;
                    i++;
                }
            }
        }
        
        return lps;
    }
    
    // Rabin-Karp Algorithm
    static rabinKarp(text, pattern) {
        // Time: O(n + m) average, Space: O(1)
        const d = 256; // Number of characters in alphabet
        const q = 101; // A prime number
        const m = pattern.length;
        const n = text.length;
        let p = 0; // hash value for pattern
        let t = 0; // hash value for text
        let h = 1;
        const matches = [];
        
        // Calculate h = d^(m-1) % q
        for (let i = 0; i < m - 1; i++) {
            h = (h * d) % q;
        }
        
        // Calculate initial hash values
        for (let i = 0; i < m; i++) {
            p = (d * p + pattern.charCodeAt(i)) % q;
            t = (d * t + text.charCodeAt(i)) % q;
        }
        
        // Slide pattern over text
        for (let i = 0; i <= n - m; i++) {
            if (p === t) {
                // Check for characters one by one
                let j;
                for (j = 0; j < m; j++) {
                    if (text[i + j] !== pattern[j]) break;
                }
                if (j === m) matches.push(i);
            }
            
            // Calculate hash for next window
            if (i < n - m) {
                t = (d * (t - text.charCodeAt(i) * h) + text.charCodeAt(i + m)) % q;
                if (t < 0) t += q;
            }
        }
        
        return matches;
    }
    
    // Minimum window substring
    static minWindow(s, t) {
        // Time: O(n), Space: O(m)
        if (s.length === 0 || t.length === 0) return "";
        
        const dictT = new Map();
        for (let char of t) {
            dictT.set(char, (dictT.get(char) || 0) + 1);
        }
        
        let required = dictT.size;
        let left = 0, right = 0;
        let formed = 0;
        const windowCounts = new Map();
        let ans = [Infinity, null, null];
        
        while (right < s.length) {
            let char = s[right];
            windowCounts.set(char, (windowCounts.get(char) || 0) + 1);
            
            if (dictT.has(char) && windowCounts.get(char) === dictT.get(char)) {
                formed++;
            }
            
            while (left <= right && formed === required) {
                char = s[left];
                
                if (right - left + 1 < ans[0]) {
                    ans = [right - left + 1, left, right];
                }
                
                windowCounts.set(char, windowCounts.get(char) - 1);
                if (dictT.has(char) && windowCounts.get(char) < dictT.get(char)) {
                    formed--;
                }
                
                left++;
            }
            
            right++;
        }
        
        return ans[0] === Infinity ? "" : s.substring(ans[1], ans[2] + 1);
    }
    
    // Edit Distance (Levenshtein Distance)
    static editDistance(word1, word2) {
        // Time: O(m * n), Space: O(m * n)
        const m = word1.length;
        const n = word2.length;
        const dp = Array(m + 1).fill(null).map(() => Array(n + 1).fill(0));
        
        for (let i = 0; i <= m; i++) dp[i][0] = i;
        for (let j = 0; j <= n; j++) dp[0][j] = j;
        
        for (let i = 1; i <= m; i++) {
            for (let j = 1; j <= n; j++) {
                if (word1[i - 1] === word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = 1 + Math.min(
                        dp[i - 1][j],     // delete
                        dp[i][j - 1],     // insert
                        dp[i - 1][j - 1]  // replace
                    );
                }
            }
        }
        
        return dp[m][n];
    }
}

// Testing
console.log(StringAlgorithms.longestPalindrome("babad")); // "bab" or "aba"
console.log(StringAlgorithms.lengthOfLongestSubstring("abcabcbb")); // 3
console.log(StringAlgorithms.kmpSearch("ABABDABACDABABCABAB", "ABABCABAB")); // [10]
```

### 1.3 Searching Algorithms

```javascript
class SearchAlgorithms {
    // Linear Search
    static linearSearch(arr, target) {
        // Time: O(n), Space: O(1)
        for (let i = 0; i < arr.length; i++) {
            if (arr[i] === target) return i;
        }
        return -1;
    }
    
    // Binary Search (Iterative)
    static binarySearch(arr, target) {
        // Time: O(log n), Space: O(1)
        let left = 0;
        let right = arr.length - 1;
        
        while (left <= right) {
            const mid = Math.floor((left + right) / 2);
            
            if (arr[mid] === target) return mid;
            if (arr[mid] < target) left = mid + 1;
            else right = mid - 1;
        }
        
        return -1;
    }
    
    // Binary Search (Recursive)
    static binarySearchRecursive(arr, target, left = 0, right = arr.length - 1) {
        // Time: O(log n), Space: O(log n)
        if (left > right) return -1;
        
        const mid = Math.floor((left + right) / 2);
        
        if (arr[mid] === target) return mid;
        if (arr[mid] < target) {
            return this.binarySearchRecursive(arr, target, mid + 1, right);
        }
        return this.binarySearchRecursive(arr, target, left, mid - 1);
    }
    
    // Find first and last position of element
    static searchRange(arr, target) {
        // Time: O(log n), Space: O(1)
        const findFirst = () => {
            let left = 0, right = arr.length - 1;
            let result = -1;
            
            while (left <= right) {
                const mid = Math.floor((left + right) / 2);
                if (arr[mid] === target) {
                    result = mid;
                    right = mid - 1;
                } else if (arr[mid] < target) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
            
            return result;
        };
        
        const findLast = () => {
            let left = 0, right = arr.length - 1;
            let result = -1;
            
            while (left <= right) {
                const mid = Math.floor((left + right) / 2);
                if (arr[mid] === target) {
                    result = mid;
                    left = mid + 1;
                } else if (arr[mid] < target) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
            
            return result;
        };
        
        return [findFirst(), findLast()];
    }
    
    // Search in rotated sorted array
    static searchRotated(arr, target) {
        // Time: O(log n), Space: O(1)
        let left = 0;
        let right = arr.length - 1;
        
        while (left <= right) {
            const mid = Math.floor((left + right) / 2);
            
            if (arr[mid] === target) return mid;
            
            // Left half is sorted
            if (arr[left] <= arr[mid]) {
                if (target >= arr[left] && target < arr[mid]) {
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }
            }
            // Right half is sorted
            else {
                if (target > arr[mid] && target <= arr[right]) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }
        
        return -1;
    }
    
    // Find peak element
    static findPeakElement(arr) {
        // Time: O(log n), Space: O(1)
        let left = 0;
        let right = arr.length - 1;
        
        while (left < right) {
            const mid = Math.floor((left + right) / 2);
            
            if (arr[mid] < arr[mid + 1]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        
        return left;
    }
    
    // Square root using binary search
    static sqrt(x) {
        // Time: O(log n), Space: O(1)
        if (x < 2) return x;
        
        let left = 1;
        let right = Math.floor(x / 2);
        
        while (left <= right) {
            const mid = Math.floor((left + right) / 2);
            const squared = mid * mid;
            
            if (squared === x) return mid;
            if (squared < x) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return right;
    }
    
    // Interpolation Search
    static interpolationSearch(arr, target) {
        // Time: O(log log n) average, O(n) worst, Space: O(1)
        let low = 0;
        let high = arr.length - 1;
        
        while (low <= high && target >= arr[low] && target <= arr[high]) {
            if (low === high) {
                if (arr[low] === target) return low;
                return -1;
            }
            
            const pos = low + Math.floor(
                ((target - arr[low]) * (high - low)) / (arr[high] - arr[low])
            );
            
            if (arr[pos] === target) return pos;
            if (arr[pos] < target) low = pos + 1;
            else high = pos - 1;
        }
        
        return -1;
    }
    
    // Exponential Search
    static exponentialSearch(arr, target) {
        // Time: O(log n), Space: O(1)
        if (arr[0] === target) return 0;
        
        let i = 1;
        while (i < arr.length && arr[i] <= target) {
            i *= 2;
        }
        
        return this.binarySearch(
            arr.slice(Math.floor(i / 2), Math.min(i, arr.length)),
            target
        ) + Math.floor(i / 2);
    }
    
    // Ternary Search
    static ternarySearch(arr, target, left = 0, right = arr.length - 1) {
        // Time: O(log₃ n), Space: O(log n)
        if (left > right) return -1;
        
        const mid1 = left + Math.floor((right - left) / 3);
        const mid2 = right - Math.floor((right - left) / 3);
        
        if (arr[mid1] === target) return mid1;
        if (arr[mid2] === target) return mid2;
        
        if (target < arr[mid1]) {
            return this.ternarySearch(arr, target, left, mid1 - 1);
        } else if (target > arr[mid2]) {
            return this.ternarySearch(arr, target, mid2 + 1, right);
        } else {
            return this.ternarySearch(arr, target, mid1 + 1, mid2 - 1);
        }
    }
}

// Testing
const sortedArr = [1, 3, 5, 7, 9, 11, 13, 15];
console.log(SearchAlgorithms.binarySearch(sortedArr, 7)); // 3
console.log(SearchAlgorithms.searchRange([5, 7, 7, 8, 8, 10], 8)); // [3, 4]
console.log(SearchAlgorithms.searchRotated([4, 5, 6, 7, 0, 1, 2], 0)); // 4
```

### 1.4 Sorting Algorithms

```javascript
class SortingAlgorithms {
    // Bubble Sort
    static bubbleSort(arr) {
        // Time: O(n²), Space: O(1)
        const n = arr.length;
        
        for (let i = 0; i < n - 1; i++) {
            let swapped = false;
            
            for (let j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
                    swapped = true;
                }
            }
            
            if (!swapped) break;
        }
        
        return arr;
    }
    
    // Selection Sort
    static selectionSort(arr) {
        // Time: O(n²), Space: O(1)
        const n = arr.length;
        
        for (let i = 0; i < n - 1; i++) {
            let minIdx = i;
            
            for (let j = i + 1; j < n; j++) {
                if (arr[j] < arr[minIdx]) {
                    minIdx = j;
                }
            }
            
            if (minIdx !== i) {
                [arr[i], arr[minIdx]] = [arr[minIdx], arr[i]];
            }
        }
        
        return arr;
    }
    
    // Insertion Sort
    static insertionSort(arr) {
        // Time: O(n²), Space: O(1)
        const n = arr.length;
        
        for (let i = 1; i < n; i++) {
            const key = arr[i];
            let j = i - 1;
            
            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }
            
            arr[j + 1] = key;
        }
        
        return arr;
    }
    
    // Merge Sort
    static mergeSort(arr) {
        // Time: O(n log n), Space: O(n)
        if (arr.length <= 1) return arr;
        
        const mid = Math.floor(arr.length / 2);
        const left = this.mergeSort(arr.slice(0, mid));
        const right = this.mergeSort(arr.slice(mid));
        
        return this.merge(left, right);
    }
    
    static merge(left, right) {
        const result = [];
        let i = 0, j = 0;
        
        while (i < left.length && j < right.length) {
            if (left[i] <= right[j]) {
                result.push(left[i++]);
            } else {
                result.push(right[j++]);
            }
        }
        
        return result.concat(left.slice(i)).concat(right.slice(j));
    }
    
    // Quick Sort
    static quickSort(arr, low = 0, high = arr.length - 1) {
        // Time: O(n log n) average, O(n²) worst, Space: O(log n)
        if (low < high) {
            const pi = this.partition(arr, low, high);
            this.quickSort(arr, low, pi - 1);
            this.quickSort(arr, pi + 1, high);
        }
        
        return arr;
    }
    
    static partition(arr, low, high) {
        const pivot = arr[high];
        let i = low - 1;
        
        for (let j = low; j < high; j++) {
            if (arr[j] < pivot) {
                i++;
                [arr[i], arr[j]] = [arr[j], arr[i]];
            }
        }
        
        [arr[i + 1], arr[high]] = [arr[high], arr[i + 1]];
        return i + 1;
    }
    
    // Quick Sort (Random Pivot)
    static quickSortRandom(arr, low = 0, high = arr.length - 1) {
        if (low < high) {
            const pi = this.randomPartition(arr, low, high);
            this.quickSortRandom(arr, low, pi - 1);
            this.quickSortRandom(arr, pi + 1, high);
        }
        
        return arr;
    }
    
    static randomPartition(arr, low, high) {
        const randomPivot = Math.floor(Math.random() * (high - low + 1)) + low;
        [arr[randomPivot], arr[high]] = [arr[high], arr[randomPivot]];
        return this.partition(arr, low, high);
    }
    
    // Heap Sort
    static heapSort(arr) {
        // Time: O(n log n), Space: O(1)
        const n = arr.length;
        
        // Build max heap
        for (let i = Math.floor(n / 2) - 1; i >= 0; i--) {
            this.heapify(arr, n, i);
        }
        
        // Extract elements from heap
        for (let i = n - 1; i > 0; i--) {
            [arr[0], arr[i]] = [arr[i], arr[0]];
            this.heapify(arr, i, 0);
        }
        
        return arr;
    }
    
    static heapify(arr, n, i) {
        let largest = i;
        const left = 2 * i + 1;
        const right = 2 * i + 2;
        
        if (left < n && arr[left] > arr[largest]) {
            largest = left;
        }
        
        if (right < n && arr[right] > arr[largest]) {
            largest = right;
        }
        
        if (largest !== i) {
            [arr[i], arr[largest]] = [arr[largest], arr[i]];
            this.heapify(arr, n, largest);
        }
    }
    
    // Counting Sort
    static countingSort(arr) {
        // Time: O(n + k), Space: O(k) where k is range
        if (arr.length === 0) return arr;
        
        const max = Math.max(...arr);
        const min = Math.min(...arr);
        const range = max - min + 1;
        const count = new Array(range).fill(0);
        const output = new Array(arr.length);
        
        // Store count of each element
        for (let i = 0; i < arr.length; i++) {
            count[arr[i] - min]++;
        }
        
        // Change count[i] to actual position
        for (let i = 1; i < count.length; i++) {
            count[i] += count[i - 1];
        }
        
        // Build output array
        for (let i = arr.length - 1; i >= 0; i--) {
            output[count[arr[i] - min] - 1] = arr[i];
            count[arr[i] - min]--;
        }
        
        return output;
    }
    
    // Radix Sort
    static radixSort(arr) {
        // Time: O(d * (n + k)), Space: O(n + k)
        const max = Math.max(...arr);
        
        for (let exp = 1; Math.floor(max / exp) > 0; exp *= 10) {
            this.countingSortByDigit(arr, exp);
        }
        
        return arr;
    }
    
    static countingSortByDigit(arr, exp) {
        const n = arr.length;
        const output = new Array(n);
        const count = new Array(10).fill(0);
        
        for (let i = 0; i < n; i++) {
            const digit = Math.floor(arr[i] / exp) % 10;
            count[digit]++;
        }
        
        for (let i = 1; i < 10; i++) {
            count[i] += count[i - 1];
        }
        
        for (let i = n - 1; i >= 0; i--) {
            const digit = Math.floor(arr[i] / exp) % 10;
            output[count[digit] - 1] = arr[i];
            count[digit]--;
        }
        
        for (let i = 0; i < n; i++) {
            arr[i] = output[i];
        }
    }
    
    // Bucket Sort
    static bucketSort(arr, bucketSize = 5) {
        // Time: O(n + k) average, Space: O(n + k)
        if (arr.length === 0) return arr;
        
        const min = Math.min(...arr);
        const max = Math.max(...arr);
        const bucketCount = Math.floor((max - min) / bucketSize) + 1;
        const buckets = Array(bucketCount).fill(null).map(() => []);
        
        // Distribute elements into buckets
        for (let i = 0; i < arr.length; i++) {
            const bucketIndex = Math.floor((arr[i] - min) / bucketSize);
            buckets[bucketIndex].push(arr[i]);
        }
        
        // Sort individual buckets and concatenate
        const result = [];
        for (let i = 0; i < buckets.length; i++) {
            this.insertionSort(buckets[i]);
            result.push(...buckets[i]);
        }
        
        return result;
    }
    
    // Shell Sort
    static shellSort(arr) {
        // Time: O(n log²n), Space: O(1)
        const n = arr.length;
        
        for (let gap = Math.floor(n / 2); gap > 0; gap = Math.floor(gap / 2)) {
            for (let i = gap; i < n; i++) {
                const temp = arr[i];
                let j = i;
                
                while (j >= gap && arr[j - gap] > temp) {
                    arr[j] = arr[j - gap];
                    j -= gap;
                }
                
                arr[j] = temp;
            }
        }
        
        return arr;
    }
    
    // Tim Sort (JavaScript's built-in sort uses a variant)
    static timSort(arr) {
        const MIN_MERGE = 32;
        const n = arr.length;
        
        // Sort individual subarrays
        for (let i = 0; i < n; i += MIN_MERGE) {
            this.insertionSort(
                arr.slice(i, Math.min(i + MIN_MERGE, n))
            );
        }
        
        // Merge subarrays
        for (let size = MIN_MERGE; size < n; size = 2 * size) {
            for (let start = 0; start < n; start += 2 * size) {
                const mid = start + size - 1;
                const end = Math.min(start + 2 * size - 1, n - 1);
                
                if (mid < end) {
                    const merged = this.merge(
                        arr.slice(start, mid + 1),
                        arr.slice(mid + 1, end + 1)
                    );
                    
                    for (let i = 0; i < merged.length; i++) {
                        arr[start + i] = merged[i];
                    }
                }
            }
        }
        
        return arr;
    }
}

// Testing
const testArr = [64, 34, 25, 12, 22, 11, 90];
console.log(SortingAlgorithms.quickSort([...testArr])); // [11, 12, 22, 25, 34, 64, 90]
console.log(SortingAlgorithms.mergeSort([...testArr])); // [11, 12, 22, 25, 34, 64, 90]
console.log(SortingAlgorithms.heapSort([...testArr])); // [11, 12, 22, 25, 34, 64, 90]
```

---

## Part 2: Intermediate Data Structures

### 2.1 Linked Lists

**Core Concept**: Linked lists are dynamic data structures where elements (nodes) are connected via pointers. Unlike arrays, they don't require contiguous memory and allow O(1) insertion/deletion at known positions.

**Advantages over Arrays**:
- Dynamic size (no pre-allocation needed)
- O(1) insertion/deletion at beginning
- No wasted memory from pre-allocation

**Disadvantages**:
- O(n) access time (must traverse from head)
- Extra memory for pointers
- No cache locality (nodes scattered in memory)

```javascript
// Node class for Singly Linked List
// Each node contains data and a reference to the next node
class ListNode {
    constructor(val = 0, next = null) {
        this.val = val;    // Data stored in node
        this.next = next;  // Pointer to next node (null if last)
    }
}

// Singly Linked List
// Head pointer tracks the first node; size tracks total nodes
class SinglyLinkedList {
    constructor() {
        this.head = null;  // Points to first node (null if empty)
        this.size = 0;     // Number of nodes in list
    }
    
    // Insert at beginning - O(1)
    // Most efficient insertion: no traversal needed
    insertAtHead(val) {
        const newNode = new ListNode(val);
        newNode.next = this.head;  // New node points to current head
        this.head = newNode;       // Update head to new node
        this.size++;
    }
    
    // Insert at end - O(n)
    // Must traverse entire list to find tail
    insertAtTail(val) {
        const newNode = new ListNode(val);
        
        if (!this.head) {
            // Empty list: new node becomes head
            this.head = newNode;
        } else {
            // Traverse to last node
            let current = this.head;
            while (current.next) {
                current = current.next;
            }
            current.next = newNode;  // Attach new node at end
        }
        
        this.size++;
    }
    
    // Insert at position - O(n)
    insertAt(val, position) {
        if (position < 0 || position > this.size) return false;
        
        if (position === 0) {
            this.insertAtHead(val);
            return true;
        }
        
        const newNode = new ListNode(val);
        let current = this.head;
        let prev = null;
        let index = 0;
        
        while (index < position) {
            prev = current;
            current = current.next;
            index++;
        }
        
        newNode.next = current;
        prev.next = newNode;
        this.size++;
        return true;
    }
    
    // Delete node by value - O(n)
    delete(val) {
        if (!this.head) return false;
        
        if (this.head.val === val) {
            this.head = this.head.next;
            this.size--;
            return true;
        }
        
        let current = this.head;
        while (current.next) {
            if (current.next.val === val) {
                current.next = current.next.next;
                this.size--;
                return true;
            }
            current = current.next;
        }
        
        return false;
    }
    
    // Delete at position - O(n)
    deleteAt(position) {
        if (position < 0 || position >= this.size) return false;
        
        if (position === 0) {
            this.head = this.head.next;
            this.size--;
            return true;
        }
        
        let current = this.head;
        let prev = null;
        let index = 0;
        
        while (index < position) {
            prev = current;
            current = current.next;
            index++;
        }
        
        prev.next = current.next;
        this.size--;
        return true;
    }
    
    // Search - O(n)
    search(val) {
        let current = this.head;
        let index = 0;
        
        while (current) {
            if (current.val === val) return index;
            current = current.next;
            index++;
        }
        
        return -1;
    }
    
    // Reverse - O(n)
    // Classic three-pointer technique to reverse links in-place
    reverse() {
        let prev = null;       // Will become new tail (null at end)
        let current = this.head;
        
        // Iterate through list, reversing each link
        while (current) {
            const next = current.next;  // Save next before changing link
            current.next = prev;        // Reverse the link
            prev = current;             // Move prev forward
            current = next;             // Move current forward
        }
        
        this.head = prev;  // Update head to last node (now first)
    }
    
    // Detect cycle - O(n)
    // Floyd's Cycle Detection: fast pointer moves 2x speed of slow pointer
    // If there's a cycle, they'll eventually meet
    hasCycle() {
        if (!this.head) return false;
        
        let slow = this.head;  // Moves 1 step at a time
        let fast = this.head;  // Moves 2 steps at a time
        
        while (fast && fast.next) {
            slow = slow.next;       // Move 1 step
            fast = fast.next.next;  // Move 2 steps
            
            if (slow === fast) return true;  // Cycle detected!
        }
        
        return false;  // Fast reached end, no cycle
    }
    
    // Find middle - O(n)
    // Fast-slow pointer technique: when fast reaches end, slow is at middle
    findMiddle() {
        if (!this.head) return null;
        
        let slow = this.head;  // Moves 1 step
        let fast = this.head;  // Moves 2 steps
        
        // When fast reaches end, slow is at middle
        while (fast && fast.next) {
            slow = slow.next;       // Move 1
            fast = fast.next.next;  // Move 2
        }
        
        return slow.val;  // Middle element
    }
    
    // Remove duplicates from sorted list - O(n)
    removeDuplicates() {
        if (!this.head) return;
        
        let current = this.head;
        
        while (current && current.next) {
            if (current.val === current.next.val) {
                current.next = current.next.next;
                this.size--;
            } else {
                current = current.next;
            }
        }
    }
    
    // Print list - O(n)
    toArray() {
        const result = [];
        let current = this.head;
        
        while (current) {
            result.push(current.val);
            current = current.next;
        }
        
        return result;
    }
}

// Doubly Linked List Node
class DoublyListNode {
    constructor(val = 0, prev = null, next = null) {
        this.val = val;
        this.prev = prev;
        this.next = next;
    }
}

// Doubly Linked List
class DoublyLinkedList {
    constructor() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }
    
    // Insert at head - O(1)
    insertAtHead(val) {
        const newNode = new DoublyListNode(val);
        
        if (!this.head) {
            this.head = this.tail = newNode;
        } else {
            newNode.next = this.head;
            this.head.prev = newNode;
            this.head = newNode;
        }
        
        this.size++;
    }
    
    // Insert at tail - O(1)
    insertAtTail(val) {
        const newNode = new DoublyListNode(val);
        
        if (!this.tail) {
            this.head = this.tail = newNode;
        } else {
            newNode.prev = this.tail;
            this.tail.next = newNode;
            this.tail = newNode;
        }
        
        this.size++;
    }
    
    // Delete from head - O(1)
    deleteFromHead() {
        if (!this.head) return null;
        
        const val = this.head.val;
        
        if (this.head === this.tail) {
            this.head = this.tail = null;
        } else {
            this.head = this.head.next;
            this.head.prev = null;
        }
        
        this.size--;
        return val;
    }
    
    // Delete from tail - O(1)
    deleteFromTail() {
        if (!this.tail) return null;
        
        const val = this.tail.val;
        
        if (this.head === this.tail) {
            this.head = this.tail = null;
        } else {
            this.tail = this.tail.prev;
            this.tail.next = null;
        }
        
        this.size--;
        return val;
    }
    
    // Reverse - O(n)
    reverse() {
        let current = this.head;
        let temp = null;
        
        while (current) {
            temp = current.prev;
            current.prev = current.next;
            current.next = temp;
            current = current.prev;
        }
        
        if (temp) {
            this.tail = this.head;
            this.head = temp.prev;
        }
    }
}

// Advanced Linked List Algorithms
class LinkedListAlgorithms {
    // Merge two sorted lists
    static mergeTwoLists(l1, l2) {
        const dummy = new ListNode(0);
        let current = dummy;
        
        while (l1 && l2) {
            if (l1.val <= l2.val) {
                current.next = l1;
                l1 = l1.next;
            } else {
                current.next = l2;
                l2 = l2.next;
            }
            current = current.next;
        }
        
        current.next = l1 || l2;
        return dummy.next;
    }
    
    // Find intersection of two lists
    static getIntersectionNode(headA, headB) {
        if (!headA || !headB) return null;
        
        let a = headA;
        let b = headB;
        
        while (a !== b) {
            a = a ? a.next : headB;
            b = b ? b.next : headA;
        }
        
        return a;
    }
    
    // Remove nth node from end
    static removeNthFromEnd(head, n) {
        const dummy = new ListNode(0);
        dummy.next = head;
        let first = dummy;
        let second = dummy;
        
        for (let i = 0; i <= n; i++) {
            first = first.next;
        }
        
        while (first) {
            first = first.next;
            second = second.next;
        }
        
        second.next = second.next.next;
        return dummy.next;
    }
    
    // Palindrome check
    static isPalindrome(head) {
        if (!head || !head.next) return true;
        
        // Find middle
        let slow = head;
        let fast = head;
        
        while (fast.next && fast.next.next) {
            slow = slow.next;
            fast = fast.next.next;
        }
        
        // Reverse second half
        let secondHalf = this.reverse(slow.next);
        let firstHalf = head;
        
        // Compare
        while (secondHalf) {
            if (firstHalf.val !== secondHalf.val) return false;
            firstHalf = firstHalf.next;
            secondHalf = secondHalf.next;
        }
        
        return true;
    }
    
    static reverse(head) {
        let prev = null;
        let current = head;
        
        while (current) {
            const next = current.next;
            current.next = prev;
            prev = current;
            current = next;
        }
        
        return prev;
    }
    
    // Reorder list (L0 → Ln → L1 → Ln-1 → ...)
    static reorderList(head) {
        if (!head || !head.next) return;
        
        // Find middle
        let slow = head;
        let fast = head;
        
        while (fast.next && fast.next.next) {
            slow = slow.next;
            fast = fast.next.next;
        }
        
        // Reverse second half
        let secondHalf = this.reverse(slow.next);
        slow.next = null;
        
        // Merge
        let firstHalf = head;
        
        while (secondHalf) {
            const temp1 = firstHalf.next;
            const temp2 = secondHalf.next;
            
            firstHalf.next = secondHalf;
            secondHalf.next = temp1;
            
            firstHalf = temp1;
            secondHalf = temp2;
        }
    }
    
    // Sort list (merge sort)
    static sortList(head) {
        if (!head || !head.next) return head;
        
        // Find middle
        let slow = head;
        let fast = head;
        let prev = null;
        
        while (fast && fast.next) {
            prev = slow;
            slow = slow.next;
            fast = fast.next.next;
        }
        
        prev.next = null;
        
        const left = this.sortList(head);
        const right = this.sortList(slow);
        
        return this.mergeTwoLists(left, right);
    }
    
    // Add two numbers represented by linked lists
    static addTwoNumbers(l1, l2) {
        const dummy = new ListNode(0);
        let current = dummy;
        let carry = 0;
        
        while (l1 || l2 || carry) {
            const sum = (l1?.val || 0) + (l2?.val || 0) + carry;
            carry = Math.floor(sum / 10);
            current.next = new ListNode(sum % 10);
            
            current = current.next;
            l1 = l1?.next;
            l2 = l2?.next;
        }
        
        return dummy.next;
    }
    
    // Copy list with random pointer
    static copyRandomList(head) {
        if (!head) return null;
        
        const map = new Map();
        let current = head;
        
        // First pass: create nodes
        while (current) {
            map.set(current, new ListNode(current.val));
            current = current.next;
        }
        
        // Second pass: assign pointers
        current = head;
        while (current) {
            const newNode = map.get(current);
            newNode.next = map.get(current.next) || null;
            newNode.random = map.get(current.random) || null;
            current = current.next;
        }
        
        return map.get(head);
    }
}

// Testing
const list = new SinglyLinkedList();
list.insertAtTail(1);
list.insertAtTail(2);
list.insertAtTail(3);
list.insertAtTail(4);
console.log(list.toArray()); // [1, 2, 3, 4]
list.reverse();
console.log(list.toArray()); // [4, 3, 2, 1]
```

### 2.2 Stacks and Queues

**Stack - LIFO (Last In, First Out)**
- Think of a stack of plates: last plate added is first removed
- Uses: Function call stack, undo/redo, expression evaluation, backtracking
- Key operations: push (add), pop (remove), peek (view top)

**Queue - FIFO (First In, First Out)**
- Think of a line at a store: first person in line is served first
- Uses: Task scheduling, BFS traversal, request handling
- Key operations: enqueue (add to rear), dequeue (remove from front)

```javascript
// Stack Implementation
class Stack {
    constructor() {
        this.items = [];  // Array-based implementation
    }
    
    // Push - O(1)
    // Add element to top of stack
    push(element) {
        this.items.push(element);  // Array push is O(1) amortized
    }
    
    // Pop - O(1)
    // Remove and return top element
    pop() {
        if (this.isEmpty()) return null;
        return this.items.pop();  // Remove from end
    }
    
    // Peek - O(1)
    // View top element without removing
    peek() {
        if (this.isEmpty()) return null;
        return this.items[this.items.length - 1];
    }
    
    // isEmpty - O(1)
    isEmpty() {
        return this.items.length === 0;
    }
    
    // Size - O(1)
    size() {
        return this.items.length;
    }
    
    // Clear - O(1)
    clear() {
        this.items = [];
    }
}

// Queue Implementation
class Queue {
    constructor() {
        this.items = [];
    }
    
    // Enqueue - O(1)
    enqueue(element) {
        this.items.push(element);
    }
    
    // Dequeue - O(n) but can be optimized
    dequeue() {
        if (this.isEmpty()) return null;
        return this.items.shift();
    }
    
    // Front - O(1)
    front() {
        if (this.isEmpty()) return null;
        return this.items[0];
    }
    
    // isEmpty - O(1)
    isEmpty() {
        return this.items.length === 0;
    }
    
    // Size - O(1)
    size() {
        return this.items.length;
    }
}

// Optimized Queue using two pointers
class OptimizedQueue {
    constructor() {
        this.items = {};
        this.front = 0;
        this.rear = 0;
    }
    
    enqueue(element) {
        this.items[this.rear] = element;
        this.rear++;
    }
    
    dequeue() {
        if (this.isEmpty()) return null;
        
        const item = this.items[this.front];
        delete this.items[this.front];
        this.front++;
        
        return item;
    }
    
    peek() {
        if (this.isEmpty()) return null;
        return this.items[this.front];
    }
    
    isEmpty() {
        return this.rear === this.front;
    }
    
    size() {
        return this.rear - this.front;
    }
}

// Circular Queue
class CircularQueue {
    constructor(k) {
        this.queue = new Array(k);
        this.size = k;
        this.front = -1;
        this.rear = -1;
    }
    
    enqueue(value) {
        if (this.isFull()) return false;
        
        if (this.isEmpty()) {
            this.front = 0;
        }
        
        this.rear = (this.rear + 1) % this.size;
        this.queue[this.rear] = value;
        return true;
    }
    
    dequeue() {
        if (this.isEmpty()) return false;
        
        if (this.front === this.rear) {
            this.front = -1;
            this.rear = -1;
        } else {
            this.front = (this.front + 1) % this.size;
        }
        
        return true;
    }
    
    Front() {
        if (this.isEmpty()) return -1;
        return this.queue[this.front];
    }
    
    Rear() {
        if (this.isEmpty()) return -1;
        return this.queue[this.rear];
    }
    
    isEmpty() {
        return this.front === -1;
    }
    
    isFull() {
        return (this.rear + 1) % this.size === this.front;
    }
}

// Deque (Double-ended Queue)
class Deque {
    constructor() {
        this.items = {};
        this.front = 0;
        this.rear = 0;
    }
    
    addFront(element) {
        this.front--;
        this.items[this.front] = element;
    }
    
    addRear(element) {
        this.items[this.rear] = element;
        this.rear++;
    }
    
    removeFront() {
        if (this.isEmpty()) return null;
        
        const item = this.items[this.front];
        delete this.items[this.front];
        this.front++;
        
        return item;
    }
    
    removeRear() {
        if (this.isEmpty()) return null;
        
        this.rear--;
        const item = this.items[this.rear];
        delete this.items[this.rear];
        
        return item;
    }
    
    peekFront() {
        if (this.isEmpty()) return null;
        return this.items[this.front];
    }
    
    peekRear() {
        if (this.isEmpty()) return null;
        return this.items[this.rear - 1];
    }
    
    isEmpty() {
        return this.front === this.rear;
    }
    
    size() {
        return this.rear - this.front;
    }
}

// Priority Queue (Min Heap)
// Elements are served based on priority, not insertion order
// Lower priority value = higher priority (served first)
class PriorityQueue {
    constructor() {
        this.heap = [];  // Binary heap stored as array
    }
    
    // Enqueue with priority - O(log n)
    // Insert element and restore heap property
    enqueue(val, priority) {
        this.heap.push({ val, priority });
        this.bubbleUp();  // Move element up to correct position
    }
    
    // Dequeue - O(log n)
    // Remove and return highest priority element (minimum)
    dequeue() {
        if (this.isEmpty()) return null;
        
        const min = this.heap[0];  // Root has minimum priority
        const end = this.heap.pop();
        
        if (this.heap.length > 0) {
            this.heap[0] = end;   // Move last element to root
            this.sinkDown();      // Restore heap property
        }
        
        return min.val;
    }
    
    // Bubble up: move element up to maintain min-heap property
    // Parent must have priority ≤ child's priority
    bubbleUp() {
        let idx = this.heap.length - 1;  // Start at last element
        const element = this.heap[idx];
        
        while (idx > 0) {
            const parentIdx = Math.floor((idx - 1) / 2);
            const parent = this.heap[parentIdx];
            
            // If element has higher priority than parent, stop
            if (element.priority >= parent.priority) break;
            
            // Swap with parent and continue
            this.heap[idx] = parent;
            idx = parentIdx;
        }
        
        this.heap[idx] = element;
    }
    
    // Sink down: move element down to maintain min-heap property
    // After removing root, restore heap by moving new root down
    sinkDown() {
        let idx = 0;
        const length = this.heap.length;
        const element = this.heap[0];
        
        while (true) {
            const leftChildIdx = 2 * idx + 1;
            const rightChildIdx = 2 * idx + 2;
            let leftChild, rightChild;
            let swap = null;  // Track which child to swap with
            
            // Check left child
            if (leftChildIdx < length) {
                leftChild = this.heap[leftChildIdx];
                if (leftChild.priority < element.priority) {
                    swap = leftChildIdx;
                }
            }
            
            // Check right child
            if (rightChildIdx < length) {
                rightChild = this.heap[rightChildIdx];
                if (
                    (swap === null && rightChild.priority < element.priority) ||
                    (swap !== null && rightChild.priority < leftChild.priority)
                ) {
                    swap = rightChildIdx;  // Right child is smaller
                }
            }
            
            if (swap === null) break;  // Element is in correct position
            
            this.heap[idx] = this.heap[swap];
            idx = swap;
        }
        
        this.heap[idx] = element;
    }
    
    isEmpty() {
        return this.heap.length === 0;
    }
}

// Stack and Queue Problems
class StackQueueProblems {
    // Valid Parentheses
    static isValid(s) {
        const stack = [];
        const pairs = {
            ')': '(',
            '}': '{',
            ']': '['
        };
        
        for (let char of s) {
            if (char === '(' || char === '{' || char === '[') {
                stack.push(char);
            } else {
                if (stack.length === 0 || stack.pop() !== pairs[char]) {
                    return false;
                }
            }
        }
        
        return stack.length === 0;
    }
    
    // Min Stack
    static MinStack = class {
        constructor() {
            this.stack = [];
            this.minStack = [];
        }
        
        push(val) {
            this.stack.push(val);
            if (this.minStack.length === 0 || val <= this.getMin()) {
                this.minStack.push(val);
            }
        }
        
        pop() {
            const val = this.stack.pop();
            if (val === this.getMin()) {
                this.minStack.pop();
            }
            return val;
        }
        
        top() {
            return this.stack[this.stack.length - 1];
        }
        
        getMin() {
            return this.minStack[this.minStack.length - 1];
        }
    };
    
    // Evaluate Reverse Polish Notation
    static evalRPN(tokens) {
        const stack = [];
        const operators = {
            '+': (a, b) => a + b,
            '-': (a, b) => a - b,
            '*': (a, b) => a * b,
            '/': (a, b) => Math.trunc(a / b)
        };
        
        for (let token of tokens) {
            if (operators[token]) {
                const b = stack.pop();
                const a = stack.pop();
                stack.push(operators[token](a, b));
            } else {
                stack.push(parseInt(token));
            }
        }
        
        return stack[0];
    }
    
    // Daily Temperatures
    static dailyTemperatures(temperatures) {
        const result = new Array(temperatures.length).fill(0);
        const stack = [];
        
        for (let i = 0; i < temperatures.length; i++) {
            while (stack.length > 0 && temperatures[i] > temperatures[stack[stack.length - 1]]) {
                const idx = stack.pop();
                result[idx] = i - idx;
            }
            stack.push(i);
        }
        
        return result;
    }
    
    // Largest Rectangle in Histogram
    static largestRectangleArea(heights) {
        const stack = [];
        let maxArea = 0;
        
        for (let i = 0; i <= heights.length; i++) {
            const h = i === heights.length ? 0 : heights[i];
            
            while (stack.length > 0 && h < heights[stack[stack.length - 1]]) {
                const height = heights[stack.pop()];
                const width = stack.length === 0 ? i : i - stack[stack.length - 1] - 1;
                maxArea = Math.max(maxArea, height * width);
            }
            
            stack.push(i);
        }
        
        return maxArea;
    }
    
    // Sliding Window Maximum
    static maxSlidingWindow(nums, k) {
        const result = [];
        const deque = [];
        
        for (let i = 0; i < nums.length; i++) {
            // Remove indices outside window
            while (deque.length > 0 && deque[0] < i - k + 1) {
                deque.shift();
            }
            
            // Remove smaller elements from back
            while (deque.length > 0 && nums[deque[deque.length - 1]] < nums[i]) {
                deque.pop();
            }
            
            deque.push(i);
            
            // Add to result when window is full
            if (i >= k - 1) {
                result.push(nums[deque[0]]);
            }
        }
        
        return result;
    }
    
    // Implement Queue using Stacks
    static MyQueue = class {
        constructor() {
            this.inStack = [];
            this.outStack = [];
        }
        
        push(x) {
            this.inStack.push(x);
        }
        
        pop() {
            this.peek();
            return this.outStack.pop();
        }
        
        peek() {
            if (this.outStack.length === 0) {
                while (this.inStack.length > 0) {
                    this.outStack.push(this.inStack.pop());
                }
            }
            return this.outStack[this.outStack.length - 1];
        }
        
        empty() {
            return this.inStack.length === 0 && this.outStack.length === 0;
        }
    };
    
    // Implement Stack using Queues
    static MyStack = class {
        constructor() {
            this.queue = [];
        }
        
        push(x) {
            this.queue.push(x);
            let size = this.queue.length;
            
            while (size > 1) {
                this.queue.push(this.queue.shift());
                size--;
            }
        }
        
        pop() {
            return this.queue.shift();
        }
        
        top() {
            return this.queue[0];
        }
        
        empty() {
            return this.queue.length === 0;
        }
    };
}

// Testing
console.log(StackQueueProblems.isValid("()[]{}"));  // true
console.log(StackQueueProblems.dailyTemperatures([73,74,75,71,69,72,76,73]));  // [1,1,4,2,1,1,0,0]
```

### 2.3 Hash Tables

```javascript
// Hash Table Implementation
class HashTable {
    constructor(size = 53) {
        this.keyMap = new Array(size);
    }
    
    // Hash function
    _hash(key) {
        let total = 0;
        const PRIME = 31;
        
        for (let i = 0; i < Math.min(key.length, 100); i++) {
            const char = key[i];
            const value = char.charCodeAt(0) - 96;
            total = (total * PRIME + value) % this.keyMap.length;
        }
        
        return total;
    }
    
    // Set key-value pair
    set(key, value) {
        const index = this._hash(key);
        
        if (!this.keyMap[index]) {
            this.keyMap[index] = [];
        }
        
        // Update if key exists
        for (let i = 0; i < this.keyMap[index].length; i++) {
            if (this.keyMap[index][i][0] === key) {
                this.keyMap[index][i][1] = value;
                return;
            }
        }
        
        this.keyMap[index].push([key, value]);
    }
    
    // Get value by key
    get(key) {
        const index = this._hash(key);
        
        if (this.keyMap[index]) {
            for (let i = 0; i < this.keyMap[index].length; i++) {
                if (this.keyMap[index][i][0] === key) {
                    return this.keyMap[index][i][1];
                }
            }
        }
        
        return undefined;
    }
    
    // Delete key
    delete(key) {
        const index = this._hash(key);
        
        if (this.keyMap[index]) {
            for (let i = 0; i < this.keyMap[index].length; i++) {
                if (this.keyMap[index][i][0] === key) {
                    this.keyMap[index].splice(i, 1);
                    return true;
                }
            }
        }
        
        return false;
    }
    
    // Get all keys
    keys() {
        const keysArr = [];
        
        for (let i = 0; i < this.keyMap.length; i++) {
            if (this.keyMap[i]) {
                for (let j = 0; j < this.keyMap[i].length; j++) {
                    keysArr.push(this.keyMap[i][j][0]);
                }
            }
        }
        
        return keysArr;
    }
    
    // Get all values
    values() {
        const valuesArr = [];
        
        for (let i = 0; i < this.keyMap.length; i++) {
            if (this.keyMap[i]) {
                for (let j = 0; j < this.keyMap[i].length; j++) {
                    if (!valuesArr.includes(this.keyMap[i][j][1])) {
                        valuesArr.push(this.keyMap[i][j][1]);
                    }
                }
            }
        }
        
        return valuesArr;
    }
}

// LRU Cache
class LRUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.cache = new Map();
    }
    
    get(key) {
        if (!this.cache.has(key)) return -1;
        
        const value = this.cache.get(key);
        // Move to end (most recently used)
        this.cache.delete(key);
        this.cache.set(key, value);
        
        return value;
    }
    
    put(key, value) {
        if (this.cache.has(key)) {
            this.cache.delete(key);
        }
        
        this.cache.set(key, value);
        
        if (this.cache.size > this.capacity) {
            // Remove first item (least recently used)
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
    }
}

// LFU Cache
class LFUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.min = 0;
        this.keyToVal = new Map();
        this.keyToFreq = new Map();
        this.freqToKeys = new Map();
    }
    
    get(key) {
        if (!this.keyToVal.has(key)) return -1;
        
        this.increaseFreq(key);
        return this.keyToVal.get(key);
    }
    
    put(key, value) {
        if (this.capacity <= 0) return;
        
        if (this.keyToVal.has(key)) {
            this.keyToVal.set(key, value);
            this.increaseFreq(key);
            return;
        }
        
        if (this.keyToVal.size >= this.capacity) {
            this.removeMinFreqKey();
        }
        
        this.keyToVal.set(key, value);
        this.keyToFreq.set(key, 1);
        
        if (!this.freqToKeys.has(1)) {
            this.freqToKeys.set(1, new Set());
        }
        this.freqToKeys.get(1).add(key);
        
        this.min = 1;
    }
    
    increaseFreq(key) {
        const freq = this.keyToFreq.get(key);
        this.keyToFreq.set(key, freq + 1);
        
        this.freqToKeys.get(freq).delete(key);
        
        if (!this.freqToKeys.has(freq + 1)) {
            this.freqToKeys.set(freq + 1, new Set());
        }
        this.freqToKeys.get(freq + 1).add(key);
        
        if (this.freqToKeys.get(freq).size === 0) {
            this.freqToKeys.delete(freq);
            if (freq === this.min) {
                this.min++;
            }
        }
    }
    
    removeMinFreqKey() {
        const keySet = this.freqToKeys.get(this.min);
        const deletedKey = keySet.values().next().value;
        
        keySet.delete(deletedKey);
        
        if (keySet.size === 0) {
            this.freqToKeys.delete(this.min);
        }
        
        this.keyToVal.delete(deletedKey);
        this.keyToFreq.delete(deletedKey);
    }
}

// Hash Map Problems
class HashMapProblems {
    // Top K Frequent Elements
    static topKFrequent(nums, k) {
        const freqMap = new Map();
        
        for (let num of nums) {
            freqMap.set(num, (freqMap.get(num) || 0) + 1);
        }
        
        return Array.from(freqMap.entries())
            .sort((a, b) => b[1] - a[1])
            .slice(0, k)
            .map(pair => pair[0]);
    }
    
    // Longest Consecutive Sequence
    static longestConsecutive(nums) {
        const numSet = new Set(nums);
        let maxLength = 0;
        
        for (let num of numSet) {
            if (!numSet.has(num - 1)) {
                let currentNum = num;
                let currentLength = 1;
                
                while (numSet.has(currentNum + 1)) {
                    currentNum++;
                    currentLength++;
                }
                
                maxLength = Math.max(maxLength, currentLength);
            }
        }
        
        return maxLength;
    }
    
    // Subarray Sum Equals K
    static subarraySum(nums, k) {
        const map = new Map([[0, 1]]);
        let count = 0;
        let sum = 0;
        
        for (let num of nums) {
            sum += num;
            if (map.has(sum - k)) {
                count += map.get(sum - k);
            }
            map.set(sum, (map.get(sum) || 0) + 1);
        }
        
        return count;
    }
    
    // Find All Anagrams
    static findAnagrams(s, p) {
        const result = [];
        const pMap = new Map();
        const sMap = new Map();
        
        for (let char of p) {
            pMap.set(char, (pMap.get(char) || 0) + 1);
        }
        
        let left = 0;
        let right = 0;
        let count = pMap.size;
        
        while (right < s.length) {
            const char = s[right];
            
            if (pMap.has(char)) {
                sMap.set(char, (sMap.get(char) || 0) + 1);
                if (sMap.get(char) === pMap.get(char)) {
                    count--;
                }
            }
            
            right++;
            
            while (count === 0) {
                if (right - left === p.length) {
                    result.push(left);
                }
                
                const leftChar = s[left];
                
                if (pMap.has(leftChar)) {
                    sMap.set(leftChar, sMap.get(leftChar) - 1);
                    if (sMap.get(leftChar) < pMap.get(leftChar)) {
                        count++;
                    }
                }
                
                left++;
            }
        }
        
        return result;
    }
}

// Testing
const ht = new HashTable();
ht.set("hello", "world");
ht.set("foo", "bar");
console.log(ht.get("hello")); // "world"
console.log(HashMapProblems.topKFrequent([1,1,1,2,2,3], 2)); // [1, 2]
```

### 2.4 Trees - Binary Trees and BST

**Binary Tree Fundamentals**:
- Each node has at most 2 children (left and right)
- **Height**: Longest path from root to leaf
- **Depth**: Distance from root to node
- **Balanced**: Height difference between left and right subtrees ≤ 1

**Binary Search Tree (BST) Properties**:
- Left subtree: all values < parent
- Right subtree: all values > parent
- Enables O(log n) search in balanced trees
- Degrades to O(n) if unbalanced (becomes a linked list)

**Traversal Types**:
- **Inorder** (Left-Root-Right): Gives sorted order for BST
- **Preorder** (Root-Left-Right): Used for tree copying
- **Postorder** (Left-Right-Root): Used for tree deletion
- **Level Order** (BFS): Level by level traversal

```javascript
// Tree Node
class TreeNode {
    constructor(val = 0, left = null, right = null) {
        this.val = val;      // Node value
        this.left = left;    // Left child pointer
        this.right = right;  // Right child pointer
    }
}

// Binary Search Tree
class BST {
    constructor() {
        this.root = null;  // Root of the tree
    }
    
    // Insert - O(log n) average, O(n) worst case (unbalanced tree)
    // Strategy: Compare with current node, go left if smaller, right if larger
    insert(val) {
        const newNode = new TreeNode(val);
        
        if (!this.root) {
            this.root = newNode;  // Empty tree: new node becomes root
            return this;
        }
        
        let current = this.root;
        
        while (true) {
            if (val === current.val) return undefined;  // Duplicate: ignore
            
            if (val < current.val) {
                // Go left
                if (!current.left) {
                    current.left = newNode;  // Found spot: insert here
                    return this;
                }
                current = current.left;  // Keep searching left
            } else {
                // Go right
                if (!current.right) {
                    current.right = newNode;  // Found spot: insert here
                    return this;
                }
                current = current.right;  // Keep searching right
            }
        }
    }
    
    // Search - O(log n) average, O(n) worst
    // BST property allows us to skip half the tree at each step
    search(val) {
        if (!this.root) return null;
        
        let current = this.root;
        
        while (current) {
            if (val === current.val) return current;  // Found it!
            // BST property: decide which subtree to search
            if (val < current.val) current = current.left;   // Search left
            else current = current.right;  // Search right
        }
        
        return null;  // Not found
    }
    
    // Delete - O(log n) average, O(n) worst
    // Three cases: no child, one child, two children
    delete(val) {
        this.root = this.deleteNode(this.root, val);
    }
    
    deleteNode(node, val) {
        if (!node) return null;  // Not found
        
        // Navigate to the node to delete
        if (val < node.val) {
            node.left = this.deleteNode(node.left, val);
        } else if (val > node.val) {
            node.right = this.deleteNode(node.right, val);
        } else {
            // Found node to delete!
            
            // Case 1: No left child - replace with right child
            if (!node.left) return node.right;
            // Case 2: No right child - replace with left child
            if (!node.right) return node.left;
            
            // Case 3: Two children - replace with inorder successor
            // (smallest node in right subtree)
            node.val = this.findMin(node.right).val;
            // Delete the inorder successor
            node.right = this.deleteNode(node.right, node.val);
        }
        
        return node;
    }
    
    // Find minimum value node (leftmost node)
    findMin(node) {
        while (node.left) {
            node = node.left;  // Keep going left
        }
        return node;
    }
    
    // Inorder Traversal (Left, Root, Right) - O(n)
    // For BST: produces values in sorted order!
    // Use case: Get all elements sorted
    inorder(node = this.root, result = []) {
        if (node) {
            this.inorder(node.left, result);   // Visit left subtree
            result.push(node.val);              // Visit root
            this.inorder(node.right, result);  // Visit right subtree
        }
        return result;
    }
    
    // Preorder Traversal (Root, Left, Right) - O(n)
    // Use case: Create a copy of the tree, prefix expression
    preorder(node = this.root, result = []) {
        if (node) {
            result.push(node.val);              // Visit root first
            this.preorder(node.left, result);   // Visit left subtree
            this.preorder(node.right, result);  // Visit right subtree
        }
        return result;
    }
    
    // Postorder Traversal (Left, Right, Root) - O(n)
    // Use case: Delete tree, postfix expression evaluation
    postorder(node = this.root, result = []) {
        if (node) {
            this.postorder(node.left, result);   // Visit left subtree
            this.postorder(node.right, result);  // Visit right subtree
            result.push(node.val);               // Visit root last
        }
        return result;
    }
    
    // Level Order Traversal (BFS) - O(n)
    // Visit nodes level by level from top to bottom
    // Use case: Find shortest path, level-wise operations
    levelOrder() {
        if (!this.root) return [];
        
        const result = [];
        const queue = [this.root];  // Use queue for BFS
        
        while (queue.length > 0) {
            const level = [];           // Current level's values
            const levelSize = queue.length;  // Number of nodes at this level
            
            // Process all nodes at current level
            for (let i = 0; i < levelSize; i++) {
                const node = queue.shift();  // Dequeue node
                level.push(node.val);
                
                // Enqueue children for next level
                if (node.left) queue.push(node.left);
                if (node.right) queue.push(node.right);
            }
            
            result.push(level);  // Add this level to result
        }
        
        return result;  // [[level0], [level1], [level2], ...]
    }
}

// Tree Algorithms
class TreeAlgorithms {
    // Maximum Depth (Height of Tree) - O(n)
    // Recursively find the longest path from root to leaf
    static maxDepth(root) {
        if (!root) return 0;  // Empty tree has depth 0
        // Depth = 1 (current node) + max depth of subtrees
        return 1 + Math.max(this.maxDepth(root.left), this.maxDepth(root.right));
    }
    
    // Minimum Depth - O(n)
    // Shortest path from root to any leaf node
    static minDepth(root) {
        if (!root) return 0;
        
        // If one subtree is empty, can't stop there (not a leaf)
        if (!root.left) return 1 + this.minDepth(root.right);
        if (!root.right) return 1 + this.minDepth(root.left);
        
        // Both subtrees exist: take minimum
        return 1 + Math.min(this.minDepth(root.left), this.minDepth(root.right));
    }
    
    // Is Balanced - O(n)
    // Check if height difference between left and right subtrees ≤ 1 at every node
    // A balanced tree ensures O(log n) operations
    static isBalanced(root) {
        const checkBalance = (node) => {
            if (!node) return 0;  // Null node has height 0
            
            const left = checkBalance(node.left);
            if (left === -1) return -1;  // Left subtree unbalanced
            
            const right = checkBalance(node.right);
            if (right === -1) return -1;  // Right subtree unbalanced
            
            // Check if current node is balanced
            if (Math.abs(left - right) > 1) return -1;  // Unbalanced!
            
            return 1 + Math.max(left, right);  // Return height
        };
        
        return checkBalance(root) !== -1;  // -1 indicates unbalanced
    }
    
    // Is Same Tree
    static isSameTree(p, q) {
        if (!p && !q) return true;
        if (!p || !q) return false;
        
        return p.val === q.val &&
               this.isSameTree(p.left, q.left) &&
               this.isSameTree(p.right, q.right);
    }
    
    // Is Symmetric
    static isSymmetric(root) {
        const isMirror = (left, right) => {
            if (!left && !right) return true;
            if (!left || !right) return false;
            
            return left.val === right.val &&
                   isMirror(left.left, right.right) &&
                   isMirror(left.right, right.left);
        };
        
        return isMirror(root, root);
    }
    
    // Invert Binary Tree - O(n)
    // Mirror the tree: swap left and right children at every node
    // Also called "Mirror Tree"
    static invertTree(root) {
        if (!root) return null;
        
        // Swap left and right children
        [root.left, root.right] = [root.right, root.left];
        // Recursively invert subtrees
        this.invertTree(root.left);
        this.invertTree(root.right);
        
        return root;
    }
    
    // Lowest Common Ancestor - O(n)
    // Find the deepest node that has both p and q as descendants
    // Used in: Finding relationships, merging operations
    static lowestCommonAncestor(root, p, q) {
        // Base cases
        if (!root || root === p || root === q) return root;
        
        // Search in both subtrees
        const left = this.lowestCommonAncestor(root.left, p, q);
        const right = this.lowestCommonAncestor(root.right, p, q);
        
        // If both subtrees return non-null, current node is LCA
        if (left && right) return root;
        // Otherwise, return the non-null subtree result
        return left || right;
    }
    
    // Path Sum
    static hasPathSum(root, targetSum) {
        if (!root) return false;
        
        if (!root.left && !root.right) {
            return root.val === targetSum;
        }
        
        return this.hasPathSum(root.left, targetSum - root.val) ||
               this.hasPathSum(root.right, targetSum - root.val);
    }
    
    // All Paths from Root to Leaf
    static binaryTreePaths(root) {
        const result = [];
        
        const dfs = (node, path) => {
            if (!node) return;
            
            path.push(node.val);
            
            if (!node.left && !node.right) {
                result.push(path.join("->"));
            } else {
                dfs(node.left, [...path]);
                dfs(node.right, [...path]);
            }
        };
        
        dfs(root, []);
        return result;
    }
    
    // Diameter of Binary Tree
    static diameterOfBinaryTree(root) {
        let diameter = 0;
        
        const height = (node) => {
            if (!node) return 0;
            
            const left = height(node.left);
            const right = height(node.right);
            
            diameter = Math.max(diameter, left + right);
            
            return 1 + Math.max(left, right);
        };
        
        height(root);
        return diameter;
    }
    
    // Validate BST - O(n)
    // Check if tree satisfies BST property: left < root < right
    // Must check against entire valid range, not just immediate children
    static isValidBST(root, min = -Infinity, max = Infinity) {
        if (!root) return true;  // Empty tree is valid BST
        
        // Current node must be within valid range
        if (root.val <= min || root.val >= max) return false;
        
        // Recursively validate subtrees with updated ranges
        // Left subtree: all values must be < root.val
        // Right subtree: all values must be > root.val
        return this.isValidBST(root.left, min, root.val) &&
               this.isValidBST(root.right, root.val, max);
    }
    
    // Kth Smallest Element in BST
    static kthSmallest(root, k) {
        const inorder = (node, result) => {
            if (!node) return;
            
            inorder(node.left, result);
            result.push(node.val);
            inorder(node.right, result);
        };
        
        const result = [];
        inorder(root, result);
        return result[k - 1];
    }
    
    // Binary Tree Right Side View
    static rightSideView(root) {
        if (!root) return [];
        
        const result = [];
        const queue = [root];
        
        while (queue.length > 0) {
            const levelSize = queue.length;
            
            for (let i = 0; i < levelSize; i++) {
                const node = queue.shift();
                
                if (i === levelSize - 1) {
                    result.push(node.val);
                }
                
                if (node.left) queue.push(node.left);
                if (node.right) queue.push(node.right);
            }
        }
        
        return result;
    }
    
    // Serialize and Deserialize Binary Tree
    static serialize(root) {
        if (!root) return "null";
        
        return `${root.val},${this.serialize(root.left)},${this.serialize(root.right)}`;
    }
    
    static deserialize(data) {
        const values = data.split(',');
        
        const buildTree = () => {
            const val = values.shift();
            
            if (val === "null") return null;
            
            const node = new TreeNode(parseInt(val));
            node.left = buildTree();
            node.right = buildTree();
            
            return node;
        };
        
        return buildTree();
    }
    
    // Binary Tree Maximum Path Sum
    static maxPathSum(root) {
        let maxSum = -Infinity;
        
        const maxGain = (node) => {
            if (!node) return 0;
            
            const leftGain = Math.max(maxGain(node.left), 0);
            const rightGain = Math.max(maxGain(node.right), 0);
            
            const priceNewPath = node.val + leftGain + rightGain;
            maxSum = Math.max(maxSum, priceNewPath);
            
            return node.val + Math.max(leftGain, rightGain);
        };
        
        maxGain(root);
        return maxSum;
    }
    
    // Construct Binary Tree from Preorder and Inorder
    static buildTree(preorder, inorder) {
        if (preorder.length === 0) return null;
        
        const rootVal = preorder[0];
        const root = new TreeNode(rootVal);
        const mid = inorder.indexOf(rootVal);
        
        root.left = this.buildTree(
            preorder.slice(1, mid + 1),
            inorder.slice(0, mid)
        );
        root.right = this.buildTree(
            preorder.slice(mid + 1),
            inorder.slice(mid + 1)
        );
        
        return root;
    }
}

// Testing
const bst = new BST();
bst.insert(10);
bst.insert(5);
bst.insert(15);
bst.insert(3);
bst.insert(7);
console.log(bst.inorder());  // [3, 5, 7, 10, 15]
console.log(bst.levelOrder());  // [[10], [5, 15], [3, 7]]
```

### 2.5 Heaps (Priority Queues)

```javascript
// Min Heap Implementation
class MinHeap {
    constructor() {
        this.heap = [];
    }
    
    // Insert - O(log n)
    insert(val) {
        this.heap.push(val);
        this.bubbleUp();
    }
    
    bubbleUp() {
        let idx = this.heap.length - 1;
        const element = this.heap[idx];
        
        while (idx > 0) {
            const parentIdx = Math.floor((idx - 1) / 2);
            const parent = this.heap[parentIdx];
            
            if (element >= parent) break;
            
            this.heap[idx] = parent;
            idx = parentIdx;
        }
        
        this.heap[idx] = element;
    }
    
    // Extract Min - O(log n)
    extractMin() {
        if (this.heap.length === 0) return null;
        if (this.heap.length === 1) return this.heap.pop();
        
        const min = this.heap[0];
        this.heap[0] = this.heap.pop();
        this.sinkDown();
        
        return min;
    }
    
    sinkDown() {
        let idx = 0;
        const length = this.heap.length;
        const element = this.heap[0];
        
        while (true) {
            const leftChildIdx = 2 * idx + 1;
            const rightChildIdx = 2 * idx + 2;
            let leftChild, rightChild;
            let swap = null;
            
            if (leftChildIdx < length) {
                leftChild = this.heap[leftChildIdx];
                if (leftChild < element) {
                    swap = leftChildIdx;
                }
            }
            
            if (rightChildIdx < length) {
                rightChild = this.heap[rightChildIdx];
                if (
                    (swap === null && rightChild < element) ||
                    (swap !== null && rightChild < leftChild)
                ) {
                    swap = rightChildIdx;
                }
            }
            
            if (swap === null) break;
            
            this.heap[idx] = this.heap[swap];
            idx = swap;
        }
        
        this.heap[idx] = element;
    }
    
    // Peek - O(1)
    peek() {
        return this.heap[0] || null;
    }
    
    // Size - O(1)
    size() {
        return this.heap.length;
    }
    
    // Build Heap from Array - O(n)
    buildHeap(arr) {
        this.heap = arr;
        
        for (let i = Math.floor(this.heap.length / 2) - 1; i >= 0; i--) {
            this.heapifyDown(i);
        }
    }
    
    heapifyDown(idx) {
        const length = this.heap.length;
        const element = this.heap[idx];
        
        while (true) {
            const leftChildIdx = 2 * idx + 1;
            const rightChildIdx = 2 * idx + 2;
            let swap = null;
            
            if (leftChildIdx < length && this.heap[leftChildIdx] < element) {
                swap = leftChildIdx;
            }
            
            if (rightChildIdx < length) {
                if (
                    (swap === null && this.heap[rightChildIdx] < element) ||
                    (swap !== null && this.heap[rightChildIdx] < this.heap[leftChildIdx])
                ) {
                    swap = rightChildIdx;
                }
            }
            
            if (swap === null) break;
            
            this.heap[idx] = this.heap[swap];
            idx = swap;
        }
        
        this.heap[idx] = element;
    }
}

// Max Heap Implementation
class MaxHeap {
    constructor() {
        this.heap = [];
    }
    
    insert(val) {
        this.heap.push(val);
        this.bubbleUp();
    }
    
    bubbleUp() {
        let idx = this.heap.length - 1;
        const element = this.heap[idx];
        
        while (idx > 0) {
            const parentIdx = Math.floor((idx - 1) / 2);
            const parent = this.heap[parentIdx];
            
            if (element <= parent) break;
            
            this.heap[idx] = parent;
            idx = parentIdx;
        }
        
        this.heap[idx] = element;
    }
    
    extractMax() {
        if (this.heap.length === 0) return null;
        if (this.heap.length === 1) return this.heap.pop();
        
        const max = this.heap[0];
        this.heap[0] = this.heap.pop();
        this.sinkDown();
        
        return max;
    }
    
    sinkDown() {
        let idx = 0;
        const length = this.heap.length;
        const element = this.heap[0];
        
        while (true) {
            const leftChildIdx = 2 * idx + 1;
            const rightChildIdx = 2 * idx + 2;
            let swap = null;
            
            if (leftChildIdx < length && this.heap[leftChildIdx] > element) {
                swap = leftChildIdx;
            }
            
            if (rightChildIdx < length) {
                if (
                    (swap === null && this.heap[rightChildIdx] > element) ||
                    (swap !== null && this.heap[rightChildIdx] > this.heap[leftChildIdx])
                ) {
                    swap = rightChildIdx;
                }
            }
            
            if (swap === null) break;
            
            this.heap[idx] = this.heap[swap];
            idx = swap;
        }
        
        this.heap[idx] = element;
    }
    
    peek() {
        return this.heap[0] || null;
    }
}

// Heap Problems
class HeapProblems {
    // Kth Largest Element
    static findKthLargest(nums, k) {
        const minHeap = new MinHeap();
        
        for (let num of nums) {
            minHeap.insert(num);
            if (minHeap.size() > k) {
                minHeap.extractMin();
            }
        }
        
        return minHeap.peek();
    }
    
    // Top K Frequent Elements
    static topKFrequent(nums, k) {
        const freqMap = new Map();
        
        for (let num of nums) {
            freqMap.set(num, (freqMap.get(num) || 0) + 1);
        }
        
        const heap = [];
        
        for (let [num, freq] of freqMap) {
            heap.push([freq, num]);
        }
        
        heap.sort((a, b) => b[0] - a[0]);
        
        return heap.slice(0, k).map(pair => pair[1]);
    }
    
    // Merge K Sorted Lists
    static mergeKLists(lists) {
        const minHeap = new MinHeap();
        const dummy = new ListNode(0);
        let current = dummy;
        
        // Add first element from each list
        for (let list of lists) {
            if (list) {
                minHeap.insert({ val: list.val, node: list });
            }
        }
        
        while (minHeap.size() > 0) {
            const { val, node } = minHeap.extractMin();
            current.next = new ListNode(val);
            current = current.next;
            
            if (node.next) {
                minHeap.insert({ val: node.next.val, node: node.next });
            }
        }
        
        return dummy.next;
    }
    
    // Find Median from Data Stream
    static MedianFinder = class {
        constructor() {
            this.maxHeap = new MaxHeap();  // Lower half
            this.minHeap = new MinHeap();  // Upper half
        }
        
        addNum(num) {
            this.maxHeap.insert(num);
            this.minHeap.insert(this.maxHeap.extractMax());
            
            if (this.maxHeap.size() < this.minHeap.size()) {
                this.maxHeap.insert(this.minHeap.extractMin());
            }
        }
        
        findMedian() {
            if (this.maxHeap.size() > this.minHeap.size()) {
                return this.maxHeap.peek();
            }
            return (this.maxHeap.peek() + this.minHeap.peek()) / 2;
        }
    };
    
    // Sliding Window Median
    static medianSlidingWindow(nums, k) {
        const result = [];
        
        for (let i = 0; i <= nums.length - k; i++) {
            const window = nums.slice(i, i + k).sort((a, b) => a - b);
            
            if (k % 2 === 0) {
                result.push((window[k / 2 - 1] + window[k / 2]) / 2);
            } else {
                result.push(window[Math.floor(k / 2)]);
            }
        }
        
        return result;
    }
}

// Testing
const minHeap = new MinHeap();
minHeap.insert(3);
minHeap.insert(1);
minHeap.insert(5);
minHeap.insert(2);
console.log(minHeap.extractMin());  // 1
console.log(minHeap.peek());  // 2
```

---

## Part 3: Advanced Data Structures

### 3.1 Graphs

**Graph Fundamentals**:
- **Graph**: Collection of vertices (nodes) connected by edges
- **Types**:
  - **Directed**: Edges have direction (A→B)
  - **Undirected**: Edges are bidirectional (A↔B)
  - **Weighted**: Edges have values/costs
  - **Unweighted**: All edges equal

**Representations**:
1. **Adjacency Matrix**: 2D array, O(V²) space, O(1) edge lookup
2. **Adjacency List**: Map/Array of lists, O(V+E) space, O(degree) lookup
3. **Edge List**: Array of edges, O(E) space

**Common Algorithms**:
- **DFS**: Explore as deep as possible first (uses stack/recursion)
- **BFS**: Explore level by level (uses queue)
- **Dijkstra**: Shortest path in weighted graphs
- **Topological Sort**: Order vertices in directed acyclic graph

```javascript
// Graph Representation - Adjacency List
// Most common representation: space-efficient for sparse graphs
class Graph {
    constructor() {
        this.adjacencyList = new Map();  // vertex -> [neighbors]
    }
    
    // Add vertex - O(1)
    addVertex(vertex) {
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);  // Initialize empty neighbor list
        }
    }
    
    // Add edge (undirected) - O(1)
    // Creates bidirectional connection between v1 and v2
    addEdge(v1, v2) {
        this.addVertex(v1);
        this.addVertex(v2);
        this.adjacencyList.get(v1).push(v2);  // v1 → v2
        this.adjacencyList.get(v2).push(v1);  // v2 → v1 (undirected)
    }
    
    // Add directed edge
    addDirectedEdge(from, to) {
        this.addVertex(from);
        this.addVertex(to);
        this.adjacencyList.get(from).push(to);
    }
    
    // Remove edge
    removeEdge(v1, v2) {
        this.adjacencyList.set(
            v1,
            this.adjacencyList.get(v1).filter(v => v !== v2)
        );
        this.adjacencyList.set(
            v2,
            this.adjacencyList.get(v2).filter(v => v !== v1)
        );
    }
    
    // Remove vertex
    removeVertex(vertex) {
        while (this.adjacencyList.get(vertex).length) {
            const adjacentVertex = this.adjacencyList.get(vertex).pop();
            this.removeEdge(vertex, adjacentVertex);
        }
        this.adjacencyList.delete(vertex);
    }
    
    // DFS Recursive - O(V + E) time, O(V) space
    // Strategy: Go deep into graph before backtracking
    // Use cases: Path finding, cycle detection, topological sort
    dfsRecursive(start) {
        const result = [];
        const visited = new Set();  // Track visited vertices
        
        const dfs = (vertex) => {
            if (!vertex) return;
            
            visited.add(vertex);  // Mark as visited
            result.push(vertex);  // Process vertex
            
            // Visit all unvisited neighbors
            for (let neighbor of this.adjacencyList.get(vertex)) {
                if (!visited.has(neighbor)) {
                    dfs(neighbor);  // Recursive call (go deeper)
                }
            }
        };
        
        dfs(start);
        return result;
    }
    
    // DFS Iterative - O(V + E) time, O(V) space
    // Same as recursive but uses explicit stack instead of call stack
    dfsIterative(start) {
        const stack = [start];  // Use array as stack
        const result = [];
        const visited = new Set([start]);
        
        while (stack.length > 0) {
            const vertex = stack.pop();  // Pop from stack (LIFO)
            result.push(vertex);
            
            // Push unvisited neighbors onto stack
            for (let neighbor of this.adjacencyList.get(vertex)) {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    stack.push(neighbor);  // Add to stack for later exploration
                }
            }
        }
        
        return result;
    }
    
    // BFS - O(V + E) time, O(V) space
    // Strategy: Explore all neighbors before going deeper
    // Use cases: Shortest path (unweighted), level-order traversal
    bfs(start) {
        const queue = [start];  // Use array as queue
        const result = [];
        const visited = new Set([start]);
        
        while (queue.length > 0) {
            const vertex = queue.shift();  // Dequeue from front (FIFO)
            result.push(vertex);
            
            // Enqueue all unvisited neighbors
            for (let neighbor of this.adjacencyList.get(vertex)) {
                if (!visited.has(neighbor)) {
                    visited.add(neighbor);
                    queue.push(neighbor);  // Add to queue for later exploration
                }
            }
        }
        
        return result;
    }
}

// Weighted Graph
class WeightedGraph {
    constructor() {
        this.adjacencyList = new Map();
    }
    
    addVertex(vertex) {
        if (!this.adjacencyList.has(vertex)) {
            this.adjacencyList.set(vertex, []);
        }
    }
    
    addEdge(v1, v2, weight) {
        this.addVertex(v1);
        this.addVertex(v2);
        this.adjacencyList.get(v1).push({ node: v2, weight });
        this.adjacencyList.get(v2).push({ node: v1, weight });
    }
    
    // Dijkstra's Algorithm
    dijkstra(start, end) {
        const distances = new Map();
        const previous = new Map();
        const pq = new PriorityQueue();
        
        // Initialize
        for (let vertex of this.adjacencyList.keys()) {
            if (vertex === start) {
                distances.set(vertex, 0);
                pq.enqueue(vertex, 0);
            } else {
                distances.set(vertex, Infinity);
            }
            previous.set(vertex, null);
        }
        
        while (pq.heap.length > 0) {
            const { val: current } = pq.dequeue();
            
            if (current === end) {
                // Build path
                const path = [];
                let temp = end;
                
                while (temp) {
                    path.push(temp);
                    temp = previous.get(temp);
                }
                
                return {
                    path: path.reverse(),
                    distance: distances.get(end)
                };
            }
            
            if (distances.get(current) !== Infinity) {
                for (let neighbor of this.adjacencyList.get(current)) {
                    const distance = distances.get(current) + neighbor.weight;
                    
                    if (distance < distances.get(neighbor.node)) {
                        distances.set(neighbor.node, distance);
                        previous.set(neighbor.node, current);
                        pq.enqueue(neighbor.node, distance);
                    }
                }
            }
        }
        
        return { path: [], distance: Infinity };
    }
}

// Graph Algorithms
class GraphAlgorithms {
    // Number of Islands (DFS)
    static numIslands(grid) {
        if (!grid || grid.length === 0) return 0;
        
        let count = 0;
        const rows = grid.length;
        const cols = grid[0].length;
        
        const dfs = (i, j) => {
            if (i < 0 || i >= rows || j < 0 || j >= cols || grid[i][j] === '0') {
                return;
            }
            
            grid[i][j] = '0';
            dfs(i + 1, j);
            dfs(i - 1, j);
            dfs(i, j + 1);
            dfs(i, j - 1);
        };
        
        for (let i = 0; i < rows; i++) {
            for (let j = 0; j < cols; j++) {
                if (grid[i][j] === '1') {
                    count++;
                    dfs(i, j);
                }
            }
        }
        
        return count;
    }
    
    // Clone Graph
    static cloneGraph(node) {
        if (!node) return null;
        
        const visited = new Map();
        
        const clone = (node) => {
            if (visited.has(node.val)) {
                return visited.get(node.val);
            }
            
            const newNode = { val: node.val, neighbors: [] };
            visited.set(node.val, newNode);
            
            for (let neighbor of node.neighbors) {
                newNode.neighbors.push(clone(neighbor));
            }
            
            return newNode;
        };
        
        return clone(node);
    }
    
    // Course Schedule (Cycle Detection)
    static canFinish(numCourses, prerequisites) {
        const graph = new Map();
        const indegree = new Array(numCourses).fill(0);
        
        // Build graph
        for (let [course, prereq] of prerequisites) {
            if (!graph.has(prereq)) {
                graph.set(prereq, []);
            }
            graph.get(prereq).push(course);
            indegree[course]++;
        }
        
        // Find all courses with no prerequisites
        const queue = [];
        for (let i = 0; i < numCourses; i++) {
            if (indegree[i] === 0) {
                queue.push(i);
            }
        }
        
        let count = 0;
        
        while (queue.length > 0) {
            const course = queue.shift();
            count++;
            
            if (graph.has(course)) {
                for (let nextCourse of graph.get(course)) {
                    indegree[nextCourse]--;
                    if (indegree[nextCourse] === 0) {
                        queue.push(nextCourse);
                    }
                }
            }
        }
        
        return count === numCourses;
    }
    
    // Topological Sort
    static topologicalSort(numCourses, prerequisites) {
        const graph = new Map();
        const indegree = new Array(numCourses).fill(0);
        
        for (let [course, prereq] of prerequisites) {
            if (!graph.has(prereq)) {
                graph.set(prereq, []);
            }
            graph.get(prereq).push(course);
            indegree[course]++;
        }
        
        const queue = [];
        for (let i = 0; i < numCourses; i++) {
            if (indegree[i] === 0) {
                queue.push(i);
            }
        }
        
        const result = [];
        
        while (queue.length > 0) {
            const course = queue.shift();
            result.push(course);
            
            if (graph.has(course)) {
                for (let nextCourse of graph.get(course)) {
                    indegree[nextCourse]--;
                    if (indegree[nextCourse] === 0) {
                        queue.push(nextCourse);
                    }
                }
            }
        }
        
        return result.length === numCourses ? result : [];
    }
    
    // Union-Find (Disjoint Set)
    static UnionFind = class {
        constructor(size) {
            this.parent = Array(size).fill(0).map((_, i) => i);
            this.rank = Array(size).fill(0);
        }
        
        find(x) {
            if (this.parent[x] !== x) {
                this.parent[x] = this.find(this.parent[x]);
            }
            return this.parent[x];
        }
        
        union(x, y) {
            const rootX = this.find(x);
            const rootY = this.find(y);
            
            if (rootX === rootY) return false;
            
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
        
        connected(x, y) {
            return this.find(x) === this.find(y);
        }
    };
    
    // Minimum Spanning Tree - Kruskal's Algorithm
    static kruskalMST(n, edges) {
        edges.sort((a, b) => a[2] - b[2]);
        const uf = new this.UnionFind(n);
        const mst = [];
        let totalWeight = 0;
        
        for (let [u, v, weight] of edges) {
            if (uf.union(u, v)) {
                mst.push([u, v, weight]);
                totalWeight += weight;
            }
        }
        
        return { mst, totalWeight };
    }
    
    // Detect Cycle in Undirected Graph
    static hasCycle(n, edges) {
        const uf = new this.UnionFind(n);
        
        for (let [u, v] of edges) {
            if (uf.connected(u, v)) {
                return true;
            }
            uf.union(u, v);
        }
        
        return false;
    }
    
    // Word Ladder
    static ladderLength(beginWord, endWord, wordList) {
        const wordSet = new Set(wordList);
        if (!wordSet.has(endWord)) return 0;
        
        const queue = [[beginWord, 1]];
        
        while (queue.length > 0) {
            const [word, level] = queue.shift();
            
            if (word === endWord) return level;
            
            for (let i = 0; i < word.length; i++) {
                for (let c = 97; c <= 122; c++) {
                    const newWord = word.slice(0, i) + String.fromCharCode(c) + word.slice(i + 1);
                    
                    if (wordSet.has(newWord)) {
                        queue.push([newWord, level + 1]);
                        wordSet.delete(newWord);
                    }
                }
            }
        }
        
        return 0;
    }
}

// Testing
const graph = new Graph();
graph.addVertex('A');
graph.addVertex('B');
graph.addVertex('C');
graph.addEdge('A', 'B');
graph.addEdge('B', 'C');
console.log(graph.dfsRecursive('A'));  // ['A', 'B', 'C']
console.log(graph.bfs('A'));  // ['A', 'B', 'C']
```

### 3.2 Tries (Prefix Trees)

```javascript
// Trie Node
class TrieNode {
    constructor() {
        this.children = new Map();
        this.isEndOfWord = false;
        this.count = 0;  // For word frequency
    }
}

// Trie Implementation
class Trie {
    constructor() {
        this.root = new TrieNode();
    }
    
    // Insert word - O(m) where m is word length
    insert(word) {
        let node = this.root;
        
        for (let char of word) {
            if (!node.children.has(char)) {
                node.children.set(char, new TrieNode());
            }
            node = node.children.get(char);
            node.count++;
        }
        
        node.isEndOfWord = true;
    }
    
    // Search for exact word - O(m)
    search(word) {
        let node = this.root;
        
        for (let char of word) {
            if (!node.children.has(char)) {
                return false;
            }
            node = node.children.get(char);
        }
        
        return node.isEndOfWord;
    }
    
    // Search for prefix - O(m)
    startsWith(prefix) {
        let node = this.root;
        
        for (let char of prefix) {
            if (!node.children.has(char)) {
                return false;
            }
            node = node.children.get(char);
        }
        
        return true;
    }
    
    // Delete word - O(m)
    delete(word) {
        const deleteHelper = (node, word, index) => {
            if (index === word.length) {
                if (!node.isEndOfWord) return false;
                node.isEndOfWord = false;
                return node.children.size === 0;
            }
            
            const char = word[index];
            const childNode = node.children.get(char);
            
            if (!childNode) return false;
            
            const shouldDeleteChild = deleteHelper(childNode, word, index + 1);
            
            if (shouldDeleteChild) {
                node.children.delete(char);
                return node.children.size === 0 && !node.isEndOfWord;
            }
            
            return false;
        };
        
        deleteHelper(this.root, word, 0);
    }
    
    // Get all words with prefix
    getAllWordsWithPrefix(prefix) {
        let node = this.root;
        
        // Navigate to prefix
        for (let char of prefix) {
            if (!node.children.has(char)) {
                return [];
            }
            node = node.children.get(char);
        }
        
        const words = [];
        
        const dfs = (node, currentWord) => {
            if (node.isEndOfWord) {
                words.push(currentWord);
            }
            
            for (let [char, childNode] of node.children) {
                dfs(childNode, currentWord + char);
            }
        };
        
        dfs(node, prefix);
        return words;
    }
    
    // Count words with prefix
    countWordsWithPrefix(prefix) {
        let node = this.root;
        
        for (let char of prefix) {
            if (!node.children.has(char)) {
                return 0;
            }
            node = node.children.get(char);
        }
        
        return node.count;
    }
    
    // Autocomplete suggestions
    autocomplete(prefix, limit = 10) {
        const words = this.getAllWordsWithPrefix(prefix);
        return words.slice(0, limit);
    }
}

// Word Dictionary with wildcards
class WordDictionary {
    constructor() {
        this.root = new TrieNode();
    }
    
    addWord(word) {
        let node = this.root;
        
        for (let char of word) {
            if (!node.children.has(char)) {
                node.children.set(char, new TrieNode());
            }
            node = node.children.get(char);
        }
        
        node.isEndOfWord = true;
    }
    
    search(word) {
        const searchHelper = (node, index) => {
            if (index === word.length) {
                return node.isEndOfWord;
            }
            
            const char = word[index];
            
            if (char === '.') {
                for (let childNode of node.children.values()) {
                    if (searchHelper(childNode, index + 1)) {
                        return true;
                    }
                }
                return false;
            } else {
                const childNode = node.children.get(char);
                if (!childNode) return false;
                return searchHelper(childNode, index + 1);
            }
        };
        
        return searchHelper(this.root, 0);
    }
}

// Trie Problems
class TrieProblems {
    // Word Search II
    static findWords(board, words) {
        const trie = new Trie();
        for (let word of words) {
            trie.insert(word);
        }
        
        const result = new Set();
        const rows = board.length;
        const cols = board[0].length;
        
        const dfs = (i, j, node, path) => {
            if (i < 0 || i >= rows || j < 0 || j >= cols) return;
            
            const char = board[i][j];
            if (char === '#' || !node.children.has(char)) return;
            
            node = node.children.get(char);
            path += char;
            
            if (node.isEndOfWord) {
                result.add(path);
            }
            
            const temp = board[i][j];
            board[i][j] = '#';
            
            dfs(i + 1, j, node, path);
            dfs(i - 1, j, node, path);
            dfs(i, j + 1, node, path);
            dfs(i, j - 1, node, path);
            
            board[i][j] = temp;
        };
        
        for (let i = 0; i < rows; i++) {
            for (let j = 0; j < cols; j++) {
                dfs(i, j, trie.root, '');
            }
        }
        
        return Array.from(result);
    }
    
    // Replace Words
    static replaceWords(dictionary, sentence) {
        const trie = new Trie();
        for (let word of dictionary) {
            trie.insert(word);
        }
        
        const findRoot = (word) => {
            let node = trie.root;
            let prefix = '';
            
            for (let char of word) {
                if (!node.children.has(char)) break;
                prefix += char;
                node = node.children.get(char);
                if (node.isEndOfWord) return prefix;
            }
            
            return word;
        };
        
        return sentence.split(' ').map(findRoot).join(' ');
    }
}

// Testing
const trie = new Trie();
trie.insert("apple");
trie.insert("app");
trie.insert("application");
console.log(trie.search("app"));  // true
console.log(trie.startsWith("app"));  // true
console.log(trie.getAllWordsWithPrefix("app"));  // ['app', 'apple', 'application']
```

### 3.3 Segment Trees and Fenwick Trees

```javascript
// Segment Tree
class SegmentTree {
    constructor(arr) {
        this.n = arr.length;
        this.tree = new Array(4 * this.n);
        this.build(arr, 0, 0, this.n - 1);
    }
    
    build(arr, node, start, end) {
        if (start === end) {
            this.tree[node] = arr[start];
            return;
        }
        
        const mid = Math.floor((start + end) / 2);
        const leftNode = 2 * node + 1;
        const rightNode = 2 * node + 2;
        
        this.build(arr, leftNode, start, mid);
        this.build(arr, rightNode, mid + 1, end);
        
        this.tree[node] = this.tree[leftNode] + this.tree[rightNode];
    }
    
    // Range sum query
    query(left, right) {
        return this.queryHelper(0, 0, this.n - 1, left, right);
    }
    
    queryHelper(node, start, end, left, right) {
        if (right < start || left > end) {
            return 0;
        }
        
        if (left <= start && end <= right) {
            return this.tree[node];
        }
        
        const mid = Math.floor((start + end) / 2);
        const leftSum = this.queryHelper(2 * node + 1, start, mid, left, right);
        const rightSum = this.queryHelper(2 * node + 2, mid + 1, end, left, right);
        
        return leftSum + rightSum;
    }
    
    // Update value at index
    update(index, value) {
        this.updateHelper(0, 0, this.n - 1, index, value);
    }
    
    updateHelper(node, start, end, index, value) {
        if (start === end) {
            this.tree[node] = value;
            return;
        }
        
        const mid = Math.floor((start + end) / 2);
        
        if (index <= mid) {
            this.updateHelper(2 * node + 1, start, mid, index, value);
        } else {
            this.updateHelper(2 * node + 2, mid + 1, end, index, value);
        }
        
        this.tree[node] = this.tree[2 * node + 1] + this.tree[2 * node + 2];
    }
}

// Fenwick Tree (Binary Indexed Tree)
class FenwickTree {
    constructor(n) {
        this.n = n;
        this.tree = new Array(n + 1).fill(0);
    }
    
    // Update value at index (add delta)
    update(index, delta) {
        index++;  // 1-indexed
        
        while (index <= this.n) {
            this.tree[index] += delta;
            index += index & (-index);
        }
    }
    
    // Get prefix sum from 0 to index
    query(index) {
        index++;  // 1-indexed
        let sum = 0;
        
        while (index > 0) {
            sum += this.tree[index];
            index -= index & (-index);
        }
        
        return sum;
    }
    
    // Get range sum from left to right
    rangeQuery(left, right) {
        return this.query(right) - (left > 0 ? this.query(left - 1) : 0);
    }
}

// Range Query Problems
class RangeQueryProblems {
    // Range Sum Query - Mutable
    static NumArray = class {
        constructor(nums) {
            this.segTree = new SegmentTree(nums);
            this.nums = nums;
        }
        
        update(index, val) {
            this.segTree.update(index, val);
            this.nums[index] = val;
        }
        
        sumRange(left, right) {
            return this.segTree.query(left, right);
        }
    };
    
    // Count of Range Sum
    static countRangeSum(nums, lower, upper) {
        const n = nums.length;
        const prefixSum = new Array(n + 1).fill(0);
        
        for (let i = 0; i < n; i++) {
            prefixSum[i + 1] = prefixSum[i] + nums[i];
        }
        
        let count = 0;
        
        for (let i = 0; i < n; i++) {
            for (let j = i + 1; j <= n; j++) {
                const sum = prefixSum[j] - prefixSum[i];
                if (sum >= lower && sum <= upper) {
                    count++;
                }
            }
        }
        
        return count;
    }
}

// Testing
const segTree = new SegmentTree([1, 3, 5, 7, 9, 11]);
console.log(segTree.query(1, 3));  // 15 (3 + 5 + 7)
segTree.update(1, 10);
console.log(segTree.query(1, 3));  // 22 (10 + 5 + 7)
```

### 3.4 Advanced Tree Structures

```javascript
// AVL Tree (Self-balancing BST)
class AVLNode {
    constructor(val) {
        this.val = val;
        this.left = null;
        this.right = null;
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
            node.height = 1 + Math.max(this.getHeight(node.left), this.getHeight(node.right));
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
    
    insert(val) {
        this.root = this.insertNode(this.root, val);
    }
    
    insertNode(node, val) {
        if (!node) return new AVLNode(val);
        
        if (val < node.val) {
            node.left = this.insertNode(node.left, val);
        } else if (val > node.val) {
            node.right = this.insertNode(node.right, val);
        } else {
            return node;
        }
        
        this.updateHeight(node);
        
        const balance = this.getBalance(node);
        
        // Left Left Case
        if (balance > 1 && val < node.left.val) {
            return this.rotateRight(node);
        }
        
        // Right Right Case
        if (balance < -1 && val > node.right.val) {
            return this.rotateLeft(node);
        }
        
        // Left Right Case
        if (balance > 1 && val > node.left.val) {
            node.left = this.rotateLeft(node.left);
            return this.rotateRight(node);
        }
        
        // Right Left Case
        if (balance < -1 && val < node.right.val) {
            node.right = this.rotateRight(node.right);
            return this.rotateLeft(node);
        }
        
        return node;
    }
}

// Red-Black Tree Node
class RBNode {
    constructor(val, color = 'RED') {
        this.val = val;
        this.color = color;
        this.left = null;
        this.right = null;
        this.parent = null;
    }
}

// B-Tree Node
class BTreeNode {
    constructor(t, leaf = true) {
        this.t = t;  // Minimum degree
        this.keys = [];
        this.children = [];
        this.leaf = leaf;
        this.n = 0;  // Current number of keys
    }
}

// Testing
const avl = new AVLTree();
avl.insert(10);
avl.insert(20);
avl.insert(30);
avl.insert(40);
avl.insert(50);
avl.insert(25);
console.log("AVL Tree created successfully");
```

---

## Part 4: Algorithm Design Paradigms

### 4.1 Divide and Conquer

```javascript
class DivideAndConquer {
    // Binary Search (D&C approach)
    static binarySearch(arr, target, left = 0, right = arr.length - 1) {
        if (left > right) return -1;
        
        const mid = Math.floor((left + right) / 2);
        
        if (arr[mid] === target) return mid;
        if (arr[mid] > target) return this.binarySearch(arr, target, left, mid - 1);
        return this.binarySearch(arr, target, mid + 1, right);
    }
    
    // Merge Sort
    static mergeSort(arr) {
        if (arr.length <= 1) return arr;
        
        const mid = Math.floor(arr.length / 2);
        const left = this.mergeSort(arr.slice(0, mid));
        const right = this.mergeSort(arr.slice(mid));
        
        return this.merge(left, right);
    }
    
    static merge(left, right) {
        const result = [];
        let i = 0, j = 0;
        
        while (i < left.length && j < right.length) {
            if (left[i] <= right[j]) {
                result.push(left[i++]);
            } else {
                result.push(right[j++]);
            }
        }
        
        return result.concat(left.slice(i)).concat(right.slice(j));
    }
    
    // Quick Select (Kth smallest element)
    static quickSelect(arr, k) {
        if (arr.length === 1) return arr[0];
        
        const pivot = arr[Math.floor(arr.length / 2)];
        const left = arr.filter(x => x < pivot);
        const middle = arr.filter(x => x === pivot);
        const right = arr.filter(x => x > pivot);
        
        if (k <= left.length) {
            return this.quickSelect(left, k);
        } else if (k <= left.length + middle.length) {
            return pivot;
        } else {
            return this.quickSelect(right, k - left.length - middle.length);
        }
    }
    
    // Maximum Subarray (D&C approach)
    static maxSubarray(arr, left = 0, right = arr.length - 1) {
        if (left === right) return arr[left];
        
        const mid = Math.floor((left + right) / 2);
        
        const leftMax = this.maxSubarray(arr, left, mid);
        const rightMax = this.maxSubarray(arr, mid + 1, right);
        const crossMax = this.maxCrossingSum(arr, left, mid, right);
        
        return Math.max(leftMax, rightMax, crossMax);
    }
    
    static maxCrossingSum(arr, left, mid, right) {
        let leftSum = -Infinity;
        let sum = 0;
        
        for (let i = mid; i >= left; i--) {
            sum += arr[i];
            leftSum = Math.max(leftSum, sum);
        }
        
        let rightSum = -Infinity;
        sum = 0;
        
        for (let i = mid + 1; i <= right; i++) {
            sum += arr[i];
            rightSum = Math.max(rightSum, sum);
        }
        
        return leftSum + rightSum;
    }
    
    // Count Inversions
    static countInversions(arr) {
        if (arr.length <= 1) return { arr, count: 0 };
        
        const mid = Math.floor(arr.length / 2);
        const left = this.countInversions(arr.slice(0, mid));
        const right = this.countInversions(arr.slice(mid));
        const merged = this.mergeAndCount(left.arr, right.arr);
        
        return {
            arr: merged.arr,
            count: left.count + right.count + merged.count
        };
    }
    
    static mergeAndCount(left, right) {
        const result = [];
        let i = 0, j = 0, count = 0;
        
        while (i < left.length && j < right.length) {
            if (left[i] <= right[j]) {
                result.push(left[i++]);
            } else {
                result.push(right[j++]);
                count += left.length - i;
            }
        }
        
        return {
            arr: result.concat(left.slice(i)).concat(right.slice(j)),
            count
        };
    }
    
    // Closest Pair of Points
    static closestPair(points) {
        points.sort((a, b) => a[0] - b[0]);
        
        const distance = (p1, p2) => {
            return Math.sqrt(Math.pow(p1[0] - p2[0], 2) + Math.pow(p1[1] - p2[1], 2));
        };
        
        const closestPairRec = (px, py) => {
            const n = px.length;
            
            if (n <= 3) {
                let minDist = Infinity;
                for (let i = 0; i < n; i++) {
                    for (let j = i + 1; j < n; j++) {
                        minDist = Math.min(minDist, distance(px[i], px[j]));
                    }
                }
                return minDist;
            }
            
            const mid = Math.floor(n / 2);
            const midPoint = px[mid];
            
            const pyl = py.filter(p => p[0] <= midPoint[0]);
            const pyr = py.filter(p => p[0] > midPoint[0]);
            
            const dl = closestPairRec(px.slice(0, mid), pyl);
            const dr = closestPairRec(px.slice(mid), pyr);
            
            const d = Math.min(dl, dr);
            
            const strip = py.filter(p => Math.abs(p[0] - midPoint[0]) < d);
            
            let minDist = d;
            for (let i = 0; i < strip.length; i++) {
                for (let j = i + 1; j < strip.length && (strip[j][1] - strip[i][1]) < minDist; j++) {
                    minDist = Math.min(minDist, distance(strip[i], strip[j]));
                }
            }
            
            return minDist;
        };
        
        const py = [...points].sort((a, b) => a[1] - b[1]);
        return closestPairRec(points, py);
    }
}

// Testing
console.log(DivideAndConquer.quickSelect([3, 2, 1, 5, 6, 4], 2));  // 2
console.log(DivideAndConquer.maxSubarray([-2, 1, -3, 4, -1, 2, 1, -5, 4]));  // 6
```

### 4.2 Dynamic Programming

**Core Concept**: Break problem into overlapping subproblems and store results to avoid recomputation.

**When to Use DP**:
1. **Optimal Substructure**: Optimal solution contains optimal solutions to subproblems
2. **Overlapping Subproblems**: Same subproblems solved multiple times

**Two Approaches**:
1. **Memoization (Top-Down)**: 
   - Start with original problem, recursively solve
   - Cache results in memo object/array
   - Easier to write, uses recursion

2. **Tabulation (Bottom-Up)**:
   - Start with smallest subproblems, build up
   - Fill DP table iteratively
   - More space-efficient, avoids recursion overhead

**Classic DP Problems**:
- Fibonacci, Climbing Stairs (1D DP)
- Knapsack, Coin Change (1D/2D DP)
- Longest Common Subsequence (2D DP)
- Edit Distance (2D DP)

```javascript
class DynamicProgramming {
    // Fibonacci - Memoization (Top-Down)
    // Time: O(n) - each subproblem solved once
    // Space: O(n) - memo object + recursion stack
    // Without memoization: O(2^n) - exponential!
    static fibMemo(n, memo = {}) {
        if (n <= 1) return n;  // Base cases: fib(0)=0, fib(1)=1
        if (memo[n]) return memo[n];  // Already computed? Return cached result
        
        // Compute and cache result: fib(n) = fib(n-1) + fib(n-2)
        memo[n] = this.fibMemo(n - 1, memo) + this.fibMemo(n - 2, memo);
        return memo[n];
    }
    
    // Fibonacci - Tabulation (Bottom-Up)
    // Time: O(n) - single loop
    // Space: O(n) - dp array
    // Build solution from smallest subproblems up
    static fibTab(n) {
        if (n <= 1) return n;
        
        const dp = [0, 1];  // Base cases
        // Build up: dp[i] = dp[i-1] + dp[i-2]
        for (let i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        
        return dp[n];
    }
    
    // Fibonacci - Space Optimized
    // Time: O(n)
    // Space: O(1) - only store last two values!
    // Observation: We only need previous 2 values, not entire array
    static fibOptimized(n) {
        if (n <= 1) return n;
        
        let prev2 = 0, prev1 = 1;  // Last two fibonacci numbers
        
        for (let i = 2; i <= n; i++) {
            const curr = prev1 + prev2;  // Current fibonacci
            prev2 = prev1;  // Shift window
            prev1 = curr;
        }
        
        return prev1;
    }
    
    // Climbing Stairs
    static climbStairs(n) {
        if (n <= 2) return n;
        
        const dp = [0, 1, 2];
        for (let i = 3; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        
        return dp[n];
    }
    
    // House Robber
    static rob(nums) {
        if (nums.length === 0) return 0;
        if (nums.length === 1) return nums[0];
        
        const dp = [nums[0], Math.max(nums[0], nums[1])];
        
        for (let i = 2; i < nums.length; i++) {
            dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
        }
        
        return dp[nums.length - 1];
    }
    
    // House Robber II (Circular)
    static robCircular(nums) {
        if (nums.length === 1) return nums[0];
        
        const robRange = (start, end) => {
            let prev2 = 0, prev1 = 0;
            
            for (let i = start; i <= end; i++) {
                const curr = Math.max(prev1, prev2 + nums[i]);
                prev2 = prev1;
                prev1 = curr;
            }
            
            return prev1;
        };
        
        return Math.max(
            robRange(0, nums.length - 2),
            robRange(1, nums.length - 1)
        );
    }
    
    // Longest Increasing Subsequence - O(n²)
    // Find longest subsequence where elements are in increasing order
    // Example: [10,9,2,5,3,7,101,18] → [2,3,7,101] (length 4)
    static lengthOfLIS(nums) {
        if (nums.length === 0) return 0;
        
        // dp[i] = length of longest increasing subsequence ending at index i
        const dp = new Array(nums.length).fill(1);  // Each element is subsequence of length 1
        let maxLength = 1;
        
        for (let i = 1; i < nums.length; i++) {
            // Check all previous elements
            for (let j = 0; j < i; j++) {
                // If current > previous, we can extend that subsequence
                if (nums[i] > nums[j]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            maxLength = Math.max(maxLength, dp[i]);
        }
        
        return maxLength;
    }
    
    // Longest Common Subsequence - O(m * n)
    // Find longest subsequence common to both strings
    // Example: "abcde", "ace" → "ace" (length 3)
    // Not necessarily contiguous!
    static longestCommonSubsequence(text1, text2) {
        const m = text1.length;
        const n = text2.length;
        // dp[i][j] = LCS length of text1[0...i-1] and text2[0...j-1]
        const dp = Array(m + 1).fill(0).map(() => Array(n + 1).fill(0));
        
        for (let i = 1; i <= m; i++) {
            for (let j = 1; j <= n; j++) {
                if (text1[i - 1] === text2[j - 1]) {
                    // Characters match: extend LCS by 1
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    // No match: take max of skipping either character
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        
        return dp[m][n];
    }
    
    // Edit Distance
    static minDistance(word1, word2) {
        const m = word1.length;
        const n = word2.length;
        const dp = Array(m + 1).fill(0).map(() => Array(n + 1).fill(0));
        
        for (let i = 0; i <= m; i++) dp[i][0] = i;
        for (let j = 0; j <= n; j++) dp[0][j] = j;
        
        for (let i = 1; i <= m; i++) {
            for (let j = 1; j <= n; j++) {
                if (word1[i - 1] === word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = 1 + Math.min(
                        dp[i - 1][j],      // delete
                        dp[i][j - 1],      // insert
                        dp[i - 1][j - 1]   // replace
                    );
                }
            }
        }
        
        return dp[m][n];
    }
    
    // Coin Change - O(amount * n)
    // Find minimum coins needed to make amount
    // Example: coins=[1,2,5], amount=11 → 3 (5+5+1)
    static coinChange(coins, amount) {
        // dp[i] = minimum coins needed to make amount i
        const dp = new Array(amount + 1).fill(Infinity);
        dp[0] = 0;  // Base case: 0 coins for amount 0
        
        for (let i = 1; i <= amount; i++) {
            // Try each coin denomination
            for (let coin of coins) {
                if (i >= coin) {
                    // Can use this coin: try using it and see if better
                    dp[i] = Math.min(dp[i], dp[i - coin] + 1);
                }
            }
        }
        
        return dp[amount] === Infinity ? -1 : dp[amount];
    }
    
    // Coin Change II (Number of combinations)
    static change(amount, coins) {
        const dp = new Array(amount + 1).fill(0);
        dp[0] = 1;
        
        for (let coin of coins) {
            for (let i = coin; i <= amount; i++) {
                dp[i] += dp[i - coin];
            }
        }
        
        return dp[amount];
    }
    
    // 0/1 Knapsack - O(n * capacity)
    // Choose items to maximize value without exceeding capacity
    // Each item can be taken 0 or 1 time (can't split items)
    static knapsack(weights, values, capacity) {
        const n = weights.length;
        // dp[i][w] = max value using first i items with capacity w
        const dp = Array(n + 1).fill(0).map(() => Array(capacity + 1).fill(0));
        
        for (let i = 1; i <= n; i++) {
            for (let w = 1; w <= capacity; w++) {
                if (weights[i - 1] <= w) {
                    // Item fits: choose max of taking it or leaving it
                    dp[i][w] = Math.max(
                        dp[i - 1][w],  // Don't take item i
                        dp[i - 1][w - weights[i - 1]] + values[i - 1]  // Take item i
                    );
                } else {
                    // Item doesn't fit: can't take it
                    dp[i][w] = dp[i - 1][w];
                }
            }
        }
        
        return dp[n][capacity];
    }
    
    // Unbounded Knapsack
    static unboundedKnapsack(weights, values, capacity) {
        const dp = new Array(capacity + 1).fill(0);
        
        for (let w = 1; w <= capacity; w++) {
            for (let i = 0; i < weights.length; i++) {
                if (weights[i] <= w) {
                    dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
                }
            }
        }
        
        return dp[capacity];
    }
    
    // Partition Equal Subset Sum
    static canPartition(nums) {
        const sum = nums.reduce((a, b) => a + b, 0);
        if (sum % 2 !== 0) return false;
        
        const target = sum / 2;
        const dp = new Array(target + 1).fill(false);
        dp[0] = true;
        
        for (let num of nums) {
            for (let i = target; i >= num; i--) {
                dp[i] = dp[i] || dp[i - num];
            }
        }
        
        return dp[target];
    }
    
    // Word Break
    static wordBreak(s, wordDict) {
        const wordSet = new Set(wordDict);
        const dp = new Array(s.length + 1).fill(false);
        dp[0] = true;
        
        for (let i = 1; i <= s.length; i++) {
            for (let j = 0; j < i; j++) {
                if (dp[j] && wordSet.has(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }
            }
        }
        
        return dp[s.length];
    }
    
    // Unique Paths
    static uniquePaths(m, n) {
        const dp = Array(m).fill(0).map(() => Array(n).fill(1));
        
        for (let i = 1; i < m; i++) {
            for (let j = 1; j < n; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        
        return dp[m - 1][n - 1];
    }
    
    // Unique Paths II (with obstacles)
    static uniquePathsWithObstacles(obstacleGrid) {
        const m = obstacleGrid.length;
        const n = obstacleGrid[0].length;
        
        if (obstacleGrid[0][0] === 1) return 0;
        
        const dp = Array(m).fill(0).map(() => Array(n).fill(0));
        dp[0][0] = 1;
        
        for (let i = 0; i < m; i++) {
            for (let j = 0; j < n; j++) {
                if (obstacleGrid[i][j] === 1) {
                    dp[i][j] = 0;
                } else if (i === 0 && j === 0) {
                    continue;
                } else {
                    const fromTop = i > 0 ? dp[i - 1][j] : 0;
                    const fromLeft = j > 0 ? dp[i][j - 1] : 0;
                    dp[i][j] = fromTop + fromLeft;
                }
            }
        }
        
        return dp[m - 1][n - 1];
    }
    
    // Minimum Path Sum
    static minPathSum(grid) {
        const m = grid.length;
        const n = grid[0].length;
        const dp = Array(m).fill(0).map(() => Array(n).fill(0));
        
        dp[0][0] = grid[0][0];
        
        for (let i = 1; i < m; i++) {
            dp[i][0] = dp[i - 1][0] + grid[i][0];
        }
        
        for (let j = 1; j < n; j++) {
            dp[0][j] = dp[0][j - 1] + grid[0][j];
        }
        
        for (let i = 1; i < m; i++) {
            for (let j = 1; j < n; j++) {
                dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
            }
        }
        
        return dp[m - 1][n - 1];
    }
    
    // Maximum Product Subarray
    static maxProduct(nums) {
        let maxProd = nums[0];
        let minProd = nums[0];
        let result = nums[0];
        
        for (let i = 1; i < nums.length; i++) {
            const temp = maxProd;
            maxProd = Math.max(nums[i], nums[i] * maxProd, nums[i] * minProd);
            minProd = Math.min(nums[i], nums[i] * temp, nums[i] * minProd);
            result = Math.max(result, maxProd);
        }
        
        return result;
    }
    
    // Palindrome Partitioning II
    static minCut(s) {
        const n = s.length;
        const isPalindrome = Array(n).fill(false).map(() => Array(n).fill(false));
        const dp = new Array(n).fill(0);
        
        for (let i = 0; i < n; i++) {
            let minCuts = i;
            
            for (let j = 0; j <= i; j++) {
                if (s[i] === s[j] && (i - j <= 1 || isPalindrome[j + 1][i - 1])) {
                    isPalindrome[j][i] = true;
                    minCuts = j === 0 ? 0 : Math.min(minCuts, dp[j - 1] + 1);
                }
            }
            
            dp[i] = minCuts;
        }
        
        return dp[n - 1];
    }
    
    // Matrix Chain Multiplication
    static matrixChainOrder(dims) {
        const n = dims.length - 1;
        const dp = Array(n).fill(0).map(() => Array(n).fill(0));
        
        for (let len = 2; len <= n; len++) {
            for (let i = 0; i < n - len + 1; i++) {
                const j = i + len - 1;
                dp[i][j] = Infinity;
                
                for (let k = i; k < j; k++) {
                    const cost = dp[i][k] + dp[k + 1][j] + dims[i] * dims[k + 1] * dims[j + 1];
                    dp[i][j] = Math.min(dp[i][j], cost);
                }
            }
        }
        
        return dp[0][n - 1];
    }
    
    // Egg Drop Problem
    static eggDrop(eggs, floors) {
        const dp = Array(eggs + 1).fill(0).map(() => Array(floors + 1).fill(0));
        
        for (let i = 1; i <= eggs; i++) {
            dp[i][0] = 0;
            dp[i][1] = 1;
        }
        
        for (let j = 1; j <= floors; j++) {
            dp[1][j] = j;
        }
        
        for (let i = 2; i <= eggs; i++) {
            for (let j = 2; j <= floors; j++) {
                dp[i][j] = Infinity;
                
                for (let k = 1; k <= j; k++) {
                    const res = 1 + Math.max(dp[i - 1][k - 1], dp[i][j - k]);
                    dp[i][j] = Math.min(dp[i][j], res);
                }
            }
        }
        
        return dp[eggs][floors];
    }
}

// Testing
console.log(DynamicProgramming.fibMemo(10));  // 55
console.log(DynamicProgramming.longestCommonSubsequence("abcde", "ace"));  // 3
console.log(DynamicProgramming.coinChange([1, 2, 5], 11));  // 3
```

### 4.3 Greedy Algorithms

```javascript
class GreedyAlgorithms {
    // Activity Selection
    static activitySelection(start, finish) {
        const n = start.length;
        const activities = [];
        
        for (let i = 0; i < n; i++) {
            activities.push({ start: start[i], finish: finish[i], index: i });
        }
        
        activities.sort((a, b) => a.finish - b.finish);
        
        const selected = [activities[0]];
        let lastFinish = activities[0].finish;
        
        for (let i = 1; i < n; i++) {
            if (activities[i].start >= lastFinish) {
                selected.push(activities[i]);
                lastFinish = activities[i].finish;
            }
        }
        
        return selected;
    }
    
    // Fractional Knapsack
    static fractionalKnapsack(weights, values, capacity) {
        const items = [];
        
        for (let i = 0; i < weights.length; i++) {
            items.push({
                weight: weights[i],
                value: values[i],
                ratio: values[i] / weights[i]
            });
        }
        
        items.sort((a, b) => b.ratio - a.ratio);
        
        let totalValue = 0;
        let remainingCapacity = capacity;
        
        for (let item of items) {
            if (remainingCapacity >= item.weight) {
                totalValue += item.value;
                remainingCapacity -= item.weight;
            } else {
                totalValue += item.ratio * remainingCapacity;
                break;
            }
        }
        
        return totalValue;
    }
    
    // Huffman Coding
    static huffmanCoding(chars, freq) {
        class HuffmanNode {
            constructor(char, freq) {
                this.char = char;
                this.freq = freq;
                this.left = null;
                this.right = null;
            }
        }
        
        const pq = chars.map((char, i) => new HuffmanNode(char, freq[i]));
        pq.sort((a, b) => a.freq - b.freq);
        
        while (pq.length > 1) {
            const left = pq.shift();
            const right = pq.shift();
            
            const parent = new HuffmanNode(null, left.freq + right.freq);
            parent.left = left;
            parent.right = right;
            
            let inserted = false;
            for (let i = 0; i < pq.length; i++) {
                if (parent.freq < pq[i].freq) {
                    pq.splice(i, 0, parent);
                    inserted = true;
                    break;
                }
            }
            
            if (!inserted) pq.push(parent);
        }
        
        const codes = new Map();
        
        const generateCodes = (node, code = '') => {
            if (!node) return;
            
            if (node.char !== null) {
                codes.set(node.char, code);
                return;
            }
            
            generateCodes(node.left, code + '0');
            generateCodes(node.right, code + '1');
        };
        
        generateCodes(pq[0]);
        return codes;
    }
    
    // Job Sequencing
    static jobSequencing(jobs) {
        jobs.sort((a, b) => b.profit - a.profit);
        
        const maxDeadline = Math.max(...jobs.map(j => j.deadline));
        const slots = new Array(maxDeadline).fill(null);
        let totalProfit = 0;
        
        for (let job of jobs) {
            for (let i = Math.min(maxDeadline, job.deadline) - 1; i >= 0; i--) {
                if (slots[i] === null) {
                    slots[i] = job;
                    totalProfit += job.profit;
                    break;
                }
            }
        }
        
        return { slots: slots.filter(j => j !== null), totalProfit };
    }
    
    // Minimum Platforms
    static findPlatform(arrivals, departures) {
        arrivals.sort((a, b) => a - b);
        departures.sort((a, b) => a - b);
        
        let platformsNeeded = 0;
        let maxPlatforms = 0;
        let i = 0, j = 0;
        
        while (i < arrivals.length && j < departures.length) {
            if (arrivals[i] <= departures[j]) {
                platformsNeeded++;
                maxPlatforms = Math.max(maxPlatforms, platformsNeeded);
                i++;
            } else {
                platformsNeeded--;
                j++;
            }
        }
        
        return maxPlatforms;
    }
    
    // Jump Game
    static canJump(nums) {
        let maxReach = 0;
        
        for (let i = 0; i < nums.length; i++) {
            if (i > maxReach) return false;
            maxReach = Math.max(maxReach, i + nums[i]);
        }
        
        return true;
    }
    
    // Jump Game II (Minimum jumps)
    static minJumps(nums) {
        let jumps = 0;
        let currentEnd = 0;
        let farthest = 0;
        
        for (let i = 0; i < nums.length - 1; i++) {
            farthest = Math.max(farthest, i + nums[i]);
            
            if (i === currentEnd) {
                jumps++;
                currentEnd = farthest;
            }
        }
        
        return jumps;
    }
    
    // Gas Station
    static canCompleteCircuit(gas, cost) {
        let totalGas = 0;
        let totalCost = 0;
        let tank = 0;
        let start = 0;
        
        for (let i = 0; i < gas.length; i++) {
            totalGas += gas[i];
            totalCost += cost[i];
            tank += gas[i] - cost[i];
            
            if (tank < 0) {
                start = i + 1;
                tank = 0;
            }
        }
        
        return totalGas >= totalCost ? start : -1;
    }
}

// Testing
console.log(GreedyAlgorithms.activitySelection([1, 3, 0, 5, 8, 5], [2, 4, 6, 7, 9, 9]));
console.log(GreedyAlgorithms.fractionalKnapsack([10, 20, 30], [60, 100, 120], 50));  // 240
```

### 4.4 Backtracking

**Core Concept**: Build solution incrementally, abandoning paths that won't lead to valid solution.

**Backtracking Template**:
```
function backtrack(current, options) {
    if (isComplete(current)) {
        result.add(current);
        return;
    }
    for (option in options) {
        if (isValid(option)) {
            make_choice(option);        // Try
            backtrack(current, options); // Explore
            undo_choice(option);        // Backtrack
        }
    }
}
```

**Common Use Cases**:
- Generate all permutations/combinations
- N-Queens, Sudoku puzzles
- Word search, path finding
- Constraint satisfaction problems

**Key Insight**: Try a choice, if it doesn't work, undo it and try next choice.

```javascript
class Backtracking {
    // Generate Permutations - O(n! * n)
    // Generate all possible orderings of elements
    // Example: [1,2,3] → [[1,2,3], [1,3,2], [2,1,3], [2,3,1], [3,1,2], [3,2,1]]
    static permute(nums) {
        const result = [];
        
        const backtrack = (current, remaining) => {
            // Base case: no more elements to add
            if (remaining.length === 0) {
                result.push([...current]);  // Found a complete permutation
                return;
            }
            
            // Try each remaining element as next choice
            for (let i = 0; i < remaining.length; i++) {
                current.push(remaining[i]);  // Make choice
                // Explore with this choice (remove used element from remaining)
                backtrack(
                    current,
                    remaining.slice(0, i).concat(remaining.slice(i + 1))
                );
                current.pop();  // Undo choice (backtrack)
            }
        };
        
        backtrack([], nums);
        return result;
    }
    
    // Generate Combinations - O(n choose k)
    // Choose k elements from n elements
    // Example: n=4, k=2 → [[1,2], [1,3], [1,4], [2,3], [2,4], [3,4]]
    static combine(n, k) {
        const result = [];
        
        const backtrack = (start, current) => {
            // Base case: we've selected k elements
            if (current.length === k) {
                result.push([...current]);
                return;
            }
            
            // Try each number from start to n
            for (let i = start; i <= n; i++) {
                current.push(i);           // Make choice
                backtrack(i + 1, current); // Explore (i+1 ensures no duplicates)
                current.pop();             // Undo choice
            }
        };
        
        backtrack(1, []);
        return result;
    }
    
    // Subsets
    static subsets(nums) {
        const result = [];
        
        const backtrack = (start, current) => {
            result.push([...current]);
            
            for (let i = start; i < nums.length; i++) {
                current.push(nums[i]);
                backtrack(i + 1, current);
                current.pop();
            }
        };
        
        backtrack(0, []);
        return result;
    }
    
    // Subsets II (with duplicates)
    static subsetsWithDup(nums) {
        nums.sort((a, b) => a - b);
        const result = [];
        
        const backtrack = (start, current) => {
            result.push([...current]);
            
            for (let i = start; i < nums.length; i++) {
                if (i > start && nums[i] === nums[i - 1]) continue;
                current.push(nums[i]);
                backtrack(i + 1, current);
                current.pop();
            }
        };
        
        backtrack(0, []);
        return result;
    }
    
    // Combination Sum
    static combinationSum(candidates, target) {
        const result = [];
        
        const backtrack = (start, current, sum) => {
            if (sum === target) {
                result.push([...current]);
                return;
            }
            
            if (sum > target) return;
            
            for (let i = start; i < candidates.length; i++) {
                current.push(candidates[i]);
                backtrack(i, current, sum + candidates[i]);
                current.pop();
            }
        };
        
        backtrack(0, [], 0);
        return result;
    }
    
    // N-Queens - O(n!)
    // Place n queens on n×n board so no two queens attack each other
    // Queens attack horizontally, vertically, and diagonally
    static solveNQueens(n) {
        const result = [];
        const board = Array(n).fill(null).map(() => Array(n).fill('.'));
        
        // Check if placing queen at (row, col) is valid
        const isValid = (row, col) => {
            // Check column (no queen above in same column)
            for (let i = 0; i < row; i++) {
                if (board[i][col] === 'Q') return false;
            }
            
            // Check diagonal (top-left to bottom-right)
            for (let i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
                if (board[i][j] === 'Q') return false;
            }
            
            // Check diagonal (top-right to bottom-left)
            for (let i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
                if (board[i][j] === 'Q') return false;
            }
            
            return true;  // Safe to place queen
        };
        
        const backtrack = (row) => {
            // Base case: placed queens in all rows
            if (row === n) {
                result.push(board.map(r => r.join('')));
                return;
            }
            
            // Try placing queen in each column of current row
            for (let col = 0; col < n; col++) {
                if (isValid(row, col)) {
                    board[row][col] = 'Q';     // Place queen
                    backtrack(row + 1);        // Move to next row
                    board[row][col] = '.';     // Remove queen (backtrack)
                }
            }
        };
        
        backtrack(0);
        return result;
    }
    
    // Sudoku Solver
    static solveSudoku(board) {
        const isValid = (row, col, num) => {
            // Check row
            for (let j = 0; j < 9; j++) {
                if (board[row][j] === num) return false;
            }
            
            // Check column
            for (let i = 0; i < 9; i++) {
                if (board[i][col] === num) return false;
            }
            
            // Check 3x3 box
            const boxRow = Math.floor(row / 3) * 3;
            const boxCol = Math.floor(col / 3) * 3;
            
            for (let i = boxRow; i < boxRow + 3; i++) {
                for (let j = boxCol; j < boxCol + 3; j++) {
                    if (board[i][j] === num) return false;
                }
            }
            
            return true;
        };
        
        const solve = () => {
            for (let i = 0; i < 9; i++) {
                for (let j = 0; j < 9; j++) {
                    if (board[i][j] === '.') {
                        for (let num = 1; num <= 9; num++) {
                            const char = num.toString();
                            
                            if (isValid(i, j, char)) {
                                board[i][j] = char;
                                
                                if (solve()) return true;
                                
                                board[i][j] = '.';
                            }
                        }
                        
                        return false;
                    }
                }
            }
            
            return true;
        };
        
        solve();
        return board;
    }
    
    // Word Search
    static wordSearch(board, word) {
        const rows = board.length;
        const cols = board[0].length;
        
        const backtrack = (i, j, index) => {
            if (index === word.length) return true;
            
            if (i < 0 || i >= rows || j < 0 || j >= cols || board[i][j] !== word[index]) {
                return false;
            }
            
            const temp = board[i][j];
            board[i][j] = '#';
            
            const found = backtrack(i + 1, j, index + 1) ||
                         backtrack(i - 1, j, index + 1) ||
                         backtrack(i, j + 1, index + 1) ||
                         backtrack(i, j - 1, index + 1);
            
            board[i][j] = temp;
            return found;
        };
        
        for (let i = 0; i < rows; i++) {
            for (let j = 0; j < cols; j++) {
                if (backtrack(i, j, 0)) return true;
            }
        }
        
        return false;
    }
    
    // Palindrome Partitioning
    static partition(s) {
        const result = [];
        
        const isPalindrome = (str) => {
            let left = 0, right = str.length - 1;
            while (left < right) {
                if (str[left] !== str[right]) return false;
                left++;
                right--;
            }
            return true;
        };
        
        const backtrack = (start, current) => {
            if (start === s.length) {
                result.push([...current]);
                return;
            }
            
            for (let i = start; i < s.length; i++) {
                const substring = s.substring(start, i + 1);
                
                if (isPalindrome(substring)) {
                    current.push(substring);
                    backtrack(i + 1, current);
                    current.pop();
                }
            }
        };
        
        backtrack(0, []);
        return result;
    }
    
    // Generate Parentheses
    static generateParenthesis(n) {
        const result = [];
        
        const backtrack = (current, open, close) => {
            if (current.length === 2 * n) {
                result.push(current);
                return;
            }
            
            if (open < n) {
                backtrack(current + '(', open + 1, close);
            }
            
            if (close < open) {
                backtrack(current + ')', open, close + 1);
            }
        };
        
        backtrack('', 0, 0);
        return result;
    }
    
    // Letter Combinations of Phone Number
    static letterCombinations(digits) {
        if (digits.length === 0) return [];
        
        const mapping = {
            '2': 'abc', '3': 'def', '4': 'ghi', '5': 'jkl',
            '6': 'mno', '7': 'pqrs', '8': 'tuv', '9': 'wxyz'
        };
        
        const result = [];
        
        const backtrack = (index, current) => {
            if (index === digits.length) {
                result.push(current);
                return;
            }
            
            const letters = mapping[digits[index]];
            
            for (let letter of letters) {
                backtrack(index + 1, current + letter);
            }
        };
        
        backtrack(0, '');
        return result;
    }
}

// Testing
console.log(Backtracking.permute([1, 2, 3]));
console.log(Backtracking.combine(4, 2));
console.log(Backtracking.generateParenthesis(3));
```

---

## Part 5: Advanced Algorithms

### 5.1 Bit Manipulation

```javascript
class BitManipulation {
    // Check if number is power of 2
    static isPowerOfTwo(n) {
        return n > 0 && (n & (n - 1)) === 0;
    }
    
    // Count set bits (1s)
    static countSetBits(n) {
        let count = 0;
        while (n > 0) {
            count += n & 1;
            n >>= 1;
        }
        return count;
    }
    
    // Brian Kernighan's Algorithm (efficient bit counting)
    static countSetBitsOptimized(n) {
        let count = 0;
        while (n > 0) {
            n &= (n - 1);
            count++;
        }
        return count;
    }
    
    // Get ith bit
    static getBit(num, i) {
        return (num & (1 << i)) !== 0 ? 1 : 0;
    }
    
    // Set ith bit
    static setBit(num, i) {
        return num | (1 << i);
    }
    
    // Clear ith bit
    static clearBit(num, i) {
        return num & ~(1 << i);
    }
    
    // Toggle ith bit
    static toggleBit(num, i) {
        return num ^ (1 << i);
    }
    
    // Single Number (XOR trick)
    static singleNumber(nums) {
        let result = 0;
        for (let num of nums) {
            result ^= num;
        }
        return result;
    }
    
    // Single Number II (appears once, others thrice)
    static singleNumberII(nums) {
        let ones = 0, twos = 0;
        
        for (let num of nums) {
            ones = (ones ^ num) & ~twos;
            twos = (twos ^ num) & ~ones;
        }
        
        return ones;
    }
    
    // Missing Number
    static missingNumber(nums) {
        let xor = nums.length;
        
        for (let i = 0; i < nums.length; i++) {
            xor ^= i ^ nums[i];
        }
        
        return xor;
    }
    
    // Reverse Bits
    static reverseBits(n) {
        let result = 0;
        
        for (let i = 0; i < 32; i++) {
            result = (result << 1) | (n & 1);
            n >>= 1;
        }
        
        return result >>> 0;
    }
    
    // Number of 1 Bits (Hamming Weight)
    static hammingWeight(n) {
        let count = 0;
        while (n !== 0) {
            count++;
            n &= (n - 1);
        }
        return count;
    }
    
    // Hamming Distance
    static hammingDistance(x, y) {
        let xor = x ^ y;
        let count = 0;
        
        while (xor !== 0) {
            count++;
            xor &= (xor - 1);
        }
        
        return count;
    }
    
    // Sum of Two Integers (without + operator)
    static getSum(a, b) {
        while (b !== 0) {
            const carry = a & b;
            a = a ^ b;
            b = carry << 1;
        }
        return a;
    }
    
    // Power Set using Bit Manipulation
    static powerSet(nums) {
        const n = nums.length;
        const totalSubsets = 1 << n;
        const result = [];
        
        for (let i = 0; i < totalSubsets; i++) {
            const subset = [];
            for (let j = 0; j < n; j++) {
                if (i & (1 << j)) {
                    subset.push(nums[j]);
                }
            }
            result.push(subset);
        }
        
        return result;
    }
}

// Testing
console.log(BitManipulation.isPowerOfTwo(16));  // true
console.log(BitManipulation.countSetBits(7));  // 3
console.log(BitManipulation.singleNumber([4,1,2,1,2]));  // 4
```

### 5.2 Advanced String Algorithms

```javascript
class AdvancedStringAlgorithms {
    // Z Algorithm (Pattern Matching)
    static zAlgorithm(text, pattern) {
        const concat = pattern + "$" + text;
        const n = concat.length;
        const z = new Array(n).fill(0);
        const matches = [];
        
        let left = 0, right = 0;
        
        for (let i = 1; i < n; i++) {
            if (i > right) {
                left = right = i;
                
                while (right < n && concat[right] === concat[right - left]) {
                    right++;
                }
                
                z[i] = right - left;
                right--;
            } else {
                const k = i - left;
                
                if (z[k] < right - i + 1) {
                    z[i] = z[k];
                } else {
                    left = i;
                    
                    while (right < n && concat[right] === concat[right - left]) {
                        right++;
                    }
                    
                    z[i] = right - left;
                    right--;
                }
            }
            
            if (z[i] === pattern.length) {
                matches.push(i - pattern.length - 1);
            }
        }
        
        return matches;
    }
    
    // Manacher's Algorithm (Longest Palindromic Substring)
    static manacher(s) {
        // Preprocess string
        let t = '#';
        for (let char of s) {
            t += char + '#';
        }
        
        const n = t.length;
        const p = new Array(n).fill(0);
        let center = 0, right = 0;
        let maxLen = 0, maxCenter = 0;
        
        for (let i = 0; i < n; i++) {
            const mirror = 2 * center - i;
            
            if (i < right) {
                p[i] = Math.min(right - i, p[mirror]);
            }
            
            // Try to expand palindrome
            let a = i + (1 + p[i]);
            let b = i - (1 + p[i]);
            
            while (a < n && b >= 0 && t[a] === t[b]) {
                p[i]++;
                a++;
                b--;
            }
            
            // Update center and right
            if (i + p[i] > right) {
                center = i;
                right = i + p[i];
            }
            
            // Track maximum
            if (p[i] > maxLen) {
                maxLen = p[i];
                maxCenter = i;
            }
        }
        
        const start = (maxCenter - maxLen) / 2;
        return s.substring(start, start + maxLen);
    }
    
    // Suffix Array Construction
    static buildSuffixArray(s) {
        const n = s.length;
        const suffixes = [];
        
        for (let i = 0; i < n; i++) {
            suffixes.push({ index: i, suffix: s.substring(i) });
        }
        
        suffixes.sort((a, b) => a.suffix.localeCompare(b.suffix));
        
        return suffixes.map(s => s.index);
    }
    
    // LCP Array (Longest Common Prefix)
    static buildLCPArray(s, suffixArray) {
        const n = s.length;
        const lcp = new Array(n).fill(0);
        const rank = new Array(n);
        
        for (let i = 0; i < n; i++) {
            rank[suffixArray[i]] = i;
        }
        
        let h = 0;
        
        for (let i = 0; i < n; i++) {
            if (rank[i] > 0) {
                const j = suffixArray[rank[i] - 1];
                
                while (i + h < n && j + h < n && s[i + h] === s[j + h]) {
                    h++;
                }
                
                lcp[rank[i]] = h;
                
                if (h > 0) h--;
            }
        }
        
        return lcp;
    }
    
    // Aho-Corasick Algorithm (Multiple Pattern Matching)
    static ahoCorasick(text, patterns) {
        class ACNode {
            constructor() {
                this.children = new Map();
                this.fail = null;
                this.output = [];
            }
        }
        
        const root = new ACNode();
        
        // Build trie
        for (let pattern of patterns) {
            let node = root;
            for (let char of pattern) {
                if (!node.children.has(char)) {
                    node.children.set(char, new ACNode());
                }
                node = node.children.get(char);
            }
            node.output.push(pattern);
        }
        
        // Build failure links (BFS)
        const queue = [];
        
        for (let child of root.children.values()) {
            child.fail = root;
            queue.push(child);
        }
        
        while (queue.length > 0) {
            const current = queue.shift();
            
            for (let [char, child] of current.children) {
                queue.push(child);
                
                let fail = current.fail;
                while (fail !== null && !fail.children.has(char)) {
                    fail = fail.fail;
                }
                
                child.fail = fail ? fail.children.get(char) : root;
                child.output.push(...child.fail.output);
            }
        }
        
        // Search
        const matches = [];
        let node = root;
        
        for (let i = 0; i < text.length; i++) {
            const char = text[i];
            
            while (node !== null && !node.children.has(char)) {
                node = node.fail;
            }
            
            node = node ? node.children.get(char) : root;
            
            for (let pattern of node.output) {
                matches.push({ pattern, index: i - pattern.length + 1 });
            }
        }
        
        return matches;
    }
}

// Testing
console.log(AdvancedStringAlgorithms.zAlgorithm("ABABDABACDABABCABAB", "ABABCABAB"));
console.log(AdvancedStringAlgorithms.manacher("babad"));
```

### 5.3 Computational Geometry

```javascript
class ComputationalGeometry {
    // Point class
    static Point = class {
        constructor(x, y) {
            this.x = x;
            this.y = y;
        }
    };
    
    // Distance between two points
    static distance(p1, p2) {
        return Math.sqrt(Math.pow(p2.x - p1.x, 2) + Math.pow(p2.y - p1.y, 2));
    }
    
    // Cross product of vectors
    static crossProduct(p1, p2, p3) {
        return (p2.x - p1.x) * (p3.y - p1.y) - (p2.y - p1.y) * (p3.x - p1.x);
    }
    
    // Orientation of ordered triplet (p, q, r)
    // 0: Collinear, 1: Clockwise, 2: Counterclockwise
    static orientation(p, q, r) {
        const val = this.crossProduct(p, q, r);
        if (val === 0) return 0;
        return val > 0 ? 1 : 2;
    }
    
    // Check if point q lies on segment pr
    static onSegment(p, q, r) {
        return q.x <= Math.max(p.x, r.x) && q.x >= Math.min(p.x, r.x) &&
               q.y <= Math.max(p.y, r.y) && q.y >= Math.min(p.y, r.y);
    }
    
    // Check if two segments intersect
    static doSegmentsIntersect(p1, q1, p2, q2) {
        const o1 = this.orientation(p1, q1, p2);
        const o2 = this.orientation(p1, q1, q2);
        const o3 = this.orientation(p2, q2, p1);
        const o4 = this.orientation(p2, q2, q1);
        
        if (o1 !== o2 && o3 !== o4) return true;
        
        if (o1 === 0 && this.onSegment(p1, p2, q1)) return true;
        if (o2 === 0 && this.onSegment(p1, q2, q1)) return true;
        if (o3 === 0 && this.onSegment(p2, p1, q2)) return true;
        if (o4 === 0 && this.onSegment(p2, q1, q2)) return true;
        
        return false;
    }
    
    // Convex Hull - Graham Scan
    static convexHull(points) {
        if (points.length < 3) return points;
        
        // Find the bottom-most point
        let bottom = 0;
        for (let i = 1; i < points.length; i++) {
            if (points[i].y < points[bottom].y ||
                (points[i].y === points[bottom].y && points[i].x < points[bottom].x)) {
                bottom = i;
            }
        }
        
        [points[0], points[bottom]] = [points[bottom], points[0]];
        const p0 = points[0];
        
        // Sort by polar angle
        points.slice(1).sort((a, b) => {
            const orient = this.orientation(p0, a, b);
            if (orient === 0) {
                return this.distance(p0, a) - this.distance(p0, b);
            }
            return orient === 2 ? -1 : 1;
        });
        
        const stack = [points[0], points[1], points[2]];
        
        for (let i = 3; i < points.length; i++) {
            while (stack.length > 1) {
                const top = stack[stack.length - 1];
                const nextToTop = stack[stack.length - 2];
                
                if (this.orientation(nextToTop, top, points[i]) !== 2) {
                    stack.pop();
                } else {
                    break;
                }
            }
            stack.push(points[i]);
        }
        
        return stack;
    }
    
    // Point in Polygon
    static isPointInPolygon(point, polygon) {
        let intersections = 0;
        const n = polygon.length;
        
        for (let i = 0; i < n; i++) {
            const p1 = polygon[i];
            const p2 = polygon[(i + 1) % n];
            
            if (p1.y === p2.y) continue;
            
            if (point.y < Math.min(p1.y, p2.y)) continue;
            if (point.y >= Math.max(p1.y, p2.y)) continue;
            
            const x = (point.y - p1.y) * (p2.x - p1.x) / (p2.y - p1.y) + p1.x;
            
            if (x > point.x) intersections++;
        }
        
        return intersections % 2 === 1;
    }
}

// Testing
const Point = ComputationalGeometry.Point;
const p1 = new Point(0, 0);
const p2 = new Point(4, 4);
console.log(ComputationalGeometry.distance(p1, p2));
```

---

## Part 6: Real-World Applications

### 6.1 System Design Data Structures

```javascript
// Rate Limiter (Sliding Window)
class RateLimiter {
    constructor(maxRequests, windowMs) {
        this.maxRequests = maxRequests;
        this.windowMs = windowMs;
        this.requests = new Map();
    }
    
    allowRequest(userId) {
        const now = Date.now();
        
        if (!this.requests.has(userId)) {
            this.requests.set(userId, []);
        }
        
        const userRequests = this.requests.get(userId);
        
        // Remove old requests outside window
        while (userRequests.length > 0 && userRequests[0] <= now - this.windowMs) {
            userRequests.shift();
        }
        
        if (userRequests.length < this.maxRequests) {
            userRequests.push(now);
            return true;
        }
        
        return false;
    }
}

// URL Shortener
class URLShortener {
    constructor() {
        this.urlToCode = new Map();
        this.codeToUrl = new Map();
        this.counter = 0;
        this.baseUrl = "https://short.url/";
    }
    
    encode(longUrl) {
        if (this.urlToCode.has(longUrl)) {
            return this.baseUrl + this.urlToCode.get(longUrl);
        }
        
        const code = this.encodeBase62(this.counter++);
        this.urlToCode.set(longUrl, code);
        this.codeToUrl.set(code, longUrl);
        
        return this.baseUrl + code;
    }
    
    decode(shortUrl) {
        const code = shortUrl.replace(this.baseUrl, "");
        return this.codeToUrl.get(code) || null;
    }
    
    encodeBase62(num) {
        const chars = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
        let code = "";
        
        while (num > 0) {
            code = chars[num % 62] + code;
            num = Math.floor(num / 62);
        }
        
        return code || "0";
    }
}

// Auto-complete System
class AutocompleteSystem {
    constructor() {
        this.trie = new TrieNode();
        this.currentInput = "";
    }
    
    addSentence(sentence, times) {
        let node = this.trie;
        
        for (let char of sentence) {
            if (!node.children.has(char)) {
                node.children.set(char, new TrieNode());
            }
            node = node.children.get(char);
        }
        
        node.isEnd = true;
        node.sentence = sentence;
        node.times += times;
    }
    
    input(c) {
        if (c === '#') {
            this.addSentence(this.currentInput, 1);
            this.currentInput = "";
            return [];
        }
        
        this.currentInput += c;
        let node = this.trie;
        
        for (let char of this.currentInput) {
            if (!node.children.has(char)) {
                return [];
            }
            node = node.children.get(char);
        }
        
        return this.getSuggestions(node);
    }
    
    getSuggestions(node) {
        const suggestions = [];
        
        const dfs = (node) => {
            if (node.isEnd) {
                suggestions.push({ sentence: node.sentence, times: node.times });
            }
            
            for (let child of node.children.values()) {
                dfs(child);
            }
        };
        
        dfs(node);
        
        return suggestions
            .sort((a, b) => {
                if (b.times !== a.times) return b.times - a.times;
                return a.sentence.localeCompare(b.sentence);
            })
            .slice(0, 3)
            .map(s => s.sentence);
    }
}

class TrieNode {
    constructor() {
        this.children = new Map();
        this.isEnd = false;
        this.sentence = "";
        this.times = 0;
    }
}

// Twitter Feed (Design)
class Twitter {
    constructor() {
        this.tweets = new Map();
        this.following = new Map();
        this.timestamp = 0;
    }
    
    postTweet(userId, tweetId) {
        if (!this.tweets.has(userId)) {
            this.tweets.set(userId, []);
        }
        this.tweets.get(userId).push({ tweetId, time: this.timestamp++ });
    }
    
    getNewsFeed(userId) {
        const heap = new MaxHeap();
        
        // Add user's own tweets
        if (this.tweets.has(userId)) {
            for (let tweet of this.tweets.get(userId)) {
                heap.insert({ ...tweet, priority: tweet.time });
            }
        }
        
        // Add followed users' tweets
        if (this.following.has(userId)) {
            for (let followeeId of this.following.get(userId)) {
                if (this.tweets.has(followeeId)) {
                    for (let tweet of this.tweets.get(followeeId)) {
                        heap.insert({ ...tweet, priority: tweet.time });
                    }
                }
            }
        }
        
        const feed = [];
        for (let i = 0; i < 10 && heap.heap.length > 0; i++) {
            feed.push(heap.extractMax().tweetId);
        }
        
        return feed;
    }
    
    follow(followerId, followeeId) {
        if (followerId === followeeId) return;
        
        if (!this.following.has(followerId)) {
            this.following.set(followerId, new Set());
        }
        this.following.get(followerId).add(followeeId);
    }
    
    unfollow(followerId, followeeId) {
        if (this.following.has(followerId)) {
            this.following.get(followerId).delete(followeeId);
        }
    }
}

// Consistent Hashing
class ConsistentHashing {
    constructor(numVirtualNodes = 150) {
        this.ring = new Map();
        this.sortedKeys = [];
        this.numVirtualNodes = numVirtualNodes;
    }
    
    hash(key) {
        let hash = 0;
        for (let i = 0; i < key.length; i++) {
            hash = ((hash << 5) - hash) + key.charCodeAt(i);
            hash |= 0;
        }
        return hash >>> 0;
    }
    
    addNode(node) {
        for (let i = 0; i < this.numVirtualNodes; i++) {
            const virtualKey = `${node}:${i}`;
            const hash = this.hash(virtualKey);
            this.ring.set(hash, node);
        }
        
        this.sortedKeys = Array.from(this.ring.keys()).sort((a, b) => a - b);
    }
    
    removeNode(node) {
        for (let i = 0; i < this.numVirtualNodes; i++) {
            const virtualKey = `${node}:${i}`;
            const hash = this.hash(virtualKey);
            this.ring.delete(hash);
        }
        
        this.sortedKeys = Array.from(this.ring.keys()).sort((a, b) => a - b);
    }
    
    getNode(key) {
        if (this.ring.size === 0) return null;
        
        const hash = this.hash(key);
        
        // Binary search for the first node >= hash
        let left = 0, right = this.sortedKeys.length - 1;
        
        while (left < right) {
            const mid = Math.floor((left + right) / 2);
            if (this.sortedKeys[mid] < hash) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        
        return this.ring.get(this.sortedKeys[left]);
    }
}

// Bloom Filter
class BloomFilter {
    constructor(size = 1000, numHashFunctions = 3) {
        this.size = size;
        this.numHashFunctions = numHashFunctions;
        this.bitArray = new Array(size).fill(0);
    }
    
    hash(item, seed) {
        let hash = 0;
        const str = item.toString();
        
        for (let i = 0; i < str.length; i++) {
            hash = ((hash << 5) - hash + str.charCodeAt(i) + seed) | 0;
        }
        
        return Math.abs(hash) % this.size;
    }
    
    add(item) {
        for (let i = 0; i < this.numHashFunctions; i++) {
            const index = this.hash(item, i);
            this.bitArray[index] = 1;
        }
    }
    
    contains(item) {
        for (let i = 0; i < this.numHashFunctions; i++) {
            const index = this.hash(item, i);
            if (this.bitArray[index] === 0) {
                return false;
            }
        }
        return true;  // Might be false positive
    }
}

// Testing
const rateLimiter = new RateLimiter(3, 1000);
console.log(rateLimiter.allowRequest('user1'));  // true

const urlShortener = new URLShortener();
const short = urlShortener.encode('https://example.com/very/long/url');
console.log(urlShortener.decode(short));
```

### 6.2 Problem-Solving Patterns

**These patterns appear repeatedly in coding interviews and real problems:**

**Pattern Recognition Guide**:
1. **Sliding Window**: Fixed/variable size subarray/substring
2. **Two Pointers**: Sorted array, finding pairs
3. **Fast & Slow**: Cycle detection, finding middle
4. **Prefix Sum**: Range queries, subarray sums
5. **Monotonic Stack**: Next greater/smaller element
6. **Cyclic Sort**: Array with numbers 1 to n
7. **Top K Elements**: Finding k largest/smallest
8. **Binary Search**: Sorted/monotonic search space
9. **Merge Intervals**: Overlapping intervals

```javascript
class ProblemSolvingPatterns {
    // Sliding Window Pattern - O(n)
    // Find maximum sum of k consecutive elements
    // Key: Slide window by removing left element and adding right element
    static maxSumSubarray(arr, k) {
        let maxSum = 0;
        let windowSum = 0;
        
        // Calculate sum of first window
        for (let i = 0; i < k; i++) {
            windowSum += arr[i];
        }
        
        maxSum = windowSum;
        
        // Slide window: remove left, add right
        for (let i = k; i < arr.length; i++) {
            windowSum = windowSum - arr[i - k] + arr[i];  // Slide!
            maxSum = Math.max(maxSum, windowSum);
        }
        
        return maxSum;
    }
    
    // Two Pointers Pattern - O(n)
    // Find pair in sorted array that sums to target
    // Key: Move pointers based on comparison with target
    static isPairSum(arr, target) {
        let left = 0;              // Start pointer
        let right = arr.length - 1; // End pointer
        
        while (left < right) {
            const sum = arr[left] + arr[right];
            
            if (sum === target) return true;  // Found!
            if (sum < target) left++;   // Need larger sum, move left right
            else right--;               // Need smaller sum, move right left
        }
        
        return false;  // No pair found
    }
    
    // Fast & Slow Pointers (Floyd's Cycle Detection) - O(n)
    // Find duplicate number in array where nums[i] is 1 to n
    // Treat array as linked list: nums[i] points to index nums[i]
    static findDuplicate(nums) {
        let slow = nums[0];  // Tortoise (moves 1 step)
        let fast = nums[0];  // Hare (moves 2 steps)
        
        // Phase 1: Detect cycle (if duplicate exists, there's a cycle)
        do {
            slow = nums[slow];           // Move 1 step
            fast = nums[nums[fast]];     // Move 2 steps
        } while (slow !== fast);  // Until they meet
        
        // Phase 2: Find cycle entrance (the duplicate number)
        slow = nums[0];  // Reset slow to start
        while (slow !== fast) {
            slow = nums[slow];  // Both move 1 step
            fast = nums[fast];
        }
        
        return slow;  // Meeting point is the duplicate
    }
    
    // Prefix Sum Pattern - O(n)
    // Count subarrays with sum equal to k
    // Key: Use cumulative sum and hash map to track sum frequencies
    static subarraySum(nums, k) {
        const prefixSum = new Map([[0, 1]]);  // sum -> frequency
        let sum = 0;   // Running cumulative sum
        let count = 0; // Number of valid subarrays
        
        for (let num of nums) {
            sum += num;  // Update cumulative sum
            
            // If (sum - k) exists, we found subarrays ending here with sum k
            // Because: currentSum - previousSum = k
            if (prefixSum.has(sum - k)) {
                count += prefixSum.get(sum - k);
            }
            
            // Store current sum frequency
            prefixSum.set(sum, (prefixSum.get(sum) || 0) + 1);
        }
        
        return count;
    }
    
    // Monotonic Stack Pattern - O(n)
    // Find next greater element for each array element
    // Stack maintains decreasing order, pop when larger found
    static nextGreaterElement(nums) {
        const result = new Array(nums.length).fill(-1);
        const stack = [];  // Stores indices of elements waiting for next greater
        
        for (let i = 0; i < nums.length; i++) {
            // While current element is greater than stack top
            while (stack.length > 0 && nums[i] > nums[stack[stack.length - 1]]) {
                const index = stack.pop();  // Found next greater for this index
                result[index] = nums[i];
            }
            stack.push(i);  // Push current index
        }
        
        return result;  // Remaining indices have no next greater (-1)
    }
    
    // Cyclic Sort Pattern
    static findMissingNumbers(nums) {
        let i = 0;
        
        while (i < nums.length) {
            const correctIndex = nums[i] - 1;
            
            if (nums[i] !== nums[correctIndex]) {
                [nums[i], nums[correctIndex]] = [nums[correctIndex], nums[i]];
            } else {
                i++;
            }
        }
        
        const missing = [];
        for (let i = 0; i < nums.length; i++) {
            if (nums[i] !== i + 1) {
                missing.push(i + 1);
            }
        }
        
        return missing;
    }
    
    // Top K Elements Pattern
    static topKFrequent(nums, k) {
        const freq = new Map();
        
        for (let num of nums) {
            freq.set(num, (freq.get(num) || 0) + 1);
        }
        
        return Array.from(freq.entries())
            .sort((a, b) => b[1] - a[1])
            .slice(0, k)
            .map(pair => pair[0]);
    }
    
    // Modified Binary Search Pattern
    static searchRotatedArray(nums, target) {
        let left = 0;
        let right = nums.length - 1;
        
        while (left <= right) {
            const mid = Math.floor((left + right) / 2);
            
            if (nums[mid] === target) return mid;
            
            if (nums[left] <= nums[mid]) {
                if (target >= nums[left] && target < nums[mid]) {
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }
            } else {
                if (target > nums[mid] && target <= nums[right]) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }
        
        return -1;
    }
    
    // Merge Intervals Pattern - O(n log n)
    // Merge all overlapping intervals
    // Key: Sort by start time, then merge if overlap
    static mergeIntervals(intervals) {
        if (intervals.length === 0) return [];
        
        // Sort by start time
        intervals.sort((a, b) => a[0] - b[0]);
        const merged = [intervals[0]];  // Start with first interval
        
        for (let i = 1; i < intervals.length; i++) {
            const current = intervals[i];
            const last = merged[merged.length - 1];
            
            // Check for overlap: current starts before last ends
            if (current[0] <= last[1]) {
                // Merge: extend end time to maximum
                last[1] = Math.max(last[1], current[1]);
            } else {
                // No overlap: add as new interval
                merged.push(current);
            }
        }
        
        return merged;
    }
}

// Testing
console.log(ProblemSolvingPatterns.maxSumSubarray([2, 1, 5, 1, 3, 2], 3));  // 9
console.log(ProblemSolvingPatterns.mergeIntervals([[1,3],[2,6],[8,10],[15,18]]));
```

---

## Summary and Best Practices

### Time Complexity Cheat Sheet

**Understanding Big O Notation:**

Big O describes how algorithm performance scales with input size. Focus on worst-case and dominant terms.

```javascript
/*
COMMON TIME COMPLEXITIES (from best to worst):

O(1)         - Constant         - Hash table lookup, array access
             Example: arr[5], hash.get(key)
             Performance: Always same time regardless of input
             
O(log n)     - Logarithmic      - Binary search, balanced tree operations
             Example: Binary search in sorted array
             Performance: Doubles input but adds only 1 operation
             1000 elements ≈ 10 operations, 1M elements ≈ 20 operations
             
O(n)         - Linear           - Array traversal, linear search
             Example: for loop through array
             Performance: Double input = double time
             
O(n log n)   - Linearithmic     - Merge sort, quick sort, heap sort
             Example: Efficient sorting algorithms
             Performance: Better than O(n²) but worse than O(n)
             Best we can do for comparison-based sorting
             
O(n²)        - Quadratic        - Nested loops, bubble sort
             Example: Check all pairs in array
             Performance: Double input = 4x time
             Avoid for large datasets!
             
O(n³)        - Cubic            - Triple nested loops
             Example: Check all triplets
             Performance: Very slow for large inputs
             
O(2^n)       - Exponential      - Recursive fibonacci (naive), subsets
             Example: Generate all subsets
             Performance: Add 1 to input = double time
             Only feasible for very small inputs (n < 25)
             
O(n!)        - Factorial        - Permutations, traveling salesman (brute force)
             Example: Try all possible orderings
             Performance: Extremely slow, only works for n < 12
             
HOW TO CALCULATE:
- Drop constants: O(2n) → O(n), O(5) → O(1)
- Drop non-dominant terms: O(n² + n) → O(n²)
- Different inputs use different variables: O(a + b), O(a * b)
- Amortized analysis: Average case over sequence of operations

SPACE COMPLEXITY CONSIDERATIONS:
- In-place algorithms: O(1) extra space (modify input directly)
- Recursive algorithms: O(h) where h is recursion depth (call stack)
- Hash tables/sets: O(n) for n elements stored
- Dynamic programming: Usually O(n) or O(n²) for memoization table
- Space-time tradeoff: Often use more space to achieve faster time
*/
```

### Problem-Solving Framework

**Master Strategy for Any Coding Problem:**

```javascript
/*
STEP 1: UNDERSTAND THE PROBLEM (5 minutes)
- Read carefully and identify inputs/outputs
  * What is the input format? (array, string, matrix, graph?)
  * What is the expected output? (number, boolean, array?)
  * What is the input size range? (affects algorithm choice)
  
- Clarify constraints and edge cases
  * Are there duplicates?
  * Can input be empty?
  * Are there negative numbers?
  * Is array sorted?
  * Memory constraints?
  
- Ask questions about ambiguous requirements
  * "Should I modify input array or create new one?"
  * "What should I return if no solution exists?"
  * "Are there time/space constraints?"

STEP 2: ANALYZE EXAMPLES (5 minutes)
- Work through examples manually
  * Start with provided example
  * Create your own simple example
  * Walk through step-by-step what you'd do
  
- Identify patterns
  * Do you see a repeating process?
  * Can problem be broken into smaller problems?
  * Is there a sorted property to exploit?
  
- Consider edge cases (IMPORTANT!)
  * Empty input: [], "", null, undefined
  * Single element: [1], "a"
  * Two elements: [1, 2]
  * Duplicates: [1, 1, 1]
  * Already sorted: [1, 2, 3]
  * Reverse sorted: [3, 2, 1]
  * All same: [5, 5, 5]
  * Large numbers: [1000000]
  * Negative numbers: [-5, -1, 3]

STEP 3: CHOOSE DATA STRUCTURE (3 minutes)
Match problem characteristics to data structures:

- Array: Fixed size, indexed access, contiguous memory
  Use when: Need random access, fixed collection size
  
- Linked List: Dynamic size, sequential access
  Use when: Frequent insertion/deletion at ends, don't need random access
  
- Stack (LIFO): Last in, first out
  Use when: Need to reverse, track nested structures, backtracking
  Keywords: "valid parentheses", "undo", "recursive", "reverse"
  
- Queue (FIFO): First in, first out
  Use when: Process in order, BFS, scheduling
  Keywords: "level order", "first", "recent", "stream"
  
- Hash Table: O(1) lookup, key-value pairs
  Use when: Need fast lookup, counting, finding pairs
  Keywords: "find pair", "count frequency", "anagram", "duplicate"
  
- Tree: Hierarchical data, BST for sorted data
  Use when: Data has hierarchy, need sorted property
  Keywords: "parent-child", "ancestor", "path", "sorted"
  
- Heap: Priority queue operations
  Use when: Need min/max quickly, k-th largest/smallest
  Keywords: "k-th largest", "median", "priority", "top k"
  
- Graph: Relationships, networks, connected components
  Use when: Data has relationships, need path finding
  Keywords: "connected", "path", "distance", "neighbors", "network"

STEP 4: CHOOSE ALGORITHM PATTERN (5 minutes)
Recognize problem patterns:

- Brute Force: Try all possibilities
  Use when: Problem size is small (n < 20), no better approach obvious
  Complexity: Often O(2^n) or O(n!)
  
- Two Pointers: Linear scan with two positions
  Use when: Array is sorted, need to find pair/triplet
  Keywords: "sorted array", "pair", "palindrome"
  Complexity: O(n)
  
- Sliding Window: Fixed or variable size subarray
  Use when: Contiguous subarray/substring problem
  Keywords: "subarray", "substring", "consecutive", "window"
  Complexity: O(n)
  
- Binary Search: Divide search space in half
  Use when: Search space is sorted or monotonic
  Keywords: "sorted", "find in log time", "search"
  Complexity: O(log n)
  
- Divide & Conquer: Break into subproblems and combine
  Use when: Problem can be split into independent subproblems
  Examples: Merge sort, quick sort, binary search
  Complexity: Often O(n log n)
  
- Dynamic Programming: Overlapping subproblems + optimal substructure
  Use when: Problem has optimal substructure, recompute same subproblems
  Keywords: "maximum", "minimum", "longest", "count ways", "optimal"
  Identify: Can you solve smaller version and build up?
  Complexity: Often O(n) or O(n²)
  
- Greedy: Make locally optimal choice at each step
  Use when: Local optimum leads to global optimum
  Keywords: "maximize", "minimize", with specific constraints
  Warning: Prove greedy choice works! Not always correct.
  Complexity: Often O(n log n) due to sorting
  
- Backtracking: Try and revert, DFS-based
  Use when: Need all solutions, constraint satisfaction
  Keywords: "all permutations", "all combinations", "generate all"
  Complexity: Often O(2^n) or O(n!)
  
- BFS: Level-order exploration (queue)
  Use when: Shortest path in unweighted graph, level-wise
  Keywords: "shortest path", "level", "minimum steps"
  Complexity: O(V + E) for graphs
  
- DFS: Go deep first (recursion/stack)
  Use when: Path finding, cycle detection, topological sort
  Keywords: "path exists", "cycle", "connected components"
  Complexity: O(V + E) for graphs

STEP 5: OPTIMIZE (2 minutes)
- Can you reduce time complexity?
  * Use hash map for O(1) lookup instead of linear search
  * Use binary search if data is sorted
  * Use prefix sum for range queries
  * Avoid nested loops if possible
  
- Can you reduce space complexity?
  * Reuse input array if allowed
  * Use two pointers instead of extra array
  * Iterative instead of recursive (save stack space)
  * Rolling array for DP (keep only needed rows)
  
- Are there redundant operations?
  * Compute once and reuse (memoization)
  * Break early when answer found
  * Skip unnecessary iterations
  
- Can you use better data structure?
  * Set for O(1) lookup instead of array O(n)
  * Heap for O(log n) insert/extract instead of array O(n)
  * Prefix sum array for O(1) range sum

STEP 6: CODE (15 minutes)
- Start with brute force if stuck
- Write clean, readable code
- Use meaningful variable names
- Add comments for complex logic
- Handle edge cases

STEP 7: TEST (5 minutes)
- Test with example inputs (verify expected output)
- Test edge cases:
  * Empty input
  * Single element  
  * Two elements
  * Duplicates
  * Large numbers
- Test performance with large inputs (mental check)
- Trace through code line by line for complex cases
*/
```

### Interview Tips

**Master Guide for Technical Interviews:**

```javascript
/*
COMMON PATTERNS TO RECOGNIZE:

1. If input is SORTED:
   Pattern: Binary Search, Two Pointers, Merge technique
   Example problems:
   - "Find element in sorted array" → Binary Search O(log n)
   - "Find pair with sum X" → Two Pointers O(n)
   - "Merge two sorted arrays" → Two Pointers O(n+m)
   Why: Sorted property gives us direction to search/eliminate

2. If asked for TOP/MAXIMUM/MINIMUM K elements:
   Pattern: Heap (Priority Queue), QuickSelect
   Example problems:
   - "K-th largest element" → Min Heap of size k O(n log k)
   - "Top K frequent elements" → Heap or Bucket Sort
   - "Median of stream" → Two Heaps (max + min)
   Why: Heap maintains k elements efficiently

3. If asked for COMMON elements:
   Pattern: Hash Table, Two Pointers (if sorted)
   Example problems:
   - "Intersection of two arrays" → Hash Set O(n+m)
   - "Find duplicates" → Hash Set O(n)
   - "Anagram check" → Hash Map for frequencies
   Why: Hash table gives O(1) lookup

4. If dealing with PERMUTATIONS/COMBINATIONS:
   Pattern: Backtracking, DFS
   Example problems:
   - "Generate all permutations" → Backtracking O(n!)
   - "All subsets" → Backtracking O(2^n)
   - "Combination sum" → Backtracking with pruning
   Why: Need to explore all possibilities

5. If TREE/GRAPH TRAVERSAL:
   Pattern: DFS (recursion or stack), BFS (queue)
   Example problems:
   - "Validate BST" → DFS with range O(n)
   - "Level order traversal" → BFS O(n)
   - "Shortest path" → BFS (unweighted) or Dijkstra (weighted)
   DFS: Stack/Recursion, go deep, uses O(h) space
   BFS: Queue, level by level, shortest path

6. If OPTIMIZATION problem (max/min):
   Pattern: Dynamic Programming, Greedy Algorithm
   Example problems:
   - "Longest increasing subsequence" → DP O(n²) or O(n log n)
   - "Coin change" → DP O(amount * coins)
   - "Activity selection" → Greedy O(n log n)
   DP: When subproblems overlap, need optimal substructure
   Greedy: When local optimum → global optimum (prove it!)

7. If STRING matching/searching:
   Pattern: KMP, Rabin-Karp, Trie
   Example problems:
   - "Find pattern in text" → KMP O(n+m)
   - "Multiple pattern matching" → Trie or Aho-Corasick
   - "Longest palindrome" → Expand from center or Manacher's
   Why: Specialized algorithms avoid O(n²) brute force

8. If SUBARRAY/SUBSTRING problem:
   Pattern: Sliding Window, Prefix Sum, Kadane's
   Example problems:
   - "Max sum subarray of size k" → Sliding Window O(n)
   - "Longest substring without repeating" → Sliding Window O(n)
   - "Subarray sum equals K" → Prefix Sum + Hash Map O(n)
   - "Maximum subarray sum" → Kadane's Algorithm O(n)
   Why: Avoid O(n²) by maintaining window/prefix

9. If IN-PLACE operation required:
   Pattern: Two Pointers, Cyclic Sort
   Example problems:
   - "Remove duplicates from sorted array" → Two Pointers O(n)
   - "Reverse array" → Two Pointers O(n)
   - "Find missing number in 1..n" → Cyclic Sort O(n)
   Why: Modify input directly, save O(n) space

10. If CYCLE detection:
    Pattern: Floyd's Algorithm (Fast & Slow), Hash Set
    Example problems:
    - "Linked list has cycle" → Floyd's O(n) time, O(1) space
    - "Find duplicate number" → Floyd's (treat as linked list)
    - "Happy number" → Floyd's or Hash Set
    Why: Floyd's detects cycle in O(1) space

COMMUNICATION TIPS:
1. **Think out loud**: Explain your thought process
2. **Ask clarifying questions**: Show you think about edge cases
3. **Start with brute force**: Show you can solve it, then optimize
4. **Discuss tradeoffs**: Time vs space, simplicity vs performance
5. **Test your code**: Walk through examples before saying "done"
6. **Don't panic if stuck**: Ask for hints, break down problem

RED FLAGS TO AVOID:
- Jumping to code without plan
- Not considering edge cases
- Not testing code
- Staying silent
- Giving up too quickly
- Not listening to hints
*/
```

### Resources for Further Learning

```javascript
/*
RECOMMENDED PRACTICE PLATFORMS:
1. LeetCode (leetcode.com)
2. HackerRank (hackerrank.com)
3. CodeForces (codeforces.com)
4. AtCoder (atcoder.jp)
5. Project Euler (projecteuler.net)

RECOMMENDED BOOKS:
1. "Introduction to Algorithms" - CLRS
2. "The Algorithm Design Manual" - Skiena
3. "Cracking the Coding Interview" - McDowell
4. "Elements of Programming Interviews" - Aziz, Lee, Prakash

RECOMMENDED COURSES:
1. MIT 6.006 - Introduction to Algorithms
2. Princeton Algorithms Course (Coursera)
3. Stanford CS161 - Design and Analysis of Algorithms

KEY CONCEPTS TO MASTER:
- Big O Analysis
- Recursion and Iteration
- Common Data Structures
- Algorithm Design Paradigms
- Problem-Solving Patterns
- System Design Basics
*/
```

---

## Conclusion

**🎓 Congratulations on completing this comprehensive JavaScript DSA tutorial!**

This tutorial has covered:

### 📚 **Part 1: Fundamentals & Beginner Level**
1. **Time & Space Complexity**: Understanding Big O notation and performance analysis
2. **Arrays & Strings**: Core operations, two-sum, three-sum, string algorithms
3. **Searching**: Linear search, binary search, interpolation search
4. **Sorting**: Bubble, selection, insertion, merge, quick, heap, radix, counting sort

**Key Takeaway**: Master these fundamentals - they're building blocks for everything else!

### 🚀 **Part 2: Intermediate Data Structures**
1. **Linked Lists**: Singly, doubly, cycle detection, reversal
2. **Stacks & Queues**: LIFO/FIFO operations, monotonic stack, priority queue
3. **Hash Tables**: O(1) operations, collision handling, LRU/LFU cache
4. **Trees & BST**: Traversals (inorder, preorder, postorder, level-order), validation
5. **Heaps**: Min/max heap, heap sort, k-th largest element

**Key Takeaway**: Choose the right data structure for O(1) or O(log n) operations!

### 💡 **Part 3: Advanced Data Structures**
1. **Graphs**: DFS, BFS, topological sort, cycle detection, Dijkstra's algorithm
2. **Tries**: Prefix trees, autocomplete, word search
3. **Segment Trees**: Range queries, interval problems
4. **Advanced Trees**: AVL trees, self-balancing operations

**Key Takeaway**: These structures solve complex problems that simple arrays can't!

### 🧠 **Part 4: Algorithm Design Paradigms**
1. **Divide & Conquer**: Binary search, merge sort, quick select
2. **Dynamic Programming**: Memoization vs tabulation, classic problems (LCS, knapsack, coin change)
3. **Greedy Algorithms**: Activity selection, Huffman coding, interval scheduling
4. **Backtracking**: Permutations, combinations, N-Queens, Sudoku

**Key Takeaway**: Recognize problem patterns to choose the right paradigm!

### ⚡ **Part 5: Advanced Algorithms**
1. **Bit Manipulation**: XOR tricks, counting bits, power of two
2. **Advanced String Algorithms**: KMP, Rabin-Karp, Z-algorithm, Manacher's
3. **Computational Geometry**: Convex hull, line intersection

**Key Takeaway**: These optimizations can turn O(n²) into O(n)!

### 🏗️ **Part 6: Real-World Applications**
1. **System Design Patterns**: Rate limiter, URL shortener, consistent hashing, Bloom filter
2. **Problem-Solving Patterns**: Sliding window, two pointers, fast-slow, prefix sum, monotonic stack

**Key Takeaway**: Apply DSA knowledge to build scalable real-world systems!

---

## 🎯 **Your Learning Path Forward**

### **Beginner Path (Weeks 1-4)**
✅ Master Big O analysis  
✅ Arrays and string manipulation  
✅ Basic searching and sorting  
✅ Solve 30-50 easy problems on LeetCode

### **Intermediate Path (Weeks 5-12)**
✅ Linked lists, stacks, queues  
✅ Trees and graph traversals  
✅ Hash tables and sets  
✅ Two pointers and sliding window  
✅ Solve 50-70 medium problems

### **Advanced Path (Weeks 13-24)**
✅ Dynamic programming (all patterns)  
✅ Advanced graph algorithms  
✅ Backtracking and pruning  
✅ System design basics  
✅ Solve 30-50 hard problems  
✅ Participate in contests

---

## 📈 **Practice Strategy**

### **Daily Practice Routine**
```javascript
// Week 1-4: Build Foundation
const beginnerRoutine = {
  daily: "1-2 easy problems (30-60 min)",
  focus: "Understanding, not speed",
  review: "Revisit problems after 3 days",
  goal: "Build problem-solving confidence"
};

// Week 5-12: Gain Momentum  
const intermediateRoutine = {
  daily: "1 medium problem (45-90 min)",
  weekly: "1 hard problem (explore, don't stress)",
  focus: "Recognize patterns, optimize solutions",
  review: "Review and explain solutions to others",
  goal: "Speed + accuracy improvement"
};

// Week 13+: Master Level
const advancedRoutine = {
  daily: "1-2 medium/hard problems (60-120 min)",
  weekly: "Participate in 1 contest",
  monthly: "Review all topics, identify weak areas",
  focus: "Interview readiness, system design",
  goal: "Consistent performance under pressure"
};
```

### **Problem Selection Strategy**
1. **By Topic**: Master one topic at a time (e.g., all array problems, then move to strings)
2. **By Pattern**: Focus on patterns (sliding window, two pointers, DP)
3. **By Company**: Practice problems from companies you're targeting
4. **Random Mix**: Test your pattern recognition ability

### **When You Get Stuck**
1. ⏰ **Spend 20-30 minutes** trying on your own
2. 💭 **Think out loud** - explain problem to rubber duck
3. 📝 **Draw diagrams** - visualize the problem
4. 🔍 **Look at hints** - not full solution immediately
5. 📖 **Read solution** - if truly stuck
6. ✍️ **Code it yourself** - don't copy-paste
7. 🔄 **Retry next day** - ensure you truly understand

---

## 🎓 **Resources for Further Learning**

### **Practice Platforms** (Ordered by Recommendation)
```javascript
const platforms = [
  {
    name: "LeetCode",
    url: "leetcode.com",
    best_for: "Interview preparation, discuss solutions",
    difficulty: "Easy to Hard",
    recommendation: "⭐⭐⭐⭐⭐ MUST USE"
  },
  {
    name: "HackerRank",  
    url: "hackerrank.com",
    best_for: "Learning basics, company hiring",
    difficulty: "Beginner friendly",
    recommendation: "⭐⭐⭐⭐ Great for beginners"
  },
  {
    name: "CodeForces",
    url: "codeforces.com", 
    best_for: "Competitive programming, contests",
    difficulty: "Medium to Very Hard",
    recommendation: "⭐⭐⭐⭐ Advanced users"
  },
  {
    name: "AtCoder",
    url: "atcoder.jp",
    best_for: "Quality problems, contests",
    difficulty: "Medium to Hard",
    recommendation: "⭐⭐⭐ Contest practice"
  },
  {
    name: "Project Euler",
    url: "projecteuler.net",
    best_for: "Mathematical problems",
    difficulty: "Varies",
    recommendation: "⭐⭐⭐ Math + algorithms"
  }
];
```

### **Essential Books**
```javascript
const books = [
  {
    title: "Introduction to Algorithms (CLRS)",
    authors: "Cormen, Leiserson, Rivest, Stein",
    level: "Comprehensive reference",
    best_for: "Deep understanding of algorithms",
    pages: 1300,
    rating: "⭐⭐⭐⭐⭐"
  },
  {
    title: "The Algorithm Design Manual",
    author: "Steven Skiena",
    level: "Practical guide",
    best_for: "Problem-solving strategies",
    pages: 700,
    rating: "⭐⭐⭐⭐⭐"
  },
  {
    title: "Cracking the Coding Interview",
    author: "Gayle Laakmann McDowell",
    level: "Interview prep",
    best_for: "Interview questions + solutions",
    pages: 700,
    rating: "⭐⭐⭐⭐⭐ MUST READ for interviews"
  },
  {
    title: "Elements of Programming Interviews in JavaScript",
    authors: "Aziz, Lee, Prakash",
    level: "Interview prep",
    best_for: "JavaScript-specific problems",
    pages: 500,
    rating: "⭐⭐⭐⭐"
  }
];
```

### **Online Courses**
```javascript
const courses = [
  {
    name: "MIT 6.006 - Introduction to Algorithms",
    platform: "MIT OpenCourseWare / YouTube",
    cost: "Free",
    duration: "~40 hours",
    level: "University level",
    rating: "⭐⭐⭐⭐⭐"
  },
  {
    name: "Princeton Algorithms Course",
    platform: "Coursera",
    instructors: "Robert Sedgewick, Kevin Wayne",
    cost: "Free to audit",
    duration: "~60 hours",
    rating: "⭐⭐⭐⭐⭐"
  },
  {
    name: "Stanford CS161 - Design and Analysis of Algorithms",
    platform: "Stanford Online",
    cost: "Free",
    duration: "~50 hours",
    rating: "⭐⭐⭐⭐⭐"
  },
  {
    name: "JavaScript Algorithms and Data Structures Masterclass",
    platform: "Udemy",
    instructor: "Colt Steele",
    cost: "~$15 on sale",
    duration: "~22 hours",
    rating: "⭐⭐⭐⭐⭐ Best for JS devs"
  }
];
```

---

## ✅ **Key Concepts to Master**

### **Core Fundamentals**
- ✅ Big O Analysis - Analyze time and space complexity
- ✅ Recursion vs Iteration - Know when to use each
- ✅ Common Data Structures - Arrays, linked lists, trees, graphs, hash tables
- ✅ Algorithm Paradigms - DP, greedy, divide & conquer, backtracking
- ✅ Problem-Solving Patterns - Sliding window, two pointers, fast-slow

### **Must-Know Problems** (Practice These!)
```javascript
const mustKnowProblems = {
  arrays: [
    "Two Sum",
    "Best Time to Buy/Sell Stock", 
    "Maximum Subarray (Kadane's)",
    "Product of Array Except Self",
    "Container With Most Water"
  ],
  strings: [
    "Valid Anagram",
    "Longest Substring Without Repeating Characters",
    "Longest Palindromic Substring",
    "Group Anagrams"
  ],
  linkedLists: [
    "Reverse Linked List",
    "Detect Cycle",
    "Merge Two Sorted Lists",
    "LRU Cache"
  ],
  trees: [
    "Maximum Depth",
    "Validate BST",
    "Lowest Common Ancestor",
    "Binary Tree Level Order Traversal",
    "Serialize/Deserialize Binary Tree"
  ],
  graphs: [
    "Number of Islands",
    "Course Schedule (Topological Sort)",
    "Clone Graph",
    "Word Ladder"
  ],
  dynamicProgramming: [
    "Climbing Stairs",
    "Coin Change",
    "Longest Increasing Subsequence",
    "0/1 Knapsack",
    "Longest Common Subsequence",
    "Edit Distance"
  ],
  backtracking: [
    "Permutations",
    "Subsets",
    "N-Queens",
    "Word Search"
  ]
};
```

---

## 🚀 **Final Tips for Success**

### **Mindset**
```javascript
const successMindset = {
  consistency: "Better to practice 1 hour daily than 10 hours on Sunday",
  patience: "Mastery takes 6-12 months of consistent practice",
  mistakes: "Every wrong solution teaches you something valuable",
  comparison: "Don't compare your day 1 to someone's day 365",
  breaks: "Take breaks when frustrated. Fresh mind solves better.",
  community: "Learn with others, explain solutions, teach to master"
};
```

### **Interview Preparation**
1. **3 months before**: Focus on fundamentals, solve 100+ easy/medium problems
2. **1 month before**: Mock interviews, time yourself, practice communication
3. **1 week before**: Review patterns, revisit hard problems, rest well
4. **Interview day**: Stay calm, think aloud, ask questions, test your code

### **Remember**
> "The expert in anything was once a beginner." - Helen Hayes

**Every algorithm master was once where you are now.** The difference? They kept practicing.

---

## 🎉 **You're Ready!**

You now have:
- ✅ **200+ code examples** with detailed explanations
- ✅ **Complete coverage** from beginner to advanced
- ✅ **Problem-solving patterns** used in real interviews
- ✅ **System design patterns** for real-world applications
- ✅ **Optimization techniques** to write efficient code

**Your journey doesn't end here - it begins!**

Start solving problems today. Be patient with yourself. Celebrate small wins. And remember:

🎯 **Practice is key!**  
🧠 **Understand, don't memorize!**  
💪 **Stay consistent, not perfect!**  
🚀 **You've got this!**

---

**Good luck with your DSA journey! May your code compile and your solutions be optimal! 🚀**

---

*Last Updated: December 2025*  
*Total Code Examples: 200+*  
*Coverage: Beginner to Advanced Level*  
*With Love for JavaScript Developers ❤️*