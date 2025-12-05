# The Complete C Programming Master Roadmap - PART 4
## Advanced Graph Algorithms & Systems Programming

---

## Table of Contents - Part 4
1. [Advanced Graph Algorithms](#1-advanced-graph-algorithms)
2. [Greedy Algorithms](#2-greedy-algorithms)
3. [Bit Manipulation](#3-bit-manipulation)
4. [File I/O](#4-file-io)
5. [Process Management](#5-process-management)
6. [Inter-Process Communication (IPC)](#6-inter-process-communication)

---

## 1. Advanced Graph Algorithms

### 1.1 Dijkstra's Shortest Path Algorithm

**Algorithm Logic:**
```
Find shortest path from source to all vertices in weighted graph

Process:
1. Initialize distances: source = 0, all others = ∞
2. Select unvisited vertex with minimum distance
3. Update distances to neighbors
4. Mark vertex as visited
5. Repeat until all visited

Time Complexity: O((V + E) log V) with min-heap
```

**Example:**
```c
#include <stdio.h>
#include <limits.h>
#include <stdbool.h>

#define V 9  // Number of vertices

int min_distance(int dist[], bool visited[]) {
    int min = INT_MAX, min_index;
    
    for (int v = 0; v < V; v++) {
        if (!visited[v] && dist[v] <= min) {
            min = dist[v];
            min_index = v;
        }
    }
    
    return min_index;
}

void print_path(int parent[], int j) {
    if (parent[j] == -1) {
        printf("%d ", j);
        return;
    }
    
    print_path(parent, parent[j]);
    printf("%d ", j);
}

void dijkstra(int graph[V][V], int src) {
    int dist[V];      // Shortest distance from source
    bool visited[V];  // Visited vertices
    int parent[V];    // Store path
    
    // Initialize
    for (int i = 0; i < V; i++) {
        dist[i] = INT_MAX;
        visited[i] = false;
        parent[i] = -1;
    }
    
    dist[src] = 0;
    
    // Find shortest path for all vertices
    for (int count = 0; count < V - 1; count++) {
        int u = min_distance(dist, visited);
        visited[u] = true;
        
        // Update distances of adjacent vertices
        for (int v = 0; v < V; v++) {
            if (!visited[v] && graph[u][v] && 
                dist[u] != INT_MAX && 
                dist[u] + graph[u][v] < dist[v]) {
                dist[v] = dist[u] + graph[u][v];
                parent[v] = u;
            }
        }
    }
    
    // Print results
    printf("Vertex\tDistance\tPath\n");
    for (int i = 0; i < V; i++) {
        printf("%d\t%d\t\t", i, dist[i]);
        print_path(parent, i);
        printf("\n");
    }
}

int main() {
    int graph[V][V] = {
        {0, 4, 0, 0, 0, 0, 0, 8, 0},
        {4, 0, 8, 0, 0, 0, 0, 11, 0},
        {0, 8, 0, 7, 0, 4, 0, 0, 2},
        {0, 0, 7, 0, 9, 14, 0, 0, 0},
        {0, 0, 0, 9, 0, 10, 0, 0, 0},
        {0, 0, 4, 14, 10, 0, 2, 0, 0},
        {0, 0, 0, 0, 0, 2, 0, 1, 6},
        {8, 11, 0, 0, 0, 0, 1, 0, 7},
        {0, 0, 2, 0, 0, 0, 6, 7, 0}
    };
    
    dijkstra(graph, 0);
    
    return 0;
}
```

### 1.2 Bellman-Ford Algorithm

**Algorithm Logic:**
```
Handles negative edge weights
Detects negative cycles

Process:
1. Initialize distances: source = 0, others = ∞
2. Relax all edges V-1 times
3. Check for negative cycles

Time Complexity: O(VE)
```

**Example:**
```c
#include <stdio.h>
#include <limits.h>
#include <stdlib.h>

typedef struct Edge {
    int src, dest, weight;
} Edge;

typedef struct Graph {
    int V, E;
    Edge *edges;
} Graph;

Graph* create_graph(int V, int E) {
    Graph *graph = malloc(sizeof(Graph));
    graph->V = V;
    graph->E = E;
    graph->edges = malloc(E * sizeof(Edge));
    return graph;
}

void bellman_ford(Graph *graph, int src) {
    int V = graph->V;
    int E = graph->E;
    int dist[V];
    
    // Initialize distances
    for (int i = 0; i < V; i++) {
        dist[i] = INT_MAX;
    }
    dist[src] = 0;
    
    // Relax edges V-1 times
    for (int i = 1; i <= V - 1; i++) {
        for (int j = 0; j < E; j++) {
            int u = graph->edges[j].src;
            int v = graph->edges[j].dest;
            int weight = graph->edges[j].weight;
            
            if (dist[u] != INT_MAX && dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
            }
        }
    }
    
    // Check for negative cycles
    for (int i = 0; i < E; i++) {
        int u = graph->edges[i].src;
        int v = graph->edges[i].dest;
        int weight = graph->edges[i].weight;
        
        if (dist[u] != INT_MAX && dist[u] + weight < dist[v]) {
            printf("Graph contains negative cycle\n");
            return;
        }
    }
    
    // Print distances
    printf("Vertex\tDistance from Source\n");
    for (int i = 0; i < V; i++) {
        printf("%d\t%d\n", i, dist[i]);
    }
}

int main() {
    int V = 5;
    int E = 8;
    Graph *graph = create_graph(V, E);
    
    // Add edges
    graph->edges[0] = (Edge){0, 1, -1};
    graph->edges[1] = (Edge){0, 2, 4};
    graph->edges[2] = (Edge){1, 2, 3};
    graph->edges[3] = (Edge){1, 3, 2};
    graph->edges[4] = (Edge){1, 4, 2};
    graph->edges[5] = (Edge){3, 2, 5};
    graph->edges[6] = (Edge){3, 1, 1};
    graph->edges[7] = (Edge){4, 3, -3};
    
    bellman_ford(graph, 0);
    
    free(graph->edges);
    free(graph);
    
    return 0;
}
```

### 1.3 Floyd-Warshall Algorithm (All Pairs Shortest Path)

**Algorithm Logic:**
```
Find shortest paths between all pairs of vertices

dp[i][j][k] = shortest path from i to j using vertices 0..k

Time Complexity: O(V³)
Space Complexity: O(V²)
```

**Example:**
```c
#include <stdio.h>
#include <limits.h>

#define V 4
#define INF 99999

void print_solution(int dist[][V]) {
    printf("Shortest distances between every pair of vertices:\n");
    printf("     ");
    for (int i = 0; i < V; i++) {
        printf("%4d", i);
    }
    printf("\n");
    
    for (int i = 0; i < V; i++) {
        printf("%4d ", i);
        for (int j = 0; j < V; j++) {
            if (dist[i][j] == INF) {
                printf(" INF");
            } else {
                printf("%4d", dist[i][j]);
            }
        }
        printf("\n");
    }
}

void floyd_warshall(int graph[][V]) {
    int dist[V][V];
    
    // Initialize solution matrix
    for (int i = 0; i < V; i++) {
        for (int j = 0; j < V; j++) {
            dist[i][j] = graph[i][j];
        }
    }
    
    // Floyd-Warshall algorithm
    for (int k = 0; k < V; k++) {
        for (int i = 0; i < V; i++) {
            for (int j = 0; j < V; j++) {
                if (dist[i][k] != INF && dist[k][j] != INF &&
                    dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    
    print_solution(dist);
}

int main() {
    int graph[V][V] = {
        {0, 5, INF, 10},
        {INF, 0, 3, INF},
        {INF, INF, 0, 1},
        {INF, INF, INF, 0}
    };
    
    floyd_warshall(graph);
    
    return 0;
}
```

### 1.4 Minimum Spanning Tree - Prim's Algorithm

**Algorithm Logic:**
```
Find minimum spanning tree (connects all vertices with minimum total edge weight)

Time Complexity: O(V²) or O(E log V) with heap
```

**Example:**
```c
#include <stdio.h>
#include <limits.h>
#include <stdbool.h>

#define V 5

int min_key(int key[], bool mst_set[]) {
    int min = INT_MAX, min_index;
    
    for (int v = 0; v < V; v++) {
        if (!mst_set[v] && key[v] < min) {
            min = key[v];
            min_index = v;
        }
    }
    
    return min_index;
}

void print_mst(int parent[], int graph[V][V]) {
    printf("Edge\tWeight\n");
    int total_weight = 0;
    
    for (int i = 1; i < V; i++) {
        printf("%d - %d\t%d\n", parent[i], i, graph[i][parent[i]]);
        total_weight += graph[i][parent[i]];
    }
    
    printf("Total weight: %d\n", total_weight);
}

void prim_mst(int graph[V][V]) {
    int parent[V];     // Store MST
    int key[V];        // Key values to pick minimum weight edge
    bool mst_set[V];   // Vertices included in MST
    
    // Initialize
    for (int i = 0; i < V; i++) {
        key[i] = INT_MAX;
        mst_set[i] = false;
    }
    
    key[0] = 0;
    parent[0] = -1;
    
    for (int count = 0; count < V - 1; count++) {
        int u = min_key(key, mst_set);
        mst_set[u] = true;
        
        // Update key values of adjacent vertices
        for (int v = 0; v < V; v++) {
            if (graph[u][v] && !mst_set[v] && graph[u][v] < key[v]) {
                parent[v] = u;
                key[v] = graph[u][v];
            }
        }
    }
    
    print_mst(parent, graph);
}

int main() {
    int graph[V][V] = {
        {0, 2, 0, 6, 0},
        {2, 0, 3, 8, 5},
        {0, 3, 0, 0, 7},
        {6, 8, 0, 0, 9},
        {0, 5, 7, 9, 0}
    };
    
    prim_mst(graph);
    
    return 0;
}
```

---

## 2. Greedy Algorithms

### 2.1 Activity Selection Problem

**Algorithm Logic:**
```
Select maximum number of activities that don't overlap

Strategy: Always pick activity that finishes first

Example:
Activities: (start, finish)
(1,4), (3,5), (0,6), (5,7), (3,9), (5,9), (6,10), (8,11), (8,12), (2,14)

Selected: (1,4), (5,7), (8,11)
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Activity {
    int start;
    int finish;
    int index;
} Activity;

int compare(const void *a, const void *b) {
    Activity *act1 = (Activity *)a;
    Activity *act2 = (Activity *)b;
    return act1->finish - act2->finish;
}

void activity_selection(Activity activities[], int n) {
    // Sort by finish time
    qsort(activities, n, sizeof(Activity), compare);
    
    printf("Selected activities: ");
    printf("A%d ", activities[0].index);
    
    int last_finish = activities[0].finish;
    
    for (int i = 1; i < n; i++) {
        if (activities[i].start >= last_finish) {
            printf("A%d ", activities[i].index);
            last_finish = activities[i].finish;
        }
    }
    printf("\n");
}

int main() {
    Activity activities[] = {
        {1, 4, 1}, {3, 5, 2}, {0, 6, 3}, {5, 7, 4},
        {3, 9, 5}, {5, 9, 6}, {6, 10, 7}, {8, 11, 8},
        {8, 12, 9}, {2, 14, 10}
    };
    
    int n = sizeof(activities) / sizeof(activities[0]);
    activity_selection(activities, n);
    
    return 0;
}
```

### 2.2 Huffman Coding

**Algorithm Logic:**
```
Optimal prefix-free encoding for data compression

Process:
1. Count frequency of each character
2. Build min-heap with frequencies
3. Extract two minimum nodes, create parent
4. Repeat until one node remains (root)

Example:
Text: "aabbbcccc"
Frequencies: a=2, b=3, c=4

Huffman Tree:
        9
       / \
      4   5
     (c) / \
        2   3
       (a) (b)

Codes: a=10, b=11, c=0
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct MinHeapNode {
    char data;
    unsigned freq;
    struct MinHeapNode *left, *right;
} MinHeapNode;

typedef struct MinHeap {
    unsigned size;
    unsigned capacity;
    MinHeapNode **array;
} MinHeap;

MinHeapNode* create_node(char data, unsigned freq) {
    MinHeapNode *node = malloc(sizeof(MinHeapNode));
    node->data = data;
    node->freq = freq;
    node->left = node->right = NULL;
    return node;
}

MinHeap* create_min_heap(unsigned capacity) {
    MinHeap *heap = malloc(sizeof(MinHeap));
    heap->size = 0;
    heap->capacity = capacity;
    heap->array = malloc(capacity * sizeof(MinHeapNode*));
    return heap;
}

void swap_nodes(MinHeapNode **a, MinHeapNode **b) {
    MinHeapNode *temp = *a;
    *a = *b;
    *b = temp;
}

void heapify(MinHeap *heap, int idx) {
    int smallest = idx;
    int left = 2 * idx + 1;
    int right = 2 * idx + 2;
    
    if (left < heap->size && 
        heap->array[left]->freq < heap->array[smallest]->freq) {
        smallest = left;
    }
    
    if (right < heap->size && 
        heap->array[right]->freq < heap->array[smallest]->freq) {
        smallest = right;
    }
    
    if (smallest != idx) {
        swap_nodes(&heap->array[smallest], &heap->array[idx]);
        heapify(heap, smallest);
    }
}

MinHeapNode* extract_min(MinHeap *heap) {
    MinHeapNode *temp = heap->array[0];
    heap->array[0] = heap->array[heap->size - 1];
    heap->size--;
    heapify(heap, 0);
    return temp;
}

void insert_min_heap(MinHeap *heap, MinHeapNode *node) {
    heap->size++;
    int i = heap->size - 1;
    
    while (i && node->freq < heap->array[(i - 1) / 2]->freq) {
        heap->array[i] = heap->array[(i - 1) / 2];
        i = (i - 1) / 2;
    }
    
    heap->array[i] = node;
}

void build_min_heap(MinHeap *heap) {
    int n = heap->size - 1;
    for (int i = (n - 1) / 2; i >= 0; i--) {
        heapify(heap, i);
    }
}

MinHeapNode* build_huffman_tree(char data[], int freq[], int size) {
    MinHeap *heap = create_min_heap(size);
    
    for (int i = 0; i < size; i++) {
        heap->array[i] = create_node(data[i], freq[i]);
    }
    heap->size = size;
    build_min_heap(heap);
    
    while (heap->size != 1) {
        MinHeapNode *left = extract_min(heap);
        MinHeapNode *right = extract_min(heap);
        
        MinHeapNode *top = create_node('$', left->freq + right->freq);
        top->left = left;
        top->right = right;
        
        insert_min_heap(heap, top);
    }
    
    return extract_min(heap);
}

void print_codes(MinHeapNode *root, int arr[], int top) {
    if (root->left) {
        arr[top] = 0;
        print_codes(root->left, arr, top + 1);
    }
    
    if (root->right) {
        arr[top] = 1;
        print_codes(root->right, arr, top + 1);
    }
    
    if (!root->left && !root->right) {
        printf("%c: ", root->data);
        for (int i = 0; i < top; i++) {
            printf("%d", arr[i]);
        }
        printf("\n");
    }
}

void huffman_codes(char data[], int freq[], int size) {
    MinHeapNode *root = build_huffman_tree(data, freq, size);
    
    int arr[100], top = 0;
    print_codes(root, arr, top);
}

int main() {
    char data[] = {'a', 'b', 'c', 'd', 'e', 'f'};
    int freq[] = {5, 9, 12, 13, 16, 45};
    int size = sizeof(data) / sizeof(data[0]);
    
    printf("Huffman Codes:\n");
    huffman_codes(data, freq, size);
    
    return 0;
}
```

---

## 3. Bit Manipulation

**Bit Operations:**
```
AND (&):  1010 & 1100 = 1000
OR  (|):  1010 | 1100 = 1110
XOR (^):  1010 ^ 1100 = 0110
NOT (~):  ~1010 = 0101
Left  (<<): 1010 << 1 = 10100
Right (>>): 1010 >> 1 = 0101
```

**Example:**
```c
#include <stdio.h>

// Check if number is power of 2
int is_power_of_2(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}

// Count set bits (Brian Kernighan's algorithm)
int count_set_bits(int n) {
    int count = 0;
    while (n) {
        n &= (n - 1);  // Removes rightmost set bit
        count++;
    }
    return count;
}

// Get bit at position
int get_bit(int n, int pos) {
    return (n >> pos) & 1;
}

// Set bit at position
int set_bit(int n, int pos) {
    return n | (1 << pos);
}

// Clear bit at position
int clear_bit(int n, int pos) {
    return n & ~(1 << pos);
}

// Toggle bit at position
int toggle_bit(int n, int pos) {
    return n ^ (1 << pos);
}

// Check if number is even/odd
int is_even(int n) {
    return (n & 1) == 0;
}

// Swap two numbers without temp variable
void swap_xor(int *a, int *b) {
    if (a != b) {  // Check for same address
        *a = *a ^ *b;
        *b = *a ^ *b;
        *a = *a ^ *b;
    }
}

// Find missing number in array [0..n]
int find_missing(int arr[], int n) {
    int xor_all = 0;
    int xor_arr = 0;
    
    for (int i = 0; i <= n; i++) {
        xor_all ^= i;
    }
    
    for (int i = 0; i < n; i++) {
        xor_arr ^= arr[i];
    }
    
    return xor_all ^ xor_arr;
}

// Find two non-repeating elements
void find_two_non_repeating(int arr[], int n) {
    int xor_all = 0;
    
    // XOR all elements
    for (int i = 0; i < n; i++) {
        xor_all ^= arr[i];
    }
    
    // Find rightmost set bit
    int set_bit_pos = xor_all & ~(xor_all - 1);
    
    int x = 0, y = 0;
    for (int i = 0; i < n; i++) {
        if (arr[i] & set_bit_pos) {
            x ^= arr[i];
        } else {
            y ^= arr[i];
        }
    }
    
    printf("Two non-repeating elements: %d and %d\n", x, y);
}

void print_binary(int n) {
    for (int i = 31; i >= 0; i--) {
        printf("%d", (n >> i) & 1);
        if (i % 4 == 0) printf(" ");
    }
    printf("\n");
}

int main() {
    int num = 12;  // 1100 in binary
    
    printf("Number: %d\n", num);
    printf("Binary: ");
    print_binary(num);
    
    printf("Is power of 2: %s\n", is_power_of_2(num) ? "Yes" : "No");
    printf("Set bits: %d\n", count_set_bits(num));
    printf("Is even: %s\n", is_even(num) ? "Yes" : "No");
    
    printf("\nBit at position 2: %d\n", get_bit(num, 2));
    printf("Set bit at position 0: %d\n", set_bit(num, 0));
    printf("Clear bit at position 2: %d\n", clear_bit(num, 2));
    printf("Toggle bit at position 1: %d\n", toggle_bit(num, 1));
    
    int a = 5, b = 7;
    printf("\nBefore swap: a=%d, b=%d\n", a, b);
    swap_xor(&a, &b);
    printf("After swap: a=%d, b=%d\n", a, b);
    
    int arr[] = {1, 2, 3, 4, 5, 6, 8};
    printf("\nMissing number: %d\n", find_missing(arr, 7));
    
    int arr2[] = {2, 3, 7, 9, 11, 2, 3, 11};
    find_two_non_repeating(arr2, 8);
    
    return 0;
}
```

---

## CONTINUED IN PART 5

This is Part 4 of the Complete C Programming Master Roadmap. It covers:
- Advanced Graph Algorithms (Dijkstra, Bellman-Ford, Floyd-Warshall, Prim's)
- Greedy Algorithms (Activity Selection, Huffman Coding)
- Bit Manipulation Techniques

**Part 5 will include:**
- File I/O (Text, Binary, Advanced Operations)
- Process Management (fork, exec, signals)
- Inter-Process Communication (Pipes, Shared Memory, Message Queues)
- Threads and Synchronization

Save this file and continue with Part 5!