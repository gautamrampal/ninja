# The Complete C Programming Master Roadmap - PART 3
## Algorithms & Optimization

---

## Table of Contents - Part 3
1. [Sorting Algorithms](#1-sorting-algorithms)
2. [Searching Algorithms](#2-searching-algorithms)
3. [Dynamic Programming](#3-dynamic-programming)
4. [Advanced Graph Algorithms](#4-advanced-graph-algorithms)
5. [Greedy Algorithms](#5-greedy-algorithms)
6. [Bit Manipulation](#6-bit-manipulation)

---

## 1. Sorting Algorithms

### 1.1 Quick Sort

**Algorithm Logic:**
```
QuickSort(arr, low, high)
1. Pick pivot element (usually last element)
2. Partition: rearrange array so elements < pivot are on left, > pivot on right
3. Recursively sort left and right subarrays

Partition Process:
Initial:  [10, 80, 30, 90, 40, 50, 70] pivot=70
          i                            j
After:    [10, 30, 40, 50, 70, 90, 80]
                         ↑
                    pivot position

Time Complexity:
- Best/Average: O(n log n)
- Worst: O(n²) when already sorted
- Space: O(log n) recursion stack
```

**Example:**
```c
#include <stdio.h>

void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int partition(int arr[], int low, int high) {
    int pivot = arr[high];  // Choose last element as pivot
    int i = low - 1;  // Index of smaller element
    
    for (int j = low; j < high; j++) {
        // If current element is smaller than pivot
        if (arr[j] < pivot) {
            i++;
            swap(&arr[i], &arr[j]);
        }
    }
    swap(&arr[i + 1], &arr[high]);
    return i + 1;
}

void quick_sort(int arr[], int low, int high) {
    if (low < high) {
        // Partitioning index
        int pi = partition(arr, low, high);
        
        // Recursively sort elements before and after partition
        quick_sort(arr, low, pi - 1);
        quick_sort(arr, pi + 1, high);
    }
}

// Randomized Quick Sort (better average case)
int randomized_partition(int arr[], int low, int high) {
    int random_index = low + rand() % (high - low + 1);
    swap(&arr[random_index], &arr[high]);
    return partition(arr, low, high);
}

void randomized_quick_sort(int arr[], int low, int high) {
    if (low < high) {
        int pi = randomized_partition(arr, low, high);
        randomized_quick_sort(arr, low, pi - 1);
        randomized_quick_sort(arr, pi + 1, high);
    }
}

void print_array(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main() {
    int arr[] = {64, 34, 25, 12, 22, 11, 90};
    int n = sizeof(arr) / sizeof(arr[0]);
    
    printf("Original array: ");
    print_array(arr, n);
    
    quick_sort(arr, 0, n - 1);
    
    printf("Sorted array: ");
    print_array(arr, n);
    
    return 0;
}
```

### 1.2 Merge Sort

**Algorithm Logic:**
```
MergeSort(arr, left, right)
1. Divide array into two halves
2. Recursively sort each half
3. Merge the two sorted halves

Merge Process:
Left:  [12, 25, 34]
Right: [11, 22, 64, 90]

Merged: [11, 12, 22, 25, 34, 64, 90]

Time Complexity: O(n log n) always
Space Complexity: O(n) for temporary arrays
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>

void merge(int arr[], int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    
    // Create temporary arrays
    int *L = malloc(n1 * sizeof(int));
    int *R = malloc(n2 * sizeof(int));
    
    // Copy data to temp arrays
    for (int i = 0; i < n1; i++) {
        L[i] = arr[left + i];
    }
    for (int j = 0; j < n2; j++) {
        R[j] = arr[mid + 1 + j];
    }
    
    // Merge the temp arrays back
    int i = 0, j = 0, k = left;
    
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }
    
    // Copy remaining elements
    while (i < n1) {
        arr[k++] = L[i++];
    }
    while (j < n2) {
        arr[k++] = R[j++];
    }
    
    free(L);
    free(R);
}

void merge_sort(int arr[], int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        
        // Sort first and second halves
        merge_sort(arr, left, mid);
        merge_sort(arr, mid + 1, right);
        
        // Merge the sorted halves
        merge(arr, left, mid, right);
    }
}

// Iterative Merge Sort (bottom-up)
void merge_sort_iterative(int arr[], int n) {
    // Start with merge subarrays of size 1, then 2, 4, 8, ...
    for (int curr_size = 1; curr_size < n; curr_size *= 2) {
        // Pick starting point of left subarray
        for (int left = 0; left < n - 1; left += 2 * curr_size) {
            int mid = left + curr_size - 1;
            int right = (left + 2 * curr_size - 1 < n - 1) ? 
                        left + 2 * curr_size - 1 : n - 1;
            
            if (mid < right) {
                merge(arr, left, mid, right);
            }
        }
    }
}

int main() {
    int arr[] = {12, 11, 13, 5, 6, 7};
    int n = sizeof(arr) / sizeof(arr[0]);
    
    printf("Original array: ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");
    
    merge_sort(arr, 0, n - 1);
    
    printf("Sorted array: ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");
    
    return 0;
}
```

### 1.3 Heap Sort

**Algorithm Logic:**
```
HeapSort:
1. Build max heap from array
2. Repeatedly extract maximum (root) and heapify

Max Heap:
       90
      /  \
    80    70
   / \    /
  50 60  40

Time Complexity: O(n log n)
Space Complexity: O(1) - in-place
```

**Example:**
```c
#include <stdio.h>

void heapify(int arr[], int n, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;
    
    // If left child is larger than root
    if (left < n && arr[left] > arr[largest]) {
        largest = left;
    }
    
    // If right child is larger than largest so far
    if (right < n && arr[right] > arr[largest]) {
        largest = right;
    }
    
    // If largest is not root
    if (largest != i) {
        int temp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = temp;
        
        // Recursively heapify the affected sub-tree
        heapify(arr, n, largest);
    }
}

void heap_sort(int arr[], int n) {
    // Build max heap
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }
    
    // Extract elements from heap one by one
    for (int i = n - 1; i > 0; i--) {
        // Move current root to end
        int temp = arr[0];
        arr[0] = arr[i];
        arr[i] = temp;
        
        // Heapify the reduced heap
        heapify(arr, i, 0);
    }
}

int main() {
    int arr[] = {12, 11, 13, 5, 6, 7};
    int n = sizeof(arr) / sizeof(arr[0]);
    
    printf("Original array: ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");
    
    heap_sort(arr, n);
    
    printf("Sorted array: ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");
    
    return 0;
}
```

### 1.4 Comparison of Sorting Algorithms

```
┌──────────────┬──────────┬──────────┬──────────┬────────┬─────────┐
│  Algorithm   │   Best   │ Average  │  Worst   │ Space  │ Stable  │
├──────────────┼──────────┼──────────┼──────────┼────────┼─────────┤
│ Bubble Sort  │  O(n)    │  O(n²)   │  O(n²)   │  O(1)  │   Yes   │
│ Selection    │  O(n²)   │  O(n²)   │  O(n²)   │  O(1)  │   No    │
│ Insertion    │  O(n)    │  O(n²)   │  O(n²)   │  O(1)  │   Yes   │
│ Merge Sort   │O(n log n)│O(n log n)│O(n log n)│  O(n)  │   Yes   │
│ Quick Sort   │O(n log n)│O(n log n)│  O(n²)   │O(log n)│   No    │
│ Heap Sort    │O(n log n)│O(n log n)│O(n log n)│  O(1)  │   No    │
└──────────────┴──────────┴──────────┴──────────┴────────┴─────────┘
```

---

## 2. Searching Algorithms

### 2.1 Binary Search

**Algorithm Logic:**
```
BinarySearch(arr, target)
Prerequisites: Array must be sorted

Process:
[1, 3, 5, 7, 9, 11, 13, 15]
 L        M              R

If target < M: search left half
If target > M: search right half
If target == M: found!

Time Complexity: O(log n)
Space: O(1) iterative, O(log n) recursive
```

**Example:**
```c
#include <stdio.h>

// Iterative binary search
int binary_search_iterative(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;  // Avoid overflow
        
        if (arr[mid] == target) {
            return mid;
        }
        
        if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return -1;  // Not found
}

// Recursive binary search
int binary_search_recursive(int arr[], int left, int right, int target) {
    if (left > right) {
        return -1;
    }
    
    int mid = left + (right - left) / 2;
    
    if (arr[mid] == target) {
        return mid;
    }
    
    if (arr[mid] < target) {
        return binary_search_recursive(arr, mid + 1, right, target);
    }
    return binary_search_recursive(arr, left, mid - 1, target);
}

// Find first occurrence
int find_first(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            result = mid;
            right = mid - 1;  // Continue searching in left half
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}

// Find last occurrence
int find_last(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            result = mid;
            left = mid + 1;  // Continue searching in right half
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return result;
}

// Find insertion position
int search_insert_position(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    
    return left;  // Position to insert
}

int main() {
    int arr[] = {1, 2, 3, 4, 4, 4, 5, 6, 7, 8};
    int n = sizeof(arr) / sizeof(arr[0]);
    
    int target = 4;
    int index = binary_search_iterative(arr, n, target);
    printf("Target %d found at index: %d\n", target, index);
    
    int first = find_first(arr, n, target);
    int last = find_last(arr, n, target);
    printf("First occurrence of %d: %d\n", target, first);
    printf("Last occurrence of %d: %d\n", target, last);
    printf("Count of %d: %d\n", target, last - first + 1);
    
    int insert_pos = search_insert_position(arr, n, 5);
    printf("Insert position for 5: %d\n", insert_pos);
    
    return 0;
}
```

---

## 3. Dynamic Programming

**DP Concept:**
```
Dynamic Programming = Recursion + Memoization
Break problem into overlapping subproblems
Store results to avoid recomputation

Two Approaches:
1. Top-Down (Memoization): Recursive with cache
2. Bottom-Up (Tabulation): Iterative, fill table
```

### 3.1 Fibonacci - Classic DP Example

**Example:**
```c
#include <stdio.h>
#include <string.h>

// Naive recursion - O(2^n)
int fib_recursive(int n) {
    if (n <= 1) return n;
    return fib_recursive(n - 1) + fib_recursive(n - 2);
}

// Top-down with memoization - O(n)
int fib_memo_helper(int n, int memo[]) {
    if (memo[n] != -1) {
        return memo[n];
    }
    
    if (n <= 1) {
        memo[n] = n;
    } else {
        memo[n] = fib_memo_helper(n - 1, memo) + fib_memo_helper(n - 2, memo);
    }
    
    return memo[n];
}

int fib_memoization(int n) {
    int memo[n + 1];
    memset(memo, -1, sizeof(memo));
    return fib_memo_helper(n, memo);
}

// Bottom-up tabulation - O(n)
int fib_tabulation(int n) {
    if (n <= 1) return n;
    
    int dp[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    
    return dp[n];
}

// Space optimized - O(1) space
int fib_optimized(int n) {
    if (n <= 1) return n;
    
    int prev2 = 0, prev1 = 1;
    int current;
    
    for (int i = 2; i <= n; i++) {
        current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }
    
    return current;
}

int main() {
    int n = 10;
    
    printf("Fibonacci(%d):\n", n);
    printf("Recursive: %d\n", fib_recursive(n));
    printf("Memoization: %d\n", fib_memoization(n));
    printf("Tabulation: %d\n", fib_tabulation(n));
    printf("Optimized: %d\n", fib_optimized(n));
    
    return 0;
}
```

### 3.2 Longest Common Subsequence (LCS)

**Algorithm Logic:**
```
LCS("ABCDGH", "AEDFHR") = "ADH"

DP Table:
      ""  A  E  D  F  H  R
  ""   0  0  0  0  0  0  0
  A    0  1  1  1  1  1  1
  B    0  1  1  1  1  1  1
  C    0  1  1  1  1  1  1
  D    0  1  1  2  2  2  2
  G    0  1  1  2  2  2  2
  H    0  1  1  2  2  3  3

If chars match: dp[i][j] = dp[i-1][j-1] + 1
Else: dp[i][j] = max(dp[i-1][j], dp[i][j-1])
```

**Example:**
```c
#include <stdio.h>
#include <string.h>

int max(int a, int b) {
    return (a > b) ? a : b;
}

// Returns length of LCS
int lcs(char *X, char *Y, int m, int n) {
    int dp[m + 1][n + 1];
    
    // Build dp table bottom-up
    for (int i = 0; i <= m; i++) {
        for (int j = 0; j <= n; j++) {
            if (i == 0 || j == 0) {
                dp[i][j] = 0;
            } else if (X[i - 1] == Y[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    
    return dp[m][n];
}

// Print LCS string
void print_lcs(char *X, char *Y, int m, int n) {
    int dp[m + 1][n + 1];
    
    for (int i = 0; i <= m; i++) {
        for (int j = 0; j <= n; j++) {
            if (i == 0 || j == 0) {
                dp[i][j] = 0;
            } else if (X[i - 1] == Y[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    
    // Backtrack to find LCS string
    int index = dp[m][n];
    char lcs_str[index + 1];
    lcs_str[index] = '\0';
    
    int i = m, j = n;
    while (i > 0 && j > 0) {
        if (X[i - 1] == Y[j - 1]) {
            lcs_str[--index] = X[i - 1];
            i--;
            j--;
        } else if (dp[i - 1][j] > dp[i][j - 1]) {
            i--;
        } else {
            j--;
        }
    }
    
    printf("LCS: %s\n", lcs_str);
}

int main() {
    char X[] = "AGGTAB";
    char Y[] = "GXTXAYB";
    
    int m = strlen(X);
    int n = strlen(Y);
    
    printf("Length of LCS: %d\n", lcs(X, Y, m, n));
    print_lcs(X, Y, m, n);
    
    return 0;
}
```

### 3.3 0/1 Knapsack Problem

**Algorithm Logic:**
```
Given: weights[], values[], capacity W
Find: Maximum value without exceeding capacity

Items:
  Item  Weight  Value
   1      2       3
   2      3       4
   3      4       5
Capacity: 5

DP Table: dp[i][w] = max value using first i items with capacity w
```

**Example:**
```c
#include <stdio.h>

int max(int a, int b) {
    return (a > b) ? a : b;
}

int knapsack(int W, int weights[], int values[], int n) {
    int dp[n + 1][W + 1];
    
    // Build table bottom-up
    for (int i = 0; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            if (i == 0 || w == 0) {
                dp[i][w] = 0;
            } else if (weights[i - 1] <= w) {
                // Include or exclude item
                dp[i][w] = max(
                    values[i - 1] + dp[i - 1][w - weights[i - 1]],
                    dp[i - 1][w]
                );
            } else {
                // Cannot include item
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    
    // Print selected items
    printf("Selected items: ");
    int w = W;
    for (int i = n; i > 0 && w > 0; i--) {
        if (dp[i][w] != dp[i - 1][w]) {
            printf("%d ", i);
            w -= weights[i - 1];
        }
    }
    printf("\n");
    
    return dp[n][W];
}

int main() {
    int values[] = {60, 100, 120};
    int weights[] = {10, 20, 30};
    int W = 50;
    int n = sizeof(values) / sizeof(values[0]);
    
    printf("Maximum value: %d\n", knapsack(W, weights, values, n));
    
    return 0;
}
```

---

## CONTINUED IN PART 4

This is Part 3 of the Complete C Programming Master Roadmap. It covers:
- Sorting Algorithms (Quick, Merge, Heap)
- Searching Algorithms (Binary Search variants)
- Dynamic Programming (Fibonacci, LCS, Knapsack)

**Part 4 will include:**
- Advanced Graph Algorithms (Dijkstra, Bellman-Ford, Floyd-Warshall)
- Greedy Algorithms
- Bit Manipulation Techniques
- Systems Programming
- File I/O and Process Management

Save this file and continue with Part 4!