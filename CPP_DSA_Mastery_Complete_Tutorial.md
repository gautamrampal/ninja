# C++ Data Structures and Algorithms Mastery Tutorial

## Complete Guide from Beginner to Advanced Level

---

## Table of Contents

1. [Introduction to DSA](#introduction-to-dsa)
2. [Time and Space Complexity Analysis](#time-and-space-complexity-analysis)
3. [Basic Data Structures](#basic-data-structures)
4. [Intermediate Data Structures](#intermediate-data-structures)
5. [Advanced Data Structures](#advanced-data-structures)
6. [Sorting Algorithms](#sorting-algorithms)
7. [Searching Algorithms](#searching-algorithms)
8. [Graph Algorithms](#graph-algorithms)
9. [Dynamic Programming](#dynamic-programming)
10. [Advanced Algorithmic Techniques](#advanced-algorithmic-techniques)
11. [Problem-Solving Patterns](#problem-solving-patterns)
12. [Practice Problems and Solutions](#practice-problems-and-solutions)

---

## Introduction to DSA

### What are Data Structures?

Data structures are specialized formats for organizing, processing, retrieving, and storing data. They enable efficient access and modification of data.

```
Data Structure Classification
│
├── Linear Data Structures
│   ├── Arrays
│   ├── Linked Lists
│   ├── Stacks
│   └── Queues
│
└── Non-Linear Data Structures
    ├── Trees
    ├── Graphs
    ├── Hash Tables
    └── Heaps
```

### What are Algorithms?

Algorithms are step-by-step procedures or formulas for solving problems.

```
Algorithm Categories
│
├── Sorting (Bubble, Quick, Merge, Heap)
├── Searching (Linear, Binary, DFS, BFS)
├── Graph (Dijkstra, Bellman-Ford, Floyd-Warshall)
├── Dynamic Programming (Knapsack, LCS, LIS)
├── Greedy (Huffman, Prim's, Kruskal's)
└── Divide and Conquer (Merge Sort, Quick Sort)
```

---

## Time and Space Complexity Analysis

### Big O Notation

Big O notation describes the upper bound of algorithm complexity.

```
Complexity Growth Chart (Best to Worst)
│
O(1)        - Constant Time
O(log n)    - Logarithmic Time
O(n)        - Linear Time
O(n log n)  - Linearithmic Time
O(n²)       - Quadratic Time
O(n³)       - Cubic Time
O(2ⁿ)       - Exponential Time
O(n!)       - Factorial Time
```

### Visual Representation

```
Time Complexity Growth
│
│                                                    O(n!)
│                                               O(2ⁿ)
│                                          O(n³)
│                                    O(n²)
│                             O(n log n)
│                       O(n)
│                 O(log n)
│           O(1)
└────────────────────────────────────────────────────────> n
```

### Common Operations Complexity

| Data Structure | Access | Search | Insertion | Deletion |
|----------------|--------|--------|-----------|----------|
| Array          | O(1)   | O(n)   | O(n)      | O(n)     |
| Linked List    | O(n)   | O(n)   | O(1)      | O(1)     |
| Stack          | O(n)   | O(n)   | O(1)      | O(1)     |
| Queue          | O(n)   | O(n)   | O(1)      | O(1)     |
| Hash Table     | N/A    | O(1)   | O(1)      | O(1)     |
| Binary Tree    | O(n)   | O(n)   | O(n)      | O(n)     |
| BST            | O(log n)| O(log n)| O(log n) | O(log n)|
| AVL Tree       | O(log n)| O(log n)| O(log n) | O(log n)|

### Analyzing Code Complexity

```cpp
// Example 1: O(1) - Constant Time
int getFirstElement(vector<int>& arr) {
    return arr[0];  // Single operation
}

// Example 2: O(n) - Linear Time
int sumArray(vector<int>& arr) {
    int sum = 0;
    for (int num : arr) {  // Loop runs n times
        sum += num;
    }
    return sum;
}

// Example 3: O(n²) - Quadratic Time
void printPairs(vector<int>& arr) {
    for (int i = 0; i < arr.size(); i++) {      // n times
        for (int j = 0; j < arr.size(); j++) {  // n times
            cout << arr[i] << ", " << arr[j] << endl;
        }
    }
}

// Example 4: O(log n) - Logarithmic Time
int binarySearch(vector<int>& arr, int target) {
    int left = 0, right = arr.size() - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}

// Example 5: O(n log n) - Linearithmic Time
void mergeSort(vector<int>& arr, int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(arr, left, mid);      // log n divisions
        mergeSort(arr, mid + 1, right); // log n divisions
        merge(arr, left, mid, right);   // n operations
    }
}
```

---

## Basic Data Structures

### 1. Arrays

**Concept**: Contiguous block of memory storing elements of the same type.

```
Array Memory Layout
┌───┬───┬───┬───┬───┐
│ 5 │ 2 │ 8 │ 1 │ 9 │
└───┴───┴───┴───┴───┘
  0   1   2   3   4    (indices)
```

**Implementation**:

```cpp
#include <iostream>
#include <vector>
using namespace std;

class ArrayOperations {
public:
    // Static Array
    void staticArrayExample() {
        int arr[5] = {5, 2, 8, 1, 9};
        
        // Access: O(1)
        cout << "Element at index 2: " << arr[2] << endl;
        
        // Traverse: O(n)
        for (int i = 0; i < 5; i++) {
            cout << arr[i] << " ";
        }
        cout << endl;
    }
    
    // Dynamic Array (Vector)
    void dynamicArrayExample() {
        vector<int> vec = {5, 2, 8, 1, 9};
        
        // Insert at end: O(1) amortized
        vec.push_back(7);
        
        // Insert at position: O(n)
        vec.insert(vec.begin() + 2, 10);
        
        // Delete from end: O(1)
        vec.pop_back();
        
        // Delete from position: O(n)
        vec.erase(vec.begin() + 3);
        
        // Search: O(n)
        auto it = find(vec.begin(), vec.end(), 8);
        if (it != vec.end()) {
            cout << "Found 8 at index: " << (it - vec.begin()) << endl;
        }
    }
    
    // Two Pointer Technique
    void reverseArray(vector<int>& arr) {
        int left = 0, right = arr.size() - 1;
        while (left < right) {
            swap(arr[left], arr[right]);
            left++;
            right--;
        }
    }
    
    // Sliding Window
    int maxSumSubarray(vector<int>& arr, int k) {
        int n = arr.size();
        if (n < k) return -1;
        
        int maxSum = 0, windowSum = 0;
        
        // Calculate sum of first window
        for (int i = 0; i < k; i++) {
            windowSum += arr[i];
        }
        maxSum = windowSum;
        
        // Slide the window
        for (int i = k; i < n; i++) {
            windowSum = windowSum - arr[i - k] + arr[i];
            maxSum = max(maxSum, windowSum);
        }
        
        return maxSum;
    }
};
```

**Sliding Window Visualization**:

```
Array: [2, 1, 5, 1, 3, 2], k = 3

Step 1: [2, 1, 5] 1, 3, 2  -> Sum = 8
Step 2:  2 [1, 5, 1] 3, 2  -> Sum = 7
Step 3:  2, 1 [5, 1, 3] 2  -> Sum = 9
Step 4:  2, 1, 5 [1, 3, 2] -> Sum = 6

Maximum Sum = 9
```

---

### 2. Linked Lists

**Concept**: Linear data structure where elements are stored in nodes, each pointing to the next.

```
Singly Linked List
┌────┬────┐    ┌────┬────┐    ┌────┬────┐    ┌────┬────┐
│ 10 │ ●──┼───>│ 20 │ ●──┼───>│ 30 │ ●──┼───>│ 40 │ NULL│
└────┴────┘    └────┴────┘    └────┴────┘    └────┴────┘
 Data  Next     Data  Next     Data  Next     Data  Next

Doubly Linked List
       ┌────┬────┬────┐    ┌────┬────┬────┐    ┌────┬────┬────┐
NULL<──┼● 10│ ●──┼───>│<───┼● 20│ ●──┼───>│<───┼● 30│ NULL│
       └────┴────┴────┘    └────┴────┴────┘    └────┴────┴────┘
       Prev Data Next      Prev Data Next      Prev Data Next
```

**Implementation**:

```cpp
#include <iostream>
using namespace std;

// Node structure for Singly Linked List
struct Node {
    int data;
    Node* next;
    
    Node(int val) : data(val), next(nullptr) {}
};

// Singly Linked List
class LinkedList {
private:
    Node* head;
    
public:
    LinkedList() : head(nullptr) {}
    
    // Insert at beginning: O(1)
    void insertAtHead(int val) {
        Node* newNode = new Node(val);
        newNode->next = head;
        head = newNode;
    }
    
    // Insert at end: O(n)
    void insertAtTail(int val) {
        Node* newNode = new Node(val);
        
        if (!head) {
            head = newNode;
            return;
        }
        
        Node* temp = head;
        while (temp->next) {
            temp = temp->next;
        }
        temp->next = newNode;
    }
    
    // Insert at position: O(n)
    void insertAtPosition(int val, int pos) {
        if (pos == 0) {
            insertAtHead(val);
            return;
        }
        
        Node* newNode = new Node(val);
        Node* temp = head;
        
        for (int i = 0; i < pos - 1 && temp; i++) {
            temp = temp->next;
        }
        
        if (temp) {
            newNode->next = temp->next;
            temp->next = newNode;
        }
    }
    
    // Delete node: O(n)
    void deleteNode(int val) {
        if (!head) return;
        
        if (head->data == val) {
            Node* temp = head;
            head = head->next;
            delete temp;
            return;
        }
        
        Node* temp = head;
        while (temp->next && temp->next->data != val) {
            temp = temp->next;
        }
        
        if (temp->next) {
            Node* toDelete = temp->next;
            temp->next = temp->next->next;
            delete toDelete;
        }
    }
    
    // Search: O(n)
    bool search(int val) {
        Node* temp = head;
        while (temp) {
            if (temp->data == val) return true;
            temp = temp->next;
        }
        return false;
    }
    
    // Reverse linked list: O(n)
    void reverse() {
        Node* prev = nullptr;
        Node* current = head;
        Node* next = nullptr;
        
        while (current) {
            next = current->next;
            current->next = prev;
            prev = current;
            current = next;
        }
        head = prev;
    }
    
    // Detect cycle: Floyd's Cycle Detection (Tortoise and Hare)
    bool hasCycle() {
        if (!head) return false;
        
        Node* slow = head;
        Node* fast = head;
        
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
            
            if (slow == fast) return true;
        }
        return false;
    }
    
    // Find middle element: O(n)
    Node* findMiddle() {
        if (!head) return nullptr;
        
        Node* slow = head;
        Node* fast = head;
        
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
        }
        return slow;
    }
    
    // Display list: O(n)
    void display() {
        Node* temp = head;
        while (temp) {
            cout << temp->data << " -> ";
            temp = temp->next;
        }
        cout << "NULL" << endl;
    }
    
    // Destructor
    ~LinkedList() {
        while (head) {
            Node* temp = head;
            head = head->next;
            delete temp;
        }
    }
};
```

**Reverse Linked List Visualization**:

```
Original:  1 -> 2 -> 3 -> 4 -> NULL

Step 1: NULL <- 1    2 -> 3 -> 4 -> NULL
             prev  curr  next

Step 2: NULL <- 1 <- 2    3 -> 4 -> NULL
                  prev  curr  next

Step 3: NULL <- 1 <- 2 <- 3    4 -> NULL
                       prev  curr  next

Step 4: NULL <- 1 <- 2 <- 3 <- 4
                            prev  curr(NULL)

Result:  4 -> 3 -> 2 -> 1 -> NULL
```

**Doubly Linked List**:

```cpp
// Node structure for Doubly Linked List
struct DNode {
    int data;
    DNode* prev;
    DNode* next;
    
    DNode(int val) : data(val), prev(nullptr), next(nullptr) {}
};

class DoublyLinkedList {
private:
    DNode* head;
    DNode* tail;
    
public:
    DoublyLinkedList() : head(nullptr), tail(nullptr) {}
    
    // Insert at beginning: O(1)
    void insertAtHead(int val) {
        DNode* newNode = new DNode(val);
        
        if (!head) {
            head = tail = newNode;
            return;
        }
        
        newNode->next = head;
        head->prev = newNode;
        head = newNode;
    }
    
    // Insert at end: O(1)
    void insertAtTail(int val) {
        DNode* newNode = new DNode(val);
        
        if (!tail) {
            head = tail = newNode;
            return;
        }
        
        tail->next = newNode;
        newNode->prev = tail;
        tail = newNode;
    }
    
    // Display forward
    void displayForward() {
        DNode* temp = head;
        while (temp) {
            cout << temp->data << " <-> ";
            temp = temp->next;
        }
        cout << "NULL" << endl;
    }
    
    // Display backward
    void displayBackward() {
        DNode* temp = tail;
        while (temp) {
            cout << temp->data << " <-> ";
            temp = temp->prev;
        }
        cout << "NULL" << endl;
    }
};
```

---

### 3. Stacks

**Concept**: LIFO (Last In First Out) data structure.

```
Stack Operations
│
│  ┌───┐
│  │ 4 │ <- Top (Last In)
│  ├───┤
│  │ 3 │
│  ├───┤
│  │ 2 │
│  ├───┤
│  │ 1 │ (First In)
│  └───┘

Push: Add element to top
Pop:  Remove element from top
Peek: View top element
```

**Implementation**:

```cpp
#include <iostream>
#include <stack>
#include <vector>
using namespace std;

// Array-based Stack Implementation
class Stack {
private:
    vector<int> arr;
    int topIndex;
    int capacity;
    
public:
    Stack(int size) : capacity(size), topIndex(-1) {
        arr.resize(capacity);
    }
    
    // Push: O(1)
    bool push(int val) {
        if (topIndex >= capacity - 1) {
            cout << "Stack Overflow" << endl;
            return false;
        }
        arr[++topIndex] = val;
        return true;
    }
    
    // Pop: O(1)
    int pop() {
        if (topIndex < 0) {
            cout << "Stack Underflow" << endl;
            return -1;
        }
        return arr[topIndex--];
    }
    
    // Peek: O(1)
    int peek() {
        if (topIndex < 0) {
            cout << "Stack is empty" << endl;
            return -1;
        }
        return arr[topIndex];
    }
    
    // Check if empty: O(1)
    bool isEmpty() {
        return topIndex < 0;
    }
    
    // Check if full: O(1)
    bool isFull() {
        return topIndex >= capacity - 1;
    }
    
    // Get size: O(1)
    int size() {
        return topIndex + 1;
    }
};

// Linked List-based Stack
class LinkedStack {
private:
    Node* top;
    
public:
    LinkedStack() : top(nullptr) {}
    
    void push(int val) {
        Node* newNode = new Node(val);
        newNode->next = top;
        top = newNode;
    }
    
    int pop() {
        if (!top) {
            cout << "Stack Underflow" << endl;
            return -1;
        }
        int val = top->data;
        Node* temp = top;
        top = top->next;
        delete temp;
        return val;
    }
    
    int peek() {
        if (!top) return -1;
        return top->data;
    }
    
    bool isEmpty() {
        return top == nullptr;
    }
};

// Stack Applications
class StackApplications {
public:
    // 1. Balanced Parentheses: O(n)
    bool isBalanced(string expr) {
        stack<char> st;
        
        for (char ch : expr) {
            if (ch == '(' || ch == '{' || ch == '[') {
                st.push(ch);
            }
            else if (ch == ')' || ch == '}' || ch == ']') {
                if (st.empty()) return false;
                
                char top = st.top();
                st.pop();
                
                if ((ch == ')' && top != '(') ||
                    (ch == '}' && top != '{') ||
                    (ch == ']' && top != '[')) {
                    return false;
                }
            }
        }
        
        return st.empty();
    }
    
    // 2. Infix to Postfix Conversion
    int precedence(char op) {
        if (op == '+' || op == '-') return 1;
        if (op == '*' || op == '/') return 2;
        if (op == '^') return 3;
        return 0;
    }
    
    string infixToPostfix(string infix) {
        stack<char> st;
        string postfix = "";
        
        for (char ch : infix) {
            if (isalnum(ch)) {
                postfix += ch;
            }
            else if (ch == '(') {
                st.push(ch);
            }
            else if (ch == ')') {
                while (!st.empty() && st.top() != '(') {
                    postfix += st.top();
                    st.pop();
                }
                if (!st.empty()) st.pop();
            }
            else {
                while (!st.empty() && precedence(st.top()) >= precedence(ch)) {
                    postfix += st.top();
                    st.pop();
                }
                st.push(ch);
            }
        }
        
        while (!st.empty()) {
            postfix += st.top();
            st.pop();
        }
        
        return postfix;
    }
    
    // 3. Next Greater Element: O(n)
    vector<int> nextGreaterElement(vector<int>& arr) {
        int n = arr.size();
        vector<int> result(n, -1);
        stack<int> st;
        
        for (int i = n - 1; i >= 0; i--) {
            while (!st.empty() && st.top() <= arr[i]) {
                st.pop();
            }
            
            if (!st.empty()) {
                result[i] = st.top();
            }
            
            st.push(arr[i]);
        }
        
        return result;
    }
    
    // 4. Stock Span Problem
    vector<int> stockSpan(vector<int>& prices) {
        int n = prices.size();
        vector<int> span(n);
        stack<int> st;
        
        for (int i = 0; i < n; i++) {
            while (!st.empty() && prices[st.top()] <= prices[i]) {
                st.pop();
            }
            
            span[i] = st.empty() ? (i + 1) : (i - st.top());
            st.push(i);
        }
        
        return span;
    }
    
    // 5. Largest Rectangle in Histogram
    int largestRectangleArea(vector<int>& heights) {
        stack<int> st;
        int maxArea = 0;
        int n = heights.size();
        
        for (int i = 0; i < n; i++) {
            while (!st.empty() && heights[st.top()] > heights[i]) {
                int height = heights[st.top()];
                st.pop();
                int width = st.empty() ? i : i - st.top() - 1;
                maxArea = max(maxArea, height * width);
            }
            st.push(i);
        }
        
        while (!st.empty()) {
            int height = heights[st.top()];
            st.pop();
            int width = st.empty() ? n : n - st.top() - 1;
            maxArea = max(maxArea, height * width);
        }
        
        return maxArea;
    }
};
```

**Next Greater Element Visualization**:

```
Array: [4, 5, 2, 10, 8]

i=4: Stack=[], arr[4]=8  -> NGE[4]=-1, Stack=[8]
i=3: Stack=[8], arr[3]=10 -> NGE[3]=-1, Stack=[10]
i=2: Stack=[10], arr[2]=2 -> NGE[2]=10, Stack=[10,2]
i=1: Stack=[10,2], arr[1]=5 -> NGE[1]=10, Stack=[10,5]
i=0: Stack=[10,5], arr[0]=4 -> NGE[0]=5, Stack=[10,5,4]

Result: [5, 10, 10, -1, -1]
```

---

### 4. Queues

**Concept**: FIFO (First In First Out) data structure.

```
Queue Operations
┌────────────────────────────┐
│  Front                Rear │
│   ↓                    ↓   │
│  ┌───┬───┬───┬───┬───┐    │
│  │ 1 │ 2 │ 3 │ 4 │ 5 │    │
│  └───┴───┴───┴───┴───┘    │
│   ↑                    ↑   │
│ Dequeue             Enqueue│
└────────────────────────────┘

Enqueue: Add element at rear
Dequeue: Remove element from front
```

**Implementation**:

```cpp
#include <iostream>
#include <queue>
using namespace std;

// Array-based Circular Queue
class CircularQueue {
private:
    vector<int> arr;
    int front, rear, capacity, count;
    
public:
    CircularQueue(int size) : capacity(size), front(0), rear(-1), count(0) {
        arr.resize(capacity);
    }
    
    // Enqueue: O(1)
    bool enqueue(int val) {
        if (count == capacity) {
            cout << "Queue Overflow" << endl;
            return false;
        }
        
        rear = (rear + 1) % capacity;
        arr[rear] = val;
        count++;
        return true;
    }
    
    // Dequeue: O(1)
    int dequeue() {
        if (count == 0) {
            cout << "Queue Underflow" << endl;
            return -1;
        }
        
        int val = arr[front];
        front = (front + 1) % capacity;
        count--;
        return val;
    }
    
    // Peek: O(1)
    int peek() {
        if (count == 0) {
            cout << "Queue is empty" << endl;
            return -1;
        }
        return arr[front];
    }
    
    bool isEmpty() { return count == 0; }
    bool isFull() { return count == capacity; }
    int size() { return count; }
};

// Linked List-based Queue
class LinkedQueue {
private:
    Node* front;
    Node* rear;
    
public:
    LinkedQueue() : front(nullptr), rear(nullptr) {}
    
    void enqueue(int val) {
        Node* newNode = new Node(val);
        
        if (!rear) {
            front = rear = newNode;
            return;
        }
        
        rear->next = newNode;
        rear = newNode;
    }
    
    int dequeue() {
        if (!front) {
            cout << "Queue Underflow" << endl;
            return -1;
        }
        
        int val = front->data;
        Node* temp = front;
        front = front->next;
        
        if (!front) rear = nullptr;
        
        delete temp;
        return val;
    }
    
    int peek() {
        if (!front) return -1;
        return front->data;
    }
    
    bool isEmpty() {
        return front == nullptr;
    }
};

// Priority Queue (Min Heap based)
class PriorityQueueExample {
public:
    void minHeapExample() {
        priority_queue<int, vector<int>, greater<int>> pq;
        
        pq.push(30);
        pq.push(10);
        pq.push(20);
        pq.push(5);
        
        cout << "Min Heap Elements: ";
        while (!pq.empty()) {
            cout << pq.top() << " ";
            pq.pop();
        }
        cout << endl;  // Output: 5 10 20 30
    }
    
    void maxHeapExample() {
        priority_queue<int> pq;
        
        pq.push(30);
        pq.push(10);
        pq.push(20);
        pq.push(5);
        
        cout << "Max Heap Elements: ";
        while (!pq.empty()) {
            cout << pq.top() << " ";
            pq.pop();
        }
        cout << endl;  // Output: 30 20 10 5
    }
};

// Deque (Double-Ended Queue)
class DequeExample {
public:
    void dequeOperations() {
        deque<int> dq;
        
        // Insert at front
        dq.push_front(10);
        dq.push_front(5);
        
        // Insert at back
        dq.push_back(20);
        dq.push_back(30);
        
        // Current: 5 10 20 30
        
        // Remove from front
        dq.pop_front();  // Removes 5
        
        // Remove from back
        dq.pop_back();   // Removes 30
        
        // Current: 10 20
    }
    
    // Sliding Window Maximum using Deque
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        deque<int> dq;
        vector<int> result;
        
        for (int i = 0; i < nums.size(); i++) {
            // Remove elements outside window
            if (!dq.empty() && dq.front() == i - k) {
                dq.pop_front();
            }
            
            // Remove smaller elements
            while (!dq.empty() && nums[dq.back()] < nums[i]) {
                dq.pop_back();
            }
            
            dq.push_back(i);
            
            // Add to result after first window
            if (i >= k - 1) {
                result.push_back(nums[dq.front()]);
            }
        }
        
        return result;
    }
};
```

**Circular Queue Visualization**:

```
Initial State (capacity=5):
  Front=0, Rear=-1, Count=0
  [_, _, _, _, _]

After Enqueue(1,2,3):
  Front=0, Rear=2, Count=3
  [1, 2, 3, _, _]
   ↑     ↑
  Front Rear

After Dequeue() twice:
  Front=2, Rear=2, Count=1
  [_, _, 3, _, _]
        ↑
   Front/Rear

After Enqueue(4,5,6,7):
  Front=2, Rear=1, Count=5 (Circular)
  [6, 7, 3, 4, 5]
        ↑     ↑
      Front  Rear
```

---

## Intermediate Data Structures

### 5. Hash Tables

**Concept**: Data structure that maps keys to values using a hash function.

```
Hash Table Structure
┌─────────────────┐
│  Hash Function  │
└────────┬────────┘
         ↓
┌───┬────────────┐
│ 0 │ NULL       │
├───┼────────────┤
│ 1 │ key1->val1 │
├───┼────────────┤
│ 2 │ key2->val2 │-> key5->val5 (Collision Chain)
├───┼────────────┤
│ 3 │ NULL       │
├───┼────────────┤
│ 4 │ key3->val3 │
└───┴────────────┘
```

**Collision Resolution**:

```
1. Chaining (Separate Chaining)
┌───┬──────────────────────────┐
│ 2 │ [key2,val2]->[key5,val5] │
└───┴──────────────────────────┘

2. Open Addressing (Linear Probing)
If collision at index i, try i+1, i+2, i+3...

3. Quadratic Probing
If collision at index i, try i+1², i+2², i+3²...

4. Double Hashing
Use second hash function to find next position
```

**Implementation**:

```cpp
#include <iostream>
#include <vector>
#include <list>
#include <unordered_map>
using namespace std;

// Hash Table with Chaining
class HashTable {
private:
    int capacity;
    list<pair<int, int>>* table;
    
    int hashFunction(int key) {
        return key % capacity;
    }
    
public:
    HashTable(int cap) : capacity(cap) {
        table = new list<pair<int, int>>[capacity];
    }
    
    // Insert: O(1) average
    void insert(int key, int value) {
        int index = hashFunction(key);
        
        // Update if key exists
        for (auto& pair : table[index]) {
            if (pair.first == key) {
                pair.second = value;
                return;
            }
        }
        
        // Insert new key-value pair
        table[index].push_back({key, value});
    }
    
    // Search: O(1) average
    int search(int key) {
        int index = hashFunction(key);
        
        for (auto& pair : table[index]) {
            if (pair.first == key) {
                return pair.second;
            }
        }
        
        return -1;  // Not found
    }
    
    // Delete: O(1) average
    void remove(int key) {
        int index = hashFunction(key);
        
        for (auto it = table[index].begin(); it != table[index].end(); ++it) {
            if (it->first == key) {
                table[index].erase(it);
                return;
            }
        }
    }
    
    // Display hash table
    void display() {
        for (int i = 0; i < capacity; i++) {
            cout << "Bucket " << i << ": ";
            for (auto& pair : table[i]) {
                cout << "[" << pair.first << ":" << pair.second << "] ";
            }
            cout << endl;
        }
    }
    
    ~HashTable() {
        delete[] table;
    }
};

// Hash Table Applications
class HashTableProblems {
public:
    // 1. Two Sum Problem: O(n)
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> map;
        
        for (int i = 0; i < nums.size(); i++) {
            int complement = target - nums[i];
            
            if (map.find(complement) != map.end()) {
                return {map[complement], i};
            }
            
            map[nums[i]] = i;
        }
        
        return {};
    }
    
    // 2. First Non-Repeating Character: O(n)
    char firstNonRepeating(string s) {
        unordered_map<char, int> freq;
        
        for (char ch : s) {
            freq[ch]++;
        }
        
        for (char ch : s) {
            if (freq[ch] == 1) {
                return ch;
            }
        }
        
        return '\0';
    }
    
    // 3. Group Anagrams: O(n * k log k)
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string, vector<string>> map;
        
        for (string& s : strs) {
            string key = s;
            sort(key.begin(), key.end());
            map[key].push_back(s);
        }
        
        vector<vector<string>> result;
        for (auto& pair : map) {
            result.push_back(pair.second);
        }
        
        return result;
    }
    
    // 4. Longest Consecutive Sequence: O(n)
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> numSet(nums.begin(), nums.end());
        int maxLength = 0;
        
        for (int num : numSet) {
            // Check if it's the start of a sequence
            if (numSet.find(num - 1) == numSet.end()) {
                int currentNum = num;
                int currentLength = 1;
                
                while (numSet.find(currentNum + 1) != numSet.end()) {
                    currentNum++;
                    currentLength++;
                }
                
                maxLength = max(maxLength, currentLength);
            }
        }
        
        return maxLength;
    }
    
    // 5. Subarray Sum Equals K: O(n)
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<int, int> prefixSum;
        prefixSum[0] = 1;
        
        int sum = 0, count = 0;
        
        for (int num : nums) {
            sum += num;
            
            if (prefixSum.find(sum - k) != prefixSum.end()) {
                count += prefixSum[sum - k];
            }
            
            prefixSum[sum]++;
        }
        
        return count;
    }
};
```

---

### 6. Trees

**Concept**: Hierarchical data structure with a root node and child nodes.

```
Binary Tree Structure
         1
       /   \
      2     3
     / \   / \
    4   5 6   7

Tree Terminology:
- Root: Node 1
- Parent: Node 2 is parent of 4 and 5
- Child: Nodes 4 and 5 are children of 2
- Leaf: Nodes 4, 5, 6, 7 (no children)
- Height: Longest path from root to leaf (3)
- Depth: Distance from root to node
- Level: Depth + 1
```

**Tree Types**:

```
1. Binary Tree
   - Each node has at most 2 children

2. Binary Search Tree (BST)
   - Left subtree < Node < Right subtree
         5
       /   \
      3     7
     / \   / \
    2   4 6   8

3. Complete Binary Tree
   - All levels filled except possibly last
   - Last level filled left to right
         1
       /   \
      2     3
     / \   /
    4   5 6

4. Perfect Binary Tree
   - All internal nodes have 2 children
   - All leaves at same level
         1
       /   \
      2     3
     / \   / \
    4   5 6   7

5. Balanced Binary Tree
   - Height difference between left and right
     subtrees is at most 1 for every node
```

**Implementation**:

```cpp
#include <iostream>
#include <queue>
#include <stack>
using namespace std;

// Tree Node
struct TreeNode {
    int data;
    TreeNode* left;
    TreeNode* right;
    
    TreeNode(int val) : data(val), left(nullptr), right(nullptr) {}
};

class BinaryTree {
public:
    TreeNode* root;
    
    BinaryTree() : root(nullptr) {}
    
    // Tree Traversals
    
    // 1. Inorder (Left-Root-Right): O(n)
    void inorder(TreeNode* node) {
        if (!node) return;
        inorder(node->left);
        cout << node->data << " ";
        inorder(node->right);
    }
    
    // 2. Preorder (Root-Left-Right): O(n)
    void preorder(TreeNode* node) {
        if (!node) return;
        cout << node->data << " ";
        preorder(node->left);
        preorder(node->right);
    }
    
    // 3. Postorder (Left-Right-Root): O(n)
    void postorder(TreeNode* node) {
        if (!node) return;
        postorder(node->left);
        postorder(node->right);
        cout << node->data << " ";
    }
    
    // 4. Level Order (BFS): O(n)
    void levelOrder(TreeNode* node) {
        if (!node) return;
        
        queue<TreeNode*> q;
        q.push(node);
        
        while (!q.empty()) {
            TreeNode* current = q.front();
            q.pop();
            
            cout << current->data << " ";
            
            if (current->left) q.push(current->left);
            if (current->right) q.push(current->right);
        }
    }
    
    // Iterative Inorder Traversal
    void inorderIterative(TreeNode* root) {
        stack<TreeNode*> st;
        TreeNode* current = root;
        
        while (current || !st.empty()) {
            while (current) {
                st.push(current);
                current = current->left;
            }
            
            current = st.top();
            st.pop();
            cout << current->data << " ";
            current = current->right;
        }
    }
    
    // Height of tree: O(n)
    int height(TreeNode* node) {
        if (!node) return 0;
        return 1 + max(height(node->left), height(node->right));
    }
    
    // Count nodes: O(n)
    int countNodes(TreeNode* node) {
        if (!node) return 0;
        return 1 + countNodes(node->left) + countNodes(node->right);
    }
    
    // Sum of all nodes: O(n)
    int sumNodes(TreeNode* node) {
        if (!node) return 0;
        return node->data + sumNodes(node->left) + sumNodes(node->right);
    }
    
    // Check if tree is balanced: O(n)
    bool isBalanced(TreeNode* node, int& height) {
        if (!node) {
            height = 0;
            return true;
        }
        
        int leftHeight = 0, rightHeight = 0;
        bool leftBalanced = isBalanced(node->left, leftHeight);
        bool rightBalanced = isBalanced(node->right, rightHeight);
        
        height = 1 + max(leftHeight, rightHeight);
        
        if (abs(leftHeight - rightHeight) > 1) return false;
        
        return leftBalanced && rightBalanced;
    }
    
    // Diameter of tree: O(n)
    int diameter(TreeNode* node, int& maxDiam) {
        if (!node) return 0;
        
        int leftHeight = diameter(node->left, maxDiam);
        int rightHeight = diameter(node->right, maxDiam);
        
        maxDiam = max(maxDiam, leftHeight + rightHeight);
        
        return 1 + max(leftHeight, rightHeight);
    }
    
    // Lowest Common Ancestor: O(n)
    TreeNode* LCA(TreeNode* node, int n1, int n2) {
        if (!node) return nullptr;
        
        if (node->data == n1 || node->data == n2) return node;
        
        TreeNode* left = LCA(node->left, n1, n2);
        TreeNode* right = LCA(node->right, n1, n2);
        
        if (left && right) return node;
        
        return left ? left : right;
    }
    
    // Print all paths from root to leaf
    void printPaths(TreeNode* node, vector<int>& path) {
        if (!node) return;
        
        path.push_back(node->data);
        
        if (!node->left && !node->right) {
            for (int val : path) {
                cout << val << " ";
            }
            cout << endl;
        }
        
        printPaths(node->left, path);
        printPaths(node->right, path);
        
        path.pop_back();
    }
    
    // Mirror tree: O(n)
    void mirror(TreeNode* node) {
        if (!node) return;
        
        swap(node->left, node->right);
        mirror(node->left);
        mirror(node->right);
    }
};
```

**Tree Traversal Visualization**:

```
Tree:
         1
       /   \
      2     3
     / \
    4   5

Inorder (Left-Root-Right):    4 2 5 1 3
Preorder (Root-Left-Right):   1 2 4 5 3
Postorder (Left-Right-Root):  4 5 2 3 1
Level Order (BFS):            1 2 3 4 5
```

**Binary Search Tree (BST)**:

```cpp
class BST {
private:
    TreeNode* root;
    
    // Insert helper: O(log n) average, O(n) worst
    TreeNode* insertHelper(TreeNode* node, int val) {
        if (!node) return new TreeNode(val);
        
        if (val < node->data) {
            node->left = insertHelper(node->left, val);
        } else if (val > node->data) {
            node->right = insertHelper(node->right, val);
        }
        
        return node;
    }
    
    // Search helper: O(log n) average, O(n) worst
    bool searchHelper(TreeNode* node, int val) {
        if (!node) return false;
        
        if (val == node->data) return true;
        
        if (val < node->data) {
            return searchHelper(node->left, val);
        } else {
            return searchHelper(node->right, val);
        }
    }
    
    // Find minimum node
    TreeNode* findMin(TreeNode* node) {
        while (node && node->left) {
            node = node->left;
        }
        return node;
    }
    
    // Delete helper: O(log n) average, O(n) worst
    TreeNode* deleteHelper(TreeNode* node, int val) {
        if (!node) return nullptr;
        
        if (val < node->data) {
            node->left = deleteHelper(node->left, val);
        } else if (val > node->data) {
            node->right = deleteHelper(node->right, val);
        } else {
            // Node to be deleted found
            
            // Case 1: No child or one child
            if (!node->left) {
                TreeNode* temp = node->right;
                delete node;
                return temp;
            } else if (!node->right) {
                TreeNode* temp = node->left;
                delete node;
                return temp;
            }
            
            // Case 2: Two children
            TreeNode* temp = findMin(node->right);
            node->data = temp->data;
            node->right = deleteHelper(node->right, temp->data);
        }
        
        return node;
    }
    
public:
    BST() : root(nullptr) {}
    
    void insert(int val) {
        root = insertHelper(root, val);
    }
    
    bool search(int val) {
        return searchHelper(root, val);
    }
    
    void remove(int val) {
        root = deleteHelper(root, val);
    }
    
    // Validate BST: O(n)
    bool isValidBST(TreeNode* node, long minVal, long maxVal) {
        if (!node) return true;
        
        if (node->data <= minVal || node->data >= maxVal) {
            return false;
        }
        
        return isValidBST(node->left, minVal, node->data) &&
               isValidBST(node->right, node->data, maxVal);
    }
    
    // Kth smallest element: O(n)
    int kthSmallest(TreeNode* node, int& k) {
        if (!node) return -1;
        
        int left = kthSmallest(node->left, k);
        if (k == 0) return left;
        
        k--;
        if (k == 0) return node->data;
        
        return kthSmallest(node->right, k);
    }
    
    // Inorder successor: O(h)
    TreeNode* inorderSuccessor(TreeNode* root, TreeNode* node) {
        if (node->right) {
            return findMin(node->right);
        }
        
        TreeNode* successor = nullptr;
        TreeNode* current = root;
        
        while (current) {
            if (node->data < current->data) {
                successor = current;
                current = current->left;
            } else if (node->data > current->data) {
                current = current->right;
            } else {
                break;
            }
        }
        
        return successor;
    }
};
```

**BST Operations Visualization**:

```
Insert 5, 3, 7, 1, 4, 6, 8

Step 1: Insert 5
    5

Step 2: Insert 3
    5
   /
  3

Step 3: Insert 7
    5
   / \
  3   7

Step 4: Insert 1
    5
   / \
  3   7
 /
1

Step 5: Insert 4
    5
   / \
  3   7
 / \
1   4

Step 6-7: Insert 6, 8
    5
   / \
  3   7
 / \ / \
1  4 6  8

Delete Node 3 (has two children):
    5
   / \
  4   7
 /   / \
1   6   8
```

---

## Advanced Data Structures

### 7. Heaps

**Concept**: Complete binary tree where parent nodes have priority over children.

```
Max Heap                    Min Heap
    100                        10
   /   \                      /   \
  90    80                  20    30
 / \   / \                 / \   / \
50 70 60 40              40 50 60 70

Property:
Max Heap: Parent ≥ Children
Min Heap: Parent ≤ Children
```

**Array Representation**:

```
Max Heap: [100, 90, 80, 50, 70, 60, 40]
Index:      0   1   2   3   4   5   6

For node at index i:
- Left child:  2*i + 1
- Right child: 2*i + 2
- Parent:      (i-1) / 2
```

**Implementation**:

```cpp
#include <iostream>
#include <vector>
using namespace std;

class MaxHeap {
private:
    vector<int> heap;
    
    int parent(int i) { return (i - 1) / 2; }
    int leftChild(int i) { return 2 * i + 1; }
    int rightChild(int i) { return 2 * i + 2; }
    
    // Heapify up: O(log n)
    void heapifyUp(int index) {
        while (index > 0 && heap[parent(index)] < heap[index]) {
            swap(heap[parent(index)], heap[index]);
            index = parent(index);
        }
    }
    
    // Heapify down: O(log n)
    void heapifyDown(int index) {
        int maxIndex = index;
        int left = leftChild(index);
        int right = rightChild(index);
        
        if (left < heap.size() && heap[left] > heap[maxIndex]) {
            maxIndex = left;
        }
        
        if (right < heap.size() && heap[right] > heap[maxIndex]) {
            maxIndex = right;
        }
        
        if (index != maxIndex) {
            swap(heap[index], heap[maxIndex]);
            heapifyDown(maxIndex);
        }
    }
    
public:
    // Insert: O(log n)
    void insert(int val) {
        heap.push_back(val);
        heapifyUp(heap.size() - 1);
    }
    
    // Extract max: O(log n)
    int extractMax() {
        if (heap.empty()) {
            throw runtime_error("Heap is empty");
        }
        
        int maxVal = heap[0];
        heap[0] = heap.back();
        heap.pop_back();
        
        if (!heap.empty()) {
            heapifyDown(0);
        }
        
        return maxVal;
    }
    
    // Get max: O(1)
    int getMax() {
        if (heap.empty()) {
            throw runtime_error("Heap is empty");
        }
        return heap[0];
    }
    
    // Build heap from array: O(n)
    void buildHeap(vector<int>& arr) {
        heap = arr;
        for (int i = heap.size() / 2 - 1; i >= 0; i--) {
            heapifyDown(i);
        }
    }
    
    bool isEmpty() { return heap.empty(); }
    int size() { return heap.size(); }
    
    void display() {
        for (int val : heap) {
            cout << val << " ";
        }
        cout << endl;
    }
};

// Heap Applications
class HeapProblems {
public:
    // 1. Kth Largest Element: O(n log k)
    int findKthLargest(vector<int>& nums, int k) {
        priority_queue<int, vector<int>, greater<int>> minHeap;
        
        for (int num : nums) {
            minHeap.push(num);
            if (minHeap.size() > k) {
                minHeap.pop();
            }
        }
        
        return minHeap.top();
    }
    
    // 2. Merge K Sorted Arrays: O(n log k)
    vector<int> mergeKSortedArrays(vector<vector<int>>& arrays) {
        priority_queue<pair<int, pair<int, int>>, 
                      vector<pair<int, pair<int, int>>>,
                      greater<pair<int, pair<int, int>>>> minHeap;
        
        // Insert first element of each array
        for (int i = 0; i < arrays.size(); i++) {
            if (!arrays[i].empty()) {
                minHeap.push({arrays[i][0], {i, 0}});
            }
        }
        
        vector<int> result;
        
        while (!minHeap.empty()) {
            auto top = minHeap.top();
            minHeap.pop();
            
            int val = top.first;
            int arrayIdx = top.second.first;
            int elemIdx = top.second.second;
            
            result.push_back(val);
            
            // Add next element from same array
            if (elemIdx + 1 < arrays[arrayIdx].size()) {
                minHeap.push({arrays[arrayIdx][elemIdx + 1], 
                             {arrayIdx, elemIdx + 1}});
            }
        }
        
        return result;
    }
    
    // 3. Top K Frequent Elements: O(n log k)
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> freq;
        for (int num : nums) {
            freq[num]++;
        }
        
        priority_queue<pair<int, int>, 
                      vector<pair<int, int>>,
                      greater<pair<int, int>>> minHeap;
        
        for (auto& p : freq) {
            minHeap.push({p.second, p.first});
            if (minHeap.size() > k) {
                minHeap.pop();
            }
        }
        
        vector<int> result;
        while (!minHeap.empty()) {
            result.push_back(minHeap.top().second);
            minHeap.pop();
        }
        
        return result;
    }
    
    // 4. Median from Data Stream
    class MedianFinder {
    private:
        priority_queue<int> maxHeap;  // Lower half
        priority_queue<int, vector<int>, greater<int>> minHeap;  // Upper half
        
    public:
        void addNum(int num) {
            if (maxHeap.empty() || num <= maxHeap.top()) {
                maxHeap.push(num);
            } else {
                minHeap.push(num);
            }
            
            // Balance heaps
            if (maxHeap.size() > minHeap.size() + 1) {
                minHeap.push(maxHeap.top());
                maxHeap.pop();
            } else if (minHeap.size() > maxHeap.size()) {
                maxHeap.push(minHeap.top());
                minHeap.pop();
            }
        }
        
        double findMedian() {
            if (maxHeap.size() == minHeap.size()) {
                return (maxHeap.top() + minHeap.top()) / 2.0;
            }
            return maxHeap.top();
        }
    };
};
```

**Heapify Process Visualization**:

```
Build Max Heap from [3, 9, 2, 1, 4, 5]

Initial Array:
    3
   / \
  9   2
 / \ /
1  4 5

Step 1: Heapify at index 2 (value 2)
    3
   / \
  9   5
 / \ /
1  4 2

Step 2: Heapify at index 1 (value 9)
    3
   / \
  9   5
 / \ /
1  4 2

Step 3: Heapify at index 0 (value 3)
    9
   / \
  4   5
 / \ /
1  3 2

Final Max Heap: [9, 4, 5, 1, 3, 2]
```

---

### 8. Tries (Prefix Trees)

**Concept**: Tree structure for storing strings, efficient for prefix searches.

```
Trie Structure for words: "cat", "car", "dog"

        root
       / | \
      c  d  ...
     /    \
    a      o
   / \      \
  t   r      g
  *   *      *
(end)(end)  (end)

* indicates end of word
```

**Implementation**:

```cpp
#include <iostream>
#include <unordered_map>
using namespace std;

class TrieNode {
public:
    unordered_map<char, TrieNode*> children;
    bool isEndOfWord;
    
    TrieNode() : isEndOfWord(false) {}
};

class Trie {
private:
    TrieNode* root;
    
public:
    Trie() {
        root = new TrieNode();
    }
    
    // Insert word: O(m) where m is word length
    void insert(string word) {
        TrieNode* node = root;
        
        for (char ch : word) {
            if (node->children.find(ch) == node->children.end()) {
                node->children[ch] = new TrieNode();
            }
            node = node->children[ch];
        }
        
        node->isEndOfWord = true;
    }
    
    // Search word: O(m)
    bool search(string word) {
        TrieNode* node = root;
        
        for (char ch : word) {
            if (node->children.find(ch) == node->children.end()) {
                return false;
            }
            node = node->children[ch];
        }
        
        return node->isEndOfWord;
    }
    
    // Check if prefix exists: O(m)
    bool startsWith(string prefix) {
        TrieNode* node = root;
        
        for (char ch : prefix) {
            if (node->children.find(ch) == node->children.end()) {
                return false;
            }
            node = node->children[ch];
        }
        
        return true;
    }
    
    // Delete word: O(m)
    bool deleteHelper(TrieNode* node, string word, int index) {
        if (index == word.length()) {
            if (!node->isEndOfWord) return false;
            
            node->isEndOfWord = false;
            return node->children.empty();
        }
        
        char ch = word[index];
        if (node->children.find(ch) == node->children.end()) {
            return false;
        }
        
        TrieNode* childNode = node->children[ch];
        bool shouldDeleteChild = deleteHelper(childNode, word, index + 1);
        
        if (shouldDeleteChild) {
            node->children.erase(ch);
            return !node->isEndOfWord && node->children.empty();
        }
        
        return false;
    }
    
    void deleteWord(string word) {
        deleteHelper(root, word, 0);
    }
    
    // Auto-complete suggestions
    void findWordsHelper(TrieNode* node, string prefix, vector<string>& results) {
        if (node->isEndOfWord) {
            results.push_back(prefix);
        }
        
        for (auto& pair : node->children) {
            findWordsHelper(pair.second, prefix + pair.first, results);
        }
    }
    
    vector<string> autoComplete(string prefix) {
        vector<string> results;
        TrieNode* node = root;
        
        // Navigate to prefix end
        for (char ch : prefix) {
            if (node->children.find(ch) == node->children.end()) {
                return results;
            }
            node = node->children[ch];
        }
        
        // Find all words with this prefix
        findWordsHelper(node, prefix, results);
        return results;
    }
    
    // Longest common prefix
    string longestCommonPrefix(vector<string>& strs) {
        if (strs.empty()) return "";
        
        // Insert all strings
        for (string& s : strs) {
            insert(s);
        }
        
        string result = "";
        TrieNode* node = root;
        
        while (node && node->children.size() == 1 && !node->isEndOfWord) {
            auto it = node->children.begin();
            result += it->first;
            node = it->second;
        }
        
        return result;
    }
};
```

---

### 9. Graphs

**Concept**: Non-linear data structure consisting of vertices (nodes) and edges.

```
Graph Representations:

1. Adjacency Matrix
     0  1  2  3
  0 [0, 1, 1, 0]
  1 [1, 0, 0, 1]
  2 [1, 0, 0, 1]
  3 [0, 1, 1, 0]

2. Adjacency List
  0 -> [1, 2]
  1 -> [0, 3]
  2 -> [0, 3]
  3 -> [1, 2]

Graph Types:
- Directed vs Undirected
- Weighted vs Unweighted
- Cyclic vs Acyclic (DAG)
- Connected vs Disconnected
```

**Implementation**:

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <stack>
#include <unordered_map>
#include <unordered_set>
using namespace std;

// Adjacency List Representation
class Graph {
private:
    int V;  // Number of vertices
    vector<vector<int>> adjList;
    bool isDirected;
    
public:
    Graph(int vertices, bool directed = false) 
        : V(vertices), isDirected(directed) {
        adjList.resize(V);
    }
    
    // Add edge: O(1)
    void addEdge(int u, int v) {
        adjList[u].push_back(v);
        if (!isDirected) {
            adjList[v].push_back(u);
        }
    }
    
    // BFS Traversal: O(V + E)
    void BFS(int start) {
        vector<bool> visited(V, false);
        queue<int> q;
        
        visited[start] = true;
        q.push(start);
        
        cout << "BFS: ";
        while (!q.empty()) {
            int vertex = q.front();
            q.pop();
            cout << vertex << " ";
            
            for (int neighbor : adjList[vertex]) {
                if (!visited[neighbor]) {
                    visited[neighbor] = true;
                    q.push(neighbor);
                }
            }
        }
        cout << endl;
    }
    
    // DFS Traversal (Iterative): O(V + E)
    void DFS(int start) {
        vector<bool> visited(V, false);
        stack<int> st;
        
        st.push(start);
        
        cout << "DFS: ";
        while (!st.empty()) {
            int vertex = st.top();
            st.pop();
            
            if (!visited[vertex]) {
                visited[vertex] = true;
                cout << vertex << " ";
                
                // Add neighbors in reverse order for correct DFS
                for (auto it = adjList[vertex].rbegin(); 
                     it != adjList[vertex].rend(); ++it) {
                    if (!visited[*it]) {
                        st.push(*it);
                    }
                }
            }
        }
        cout << endl;
    }
    
    // DFS Recursive
    void DFSRecursiveHelper(int vertex, vector<bool>& visited) {
        visited[vertex] = true;
        cout << vertex << " ";
        
        for (int neighbor : adjList[vertex]) {
            if (!visited[neighbor]) {
                DFSRecursiveHelper(neighbor, visited);
            }
        }
    }
    
    void DFSRecursive(int start) {
        vector<bool> visited(V, false);
        cout << "DFS Recursive: ";
        DFSRecursiveHelper(start, visited);
        cout << endl;
    }
    
    // Detect Cycle in Undirected Graph: O(V + E)
    bool hasCycleUndirected() {
        vector<bool> visited(V, false);
        
        for (int i = 0; i < V; i++) {
            if (!visited[i]) {
                if (hasCycleUtil(i, visited, -1)) {
                    return true;
                }
            }
        }
        return false;
    }
    
    bool hasCycleUtil(int v, vector<bool>& visited, int parent) {
        visited[v] = true;
        
        for (int neighbor : adjList[v]) {
            if (!visited[neighbor]) {
                if (hasCycleUtil(neighbor, visited, v)) {
                    return true;
                }
            } else if (neighbor != parent) {
                return true;
            }
        }
        return false;
    }
    
    // Topological Sort (DFS-based): O(V + E)
    void topologicalSortUtil(int v, vector<bool>& visited, stack<int>& st) {
        visited[v] = true;
        
        for (int neighbor : adjList[v]) {
            if (!visited[neighbor]) {
                topologicalSortUtil(neighbor, visited, st);
            }
        }
        
        st.push(v);
    }
    
    void topologicalSort() {
        stack<int> st;
        vector<bool> visited(V, false);
        
        for (int i = 0; i < V; i++) {
            if (!visited[i]) {
                topologicalSortUtil(i, visited, st);
            }
        }
        
        cout << "Topological Sort: ";
        while (!st.empty()) {
            cout << st.top() << " ";
            st.pop();
        }
        cout << endl;
    }
    
    // Shortest Path in Unweighted Graph (BFS): O(V + E)
    void shortestPath(int start, int end) {
        vector<int> distance(V, -1);
        vector<int> parent(V, -1);
        queue<int> q;
        
        distance[start] = 0;
        q.push(start);
        
        while (!q.empty()) {
            int vertex = q.front();
            q.pop();
            
            if (vertex == end) break;
            
            for (int neighbor : adjList[vertex]) {
                if (distance[neighbor] == -1) {
                    distance[neighbor] = distance[vertex] + 1;
                    parent[neighbor] = vertex;
                    q.push(neighbor);
                }
            }
        }
        
        if (distance[end] == -1) {
            cout << "No path exists" << endl;
            return;
        }
        
        // Reconstruct path
        vector<int> path;
        int current = end;
        while (current != -1) {
            path.push_back(current);
            current = parent[current];
        }
        
        reverse(path.begin(), path.end());
        
        cout << "Shortest path from " << start << " to " << end << ": ";
        for (int v : path) {
            cout << v << " ";
        }
        cout << "(Distance: " << distance[end] << ")" << endl;
    }
    
    // Connected Components: O(V + E)
    int countConnectedComponents() {
        vector<bool> visited(V, false);
        int count = 0;
        
        for (int i = 0; i < V; i++) {
            if (!visited[i]) {
                DFSRecursiveHelper(i, visited);
                count++;
            }
        }
        
        return count;
    }
    
    void display() {
        for (int i = 0; i < V; i++) {
            cout << i << " -> ";
            for (int neighbor : adjList[i]) {
                cout << neighbor << " ";
            }
            cout << endl;
        }
    }
};

// Weighted Graph
class WeightedGraph {
private:
    int V;
    vector<vector<pair<int, int>>> adjList;  // {vertex, weight}
    
public:
    WeightedGraph(int vertices) : V(vertices) {
        adjList.resize(V);
    }
    
    void addEdge(int u, int v, int weight) {
        adjList[u].push_back({v, weight});
        adjList[v].push_back({u, weight});  // For undirected
    }
    
    // Dijkstra's Shortest Path: O((V + E) log V)
    vector<int> dijkstra(int start) {
        vector<int> dist(V, INT_MAX);
        priority_queue<pair<int, int>, 
                      vector<pair<int, int>>,
                      greater<pair<int, int>>> pq;
        
        dist[start] = 0;
        pq.push({0, start});
        
        while (!pq.empty()) {
            int u = pq.top().second;
            int d = pq.top().first;
            pq.pop();
            
            if (d > dist[u]) continue;
            
            for (auto& edge : adjList[u]) {
                int v = edge.first;
                int weight = edge.second;
                
                if (dist[u] + weight < dist[v]) {
                    dist[v] = dist[u] + weight;
                    pq.push({dist[v], v});
                }
            }
        }
        
        return dist;
    }
    
    // Bellman-Ford Algorithm: O(V * E)
    vector<int> bellmanFord(int start) {
        vector<int> dist(V, INT_MAX);
        dist[start] = 0;
        
        // Relax edges V-1 times
        for (int i = 0; i < V - 1; i++) {
            for (int u = 0; u < V; u++) {
                for (auto& edge : adjList[u]) {
                    int v = edge.first;
                    int weight = edge.second;
                    
                    if (dist[u] != INT_MAX && dist[u] + weight < dist[v]) {
                        dist[v] = dist[u] + weight;
                    }
                }
            }
        }
        
        // Check for negative cycles
        for (int u = 0; u < V; u++) {
            for (auto& edge : adjList[u]) {
                int v = edge.first;
                int weight = edge.second;
                
                if (dist[u] != INT_MAX && dist[u] + weight < dist[v]) {
                    cout << "Negative cycle detected!" << endl;
                    return {};
                }
            }
        }
        
        return dist;
    }
    
    // Prim's Minimum Spanning Tree: O(E log V)
    int primMST() {
        vector<bool> inMST(V, false);
        priority_queue<pair<int, int>,
                      vector<pair<int, int>>,
                      greater<pair<int, int>>> pq;
        
        int mstWeight = 0;
        pq.push({0, 0});  // {weight, vertex}
        
        while (!pq.empty()) {
            int u = pq.top().second;
            int weight = pq.top().first;
            pq.pop();
            
            if (inMST[u]) continue;
            
            inMST[u] = true;
            mstWeight += weight;
            
            for (auto& edge : adjList[u]) {
                int v = edge.first;
                int w = edge.second;
                
                if (!inMST[v]) {
                    pq.push({w, v});
                }
            }
        }
        
        return mstWeight;
    }
};
```

**Graph Traversal Visualization**:

```
Graph:
    0 --- 1
    |     |
    2 --- 3

BFS from 0: 0 -> 1 -> 2 -> 3
DFS from 0: 0 -> 1 -> 3 -> 2

Queue/Stack states:
BFS: [0] -> [1,2] -> [2,3] -> [3] -> []
DFS: [0] -> [1,2] -> [3,2] -> [2] -> []
```

---

## Sorting Algorithms

### Complete Sorting Implementation

```cpp
#include <iostream>
#include <vector>
using namespace std;

class SortingAlgorithms {
public:
    // 1. Bubble Sort: O(n²)
    void bubbleSort(vector<int>& arr) {
        int n = arr.size();
        for (int i = 0; i < n - 1; i++) {
            bool swapped = false;
            for (int j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    swap(arr[j], arr[j + 1]);
                    swapped = true;
                }
            }
            if (!swapped) break;  // Already sorted
        }
    }
    
    // 2. Selection Sort: O(n²)
    void selectionSort(vector<int>& arr) {
        int n = arr.size();
        for (int i = 0; i < n - 1; i++) {
            int minIdx = i;
            for (int j = i + 1; j < n; j++) {
                if (arr[j] < arr[minIdx]) {
                    minIdx = j;
                }
            }
            swap(arr[i], arr[minIdx]);
        }
    }
    
    // 3. Insertion Sort: O(n²)
    void insertionSort(vector<int>& arr) {
        int n = arr.size();
        for (int i = 1; i < n; i++) {
            int key = arr[i];
            int j = i - 1;
            
            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }
            arr[j + 1] = key;
        }
    }
    
    // 4. Merge Sort: O(n log n)
    void merge(vector<int>& arr, int left, int mid, int right) {
        int n1 = mid - left + 1;
        int n2 = right - mid;
        
        vector<int> L(n1), R(n2);
        
        for (int i = 0; i < n1; i++)
            L[i] = arr[left + i];
        for (int i = 0; i < n2; i++)
            R[i] = arr[mid + 1 + i];
        
        int i = 0, j = 0, k = left;
        
        while (i < n1 && j < n2) {
            if (L[i] <= R[j]) {
                arr[k++] = L[i++];
            } else {
                arr[k++] = R[j++];
            }
        }
        
        while (i < n1) arr[k++] = L[i++];
        while (j < n2) arr[k++] = R[j++];
    }
    
    void mergeSort(vector<int>& arr, int left, int right) {
        if (left < right) {
            int mid = left + (right - left) / 2;
            mergeSort(arr, left, mid);
            mergeSort(arr, mid + 1, right);
            merge(arr, left, mid, right);
        }
    }
    
    // 5. Quick Sort: O(n log n) average
    int partition(vector<int>& arr, int low, int high) {
        int pivot = arr[high];
        int i = low - 1;
        
        for (int j = low; j < high; j++) {
            if (arr[j] < pivot) {
                i++;
                swap(arr[i], arr[j]);
            }
        }
        swap(arr[i + 1], arr[high]);
        return i + 1;
    }
    
    void quickSort(vector<int>& arr, int low, int high) {
        if (low < high) {
            int pi = partition(arr, low, high);
            quickSort(arr, low, pi - 1);
            quickSort(arr, pi + 1, high);
        }
    }
    
    // 6. Heap Sort: O(n log n)
    void heapify(vector<int>& arr, int n, int i) {
        int largest = i;
        int left = 2 * i + 1;
        int right = 2 * i + 2;
        
        if (left < n && arr[left] > arr[largest])
            largest = left;
        
        if (right < n && arr[right] > arr[largest])
            largest = right;
        
        if (largest != i) {
            swap(arr[i], arr[largest]);
            heapify(arr, n, largest);
        }
    }
    
    void heapSort(vector<int>& arr) {
        int n = arr.size();
        
        // Build max heap
        for (int i = n / 2 - 1; i >= 0; i--)
            heapify(arr, n, i);
        
        // Extract elements
        for (int i = n - 1; i > 0; i--) {
            swap(arr[0], arr[i]);
            heapify(arr, i, 0);
        }
    }
    
    // 7. Counting Sort: O(n + k)
    void countingSort(vector<int>& arr) {
        if (arr.empty()) return;
        
        int maxVal = *max_element(arr.begin(), arr.end());
        int minVal = *min_element(arr.begin(), arr.end());
        int range = maxVal - minVal + 1;
        
        vector<int> count(range, 0);
        vector<int> output(arr.size());
        
        for (int num : arr)
            count[num - minVal]++;
        
        for (int i = 1; i < range; i++)
            count[i] += count[i - 1];
        
        for (int i = arr.size() - 1; i >= 0; i--) {
            output[count[arr[i] - minVal] - 1] = arr[i];
            count[arr[i] - minVal]--;
        }
        
        arr = output;
    }
    
    // 8. Radix Sort: O(d * (n + k))
    void countingSortForRadix(vector<int>& arr, int exp) {
        int n = arr.size();
        vector<int> output(n);
        vector<int> count(10, 0);
        
        for (int i = 0; i < n; i++)
            count[(arr[i] / exp) % 10]++;
        
        for (int i = 1; i < 10; i++)
            count[i] += count[i - 1];
        
        for (int i = n - 1; i >= 0; i--) {
            output[count[(arr[i] / exp) % 10] - 1] = arr[i];
            count[(arr[i] / exp) % 10]--;
        }
        
        arr = output;
    }
    
    void radixSort(vector<int>& arr) {
        if (arr.empty()) return;
        
        int maxVal = *max_element(arr.begin(), arr.end());
        
        for (int exp = 1; maxVal / exp > 0; exp *= 10)
            countingSortForRadix(arr, exp);
    }
};
```

---

## Dynamic Programming

### Classic DP Problems

```cpp
#include <iostream>
#include <vector>
#include <climits>
using namespace std;

class DynamicProgramming {
public:
    // 1. Fibonacci: O(n)
    int fibonacci(int n) {
        if (n <= 1) return n;
        
        vector<int> dp(n + 1);
        dp[0] = 0;
        dp[1] = 1;
        
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        
        return dp[n];
    }
    
    // 2. Longest Common Subsequence: O(m * n)
    int LCS(string s1, string s2) {
        int m = s1.length();
        int n = s2.length();
        
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
        
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1[i - 1] == s2[j - 1]) {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        
        return dp[m][n];
    }
    
    // 3. 0/1 Knapsack: O(n * W)
    int knapsack(vector<int>& weights, vector<int>& values, int W) {
        int n = weights.size();
        vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));
        
        for (int i = 1; i <= n; i++) {
            for (int w = 1; w <= W; w++) {
                if (weights[i - 1] <= w) {
                    dp[i][w] = max(values[i - 1] + dp[i - 1][w - weights[i - 1]],
                                  dp[i - 1][w]);
                } else {
                    dp[i][w] = dp[i - 1][w];
                }
            }
        }
        
        return dp[n][W];
    }
    
    // 4. Coin Change (Minimum coins): O(n * amount)
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, INT_MAX);
        dp[0] = 0;
        
        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (coin <= i && dp[i - coin] != INT_MAX) {
                    dp[i] = min(dp[i], dp[i - coin] + 1);
                }
            }
        }
        
        return dp[amount] == INT_MAX ? -1 : dp[amount];
    }
    
    // 5. Longest Increasing Subsequence: O(n²)
    int LIS(vector<int>& arr) {
        int n = arr.size();
        if (n == 0) return 0;
        
        vector<int> dp(n, 1);
        
        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (arr[j] < arr[i]) {
                    dp[i] = max(dp[i], dp[j] + 1);
                }
            }
        }
        
        return *max_element(dp.begin(), dp.end());
    }
    
    // 6. Edit Distance: O(m * n)
    int editDistance(string s1, string s2) {
        int m = s1.length();
        int n = s2.length();
        
        vector<vector<int>> dp(m + 1, vector<int>(n + 1));
        
        for (int i = 0; i <= m; i++)
            dp[i][0] = i;
        
        for (int j = 0; j <= n; j++)
            dp[0][j] = j;
        
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1[i - 1] == s2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = 1 + min({dp[i - 1][j],      // Delete
                                       dp[i][j - 1],        // Insert
                                       dp[i - 1][j - 1]});  // Replace
                }
            }
        }
        
        return dp[m][n];
    }
    
    // 7. Matrix Chain Multiplication: O(n³)
    int matrixChainMultiplication(vector<int>& dims) {
        int n = dims.size() - 1;
        vector<vector<int>> dp(n, vector<int>(n, 0));
        
        for (int len = 2; len <= n; len++) {
            for (int i = 0; i < n - len + 1; i++) {
                int j = i + len - 1;
                dp[i][j] = INT_MAX;
                
                for (int k = i; k < j; k++) {
                    int cost = dp[i][k] + dp[k + 1][j] + 
                              dims[i] * dims[k + 1] * dims[j + 1];
                    dp[i][j] = min(dp[i][j], cost);
                }
            }
        }
        
        return dp[0][n - 1];
    }
    
    // 8. Subset Sum: O(n * sum)
    bool subsetSum(vector<int>& arr, int sum) {
        int n = arr.size();
        vector<vector<bool>> dp(n + 1, vector<bool>(sum + 1, false));
        
        for (int i = 0; i <= n; i++)
            dp[i][0] = true;
        
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= sum; j++) {
                dp[i][j] = dp[i - 1][j];
                if (j >= arr[i - 1]) {
                    dp[i][j] = dp[i][j] || dp[i - 1][j - arr[i - 1]];
                }
            }
        }
        
        return dp[n][sum];
    }
};
```

---

## Problem-Solving Patterns

### 1. Two Pointers Pattern

```cpp
class TwoPointers {
public:
    // Remove duplicates from sorted array
    int removeDuplicates(vector<int>& arr) {
        if (arr.empty()) return 0;
        
        int slow = 0;
        for (int fast = 1; fast < arr.size(); fast++) {
            if (arr[fast] != arr[slow]) {
                slow++;
                arr[slow] = arr[fast];
            }
        }
        
        return slow + 1;
    }
    
