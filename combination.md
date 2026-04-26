# Combination (nCr) Under Modular Arithmetic

## When to Use

- Compute `C(n, r) % MOD` — the number of ways to choose `r` items from `n`
- Common in counting problems, DP transitions, and combinatorial arguments

## Approach Comparison

| Approach | Precomputation | Per Query | Space | Best For |
|----------|---------------|-----------|-------|----------|
| Pascal's Triangle | O(n²) | O(1) | O(n²) | Small n (≤ 5000) |
| Factorial + Modular Inverse | O(n) | O(1) | O(n) | Large n (≤ 10⁶), MOD is prime |
| On-the-fly | None | O(r) | O(1) | Single query, no precomputation |

---

## Approach 1: Pascal's Triangle

Build the full table using `C(i, j) = C(i-1, j-1) + C(i-1, j)`.

```java
static final long MOD = 1_000_000_007L;

private static long[][] buildCombination(int maxN) {
    long[][] c = new long[maxN + 1][maxN + 1];

    for (int i = 0; i <= maxN; i++) {
        c[i][0] = 1;
        c[i][i] = 1;
        for (int j = 1; j < i; j++) {
            c[i][j] = (c[i - 1][j - 1] + c[i - 1][j]) % MOD;
        }
    }

    return c;
}
```

**Pros**: Simple, no modular inverse needed, works with any MOD.
**Cons**: O(n²) time and space — impractical when n > 5000.

---

## Approach 2: Factorial + Modular Inverse (Recommended)

`C(n, r) = n! / (r! × (n-r)!) = fact[n] × invFact[r] × invFact[n-r] % MOD`

Precompute `fact[]` and `invFact[]` in O(n), then answer any query in O(1).

```java
static final long MOD = 1_000_000_007L;

private static long[] fact;
private static long[] invFact;

private static void precompute(int maxN) {
    fact = new long[maxN + 1];
    invFact = new long[maxN + 1];

    fact[0] = 1;
    for (int i = 1; i <= maxN; i++) {
        fact[i] = fact[i - 1] * i % MOD;
    }

    invFact[maxN] = modPow(fact[maxN], MOD - 2);
    for (int i = maxN - 1; i >= 0; i--) {
        invFact[i] = invFact[i + 1] * (i + 1) % MOD;
    }
}

private static long nCr(int n, int r) {
    if (r < 0 || r > n) return 0;
    return fact[n] % MOD * invFact[r] % MOD * invFact[n - r] % MOD;
}

private static long modPow(long base, long exp) {
    long result = 1;
    base %= MOD;
    while (exp > 0) {
        if ((exp & 1) == 1) {
            result = result * base % MOD;
        }
        base = base * base % MOD;
        exp >>= 1;
    }
    return result;
}
```

**Pros**: O(n) precomputation, O(1) per query, handles n up to 10⁶+.
**Cons**: Requires MOD to be prime (for Fermat's little theorem).

### Why Compute `invFact` in Reverse?

Only one `modPow` call is needed — compute `invFact[maxN]`, then derive the rest:

`invFact[i] = invFact[i+1] × (i+1)`

This works because `1/i! = 1/(i+1)! × (i+1)`.

---

## Approach 3: On-the-fly (No Precomputation)

Compute a single `C(n, r)` in O(r) time without any precomputation.

```java
static final long MOD = 1_000_000_007L;

private static long nCr(int n, int r) {
    if (r < 0 || r > n) return 0;
    if (r > n - r) r = n - r;

    long num = 1, den = 1;
    for (int i = 0; i < r; i++) {
        num = num * ((n - i) % MOD) % MOD;
        den = den * ((i + 1) % MOD) % MOD;
    }

    return num % MOD * modPow(den, MOD - 2) % MOD;
}
```

**Pros**: No precomputation, O(1) space.
**Cons**: O(r) per query — slow if called many times.

---

## Complexity Summary

| | Pascal's Triangle | Factorial + Inverse | On-the-fly |
|---|---|---|---|
| Precompute | O(n²) | O(n) | None |
| Query | O(1) | O(1) | O(r) |
| Space | O(n²) | O(n) | O(1) |
| MOD constraint | Any | Must be prime | Must be prime |

## Quick Notes

- **Default choice**: Factorial + Modular Inverse — it covers the vast majority of competitive programming problems
- Pascal's Triangle is useful when MOD is not prime, or when n is very small
- Remember to handle edge cases: `r < 0`, `r > n`, and `C(0, 0) = 1`
- For `C(n, r)` where n is huge but r is small (e.g., n = 10^18, r ≤ 30), use on-the-fly
- Common identity: `C(n, r) = C(n, n-r)` — always pick `min(r, n-r)` to reduce work

## Related Problems

| # | Problem | Notes |
|---|---------|-------|
| 62 | Unique Paths | `C(m+n-2, m-1)` |
| 96 | Unique Binary Search Trees | Catalan number via combinations |
| 1359 | Count All Valid Pickup and Delivery Options | Factorial + combinatorics |
| 1569 | Number of Ways to Reorder Array to Get Same BST | Pascal's Triangle in tree recursion |
| 2400 | Number of Ways to Reach a Position After Exactly k Steps | `C(k, (k + abs(diff)) / 2)` |
| 2842 | Count K-Subsequences of a String With Maximum Beauty | nCr for selection |
