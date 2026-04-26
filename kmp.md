# KMP Algorithm — String Pattern Matching

## When to Use

- Find the first (or all) occurrence(s) of a **pattern** string in a **text** string in linear time
- Avoid the naive O(N × M) approach by never re-scanning matched characters in the text
- Typical problem: [28. Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/)

## Core Idea

1. **Preprocess** the pattern to build a `next[]` (failure function / partial match table)
2. `next[j]` stores the length of the longest **proper prefix** of `pattern[0..j-1]` that is also a suffix
3. During matching, when a mismatch occurs at `pattern[j]`, instead of restarting from `j = 0`, jump to `j = next[j]` — skipping characters that are guaranteed to match

> **Key Insight**: The `next[]` table encodes how much of the pattern we can "reuse" after a mismatch. This avoids backtracking in the text string, keeping the text pointer `i` always moving forward.

## Template Code (Java)

```java
class Solution {
    public int strStr(String haystack, String needle) {
        int i = 0;
        int j = 0;

        int[] next = new int[needle.length() + 1];
        getNext(needle, next);

        while (i < haystack.length() && j < needle.length()) {
            if (j == -1 || haystack.charAt(i) == needle.charAt(j)) {
                i++;
                j++;
            } else {
                j = next[j];
            }
        }

        if (j == needle.length())
            return i - j;
        else
            return -1;
    }

    void getNext(String p, int[] next) {
        int i = 0;
        int j = -1;
        next[0] = -1;

        while (i < p.length()) {
            if (j == -1 || p.charAt(i) == p.charAt(j)) {
                i++;
                j++;
                next[i] = j;
            } else {
                j = next[j];
            }
        }
    }
}
```

## How `next[]` Works

For pattern `"ABCABD"`:

| j | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| char | A | B | C | A | B | D |
| next[j] | -1 | 0 | 0 | 0 | 1 | 2 |

- `next[0] = -1`: sentinel value, means "advance both pointers"
- `next[4] = 1`: the prefix `"ABCA"` has longest proper prefix-suffix `"A"` of length 1
- `next[5] = 2`: the prefix `"ABCAB"` has longest proper prefix-suffix `"AB"` of length 2

When mismatch at `j = 5` (char `D`), jump to `j = 2` (char `C`) because `"AB"` already matches.

## The `getNext` Construction — Step by Step

The `next[]` table is built by **matching the pattern against itself**:
- `i` scans the pattern from left to right
- `j` tracks the length of the current matching prefix-suffix
- On mismatch, `j` falls back to `next[j]` (same logic as the main matching)

This self-matching property is what makes KMP elegant — the same algorithm builds the table and performs the search.

## Complexity

| | Value |
|---|---|
| Time | O(N + M) — N = text length, M = pattern length |
| Space | O(M) for the `next[]` table |

## Why `j = -1` Instead of `j = 0`?

- `next[0] = -1` acts as a sentinel: when `j == -1`, both `i` and `j` increment, effectively saying "no prefix matches, move to the next character in text"
- This avoids a special case check and keeps the loop clean

## Quick Notes

- The text pointer `i` **never backtracks** — it only moves forward, guaranteeing O(N) text scanning
- To find **all occurrences**, don't return on the first match — instead set `j = next[j]` and continue
- KMP is the foundation for more advanced algorithms (Aho-Corasick for multi-pattern, etc.)
- The `next[]` table itself solves many prefix-suffix problems (e.g., shortest repeating unit)

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 28 | Find the Index of the First Occurrence in a String | Classic KMP |
| 214 | Shortest Palindrome | Build `next[]` on `s + "#" + reverse(s)` |
| 459 | Repeated Substring Pattern | Check if `len % (len - next[len]) == 0` |
| 686 | Repeated String Matching | KMP search on repeated text |
| 1392 | Longest Happy Prefix | Directly return `next[len]` |
