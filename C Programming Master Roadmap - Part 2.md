# The Complete C Programming Master Roadmap - PART 2
## Data Structures & Algorithms (Continued)

---

## Table of Contents - Part 2
1. [Trees (Binary Trees, BST, AVL)](#1-trees)
2. [Hash Tables](#2-hash-tables)
3. [Graphs](#3-graphs)
4. [Sorting Algorithms](#4-sorting-algorithms)
5. [Searching Algorithms](#5-searching-algorithms)
6. [Dynamic Programming](#6-dynamic-programming)

---

## 1. Trees

### 1.1 Binary Trees

**Binary Tree Structure:**
```
         1
       /   \
      2     3
     / \   / \
    4   5 6   7

Properties:
- Each node has at most 2 children
- Height: longest path from root to leaf
- Complete: all levels filled except possibly last
- Full: all nodes have 0 or 2 children
- Perfect: all internal nodes have 2 children, all leaves at same level
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct TreeNode {
    int data;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

TreeNode* create_node(int data) {
    TreeNode *node = malloc(sizeof(TreeNode));
    node->data = data;
    node->left = NULL;
    node->right = NULL;
    return node;
}

// Tree Traversals

// Inorder: Left -> Root -> Right
void inorder(TreeNode *root) {
    if (root == NULL) return;
    inorder(root->left);
    printf("%d ", root->data);
    inorder(root->right);
}

// Preorder: Root -> Left -> Right
void preorder(TreeNode *root) {
    if (root == NULL) return;
    printf("%d ", root->data);
    preorder(root->left);
    preorder(root->right);
}

// Postorder: Left -> Right -> Root
void postorder(TreeNode *root) {
    if (root == NULL) return;
    postorder(root->left);
    postorder(root->right);
    printf("%d ", root->data);
}

// Level-order traversal (BFS)
void level_order(TreeNode *root) {
    if (root == NULL) return;
    
    // Use queue for level-order
    TreeNode *queue[100];
    int front = 0, rear = 0;
    
    queue[rear++] = root;
    
    while (front < rear) {
        TreeNode *current = queue[front++];
        printf("%d ", current->data);
        
        if (current->left) queue[rear++] = current->left;
        if (current->right) queue[rear++] = current->right;
    }
}

// Calculate height of tree
int height(TreeNode *root) {
    if (root == NULL) return 0;
    
    int left_height = height(root->left);
    int right_height = height(root->right);
    
    return 1 + (left_height > right_height ? left_height : right_height);
}

// Count nodes in tree
int count_nodes(TreeNode *root) {
    if (root == NULL) return 0;
    return 1 + count_nodes(root->left) + count_nodes(root->right);
}

// Find maximum value in tree
int find_max(TreeNode *root) {
    if (root == NULL) return -1;
    
    int max = root->data;
    int left_max = find_max(root->left);
    int right_max = find_max(root->right);
    
    if (left_max > max) max = left_max;
    if (right_max > max) max = right_max;
    
    return max;
}

// Check if two trees are identical
int are_identical(TreeNode *t1, TreeNode *t2) {
    if (t1 == NULL && t2 == NULL) return 1;
    if (t1 == NULL || t2 == NULL) return 0;
    
    return (t1->data == t2->data) &&
           are_identical(t1->left, t2->left) &&
           are_identical(t1->right, t2->right);
}

// Mirror a tree
void mirror(TreeNode *root) {
    if (root == NULL) return;
    
    // Swap left and right children
    TreeNode *temp = root->left;
    root->left = root->right;
    root->right = temp;
    
    mirror(root->left);
    mirror(root->right);
}

void free_tree(TreeNode *root) {
    if (root == NULL) return;
    free_tree(root->left);
    free_tree(root->right);
    free(root);
}

int main() {
    // Create tree:
    //       1
    //      / \
    //     2   3
    //    / \
    //   4   5
    
    TreeNode *root = create_node(1);
    root->left = create_node(2);
    root->right = create_node(3);
    root->left->left = create_node(4);
    root->left->right = create_node(5);
    
    printf("Inorder: ");
    inorder(root);
    printf("\n");
    
    printf("Preorder: ");
    preorder(root);
    printf("\n");
    
    printf("Postorder: ");
    postorder(root);
    printf("\n");
    
    printf("Level-order: ");
    level_order(root);
    printf("\n");
    
    printf("Height: %d\n", height(root));
    printf("Node count: %d\n", count_nodes(root));
    printf("Max value: %d\n", find_max(root));
    
    free_tree(root);
    return 0;
}
```

### 1.2 Binary Search Tree (BST)

**BST Property:**
```
For each node:
- All values in left subtree < node value
- All values in right subtree > node value

Example BST:
        50
       /  \
      30   70
     / \   / \
    20 40 60 80

Inorder traversal gives sorted sequence: 20,30,40,50,60,70,80
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct BSTNode {
    int data;
    struct BSTNode *left;
    struct BSTNode *right;
} BSTNode;

BSTNode* create_bst_node(int data) {
    BSTNode *node = malloc(sizeof(BSTNode));
    node->data = data;
    node->left = NULL;
    node->right = NULL;
    return node;
}

// Insert into BST
BSTNode* insert(BSTNode *root, int data) {
    if (root == NULL) {
        return create_bst_node(data);
    }
    
    if (data < root->data) {
        root->left = insert(root->left, data);
    } else if (data > root->data) {
        root->right = insert(root->right, data);
    }
    // If equal, don't insert (no duplicates)
    
    return root;
}

// Search in BST
BSTNode* search(BSTNode *root, int key) {
    if (root == NULL || root->data == key) {
        return root;
    }
    
    if (key < root->data) {
        return search(root->left, key);
    }
    return search(root->right, key);
}

// Find minimum value node
BSTNode* find_min(BSTNode *root) {
    if (root == NULL) return NULL;
    
    while (root->left != NULL) {
        root = root->left;
    }
    return root;
}

// Delete from BST
BSTNode* delete(BSTNode *root, int key) {
    if (root == NULL) return NULL;
    
    if (key < root->data) {
        root->left = delete(root->left, key);
    } else if (key > root->data) {
        root->right = delete(root->right, key);
    } else {
        // Node to be deleted found
        
        // Case 1: No child or one child
        if (root->left == NULL) {
            BSTNode *temp = root->right;
            free(root);
            return temp;
        } else if (root->right == NULL) {
            BSTNode *temp = root->left;
            free(root);
            return temp;
        }
        
        // Case 2: Two children
        // Get inorder successor (smallest in right subtree)
        BSTNode *temp = find_min(root->right);
        root->data = temp->data;
        root->right = delete(root->right, temp->data);
    }
    
    return root;
}

// Check if tree is BST
int is_bst_util(BSTNode *root, int min, int max) {
    if (root == NULL) return 1;
    
    if (root->data <= min || root->data >= max) {
        return 0;
    }
    
    return is_bst_util(root->left, min, root->data) &&
           is_bst_util(root->right, root->data, max);
}

int is_bst(BSTNode *root) {
    return is_bst_util(root, -2147483648, 2147483647);
}

// Find kth smallest element
void kth_smallest_util(BSTNode *root, int *k, int *result) {
    if (root == NULL) return;
    
    kth_smallest_util(root->left, k, result);
    
    (*k)--;
    if (*k == 0) {
        *result = root->data;
        return;
    }
    
    kth_smallest_util(root->right, k, result);
}

int kth_smallest(BSTNode *root, int k) {
    int result = -1;
    kth_smallest_util(root, &k, &result);
    return result;
}

// Inorder traversal
void inorder_bst(BSTNode *root) {
    if (root == NULL) return;
    inorder_bst(root->left);
    printf("%d ", root->data);
    inorder_bst(root->right);
}

void free_bst(BSTNode *root) {
    if (root == NULL) return;
    free_bst(root->left);
    free_bst(root->right);
    free(root);
}

int main() {
    BSTNode *root = NULL;
    
    // Insert elements
    int elements[] = {50, 30, 70, 20, 40, 60, 80};
    for (int i = 0; i < 7; i++) {
        root = insert(root, elements[i]);
    }
    
    printf("Inorder traversal: ");
    inorder_bst(root);
    printf("\n");
    
    printf("Is BST? %s\n", is_bst(root) ? "Yes" : "No");
    
    int key = 40;
    BSTNode *found = search(root, key);
    printf("Search %d: %s\n", key, found ? "Found" : "Not found");
    
    printf("3rd smallest: %d\n", kth_smallest(root, 3));
    
    root = delete(root, 30);
    printf("After deleting 30: ");
    inorder_bst(root);
    printf("\n");
    
    free_bst(root);
    return 0;
}
```

### 1.3 AVL Tree (Self-Balancing BST)

**AVL Tree Balance:**
```
Balance Factor = height(left subtree) - height(right subtree)
Balance Factor ∈ {-1, 0, 1} for all nodes

Rotations:

Left Rotation:
    y                x
   / \              / \
  x   C    →       A   y
 / \                  / \
A   B                B   C

Right Rotation:
    x                y
   / \              / \
  y   C    →       A   x
 / \                  / \
A   B                B   C
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct AVLNode {
    int data;
    struct AVLNode *left;
    struct AVLNode *right;
    int height;
} AVLNode;

int max(int a, int b) {
    return (a > b) ? a : b;
}

int get_height(AVLNode *node) {
    if (node == NULL) return 0;
    return node->height;
}

int get_balance(AVLNode *node) {
    if (node == NULL) return 0;
    return get_height(node->left) - get_height(node->right);
}

AVLNode* create_avl_node(int data) {
    AVLNode *node = malloc(sizeof(AVLNode));
    node->data = data;
    node->left = NULL;
    node->right = NULL;
    node->height = 1;
    return node;
}

// Right rotate subtree rooted with y
AVLNode* right_rotate(AVLNode *y) {
    AVLNode *x = y->left;
    AVLNode *T2 = x->right;
    
    // Perform rotation
    x->right = y;
    y->left = T2;
    
    // Update heights
    y->height = max(get_height(y->left), get_height(y->right)) + 1;
    x->height = max(get_height(x->left), get_height(x->right)) + 1;
    
    return x;
}

// Left rotate subtree rooted with x
AVLNode* left_rotate(AVLNode *x) {
    AVLNode *y = x->right;
    AVLNode *T2 = y->left;
    
    // Perform rotation
    y->left = x;
    x->right = T2;
    
    // Update heights
    x->height = max(get_height(x->left), get_height(x->right)) + 1;
    y->height = max(get_height(y->left), get_height(y->right)) + 1;
    
    return y;
}

// Insert node and balance
AVLNode* avl_insert(AVLNode *node, int data) {
    // Standard BST insertion
    if (node == NULL) {
        return create_avl_node(data);
    }
    
    if (data < node->data) {
        node->left = avl_insert(node->left, data);
    } else if (data > node->data) {
        node->right = avl_insert(node->right, data);
    } else {
        return node;  // Duplicates not allowed
    }
    
    // Update height
    node->height = 1 + max(get_height(node->left), get_height(node->right));
    
    // Get balance factor
    int balance = get_balance(node);
    
    // Left Left Case
    if (balance > 1 && data < node->left->data) {
        return right_rotate(node);
    }
    
    // Right Right Case
    if (balance < -1 && data > node->right->data) {
        return left_rotate(node);
    }
    
    // Left Right Case
    if (balance > 1 && data > node->left->data) {
        node->left = left_rotate(node->left);
        return right_rotate(node);
    }
    
    // Right Left Case
    if (balance < -1 && data < node->right->data) {
        node->right = right_rotate(node->right);
        return left_rotate(node);
    }
    
    return node;
}

void print_inorder_avl(AVLNode *root) {
    if (root == NULL) return;
    print_inorder_avl(root->left);
    printf("%d(h=%d) ", root->data, root->height);
    print_inorder_avl(root->right);
}

void free_avl(AVLNode *root) {
    if (root == NULL) return;
    free_avl(root->left);
    free_avl(root->right);
    free(root);
}

int main() {
    AVLNode *root = NULL;
    
    // Insert elements that would create imbalance in regular BST
    int elements[] = {10, 20, 30, 40, 50, 25};
    
    for (int i = 0; i < 6; i++) {
        root = avl_insert(root, elements[i]);
        printf("After inserting %d: ", elements[i]);
        print_inorder_avl(root);
        printf("\n");
    }
    
    printf("\nFinal AVL tree (balanced): ");
    print_inorder_avl(root);
    printf("\n");
    
    free_avl(root);
    return 0;
}
```

---

## 2. Hash Tables

**Hash Table Concept:**
```
Hash Function: key → index
Collision Resolution: Chaining or Open Addressing

Chaining Example:
┌───┬───────────────┐
│ 0 │ → 10 → 20     │
├───┼───────────────┤
│ 1 │ → 31          │
├───┼───────────────┤
│ 2 │ NULL          │
├───┼───────────────┤
│ 3 │ → 13 → 23     │
└───┴───────────────┘

Time Complexity:
- Average: O(1) insert, delete, search
- Worst: O(n) when all keys hash to same index
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define TABLE_SIZE 10

// Node for chaining
typedef struct HashNode {
    char *key;
    int value;
    struct HashNode *next;
} HashNode;

typedef struct {
    HashNode *table[TABLE_SIZE];
    int size;
} HashTable;

// Hash function (djb2 algorithm)
unsigned long hash(const char *str) {
    unsigned long hash = 5381;
    int c;
    
    while ((c = *str++)) {
        hash = ((hash << 5) + hash) + c;  // hash * 33 + c
    }
    
    return hash % TABLE_SIZE;
}

HashTable* create_hash_table() {
    HashTable *ht = malloc(sizeof(HashTable));
    for (int i = 0; i < TABLE_SIZE; i++) {
        ht->table[i] = NULL;
    }
    ht->size = 0;
    return ht;
}

// Insert key-value pair
void insert(HashTable *ht, const char *key, int value) {
    unsigned long index = hash(key);
    
    // Check if key already exists
    HashNode *current = ht->table[index];
    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            current->value = value;  // Update value
            return;
        }
        current = current->next;
    }
    
    // Create new node
    HashNode *new_node = malloc(sizeof(HashNode));
    new_node->key = strdup(key);
    new_node->value = value;
    new_node->next = ht->table[index];
    ht->table[index] = new_node;
    ht->size++;
}

// Search for key
int search(HashTable *ht, const char *key, int *value) {
    unsigned long index = hash(key);
    
    HashNode *current = ht->table[index];
    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            *value = current->value;
            return 1;
        }
        current = current->next;
    }
    
    return 0;  // Not found
}

// Delete key
int delete(HashTable *ht, const char *key) {
    unsigned long index = hash(key);
    
    HashNode *current = ht->table[index];
    HashNode *prev = NULL;
    
    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            if (prev == NULL) {
                ht->table[index] = current->next;
            } else {
                prev->next = current->next;
            }
            free(current->key);
            free(current);
            ht->size--;
            return 1;
        }
        prev = current;
        current = current->next;
    }
    
    return 0;  // Not found
}

// Print hash table
void print_hash_table(HashTable *ht) {
    printf("\nHash Table Contents:\n");
    for (int i = 0; i < TABLE_SIZE; i++) {
        printf("[%d]: ", i);
        HashNode *current = ht->table[i];
        while (current != NULL) {
            printf("(%s: %d) -> ", current->key, current->value);
            current = current->next;
        }
        printf("NULL\n");
    }
    printf("Size: %d\n\n", ht->size);
}

void free_hash_table(HashTable *ht) {
    for (int i = 0; i < TABLE_SIZE; i++) {
        HashNode *current = ht->table[i];
        while (current != NULL) {
            HashNode *temp = current;
            current = current->next;
            free(temp->key);
            free(temp);
        }
    }
    free(ht);
}

int main() {
    HashTable *ht = create_hash_table();
    
    // Insert key-value pairs
    insert(ht, "apple", 5);
    insert(ht, "banana", 7);
    insert(ht, "orange", 3);
    insert(ht, "grape", 12);
    insert(ht, "mango", 8);
    
    print_hash_table(ht);
    
    // Search
    int value;
    if (search(ht, "banana", &value)) {
        printf("Found banana: %d\n", value);
    }
    
    // Update
    insert(ht, "banana", 10);
    printf("\nAfter updating banana to 10:\n");
    print_hash_table(ht);
    
    // Delete
    delete(ht, "orange");
    printf("After deleting orange:\n");
    print_hash_table(ht);
    
    free_hash_table(ht);
    return 0;
}
```

---

## 3. Graphs

**Graph Representations:**
```
1. Adjacency Matrix:
     0  1  2  3
  0 [0  1  1  0]
  1 [1  0  1  1]
  2 [1  1  0  1]
  3 [0  1  1  0]

2. Adjacency List:
  0: → 1 → 2
  1: → 0 → 2 → 3
  2: → 0 → 1 → 3
  3: → 1 → 2
```

**Example:**
```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_VERTICES 100

// Adjacency List Node
typedef struct AdjListNode {
    int dest;
    int weight;
    struct AdjListNode *next;
} AdjListNode;

// Adjacency List
typedef struct {
    AdjListNode *head;
} AdjList;

// Graph structure
typedef struct {
    int num_vertices;
    AdjList *array;
} Graph;

// Create graph
Graph* create_graph(int vertices) {
    Graph *graph = malloc(sizeof(Graph));
    graph->num_vertices = vertices;
    graph->array = malloc(vertices * sizeof(AdjList));
    
    for (int i = 0; i < vertices; i++) {
        graph->array[i].head = NULL;
    }
    
    return graph;
}

// Add edge to undirected graph
void add_edge(Graph *graph, int src, int dest, int weight) {
    // Add edge from src to dest
    AdjListNode *new_node = malloc(sizeof(AdjListNode));
    new_node->dest = dest;
    new_node->weight = weight;
    new_node->next = graph->array[src].head;
    graph->array[src].head = new_node;
    
    // Add edge from dest to src (undirected)
    new_node = malloc(sizeof(AdjListNode));
    new_node->dest = src;
    new_node->weight = weight;
    new_node->next = graph->array[dest].head;
    graph->array[dest].head = new_node;
}

// DFS utility
void dfs_util(Graph *graph, int vertex, int visited[]) {
    visited[vertex] = 1;
    printf("%d ", vertex);
    
    AdjListNode *current = graph->array[vertex].head;
    while (current != NULL) {
        if (!visited[current->dest]) {
            dfs_util(graph, current->dest, visited);
        }
        current = current->next;
    }
}

// Depth First Search
void dfs(Graph *graph, int start) {
    int visited[MAX_VERTICES] = {0};
    printf("DFS starting from %d: ", start);
    dfs_util(graph, start, visited);
    printf("\n");
}

// Breadth First Search
void bfs(Graph *graph, int start) {
    int visited[MAX_VERTICES] = {0};
    int queue[MAX_VERTICES];
    int front = 0, rear = 0;
    
    visited[start] = 1;
    queue[rear++] = start;
    
    printf("BFS starting from %d: ", start);
    
    while (front < rear) {
        int vertex = queue[front++];
        printf("%d ", vertex);
        
        AdjListNode *current = graph->array[vertex].head;
        while (current != NULL) {
            if (!visited[current->dest]) {
                visited[current->dest] = 1;
                queue[rear++] = current->dest;
            }
            current = current->next;
        }
    }
    printf("\n");
}

// Check if graph has cycle (DFS-based)
int has_cycle_util(Graph *graph, int vertex, int visited[], int rec_stack[]) {
    visited[vertex] = 1;
    rec_stack[vertex] = 1;
    
    AdjListNode *current = graph->array[vertex].head;
    while (current != NULL) {
        if (!visited[current->dest]) {
            if (has_cycle_util(graph, current->dest, visited, rec_stack)) {
                return 1;
            }
        } else if (rec_stack[current->dest]) {
            return 1;
        }
        current = current->next;
    }
    
    rec_stack[vertex] = 0;
    return 0;
}

int has_cycle(Graph *graph) {
    int visited[MAX_VERTICES] = {0};
    int rec_stack[MAX_VERTICES] = {0};
    
    for (int i = 0; i < graph->num_vertices; i++) {
        if (!visited[i]) {
            if (has_cycle_util(graph, i, visited, rec_stack)) {
                return 1;
            }
        }
    }
    return 0;
}

void print_graph(Graph *graph) {
    printf("\nGraph Adjacency List:\n");
    for (int i = 0; i < graph->num_vertices; i++) {
        printf("Vertex %d: ", i);
        AdjListNode *current = graph->array[i].head;
        while (current != NULL) {
            printf("→ %d(w:%d) ", current->dest, current->weight);
            current = current->next;
        }
        printf("\n");
    }
}

void free_graph(Graph *graph) {
    for (int i = 0; i < graph->num_vertices; i++) {
        AdjListNode *current = graph->array[i].head;
        while (current != NULL) {
            AdjListNode *temp = current;
            current = current->next;
            free(temp);
        }
    }
    free(graph->array);
    free(graph);
}

int main() {
    Graph *graph = create_graph(5);
    
    add_edge(graph, 0, 1, 2);
    add_edge(graph, 0, 4, 1);
    add_edge(graph, 1, 2, 3);
    add_edge(graph, 1, 3, 2);
    add_edge(graph, 1, 4, 4);
    add_edge(graph, 2, 3, 1);
    add_edge(graph, 3, 4, 2);
    
    print_graph(graph);
    
    dfs(graph, 0);
    bfs(graph, 0);
    
    printf("Has cycle: %s\n", has_cycle(graph) ? "Yes" : "No");
    
    free_graph(graph);
    return 0;
}
```

---

## CONTINUED IN PART 3

This is Part 2 of the Complete C Programming Master Roadmap. It covers:
- Trees (Binary Trees, BST, AVL)
- Hash Tables with collision resolution
- Graphs (DFS, BFS, Cycle Detection)

**Part 3 will include:**
- Sorting Algorithms (Quick, Merge, Heap)
- Searching Algorithms
- Dynamic Programming
- Advanced Graph Algorithms (Dijkstra, Floyd-Warshall)
- Systems Programming basics

Save this file and continue with Part 3!