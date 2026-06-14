
### **1. The Core Problem**

You have two nodes, $u$ and $v$. You need to find the lowest node in the tree that is an ancestor to both of them.

Because we are using Binary Lifting, queries will take $O(\log N)$ time after $O(N \log N)$ preprocessing.

### **2. The State Definition (What we need to store)**

We need the exact same `up` table from the $K$-th ancestor technique:

- `up[u][j]` = The $2^j$-th ancestor of node $u$.


We also need **one new piece of data**:

- `depth[u]` = The distance from the root to node $u$. The root is at depth 0.

    We need this because comparing nodes at different heights is structurally impossible.

### **3. The 3-Step LCA Algorithm**

**Step 1: Level the Playing Field**

If $u$ and $v$ are at different depths, we cannot jump them together.

- Assume $u$ is deeper than $v$.
    
- Calculate the difference: $diff = depth[u] - depth[v]$.
    
- Jump $u$ up exactly $diff$ steps (using the standard $K$-th ancestor logic).
    
- _Edge Case Check:_ If $u$ and $v$ are now the exact same node, we are done. The original $v$ was a direct ancestor of $u$.
    

**Step 2: Walk the "Maximum Safe Distance"**

Now $u$ and $v$ are at the same depth. We want to jump them up together, but **we cannot trust equality**. If `up[u][j] == up[v][j]`, we don't know if we landed exactly on the LCA, or if we overshot it by 50 steps.

- Therefore, we only jump if the branches are distinct: `up[u][j] != up[v][j]`.
    
- We test our jumps from largest to smallest (e.g., from $j = 19$ down to $j = 0$).
    
- Because we test in descending order of binary powers, we greedily accumulate the absolute maximum distance possible without merging.
    
- By the time the loop finishes, we are mathematically guaranteed to be standing **exactly 1 step below the true LCA**.
    

**Step 3: One Final Step**

Since $u$ is now exactly one step below the LCA, the answer is simply the direct parent of $u$, which is `up[u][0]`.

---

## **HIGH-PERFORMANCE C++ TEMPLATE**

This C++ code uses `std::array` for optimal CPU caching and relies entirely on standard bitwise operations. It is heavily optimized against Time Limit Exceeded (TLE) verdicts.

```cpp
#include <iostream>
#include <vector>
#include <array>
#include <algorithm>

using namespace std;

// 20 bits supports trees up to 10^6 nodes.
const int MAX_BITS = 20; 

struct LCA {
    int n;
    vector<array<int, MAX_BITS>> up;
    vector<int> depth;
    vector<vector<int>> adj;

    // Constructor: Pre-allocates all contiguous memory instantly
    LCA(int nodes) : n(nodes) {
        up.assign(n + 1, array<int, MAX_BITS>{}); 
        depth.assign(n + 1, 0);
        adj.resize(n + 1);
    }

    void add_edge(int u, int v) {
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // Precompute the DP table and depths. Call this once: dfs(root, 0, 0)
    void dfs(int u, int p, int d) {
        up[u][0] = p;
        depth[u] = d;

        // Build the binary lifting table for node u
        for (int i = 1; i < MAX_BITS; i++) {
            up[u][i] = up[ up[u][i - 1] ][ i - 1 ];
        }

        // DFS to children
        for (int v : adj[u]) {
            if (v != p) {
                dfs(v, u, d + 1);
            }
        }
    }

    // Standard K-th ancestor logic used for Step 1
    int get_kth_ancestor(int u, int k) {
        for (int i = 0; i < MAX_BITS; i++) {
            if ((k >> i) & 1) {
                u = up[u][i];
                if (u == 0) return 0; // Universal sink
            }
        }
        return u;
    }

    // Returns the Lowest Common Ancestor of nodes u and v
    int get_lca(int u, int v) {
        // Step 1: Force u to be the deeper node
        if (depth[u] < depth[v]) {
            swap(u, v);
        }

        // Level them out to the exact same depth
        u = get_kth_ancestor(u, depth[u] - depth[v]);

        // If they collided during leveling, v was the LCA
        if (u == v) {
            return u;
        }

        // Step 2: Maximum Safe Distance
        // MUST loop backwards from largest jump down to smallest jump (j = 0)
        for (int i = MAX_BITS - 1; i >= 0; i--) {
            // Only jump if we stay on separate branches
            if (up[u][i] != up[v][i]) {
                u = up[u][i];
                v = up[v][i];
            }
        }

        // Step 3: u is now exactly 1 step below the LCA. Return its parent.
        retu up[u][0];
    }
};
```

---

### **Design Principles in this Code:**

1. **Enforcing State (`swap(u, v)`):** Instead of writing messy `if/else` blocks to figure out whether to pull $u$ up or $v$ up during Step 1, we write one `if (depth[u] < depth[v]) swap(u, v);`. This guarantees that $u$ is _always_ the deeper node. This flattens the logic and eliminates branching in the rest of the function.
    
2. **The Universal Sink (`return 0`):** Just like the $K$-th ancestor template, if you ask for the LCA of disconnected components or make a mistake that pushes a node above the root, it resolves to `0`. `up[0][i]` is securely zero-initialized, preventing segmentation faults.
    
3. **Descending Loop:** The loop in Step 2 is `for (int i = MAX_BITS - 1; i >= 0; i--)`. In competitive programming, a common bug is writing `i++` out of habit. Binary lifting **will absolutely fail** if you iterate upwards, because taking a small jump first might force you to skip a valid larger jump later, ruining the binary decomposition geometry. You must test the largest boosters first.