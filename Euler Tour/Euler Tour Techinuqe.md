
The **Euler Tour Technique (ETT)**: a method to flatten a tree into an array while preserving its structural properties (subtrees and paths).

### 1. The Core Intuition: Flattening the Tree

Imagine a tree. If someone asks you to add $x$ to all nodes in the subtree of node $u$, doing a DFS every time is $O(N)$, which is too slow for CP queries.

However, if those nodes were situated contiguously in an array, you could do a simple range update using a Segment Tree or Fenwick Tree in $O(\log N)$ time. ETT makes this possible. By keeping track of a global `timer`during a Depth-First Search (DFS), we can record exactly when we **enter** a node and when we **exit** it.

### 2. The Mechanics: Entry and Exit Times

During the DFS:

1. When you first visit node $u$, record the current time as its **entry time** (`in[u]`), then increment the timer.
    
2. Visit all its children.
    
3. Once all children are processed, record the current time as its **exit time** (`out[u]`).
    

**The Golden Property:** Every node $v$ in the subtree of $u$ will be visited _after_ we enter $u$ and _before_ we exit $u$. Therefore, the subtree of $u$ maps exactly to the contiguous array range $[ \text{in}[u], \text{out}[u] - 1 ]$.

### 3. C++ Implementation

Here is the cleanest, most CP-optimized way to write a standard subtree Euler Tour.

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 2e5 + 5;
vector<int> adj[MAXN];
int timer = 0;

// in[u] stores the flattened array index for node u
int in[MAXN], out[MAXN]; 
int flat_tree[MAXN]; // Optional: Maps the new index back to the node values

void dfs(int u, int p = -1) {
    in[u] = timer;           // Record entry time
    flat_tree[timer] = u;    // Map timer back to node (useful for Segment Trees)
    timer++;                 
    
    for (int v : adj[u]) {
        if (v != p) {
            dfs(v, u);
        }
    }
    out[u] = timer;          // Record exit time (non-inclusive)
}

int main() {
    int n; // number of nodes
    cin >> n;
    
    for(int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }
    
    // 1-indexed nodes assumed
    dfs(1);
    
    // Now, the subtree of node 'u' is perfectly represented by 
    // the contiguous range [ in[u], out[u] - 1 ]
    return 0;
}
```

To truly master ETT, you must realize that the "Subtree Tour" shown above is only **Type 1**. There are three distinct variations of the Euler Tour, each solving a different class of problems.


### Variation A: The Subtree Tour (Size $N$)

- **What it does:** Records `in` and `out` times as shown above.
    
- **Use Cases:**
    
    - Point update, Subtree sum query (Fenwick).
        
    - Subtree update, Subtree max/min query (Lazy Segment Tree).
        
    - Checking if node $u$ is an ancestor of node $v$ in $O(1)$ time: $u$ is an ancestor of $v$ if and only if $\text{in}[u] \le \text{in}[v]$ and $\text{out}[u] \ge \text{out}[v]$.




# Problem (CSES)

# Subtree Queries

You are given a rooted tree consisting of nnn nodes. The nodes are numbered 1,2,…,n and node 1 is the root. Each node has a value.

Your task is to process following types of queries:

1. change the value of node s to x
2. calculate the sum of values in the subtree of node s

# Input

The first input line contains two integers nnn and qqq: the number of nodes and queries. The nodes are numbered 1,2,…,n.

The next line has nnn integers v1,v2,…,vn: the value of each node.

Then there are n−1 lines describing the edges. Each line contans two integers a and b: there is an edge between nodes a and b.

Finally, there are q lines describing the queries. Each query is either of the form "1 s x" or "2 s".

# Output

Print the answer to each query of type 2.

# Constraints

- 1 ≤ n, q ≤ 2⋅10^5
- 1 ≤ a, b, s ≤ n
- 1 ≤ vi, x ≤ 10^9

# Example

Input:

5 3
4 2 5 2 1
1 2
1 3
3 4
3 5
2 3
1 5 3
2 3

Output:

8
10





```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

int n, q;
vector<ll> val;
vector<vector<int>> adj;

vector<int> in, out;
int timer = 0;

vector<ll> bit;

void update(int i, ll delta) {
    for(; i <= n; i += i & -i) {
        bit[i] += delta;
    }
}

ll query(int i) {
    ll res = 0;
    for(; i > 0; i -= i & -i) {
        res += bit[i];
    }
    return res;
}

void dfs(int u, int p) {
    in[u] = ++timer;

    for(int v : adj[u]) {
        if(v != p) {
            dfs(v, u);
        }
    }

    out[u] = timer;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    cin >> n >> q;

    val.resize(n + 1);
    for(int i = 1; i <= n; ++i) {
        cin >> val[i];
    }

    adj.resize(n + 1);
    for(int i = 0; i < n - 1; ++i) {
        int u, v;
        cin >> u >> v;
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    in.resize(n + 1);
    out.resize(n + 1);

    dfs(1, 0);

    bit.assign(n + 1, 0);
    for(int i = 1; i <= n; ++i) {
        update(in[i], val[i]);
    }

    while(q--) {
        int type;
        cin >> type;

        if(type == 1) {
            int s;
            ll x;
            cin >> s >> x;

            ll delta = x - val[s];
            update(in[s], delta);
            val[s] = x;
        } else {
            int s;
            cin >> s;
            cout << query(out[s]) - query(in[s] - 1) << '\n';
        }
    }

    return 0;
}

```