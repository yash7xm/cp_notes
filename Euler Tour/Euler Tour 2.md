
If the Subtree Tour is about laying out memory contiguously, the Path Tour is about simulating the **call stack** of your program.

When you traverse a tree to find a path between two nodes, the path is rarely a straight line downwards. You usually have to go _up_ from the starting node to a Lowest Common Ancestor (LCA), and then _down_ to the target node. This "up-and-down" topology is a nightmare for standard data structures. The Path Tour solves this by using a mathematical property of inverses: $X + (-X) = 0$.

Here is the first-principles breakdown of how to flatten tree paths into an array.

### 1. The Core Intuition: The Call Stack

Imagine the DFS as an execution thread pushing and popping frames onto a stack.

- **Entry (`in[u]`):** You step into node $u$. You push it onto the stack and activate its value ($+1$).
    
- **Exit (`out[u]`):** You are done with node $u$ and its descendants. You step backward, popping it off the stack and deactivating its value ($-1$).
    

If you record this exact sequence in an array of size $2N$, you create a chronological ledger of state changes.

**The Golden Rule of the Path Tour:** At any exact moment during the DFS (at time $T = \text{in}[u]$), the prefix sum of the flattened array from index $1$ to $\text{in}[u]$ yields exactly the nodes that are currently active on the stack.

Everything else—every branch, subtree, or dead end you explored and left prior to reaching $u$—has been both added ($+1$) and subtracted ($-1$), cleanly zeroing itself out.

The prefix sum up to $\text{in}[u]$ gives you the direct, vertical path from the Root down to $u$.

### 2. The Magic of Cancellation: Node-to-Node Queries

Querying from the root is easy, but competitive programming problems will ask you to query the path between two arbitrary nodes, $u$ and $v$.

Assume $\text{in}[u] < \text{in}[v]$ (we discovered $u$ first). There are exactly two topological cases.

**Case 1: $u$ is an ancestor of $v$ (The Straight Line)**

- The path from $u$ to $v$ is a straight downward line.
    
- You query the contiguous array range $[\text{in}[u], \text{in}[v]]$.
    
- **Why it works:** You entered $u$, explored some side branches, and eventually reached $v$. Any node $w$ in those side branches was fully explored and exited. Therefore, $w$ appears in your range exactly twice: once as $+1$ and once as $-1$. It cancels out. The only nodes that remain at $+1$ are the ones you entered but have _not yet exited_—which is exactly the direct path from $u$ down to $v$.
    

**Case 2: $u$ is not an ancestor of $v$ (The Cross-Branch Path)**

- This is the classic "up-and-over" path. You have to travel up from $u$ to $\text{LCA}(u, v)$, and then down to $v$.
    
- You query the contiguous array range $[\text{out}[u], \text{in}[v]]$.
    
- **Why it works:** Let's trace the DFS timeline. You finish $u$ (recording $\text{out}[u]$), you climb up the tree, you explore some other subtrees, and finally you discover $v$ (recording $\text{in}[v]$).
    
- What happens to the nodes in this range?
    
    1. **Nodes fully explored in between:** They appear as both $+1$ and $-1$. They cancel out.
        
    2. **Nodes on the path going UP from $u$:** Their entry time was _before_ $\text{out}[u]$. So we only see their exit ($-1$) in our range.
        
    3. **Nodes on the path going DOWN to $v$:** Their entry time ($+1$) is in our range, but their exit is _after_ $\text{in}[v]$.
        
    4. **The result:** Every node on the path between $u$ and $v$ appears exactly once in the range $[\text{out}[u], \text{in}[v]]$, either as an entry or an exit.
        

### 3. The LCA Exception (The Missing Link)

There is one structural flaw in Case 2 that you must manually patch.

Look at the Lowest Common Ancestor. We entered the LCA long before we reached $u$, and we will exit the LCA long after we process $v$.

Therefore, the LCA's entry is to the left of our range, and its exit is to the right of our range. **The LCA does not appear in the range $[\text{out}[u], \text{in}[v]]$ at all.**

When you calculate the path sum or XOR for Case 2 using the flattened array, you must manually add the value of $\text{LCA}(u, v)$ to your final answer.


#### Application 1: Mo’s Algorithm on Trees (The Crown Jewel)

This is by far the most important reason the $2N$ Path Tour exists in modern competitive programming.