    // Container with most water
    int maxArea(vector<int>& height) {
        int left = 0, right = height.size() - 1;
        int maxWater = 0;
        
        while (left < right) {
            int h = min(height[left], height[right]);
            int width = right - left;
            maxWater = max(maxWater, h * width);
            
            if (height[left] < height[right]) {
                left++;
            } else {
                right--;
            }
        }
        
        return maxWater;
    }
};
```

### 2. Sliding Window Pattern

```cpp
class SlidingWindow {
public:
    // Maximum sum subarray of size k
    int maxSumSubarray(vector<int>& arr, int k) {
        int n = arr.size();
        if (n < k) return -1;
        
        int windowSum = 0;
        for (int i = 0; i < k; i++)
            windowSum += arr[i];
        
        int maxSum = windowSum;
        
        for (int i = k; i < n; i++) {
            windowSum = windowSum - arr[i - k] + arr[i];
            maxSum = max(maxSum, windowSum);
        }
        
        return maxSum;
    }
    
    // Longest substring without repeating characters
    int lengthOfLongestSubstring(string s) {
        unordered_map<char, int> charIndex;
        int maxLen = 0;
        int start = 0;
        
        for (int end = 0; end < s.length(); end++) {
            if (charIndex.find(s[end]) != charIndex.end()) {
                start = max(start, charIndex[s[end]] + 1);
            }
            
            charIndex[s[end]] = end;
            maxLen = max(maxLen, end - start + 1);
        }
        
        return maxLen;
    }
};
```

### 3. Fast & Slow Pointers (Floyd's Algorithm)

```cpp
class FloydAlgorithm {
public:
    // Detect cycle in linked list
    bool hasCycle(Node* head) {
        if (!head) return false;
        
        Node* slow = head;
        Node* fast = head;
        
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
            
            if (slow == fast) return true;
        }
        
