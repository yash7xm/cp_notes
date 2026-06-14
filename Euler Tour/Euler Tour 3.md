
### Variation C: The LCA Tour (Size $2N - 1$)

- **What it does:** You add the node to the array every single time you visit it (when you go down to a child, and when you come back up from a child).
    
- **Use Cases:** Finding the Lowest Common Ancestor (LCA) in $O(1)$ time. The LCA of $u$ and $v$ is simply the node with the minimum depth in the flattened array between the first occurrence of $u$ and the first occurrence of $v$. You solve this using a Range Minimum Query (RMQ) Sparse Table.

#### 1. The Core Intuition: The Physical Walk

Imagine you are physically walking through the tree. You start at the Root. You walk down an edge to a child, you walk back up, you walk down to another child, and so on.

If you write down the name of the node you are standing on _every single time you take a step_ (including backtracking), you create a sequence.

**The Golden Property of the LCA Walk:**

Look at any two nodes, $u$ and $v$. Think about the physical path the DFS takes to get from the first time it sees $u$ to the first time it sees $v$. Because it is a tree, the only way to get from $u$'s branch to $v$'s branch is to climb _up_ to their Lowest Common Ancestor, and then walk _down_ to $v$.

Therefore, in your written sequence of steps between $u$ and $v$, the LCA is guaranteed to be the node that is closest to the Root. In other words, it is the node with the **minimum depth**.

#### 2. The Mechanics: The $2N - 1$ Array

Why is the array exactly size $2N - 1$?

A tree with $N$ nodes has exactly $N - 1$ edges. A full DFS traverses every single edge exactly twice (once going down, once coming back up).

$2 \times (N - 1)$ edge traversals $= 2N - 2$ steps. Add the very first time you stand on the root ($+1$), and you get exactly $2N - 1$ node visits.

To make this work, we need to maintain three parallel arrays during our DFS:

1. **`tour` array:** Records the sequence of node IDs as we walk up and down.
    
2. **`depth` array:** Records the depth of the node at that specific step.
    
3. **`first` array:** A lookup table (size $N$) that stores the index in the `tour` array where node $X$ appears for the very first time.
    

#### 3. The Transformation to RMQ

Once the tree is flattened, finding $\text{LCA}(u, v)$ takes exactly three steps:

1. Look up $\text{first}[u]$ and $\text{first}[v]$. Let's say $\text{first}[u] < \text{first}[v]$.
    
2. Look at the `depth` array in the contiguous range $[\text{first}[u], \text{first}[v]]$.
    
3. Find the index of the **minimum value** in that depth range. The node at that index in the `tour` array is your LCA.
    

Because the array is static (the tree structure doesn't change), we can answer Range Minimum Queries in $O(1)$ time by precomputing a **Sparse Table**.

#### 4. C++ Implementation

This is the standard, highly-optimized template for $O(1)$ LCA using a Sparse Table. Notice how we only push to the arrays when we enter a node and when we return from a child.

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 2e5 + 5;
const int LOG = 20; // log2(2 * MAXN)

vector<int> adj[MAXN];

// ETT State
int tour[MAXN * 2];
int depth[MAXN * 2];
int first[MAXN];
int timer = 0;

// Sparse Table: st[i][j] stores the *index* of the minimum depth 
// in the range starting at i with length 2^j
int st[MAXN * 2][LOG];
int log_table[MAXN * 2];

void dfs(int u, int p, int d) {
    // 1. Enter the node
    first[u] = timer;
    tour[timer] = u;
    depth[timer] = d;
    timer++;

    for (int v : adj[u]) {
        if (v != p) {
            dfs(v, u, d + 1);
            
            // 2. Return from the child (backtrack step)
            tour[timer] = u;
            depth[timer] = d;
            timer++;
        }
    }
}

void build_sparse_table(int m) {
    // Precompute logarithms for O(1) RMQ
    log_table[1] = 0;
    for (int i = 2; i <= m; i++)
        log_table[i] = log_table[i/2] + 1;

    // Initialize base cases (intervals of length 1)
    for (int i = 0; i < m; i++)
        st[i][0] = i;

    // Build the table: O(N log N)
    for (int j = 1; (1 << j) <= m; j++) {
        for (int i = 0; i + (1 << j) <= m; i++) {
            int left_idx = st[i][j - 1];
            int right_idx = st[i + (1 << (j - 1))][j - 1];
            
            // We want to store the index of the minimum depth
            if (depth[left_idx] < depth[right_idx])
                st[i][j] = left_idx;
            else
                st[i][j] = right_idx;
        }
    }
}

// Strictly O(1) query
int get_lca(int u, int v) {
    int L = first[u];
    int R = first[v];
    if (L > R) swap(L, R);

    // Find the largest power of 2 that fits in the range
    int j = log_table[R - L + 1];
    
    // Combine two overlapping intervals of size 2^j
    int left_idx = st[L][j];
    int right_idx = st[R - (1 << j) + 1][j];

    if (depth[left_idx] < depth[right_idx])
        return tour[left_idx];
    else
        return tour[right_idx];
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int n, q;
    cin >> n >> q;

    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // 1. Flatten the tree
    dfs(1, 1, 0);

    // 2. The total number of steps is timer (which will be 2N - 1)
    build_sparse_table(timer);

    // 3. Process O(1) queries
    while (q--) {
        int u, v;
        cin >> u >> v;
        cout << get_lca(u, v) << "\n";
    }

    return 0;
}
```


### 5. Why this matters

This specific technique is the undisputed prerequisite for **Virtual Trees (Auxiliary Trees)**.

In some CP problems, you are given a tree of size $10^5$, and $Q$ queries. Each query gives you a subset of $K$ nodes (where $\sum K \le 10^5$) and asks a complex question about _only those nodes_. You cannot do a full DFS for every query.

Instead, you use the $O(1)$ LCA Sparse Table to instantly compress the giant tree into a "Virtual Tree" that contains only the $K$ nodes and their pairwise LCAs. This reduces the time complexity for each query from $O(N)$ down to $O(K \log K)$.
