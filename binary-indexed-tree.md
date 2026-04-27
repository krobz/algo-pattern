# Binary Indexed Tree (Fenwick Tree)

## When to Use

- **Point update + range query** on an array: update a single element and query prefix/range sums efficiently
- Need O(log N) for both operations (vs. O(N) update for prefix sum array, or O(N) query for raw array)
- Typical problem: [307. Range Sum Query - Mutable](https://leetcode.com/problems/range-sum-query-mutable/)

## Core Idea

1. Store partial sums in a 1-indexed array `tree[]`, where each position is responsible for a range of elements determined by its **lowest set bit**
2. `tree[i]` holds the sum of `nums[j]` for `j ∈ (i - lowbit(i), i]`, where `lowbit(i) = i & -i`
3. **Update**: add delta to `tree[i]`, then climb upward by adding `lowbit(i)` to the index
4. **Query**: accumulate `tree[i]`, then strip the lowest set bit to move to the next responsible node

> **Key Insight**: The binary representation of the index determines the range each node covers. `lowbit(i) = i & -i` extracts the lowest set bit, which equals the size of the range that `tree[i]` is responsible for. This creates an implicit tree structure with O(log N) height.

## Template Code (Java)

```java
class NumArray {
    private int[] nums;
    private int[] tree;

    public NumArray(int[] nums) {
        int n = nums.length;
        this.nums = new int[n];
        tree = new int[n + 1];
        for (int i = 0; i < n; i++) {
            update(i, nums[i]);
        }
    }

    public void update(int index, int val) {
        int delta = val - nums[index];
        nums[index] = val;
        for (int i = index + 1; i < tree.length; i += i & -i) {
            tree[i] += delta;
        }
    }

    private int prefixSum(int i) {
        int s = 0;
        for (; i > 0; i &= i - 1) {
            s += tree[i];
        }
        return s;
    }

    public int sumRange(int left, int right) {
        return prefixSum(right + 1) - prefixSum(left);
    }
}
```

## How the Index Structure Works

BIT is **1-indexed**. For index `i`, `lowbit(i) = i & -i` determines the range it covers:

| i (decimal) | i (binary) | lowbit(i) | tree[i] covers |
|-------------|-----------|-----------|----------------|
| 1 | 0001 | 1 | nums[0..0] |
| 2 | 0010 | 2 | nums[0..1] |
| 3 | 0011 | 1 | nums[2..2] |
| 4 | 0100 | 4 | nums[0..3] |
| 5 | 0101 | 1 | nums[4..4] |
| 6 | 0110 | 2 | nums[4..5] |
| 7 | 0111 | 1 | nums[6..6] |
| 8 | 1000 | 8 | nums[0..7] |

## Operations Explained

### Update — Climb Up

To update index `i`, add delta to all `tree[]` nodes whose range includes `i`:

```
i = 3 (binary: 011)
  → tree[3]  (011)          add lowbit → 011 + 001 = 100
  → tree[4]  (100)          add lowbit → 100 + 100 = 1000
  → tree[8]  (1000)         add lowbit → done (out of range)
```

### Query — Strip Lowest Bit

To compute `prefixSum(7)`, accumulate tree nodes and strip the lowest set bit:

```
i = 7 (binary: 111)
  → tree[7]  (111)  covers nums[6..6]     strip → 111 & 110 = 110
  → tree[6]  (110)  covers nums[4..5]     strip → 110 & 101 = 100
  → tree[4]  (100)  covers nums[0..3]     strip → 100 & 011 = 000
  → done
  Total = tree[7] + tree[6] + tree[4]
```

`i &= i - 1` is equivalent to `i -= i & -i` — both strip the lowest set bit.

## Complexity

| | Value |
|---|---|
| Build | O(N log N) via N individual updates |
| Update | O(log N) |
| Prefix Query | O(log N) |
| Range Query | O(log N) — `prefixSum(right + 1) - prefixSum(left)` |
| Space | O(N) |

## O(N) Build Optimization

Instead of N individual updates, build in-place by propagating each node to its direct parent:

```java
public NumArray(int[] nums) {
    int n = nums.length;
    this.nums = Arrays.copyOf(nums, n);
    tree = new int[n + 1];
    for (int i = 1; i <= n; i++) {
        tree[i] += nums[i - 1];
        int parent = i + (i & -i);
        if (parent <= n) {
            tree[parent] += tree[i];
        }
    }
}
```

## BIT vs. Segment Tree

| | BIT | Segment Tree |
|---|-----|-------------|
| Code complexity | ~15 lines | ~50 lines |
| Space | N + 1 | 4N |
| Constant factor | Very small | Larger |
| Point update + range query | Yes | Yes |
| Range update + range query | With 2 BITs | Yes |
| Arbitrary range operations (min/max) | No | Yes |
| Lazy propagation | No | Yes |

**Rule of thumb**: if the operation has an inverse (sum → subtraction), BIT is sufficient and preferred. For non-invertible operations (min, max, gcd), use a Segment Tree.

## Common Variations

| Variation | Modification |
|-----------|-------------|
| **Count inversions** | Discretize values, insert from right, query prefix count |
| **Range update + point query** | Store deltas in BIT, query = prefix sum of deltas |
| **Range update + range query** | Use two BITs to maintain `B1` and `B2` such that `prefixSum(i) = B1[i] * i - B2[i]` |
| **2D BIT** | Nest two BIT loops for 2D point update + rectangle sum |
| **Order statistics** | BIT on value domain + binary lifting to find k-th smallest |

## Quick Notes

- BIT is **1-indexed** — always convert 0-indexed input with `i + 1`
- `i & -i` works because `-i` is the two's complement of `i`, which flips all bits and adds 1
- For **long** values, change `int[] tree` to `long[] tree`
- BIT cannot efficiently support min/max queries — the prefix decomposition only works for invertible operations
- When counting inversions, iterate right-to-left and query how many previously inserted values are less than current

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 307 | Range Sum Query - Mutable | Classic point update + range query |
| 315 | Count of Smaller Numbers After Self | BIT on discretized values |
| 327 | Count of Range Sum | BIT + coordinate compression |
| 493 | Reverse Pairs | BIT or merge sort |
| 1649 | Create Sorted Array through Instructions | BIT to count less/greater |
| 2179 | Count Good Triplets in an Array | BIT for prefix counting |
| 308 | Range Sum Query 2D - Mutable | 2D BIT |
