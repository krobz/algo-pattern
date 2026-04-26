# Dijkstra's Algorithm — Single Source Shortest Path

## When to Use

- Find the shortest path from a **single source** to all other nodes in a **weighted graph with non-negative edges**
- Works on both directed and undirected graphs
- If edges can be negative, use Bellman-Ford instead

## Core Idea

1. Maintain a distance array `distTo[]` initialized to `∞`, with `distTo[source] = 0`
2. Use a **min-heap (priority queue)** to always process the node with the smallest known distance
3. For each node polled, relax all its neighbors — if a shorter path is found, update `distTo[]` and push the new state
4. Skip stale entries: if the polled distance is greater than `distTo[node]`, this entry is outdated

> **Key Insight**: The greedy choice is correct because all edge weights are non-negative — once a node is polled from the min-heap with distance `d`, no future path to that node can be shorter than `d`.

## Template Code (Java)

```java
class Solution {
    public int networkDelayTime(int[][] times, int n, int k) {
        List<int[]>[] graph = new List[n + 1];
        for (int i = 0; i <= n; i++) {
            graph[i] = new ArrayList<>();
        }
        for (int[] time : times) {
            graph[time[0]].add(new int[]{time[1], time[2]});
        }

        int[] distTo = new int[n + 1];
        Arrays.fill(distTo, Integer.MAX_VALUE);
        distTo[k] = 0;

        Queue<State> pq = new PriorityQueue<>((a, b) -> a.distFromStart - b.distFromStart);
        pq.offer(new State(k, 0));

        while (!pq.isEmpty()) {
            State cur = pq.poll();
            int id = cur.id;
            int distFromStart = cur.distFromStart;

            if (distTo[id] < distFromStart) {
                continue;
            }

            for (int[] next : graph[id]) {
                int nextId = next[0];
                int nextW = next[1];
                int toNext = distFromStart + nextW;
                if (toNext < distTo[nextId]) {
                    distTo[nextId] = toNext;
                    pq.offer(new State(nextId, toNext));
                }
            }
        }

        int minPath = -1;
        for (int i = 1; i <= n; i++) {
            minPath = Math.max(minPath, distTo[i]);
        }
        return minPath == Integer.MAX_VALUE ? -1 : minPath;
    }
}

class State {
    int id;
    int distFromStart;

    State(int id, int distFromStart) {
        this.id = id;
        this.distFromStart = distFromStart;
    }
}
```

## Complexity

| | Value |
|---|---|
| Time | O((V + E) log V) with binary heap |
| Space | O(V + E) |

## The "Stale Entry" Trick

Instead of using a `visited[]` set or `decrease-key` operation, we simply:
1. Allow duplicate entries in the priority queue
2. Skip processing when `distTo[id] < distFromStart`

This is simpler to implement and has the same asymptotic complexity.

## Dijkstra vs. BFS vs. Bellman-Ford

| Algorithm | Edge Weights | Graph Type | Time |
|-----------|-------------|------------|------|
| BFS | Unweighted (or weight = 1) | Any | O(V + E) |
| Dijkstra | Non-negative | Any | O((V + E) log V) |
| Bellman-Ford | Any (handles negative) | Any | O(V × E) |
| 0-1 BFS | Weights are 0 or 1 | Any | O(V + E) |

## Quick Notes

- Never use Dijkstra with negative edge weights — the greedy assumption breaks
- The priority queue may contain up to O(E) entries due to duplicate states
- For **early termination** (single target), return as soon as the target node is polled
- Wrapping `(id, dist)` into a `State` class is cleaner than using `int[]` with comparators
- No need to process all nodes in the same "level" together — just poll one at a time

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 743 | Network Delay Time | Classic single-source shortest path |
| 787 | Cheapest Flights Within K Stops | Dijkstra + constraint (at most K stops) |
| 1514 | Path with Maximum Probability | Max-heap variant (maximize product) |
| 1631 | Path With Minimum Effort | Dijkstra on grid, minimize max edge |
| 778 | Swim in Rising Water | Dijkstra on grid, minimize max node |
| 882 | Reachable Nodes In Subdivided Graph | Modified relaxation |
| 2577 | Minimum Time to Visit a Cell In a Grid | Dijkstra + parity trick |
