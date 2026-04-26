# Tarjan's Algorithm — Strongly Connected Components (SCC)

## When to Use

- Find all **Strongly Connected Components** in a **directed graph**
- SCC: a maximal subgraph where every vertex is reachable from every other vertex
- Applications: graph condensation (DAG reduction), 2-SAT, bridge/articulation point variants

## Core Idea

1. Run DFS and maintain two values for each node:
   - `disc[u]`: discovery timestamp of node u
   - `low[u]`: smallest timestamp reachable from u through the DFS subtree and back edges
2. Use a **stack** to track nodes on the current DFS path
3. When backtracking to node u, if `disc[u] == low[u]`, then u is the **root** of an SCC — pop the stack down to u to collect the component

> **Key Insight**: `low[u] == disc[u]` means u cannot reach any node discovered earlier through its subtree's back edges, making u the first-discovered node in its SCC (the root).

## Template Code (Java)

```java
class Solution {
    private List<Integer>[] graph;
    private int[] disc;
    private int[] low;
    private boolean[] inStack;
    private Deque<Integer> stack;

    private int time;
    private List<List<Integer>> sccList;

    public List<List<Integer>> getSCCs(int V, int[][] edges) {
        graph = new ArrayList[V];
        for (int i = 0; i < V; i++) {
            graph[i] = new ArrayList<>();
        }
        for (int[] edge : edges) {
            graph[edge[0]].add(edge[1]);
        }

        disc = new int[V];
        low = new int[V];
        inStack = new boolean[V];
        stack = new ArrayDeque<>();
        sccList = new ArrayList<>();
        Arrays.fill(disc, -1);
        time = 0;

        for (int i = 0; i < V; i++) {
            if (disc[i] == -1) {
                dfs(i);
            }
        }
        return sccList;
    }

    private void dfs(int u) {
        disc[u] = low[u] = time++;
        stack.push(u);
        inStack[u] = true;

        for (int v : graph[u]) {
            if (disc[v] == -1) {
                dfs(v);
                low[u] = Math.min(low[u], low[v]);
            } else if (inStack[v]) {
                low[u] = Math.min(low[u], disc[v]);
            }
        }

        if (disc[u] == low[u]) {
            List<Integer> component = new ArrayList<>();
            while (true) {
                int node = stack.pop();
                inStack[node] = false;
                component.add(node);
                if (node == u) break;
            }
            sccList.add(component);
        }
    }
}
```

## Complexity

| | Value |
|---|---|
| Time | O(V + E) |
| Space | O(V) |

## Key Variables Explained

| Variable | Meaning |
|----------|---------|
| `disc[u]` | Timestamp when node u was first discovered |
| `low[u]` | Smallest `disc` value reachable from u via its subtree and back edges |
| `inStack[v]` | Whether node v is still on the stack (still a candidate for the current SCC) |
| `disc[u] == low[u]` | u is the root of its SCC — trigger stack pop to collect the component |

## Edge Types in DFS

| Edge Type | Condition | Action |
|-----------|-----------|--------|
| Tree edge | `disc[v] == -1` | Recurse DFS, then `low[u] = min(low[u], low[v])` |
| Back edge | `disc[v] != -1 && inStack[v]` | `low[u] = min(low[u], disc[v])` |
| Cross / Forward edge | `disc[v] != -1 && !inStack[v]` | Ignore (v belongs to an already-finalized SCC) |

## Quick Notes

- `inStack` is the key to distinguishing back edges from cross edges — only nodes still on the stack can belong to the same SCC as the current node
- Each node is pushed and popped exactly once, ensuring linear time overall
- Tarjan finds all SCCs in a single DFS pass; Kosaraju requires two passes
- SCCs are popped in **reverse topological order** (useful for DAG analysis after condensation)

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 1192 | Critical Connections in a Network | Tarjan for bridges (variant: no inStack, check `low[v] > disc[u]`) |
| 685 | Redundant Connection II | Directed graph redundant edge, SCC reasoning |
| 928 | Minimize Malware Spread II | Graph condensation + analysis |
| 1489 | Find Critical and Pseudo-Critical Edges in MST | Graph structure analysis |
