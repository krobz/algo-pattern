# Kruskal's Algorithm — Minimum Spanning Tree (MST)

## When to Use

- Find the **Minimum Spanning Tree** of an undirected, weighted graph
- MST connects all vertices with the minimum total edge weight and exactly `V - 1` edges
- Kruskal's is particularly efficient when you already have an **edge list** (vs. adjacency list)

## Core Idea

1. Sort all edges by weight (ascending)
2. Iterate through edges greedily — for each edge, if it connects two **different components**, add it to the MST
3. Use **Union-Find** to efficiently check and merge components
4. Stop after adding `V - 1` edges (MST is complete)

> **Key Insight**: By processing edges in sorted order, the first edge that connects two separate components is always safe to include — this is the **Cut Property** of MSTs.

## Template Code (Java)

```java
class Solution {
    public int minCostConnectPoints(int[][] points) {
        int n = points.length;
        List<int[]> edges = new ArrayList<>();

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int dist = Math.abs(points[i][0] - points[j][0])
                         + Math.abs(points[i][1] - points[j][1]);
                edges.add(new int[]{i, j, dist});
            }
        }

        edges.sort((a, b) -> a[2] - b[2]);

        int mst = 0;
        UF uf = new UF(n);

        for (int[] edge : edges) {
            int u = edge[0];
            int v = edge[1];
            if (uf.isConnected(u, v)) continue;
            uf.union(u, v);
            mst += edge[2];
        }

        return mst;
    }
}
```

## Union-Find (with Path Compression)

```java
class UF {
    private int[] parent;

    public UF(int n) {
        parent = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ) return;
        parent[rootP] = rootQ;
    }

    public int find(int x) {
        if (x != parent[x]) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    public boolean isConnected(int p, int q) {
        return find(p) == find(q);
    }
}
```

## Complexity

| | Value |
|---|---|
| Time | O(E log E) — dominated by sorting |
| Space | O(V + E) |

> With path compression + union by rank, each Union-Find operation is nearly O(1) amortized — O(α(n)), where α is the inverse Ackermann function.

## Kruskal vs. Prim

| | Kruskal | Prim |
|---|---------|------|
| Data structure | Union-Find | Min-Heap / Priority Queue |
| Best for | Sparse graphs (E ≈ V) | Dense graphs (E ≈ V²) |
| Input format | Edge list | Adjacency list |
| Time | O(E log E) | O((V + E) log V) |
| Approach | Global edge sorting | Grow tree from a source |

## Union-Find Optimizations

| Optimization | Effect |
|-------------|--------|
| **Path compression** | `find(x)` flattens the tree — nearly O(1) per operation |
| **Union by rank/size** | Attach smaller tree under larger — keeps tree shallow |
| Both combined | O(α(n)) per operation (effectively constant) |

## Quick Notes

- Always sort edges first — the greedy correctness depends on processing in non-decreasing weight order
- For complete graphs (like connecting points), the edge list has O(V²) entries
- Can **early-terminate** after adding exactly `V - 1` edges
- Union-Find with path compression alone (without union by rank) is sufficient for most competitive programming

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 1584 | Min Cost to Connect All Points | Complete graph MST |
| 1135 | Connecting Cities With Minimum Cost | Classic MST |
| 1489 | Find Critical and Pseudo-Critical Edges in MST | Enumerate edge inclusion/exclusion |
| 1168 | Optimize Water Distribution in a Village | Virtual node + MST |
| 1579 | Remove Max Number of Edges to Keep Graph Fully Traversable | Dual Union-Find |
| 1697 | Checking Existence of Edge Length Limited Paths | Offline queries + sorted Kruskal |
