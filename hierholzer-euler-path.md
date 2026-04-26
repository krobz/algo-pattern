# Hierholzer's Algorithm — Euler Path / Euler Circuit

## When to Use

- Find a path (or circuit) that visits **every edge exactly once** in a graph
- Typical problem: [332. Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/)

## Prerequisites

| Type | Undirected Graph | Directed Graph |
|------|-----------------|----------------|
| Euler Circuit | All vertices have even degree | All vertices have in-degree == out-degree |
| Euler Path | Exactly 2 vertices have odd degree | Exactly 1 vertex with out - in == 1 (start), 1 vertex with in - out == 1 (end), rest equal |

## Core Idea

1. Start DFS from the source and **immediately remove** each edge as you traverse it (`poll` / `remove`)
2. When a node has no more outgoing edges, add it to the **front** of the result (post-order insertion)
3. The final result is the Euler path / circuit

> **Key Insight**: Post-order insertion guarantees that "dead-end" nodes always end up at the tail of the path, preventing them from blocking traversal of other edges.

## Template Code (Java)

```java
class Solution {
    Map<String, Queue<String>> graph = new HashMap<>();
    List<String> ans = new ArrayList<>();

    public List<String> findItinerary(List<List<String>> tickets) {
        for (List<String> e : tickets) {
            String from = e.get(0);
            String to = e.get(1);
            graph.computeIfAbsent(from, k -> new PriorityQueue<>()).offer(to);
        }
        dfs("JFK");
        return ans;
    }

    private void dfs(String cur) {
        Queue<String> currQ = graph.get(cur);
        while (currQ != null && !currQ.isEmpty()) {
            dfs(currQ.poll());
        }
        ans.add(0, cur);
    }
}
```

## Complexity

| | Value |
|---|---|
| Time | O(E log E) with sorted adjacency list, or O(E) without sorting |
| Space | O(V + E) |

## Quick Notes

- `PriorityQueue` ensures lexicographically smallest path; swap to `LinkedList` if ordering doesn't matter
- `ans.add(0, cur)` is equivalent to reversing a post-order traversal; alternatively use `LinkedList.addFirst` or `Collections.reverse` at the end
- Remove edges on the fly (`poll`) to avoid revisiting
- This algorithm naturally handles multigraphs (multiple edges between the same pair of nodes)

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 332 | Reconstruct Itinerary | Directed Euler path, lexicographic order |
| 753 | Cracking the Safe | Euler circuit on de Bruijn graph |
| 2097 | Valid Arrangement of Pairs | Directed Euler path |
