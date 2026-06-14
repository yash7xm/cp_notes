
### **1. The Core Problem**

You are at node `u`. You need to find its $k$-th ancestor (jumping $k$ steps up towards the root).

- **Naive Approach:** Walk up one parent at a time. Takes $O(k)$ time. If a problem asks you to do this $10^5$ times, you will get Time Limit Exceeded (TLE).
    
- **Optimised Approach:** Represent the jump distance $k$ in binary. Break the massive jump into a few "power of 2" jumps Takes $O(\log k)$ time.
 

### **2. The State Definition**

We need a memory table to store our precomputed jumps. We call this table `up`.

`up[u][j]` = The node you reach if you start at `u` and jump exactly $2^j$ steps up.

- `up[u][0]` = Jump $2^0$ (1 step). This is just the **direct parent**.
    
- `up[u][1]` = Jump $2^1$ (2 steps).
    
- `up[u][2]` = Jump $2^2$ (4 steps).
    
- `up[u][3]` = Jump $2^3$ (8 steps).
    

### **3. Building the Table (The Transition)**

You cannot compute `up[u][3]` (an 8-step jump) directly. But if you already know how to make 4-step jumps, you can split it into two halves:

1. Jump 4 steps. (You land on an intermediate node).
    
2. From that intermediate node, jump another 4 steps.
    

**The Golden Formula:**

`up[u][j] = up[ up[u][j-1] ][ j-1 ]`

_Translation in plain English:_ "My destination after jumping $2^j$ steps is found by first jumping $2^{j-1}$ steps, looking at where I landed, and then jumping another $2^{j-1}$ steps from there."

### **4. Making a Query (The Jumps)**

To jump $k$ steps, you look at the binary bits of $k$.

If $k = 13$, in binary it is `1101`.

`1101` means: $8 + 4 + 0 + 1$.

You just loop through the bits of $k$. If the $i$-th bit is `1`, you instantly jump $2^i$ steps using your precomputed `up` table.


## **HIGH-PERFORMANCE C++ TEMPLATE**

This implementation abandons slow `vector<vector<int>>` in favor of `vector<array<int, MAX_BITS>>`. In C++, an `array` inside a `vector` guarantees the data for a single node is kept strictly contiguous in the CPU cache, drastically reducing cache misses during the inner loops.

```cpp
#include <iostream>
#include <vector>
#include <array>

using namespace std;

// 20 bits is enough for 1,048,576 nodes. 
// Do not make this dynamically sized. Constant size allows compiler unrolling.
const int MAX_BITS = 20; 

struct BinaryLifting {
    int n;
    // vector of arrays is significantly faster than vector of vectors for CPU caching
    vector<array<int, MAX_BITS>> up;
    vector<vector<int>> adj;

    // Constructor: Pre-allocates all necessary memory instantly.
    BinaryLifting(int nodes) : n(nodes) {
        // array initializes with garbage, so we explicitly zero-fill it
        up.assign(n + 1, array<int, MAX_BITS>{}); 
        adj.resize(n + 1);
    }

    void add_edge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // Call this exactly once from the root before answering queries.
    // Usually: dfs(root, 0)
    void dfs(int u, int p) {
        // Base case: The 2^0 ancestor (1 step up) is the parent 'p'
        up[u][0] = p;

        // DP Transition: Build all power-of-2 jumps for node 'u'
        for (int i = 1; i < MAX_BITS; i++) {
            up[u][i] = up[ up[u][i - 1] ][ i - 1 ];
        }

        // Continue DFS down the tree
        for (int v : adj[u]) {
            if (v != p) {
                dfs(v, u);
            }
        }
    }

    // Query: Find the k-th ancestor of node 'u'
    int get_kth_ancestor(int u, int k) {
        for (int i = 0; i < MAX_BITS; i++) {
            // Check if the i-th bit of k is a 1
            if ((k >> i) & 1) {
                u = up[u][i]; // Take the jump
                
                // If u becomes 0, we jumped higher than the root.
                // We can stop early to save time.
                if (u == 0) return 0; 
            }
        }
        return u;
    }
};
```



- **The Universal Sink (Node `0`):** We use 1-based indexing for the nodes. This leaves index `0` unused. We deliberately set `up[0][i] = 0` (which happens automatically by zero-filling the arrays). _Why?_ If you are at node 2 and jump 100 steps up, you will hit the root and keep going. Instead of writing messy `if (node == root)` boundary checks, the jump just lands in `0`. Once you are in `0`, jumping any higher simply keeps you trapped in `0`. Node `0` safely absorbs all out-of-bounds jumps without crashing your program.
    
- **`std::array` over `std::vector`:** A `vector` stores a pointer to data held elsewhere in the heap. `vector<vector<int>>` means memory is fragmented everywhere. An `array` stores data directly inside the struct. `vector<array>` guarantees a single, unbreakable block of memory. When your CPU reads `up[u][0]`, it automatically pulls `up[u][1]` through `up[u][19]` into the L1 cache. This makes your table building virtually instantaneous.