**The Problem:** You are given a tree where each node has a color. You are given $Q$ offline queries of the form $(u, v)$: "How many _distinct_ colors exist on the simple path between $u$ and $v$?"

**Why it's hard:** There is no easy way to merge "distinct colors" in a Segment Tree or Fenwick Tree.

**The $2N$ ETT Solution:**

1. Flatten the tree into a $2N$ array using the exact Entry/Exit timer we discussed.
    
2. Convert the tree queries into array queries:
    
    - If $u$ is an ancestor of $v$, the query is $[\text{in}[u], \text{in}[v]]$.
        
    - If not, the query is $[\text{out}[u], \text{in}[v]]$. (And remember to manually check the LCA!).
        
3. Sort these array intervals using Mo's Algorithm (grouping by $\sqrt{N}$ blocks).
    
4. **The Toggling Magic:** As you expand and contract your Mo's pointers over the $2N$ array, you maintain a `frequency` array of colors and a `visited` boolean array for the nodes.
    
    - When your pointer hits a node in the array, you flip its `visited` state.
        
    - If `visited` becomes `true` (first time seeing it), you add its color to your frequency map.
        
    - If `visited` becomes `false` (second time seeing it, meaning we are exiting the subtree), you _remove_ its color from your frequency map.
        
    - This perfectly simulates the path! Every node not on the path gets toggled `true` then `false`, vanishing from your frequency map.
        

#### Application 2: Dynamic Path XOR Queries

**The Problem:** Nodes have integer values. You have two queries: update the value of node $u$, and find the Bitwise XOR of all nodes on the path between $u$ and $v$.

**The $2N$ ETT Solution:**

This is identical to the Path Sum we coded, but it works even better because in XOR math, a number is its own inverse: $X \oplus X = 0$.

- When entering node $u$, add $W[u]$ to the Fenwick Tree using XOR.
    
- When exiting node $u$, add $W[u]$ again to the Fenwick Tree using XOR.
    
- The prefix XOR up to $\text{in}[X]$ gives the exact XOR from the Root to $X$. All side-branches cancel themselves out because $W[\text{branch}] \oplus W[\text{branch}] = 0$.
    
