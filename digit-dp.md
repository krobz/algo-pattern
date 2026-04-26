# Digit DP

## When to Use

- Count numbers in a range `[0, N]` (or `[L, R]`) that satisfy some digit-level constraint
- The constraint depends on individual digits (e.g., digit sum, specific digit presence, divisibility by digit properties)
- Typical pattern: convert N to a character array, then DFS from the most significant digit to the least

## Core Idea

1. Process the number **digit by digit** from left to right
2. Track a `tight` flag — whether previous digits have all matched the upper bound, constraining the current digit's max value
3. Track an `isNum` flag — whether we have started placing non-zero digits (handles leading zeros)
4. Memoize on `(pos, state, isNum)` — only cache when `tight == false`, since tight states are unique paths

> **Key Insight**: The `tight` constraint propagates like a chain — once any digit is chosen strictly less than the upper bound digit, all subsequent digits are free to be 0–9. This is why we only memoize the non-tight states.

## Template Code (Java)

```java
class Solution {
    private char[] s;
    private long[][][] memo;
    private int target;

    public int countLargestGroup(int n) {
        s = String.valueOf(n).toCharArray();

        int len = s.length;
        int maxDigitSum = 9 * len;

        long maxSize = 0;
        int ans = 0;

        for (int t = 1; t <= maxDigitSum; t++) {
            target = t;
            memo = new long[len][target + 1][2];

            for (int i = 0; i < len; i++) {
                for (int j = 0; j <= target; j++) {
                    Arrays.fill(memo[i][j], -1);
                }
            }

            long size = dfs(0, 0, false, true);

            if (size > maxSize) {
                maxSize = size;
                ans = 1;
            } else if (size == maxSize) {
                ans++;
            }
        }

        return ans;
    }

    private long dfs(int pos, int sum, boolean isNum, boolean tight) {
        if (sum > target) {
            return 0;
        }

        if (pos == s.length) {
            return isNum && sum == target ? 1 : 0;
        }

        int started = isNum ? 1 : 0;

        if (!tight && memo[pos][sum][started] != -1) {
            return memo[pos][sum][started];
        }

        int up = tight ? s[pos] - '0' : 9;
        long res = 0;

        for (int d = 0; d <= up; d++) {
            boolean nextIsNum = isNum || d != 0;
            int nextSum = nextIsNum ? sum + d : sum;
            boolean nextTight = tight && d == up;
            res += dfs(pos + 1, nextSum, nextIsNum, nextTight);
        }

        if (!tight) {
            memo[pos][sum][started] = res;
        }

        return res;
    }
}
```

## DFS Parameters Explained

| Parameter | Meaning |
|-----------|---------|
| `pos` | Current digit position (0-indexed from the most significant digit) |
| `sum` | Accumulated state (digit sum in this example; could be any constraint) |
| `isNum` | Whether we've placed at least one non-zero digit (handles leading zeros) |
| `tight` | Whether all previously chosen digits equal the upper bound's digits |

## Why Only Memoize `tight == false`?

- When `tight == true`, the current path is **uniquely determined** by the upper bound — it's a single path down the DFS tree, so caching it provides no reuse
- When `tight == false`, the subtree result depends only on `(pos, state, isNum)` and can be reused across many parent paths

## Complexity

| | Value |
|---|---|
| Time | O(pos × state_space × 10) per query |
| Space | O(pos × state_space) for memo table |

## Common Variations

| Variation | State to Track |
|-----------|---------------|
| Digit sum equals K | `sum` |
| Number divisible by K | `remainder % K` |
| Count of a specific digit | `count` |
| No repeated digits | Bitmask of used digits |
| Range `[L, R]` | `f(R) - f(L - 1)` |

## Quick Notes

- Always convert the number to a `char[]` or `String` first — never do arithmetic extraction during DFS
- `isNum` is crucial for problems where leading zeros matter (e.g., "count numbers with no repeated digits")
- For range `[L, R]` queries, use prefix subtraction: `solve(R) - solve(L - 1)`
- The `tight` flag is never part of the memo key — it's a control flag that determines whether to memoize

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 233 | Number of Digit One | Count digit 1 in [1, n] |
| 357 | Count Numbers with Unique Digits | Bitmask state |
| 600 | Non-negative Integers without Consecutive Ones | Binary digit DP |
| 902 | Numbers At Most N Given Digit Set | Restricted digit set |
| 1012 | Numbers With Repeated Digits | Bitmask + complement counting |
| 1397 | Find All Good Strings | Digit DP + KMP |
| 2376 | Count Special Integers | No repeated digits |