        return false;
    }
    
    // Find cycle start
    Node* detectCycleStart(Node* head) {
        Node* slow = head;
        Node* fast = head;
        
        while (fast && fast->next) {
            slow = slow->next;
            fast = fast->next->next;
            
            if (slow == fast) {
                slow = head;
                while (slow != fast) {
                    slow = slow->next;
                    fast = fast->next;
                }
                return slow;
            }
        }
        
        return nullptr;
    }
};
```

---

## Practice Problems and Solutions

### LeetCode Essential Problems by Topic

**Arrays:**
1. Two Sum (Easy)
2. Best Time to Buy and Sell Stock (Easy)
3. Contains Duplicate (Easy)
4. Maximum Subarray (Easy)
5. Merge Intervals (Medium)
6. 3Sum (Medium)
7. Product of Array Except Self (Medium)

**Strings:**
1. Valid Anagram (Easy)
2. Valid Parentheses (Easy)
3. Longest Substring Without Repeating Characters (Medium)
4. Longest Palindromic Substring (Medium)
5. Group Anagrams (Medium)

**Linked Lists:**
1. Reverse Linked List (Easy)
2. Merge Two Sorted Lists (Easy)
3. Linked List Cycle (Easy)
4. Remove Nth Node From End (Medium)
5. Reorder List (Medium)

**Trees:**
1. Maximum Depth of Binary Tree (Easy)
2. Same Tree (Easy)
3. Invert Binary Tree (Easy)
4. Binary Tree Level Order Traversal (Medium)
5. Validate Binary Search Tree (Medium)
6. Lowest Common Ancestor (Medium)

**Dynamic Programming:**
1. Climbing Stairs (Easy)
2. House Robber (Medium)
3. Coin Change (Medium)
4. Longest Increasing Subsequence (Medium)
5. Word Break (Medium)

**Graphs:**
1. Number of Islands (Medium)
2. Clone Graph (Medium)
3. Course Schedule (Medium)
4. Pacific Atlantic Water Flow (Medium)
5. Graph Valid Tree (Medium)

---

## Conclusion

This comprehensive tutorial covers all essential data structures and algorithms needed for:
- Technical interviews (FAANG companies)
- Competitive programming
- Software development
- Academic coursework

### Study Plan

**Week 1-2:** Arrays, Strings, Basic Operations
**Week 3-4:** Linked Lists, Stacks, Queues
**Week 5-6:** Trees, Binary Search Trees
**Week 7-8:** Heaps, Hash Tables, Tries
**Week 9-10:** Graphs, BFS, DFS
**Week 11-12:** Sorting Algorithms
**Week 13-14:** Dynamic Programming
**Week 15-16:** Advanced Algorithms & Problem Solving

### Resources

1. **Books:**
   - "Introduction to Algorithms" (CLRS)
   - "Cracking the Coding Interview"
   - "Elements of Programming Interviews"

2. **Online Platforms:**
   - LeetCode
   - HackerRank
   - Codeforces
   - GeeksforGeeks

3. **Practice:**
   - Solve at least 200+ problems
   - Focus on understanding patterns
   - Write code from scratch
   - Analyze time and space complexity

### Final Tips

1. **Understand, don't memorize** - Focus on problem-solving patterns
2. **Practice regularly** - Consistency is key
3. **Time yourself** - Practice under interview conditions
4. **Explain your approach** - Verbalize your thought process
5. **Optimize** - Always look for better solutions
6. **Test thoroughly** - Consider edge cases
7. **Learn from mistakes** - Review failed attempts

**Good luck on your DSA journey!** 🚀

---

*This tutorial is comprehensive and covers all essential topics. Keep practicing and never stop learning!*