- The formula for the path becomes even cleaner because subtraction is just XOR:
    
    $$\text{Path}(u, v) = \text{Query}(u) \oplus \text{Query}(v) \oplus W[\text{LCA}]$$
    
    _(Note: We don't multiply the LCA by 2 like in addition, because in XOR, $X \oplus X = 0$, so the shared path from the root entirely obliterates itself, taking the LCA with it. We just put the LCA back once)._
    

#### Application 3: Edge Weight Queries

**The Problem:** Instead of node weights, the _edges_ have weights. You need to update an edge weight and query the sum/XOR of a path.

**The $2N$ ETT Solution:**

In trees, every edge connects a parent to a child. Because every node (except the root) has exactly one parent, you can "push" the edge weight down into the lower node.

- If there is an edge between $u$ and $v$ with weight $W$, and 
	depth$[v] > \text{depth}[u]$, you simply assign $W[v] = W$.
    
- The problem is now entirely reduced to standard Node Weight queries (Application 2), with one minor tweak: when calculating the final path answer, you do _not_ include the LCA's weight, because the LCA's weight represents the edge above the LCA, which is not part of the path between $u$ and $v$.
    

#### Application 4: "Is Node X on the Path between U and V?"

**The Problem:** Given multiple offline queries $(u, v, x)$, determine if node $x$ lies on the simple path between $u$ and $v$ in $O(1)$ time per query.

**The $2N$ ETT Solution:**

You can solve this using raw LCA calls, but it requires multiple checks. The $2N$ array allows for a beautiful topological check.

A node $x$ is on the path between $u$ and $v$ if and only if it satisfies the structural boundaries of the flattened array.

Using the Entry and Exit times, you can definitively prove if $x$ is an ancestor of $u$ or $v$, and if its branching depth aligns with the LCA of $u$ and $v$. By checking if $\text{in}[x]$ falls between specific boundaries, you can answer this in strict $O(1)$ without doing any path traversal.

#### When to Abandon the $2N$ Tour

If a problem asks: "What is the maximum weight on the path between $u$ and $v$?", you **cannot** use the $2N$ ETT.

Why? Because the `MAX()` function has no inverse. If you enter a branch and see a weight of `100`, and then you exit that branch, how do you remove the `100` from your prefix maximum? You can't. You cannot "un-max" a value.

Whenever you face a path query involving non-invertible operations (Max, Min, Matrix Multiplication, string concatenation), you must immediately abandon the $2N$ Path Tour and build a **Heavy-Light Decomposition (HLD)**.


### Problem (CSES)

# Path Queries

You are given a rooted tree consisting of nnn nodes. The nodes are numbered 1,2,…,n and node 1 is the root. Each node has a value.

Your task is to process following types of queries:

1. change the value of node sss to x
2. calculate the sum of values on the path from the root to node s

# Input

The first input line contains two integers nnn and qqq: the number of nodes and queries. The nodes are numbered 1,2,…,n.

The next line has nnn integers v1,v2,…,vn: the value of each node.

Then there are n−1 lines describing the edges. Each line contains two integers a and b: there is an edge between nodes a and b.

Finally, there are qqq lines describing the queries. Each query is either of the form "1 sss x" or "2 s".

# Output

Print the answer to each query of type 2.

# Constraints

- 1 ≤ n, q ≤ 2⋅10^5
- 1 ≤ a, b, s ≤n
- 1 ≤ vi, x ≤ 10^9

# Example

Input:

5 3
4 2 5 2 1
1 2
1 3
3 4
3 5
2 4
1 3 2
2 4

Output:

11
8





```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 2e5 + 5;
const int LOG = 18;

vector<int> adj[MAXN];
int W[MAXN]; // Node weights

// Euler Tour state
int in[MAXN], out[MAXN], timer = 0;

// Binary Lifting state
int up[MAXN][LOG], depth[MAXN];

// Fenwick Tree (Binary Indexed Tree) sized for 2N
long long bit[MAXN * 2];

void add(int idx, long long val) {
    for (; idx < MAXN * 2; idx += idx & -idx)
        bit[idx] += val;
}

long long query(int idx) {
    long long sum = 0;
    for (; idx > 0; idx -= idx & -idx)
        sum += bit[idx];
    return sum;
}

// DFS to populate ETT, Depth, and Binary Lifting table
void dfs(int u, int p) {
    in[u] = ++timer;
    
    // Add positive weight on entry
    add(in[u], W[u]);
    
    up[u][0] = p;
    for (int i = 1; i < LOG; i++) {
        up[u][i] = up[up[u][i-1]][i-1];
    }
    
    for (int v : adj[u]) {
        if (v != p) {
            depth[v] = depth[u] + 1;
            dfs(v, u);
        }
    }
    
    out[u] = ++timer;
    
    // Add negative weight on exit
    add(out[u], -W[u]);
}

// Standard O(log N) LCA using Binary Lifting
int get_lca(int u, int v) {
    if (depth[u] < depth[v]) swap(u, v);
    int diff = depth[u] - depth[v];
    for (int i = 0; i < LOG; i++) {
        if ((diff >> i) & 1) {
            u = up[u][i];
        }
    }
    if (u == v) return u;
    for (int i = LOG - 1; i >= 0; i--) {
        if (up[u][i] != up[v][i]) {
            u = up[u][i];
            v = up[v][i];
        }
    }
    return up[u][0];
}

// Function to update a node's weight dynamically
void update_node(int u, int new_val) {
    int diff = new_val - W[u];
    W[u] = new_val;
    
    // Update the +1 state and the -1 state
    add(in[u], diff);
    add(out[u], -diff);
}

// Function to get the sum of the path between u and v
long long get_path_sum(int u, int v) {
    int lca = get_lca(u, v);
    long long sum_u = query(in[u]);
    long long sum_v = query(in[v]);
    long long sum_lca = query(in[lca]);
    
    return sum_u + sum_v - 2 * sum_lca + W[lca];
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int n, q;
    cin >> n >> q;

    for (int i = 1; i <= n; i++) {
        cin >> W[i];
    }

    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    // Initialize Root as node 1
    depth[1] = 0;
    dfs(1, 1);

    // Process Queries
    while (q--) {
        int type;
        cin >> type;
        if (type == 1) {
            // Update node u to new_val
            int u, new_val;
            cin >> u >> new_val;
            update_node(u, new_val);
        } else if (type == 2) {
            // Query path sum between u and v
            int u = 1;
            int v;
            cin >> v;
            cout << get_path_sum(u, v) << "\n";
        }
    }

    return 0;
}
```
