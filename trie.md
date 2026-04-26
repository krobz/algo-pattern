# Trie (Prefix Tree)

## When to Use

- Efficiently store and retrieve strings with **common prefixes**
- Fast prefix lookup, autocomplete, word search, and dictionary operations
- Typical problem: [208. Implement Trie](https://leetcode.com/problems/implement-trie-prefix-tree/)

## Core Idea

1. Each node represents a single character and has up to 26 children (for lowercase English letters)
2. A path from root to any node spells out a prefix; a path ending at a node marked `isEnd` spells out a complete word
3. Insert, search, and prefix check all traverse the tree character by character in O(L) time, where L is the string length

> **Key Insight**: Trie exploits shared prefixes — words like "app", "apple", "apply" share the path `a → p → p`, storing each common prefix only once.

## Template Code (Java)

```java
class Trie {

    class TrieNode {
        boolean isEnd;
        TrieNode[] tns = new TrieNode[26];
    }

    TrieNode root = new TrieNode();

    public Trie() {}

    public void insert(String word) {
        TrieNode p = root;
        for (char ch : word.toCharArray()) {
            int idx = ch - 'a';
            if (p.tns[idx] == null) {
                p.tns[idx] = new TrieNode();
            }
            p = p.tns[idx];
        }
        p.isEnd = true;
    }

    public boolean search(String word) {
        TrieNode p = root;
        for (char ch : word.toCharArray()) {
            int idx = ch - 'a';
            if (p.tns[idx] == null) {
                return false;
            }
            p = p.tns[idx];
        }
        return p.isEnd;
    }

    public boolean startsWith(String prefix) {
        TrieNode p = root;
        for (char ch : prefix.toCharArray()) {
            int idx = ch - 'a';
            if (p.tns[idx] == null) {
                return false;
            }
            p = p.tns[idx];
        }
        return true;
    }
}
```

## Operations

| Operation | Logic | Time |
|-----------|-------|------|
| `insert(word)` | Walk the tree, create missing nodes, mark the last node `isEnd = true` | O(L) |
| `search(word)` | Walk the tree, return `false` if any node is missing, check `isEnd` at the end | O(L) |
| `startsWith(prefix)` | Same as `search`, but return `true` as long as the path exists (ignore `isEnd`) | O(L) |

## Complexity

| | Value |
|---|---|
| Time | O(L) per operation, where L = string length |
| Space | O(N × 26) worst case, N = total characters across all inserted words |

## Common Variations

| Variation | Modification |
|-----------|-------------|
| **HashMap children** | Replace `TrieNode[26]` with `Map<Character, TrieNode>` for large or variable character sets |
| **Count prefix** | Add a `prefixCount` field, increment on insert — enables "count words with prefix" |
| **Count word** | Add a `wordCount` field instead of `boolean isEnd` — handles duplicate insertions |
| **Delete** | Decrement counts or recursively remove nodes with no children |
| **Wildcard search** | On `.` character, branch into all 26 children (see problem 211) |
| **XOR Trie** | Store binary representations of numbers — used for max XOR problems |

## Quick Notes

- `search` vs `startsWith`: the only difference is whether you check `isEnd` at the final node
- Array-based children (`TrieNode[26]`) are faster than HashMap but use more memory
- Trie + DFS/backtracking is a powerful combo for word search in grids (problem 212)
- For problems involving bitwise operations (max XOR), build a binary Trie where each node has children `[0]` and `[1]`

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 208 | Implement Trie (Prefix Tree) | Classic Trie implementation |
| 211 | Design Add and Search Words Data Structure | Trie + wildcard DFS |
| 212 | Word Search II | Trie + backtracking on grid |
| 648 | Replace Words | Prefix lookup with Trie |
| 720 | Longest Word in Dictionary | Trie + BFS/DFS for valid chain |
| 421 | Maximum XOR of Two Numbers in an Array | Binary (XOR) Trie |
| 1268 | Search Suggestions System | Trie + DFS for autocomplete |
| 677 | Map Sum Pairs | Trie with value accumulation |
